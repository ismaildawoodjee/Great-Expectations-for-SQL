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

and type `\d` to list the populated tables. Quit the database using `\q`.

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
    column, and ensured uniqueness and non-null values for the ID columns.
