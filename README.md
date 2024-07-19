# LearnGX

I think that the process should be:


- use pandas
- read Yellow Taxi CSV into a dataframe
- get the schema from the df
- connect to Postgres and create the table
- load the data
- continue on with the GX instructions

### Build the Python container

```bash
docker build . -f Dockerfile -t my_python_env:3.10
```

### Start the environment

```bash
docker compose up --detach --wait --wait-timeout 120
```

### Open a shell in the `python-shell` service

```bash
docker compose exec python-shell bash
```

### Start the Python interpreter

```bash
python
```

### Create the table and load data

The database `taxi_db` was created, and the password for user `postgres` was set to `example` during the initialization of the Postgres instance because the Docker Compose file includes sets these environment variables:

```yaml
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: example
      POSTGRES_DB: taxi_db
```

This code can be run to examine the schema of the Yellow Taxi data and create a table in the `taxi_db` database of Postgresql sserver `db`:

```python
import pandas as pd

df = pd.read_csv(
    'taxi-data/yellow_tripdata_sample_2020-09.csv', 
    nrows=100, 
    parse_dates=['pickup_datetime', 'dropoff_datetime'], 
)

print(pd.io.sql.get_schema(df, name='yellow_taxi_data'))

from sqlalchemy import create_engine

engine = create_engine('postgresql://postgres:example@db:5432/taxi_db')

print(pd.io.sql.get_schema(df, name='yellow_taxi_data', con=engine)) 

df.head(n=0).to_sql(name='yellow_taxi_data', con=engine, if_exists='replace')
```

### Load the data

```python
df = pd.read_csv( 
    'taxi-data/yellow_tripdata_sample_2020-09.csv', 
    parse_dates=['pickup_datetime', 'dropoff_datetime'] 
)

df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append', chunksize=1000)
```

### Import and setup Postgresql connection

```python
import great_expectations as gx

from great_expectations.checkpoint import Checkpoint

from sqlalchemy import create_engine

from sqlalchemy_utils import database_exists, create_database

context = gx.get_context()

PG_CONNECTION_STRING = "postgresql+psycopg2://postgres:example@db/taxi_db"

pg_datasource = context.sources.add_postgres(
    name="pg_datasource", connection_string=PG_CONNECTION_STRING
)
```

```python
pg_datasource.add_table_asset(
    name="yellow_taxi_data", table_name="yellow_taxi_data"
)

batch_request = pg_datasource.get_asset("yellow_taxi_data").build_batch_request()

expectation_suite_name = "insert_your_expectation_suite_name_here"
context.add_or_update_expectation_suite(expectation_suite_name=expectation_suite_name)
validator = context.get_validator(
    batch_request=batch_request,
    expectation_suite_name=expectation_suite_name,
)

print(validator.head())

validator.expect_column_values_to_not_be_null(column="passenger_count")

validator.expect_column_values_to_be_between(
    column="congestion_surcharge", min_value=0, max_value=1000
)

validator.save_expectation_suite(discard_failed_expectations=False)

my_checkpoint_name = "my_sql_checkpoint"

checkpoint = Checkpoint(
    name=my_checkpoint_name,
    run_name_template="%Y%m%d-%H%M%S-my-run-name-template",
    data_context=context,
    batch_request=batch_request,
    expectation_suite_name=expectation_suite_name,
    action_list=[
        {
            "name": "store_validation_result",
            "action": {"class_name": "StoreValidationResultAction"},
        },
        {"name": "update_data_docs", "action": {"class_name": "UpdateDataDocsAction"}},
    ],
)

checkpoint_result = checkpoint.run()

print(checkpoint.get_config().to_yaml_str())

# This may need to be run in a Jupyter notebook
context.open_data_docs()

```
