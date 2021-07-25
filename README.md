# Great Expectations with SQL

Reference: [fill PostgresDB with data](https://levelup.gitconnected.com/creating-and-filling-a-postgres-db-with-docker-compose-e1607f6f882f)

Run

    docker-compose up -d

Verify that database is up and running

    docker exec -it <CONTAINER_ID> /bin/bash

Inside the container, run

    psql -U postgres

and type `\d` to list the populated tables.