# LearnGX

I think that the process should be:


- use pandas
- read Yellow Taxi CSV into a dataframe
- get the schema from the df
- connect to Postgres and create the table
- load the data
- continue on with the GX instructions

### Build the Jupyter container

```bash
docker compose build
```

### Start the environment

```bash
docker compose up --detach --wait --wait-timeout 120
```

### Connect to Jupyter server

http://127.0.0.1:8889/

> Tip
>
> The token is `my-token`

### Work with the data

Follow along with the notebook `YellowCab.ipynb`. I have not figured out why the output of `context.open_data_docs()` is not published.
