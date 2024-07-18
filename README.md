### Build the Python container

```bash
docker build . -f Dockerfile -t my_python_env:3.10
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

context = gx.get_context()

PG_CONNECTION_STRING = "postgresql+psycopg2://postgres:example@db/"

pg_datasource = context.sources.add_postgres(
    name="pg_datasource", connection_string=PG_CONNECTION_STRING
)
```
# LearnGX
