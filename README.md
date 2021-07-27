# Great Expectations with SQL

## Table of Contents

## Introduction

Great Expectations (GE) makes use of SQLAlchemy together with the Python library appropriate for the specific flavour of SQL database
that want to use. For example, if I want to connect GE to a PostgreSQL database, we will need the `sqlalchemy` and `psycopg2` libraries.
GE will prompt the user to install them if they are note installed yet. The credentials for connecting to the SQL database will be
specified in the `config_variables.yml` file in the `great_expectations/uncommitted` folder after configuring a Datasource.

The data preparation step is described in the [PostgresDB Setup](#postgresdb-setup) section. I used a Docker container to host
a Postgres database, following this [useful reference](https://levelup.gitconnected.com/creating-and-filling-a-postgres-db-with-docker-compose-e1607f6f882f).
The setup populates the database with dummy e-commerce data, which can then be connected to Great Expectations for profiling and
validation.

## Setup

### PostgresDB Setup

Clone this repo, set up a Python virtual environment, and run the `docker-compose` file in the background:

    git clone https://github.com/ismaildawoodjee/Great-Expectations-for-SQL

    cd Great-Expectations-for-SQL

    python -m venv .venv

    source .venv/bin/activate

    docker-compose up -d

The `docker-compose` file pulls a Postgres v10.5 image, mounts a volume `postgres-data` to persist data (after the container
is destroyed), and copies the SQL scripts from the `sql` folder to create tables and populate them with dummy e-commerce data.
The local port on your local host is 5438, which will be needed when connecting Great Expectations to this Postgres container.
The Postgres default port is 5432. The credentials for the username, password and database name should be provided in the
Jupyter notebook when configuring a Datasource.

To verify that the database is up and running, use the CLI command (with container ID):

    docker exec -it <CONTAINER_ID> /bin/bash

Inside the container, run

    psql -U postgres

and type `\d` to list the populated tables. Here, SQL queries can be written to explore the database, and add or remove rows
from the tables. Quit the database using `\q`.

### Great Expectations Setup

1. Install Great Expectations with

        pip install great_expectations

2. Initialize a Data Context with

        great_expectations --v3-api init

3. Configure a new Datasource. GE will prompt you to install `sqlalchemy` and `psycopg2` if they are not already install in the venv:

        great_expectations --v3-api datasource new

        pip install sqlalchemy psycopg2

4. Running the previous command will open up a Jupyter Notebook, where we can provide the credentials to connect to the
Postgres database (the Datasource name can be anything, I gave the name `postgres_data`). These can be changed as you wish
when initially setting up the Postgres database:

        host = "localhost"
        port = "5438"
        username = "postgres"
        password = "postgres"
        database = "postgres"

5. After ensuring that the connection is successful, we create a new Expectation Suite:

        great_expectations --v3-api suite new

    Here, we will have to choose which table (out of the 8 tables that were populated using the SQL scripts) to carry out
    profiling. I chose the `sale` table, and named the Suite `sale_suite`.

6. The Expectations generated from this initial profiling will not be ideal. We can edit the Expectations using the command:

        great_expectations --v3-api suite edit sale_suite --interactive

    Just like the JSON and CSV evaluations, we can add/remove/change the Expectations within the Jupyter Notebook and
    inspect the Data Doc that is opened after running the last cell. Here, I just implemented some range checks for the `amount`
    column, and ensured consistent type, uniqueness and non-null values for the ID columns. Rerun the Jupyter Notebook cells
    after editing the Expectations and check out the newly generated Data Doc.

## Validating New Data

To validate a new batch of data, more data has to be first inserted into the `sale` table and then we can create a checkpoint
to pair up new data with the edited Expectation Suite. I added a new row to the `sale` table that violates the expected
`amount` range between [0,10,000]:

    INSERT INTO public.sale(sale_id, amount, date_sale, product_id, user_id, store_id)
    VALUES ('7914a1d1-5148-4d2a-93ef-aeb24687fc08', 10010.10, '2019-07-02 12:15:43.123456', 72, 20111, 43);

Note that the only single quotes should be used in SQL, not double scripts.

Next, I create a checkpoint to pair up the table with the Expectation Suite using the CLI command

    great_expectations --v3-api checkpoint new sale_checkpoint

Change the `data_asset_name` to be the `sale` table that we want to validate and then run all cells in the Notebook.
This should open up a Data Doc (if it's not already opened) and then we can see the validation results.
