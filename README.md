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
# Note: the module name is psycopg, not psycopg3
import psycopg

# Connect to an existing database
with psycopg.connect("dbname=test user=postgres") as conn:

    # Open a cursor to perform database operations
    with conn.cursor() as cur:

        # Execute a command: this creates a new table
        cur.execute("""
            CREATE TABLE test (
                id serial PRIMARY KEY,
                num integer,
                data text)
            """)

        # Pass data to fill a query placeholders and let Psycopg perform
        # the correct conversion (no SQL injections!)
        cur.execute(
            "INSERT INTO test (num, data) VALUES (%s, %s)",
            (100, "abc'def"))

        # Query the database and obtain data as Python objects.
        cur.execute("SELECT * FROM test")
        cur.fetchone()
        # will return (1, 100, "abc'def")

        # You can use `cur.fetchmany()`, `cur.fetchall()` to return a list
        # of several records, or even iterate on the cursor
        for record in cur:
            print(record)

        # Make the changes to the database persistent
        conn.commit()
```
