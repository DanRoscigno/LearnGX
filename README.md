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

It looks like I need to do a `build` of the docs and then copy the content generated to a place where it can be served:

<img width="450" alt="image" src="https://github.com/user-attachments/assets/ca8fea07-a33a-4bbb-ae53-2f5059ccd5df">

The Jupyter server has a web server running, possibly I can put the files from `/tmp` in a spot where it will serve them.
