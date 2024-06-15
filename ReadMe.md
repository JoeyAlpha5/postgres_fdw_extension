# **postgres_fdw**

The `postgres_fdw` foreign data wrapper extension
allows you to access data stored in a PostgreSQL database on a remote server.

The docker-compose file in this directory sets up two PostgreSQL servers, `database_1` and `database_2`, 
and creates a foreign data wrapper in `database_1` that allows it to access tables on `database_2`.

To run the example, execute the following command:

1. Start the two PostgreSQL servers, `database_1` and `database_2`. The command will also launch pgAdmin on port 8001.
```bash
docker-compose up -d
```

2. In pgAdmin, register the two servers:
   - `database_1`:
     - Hostname: `db-1`
     - Username: `database_1`
     - Password: `database_1`
   - `database_2`:
     - Hostname: `db-1`
     - Username: `database_2`
     - Password: `database_2`

3. Connect to `database_1` and create the foreign data wrapper:
```sql
CREATE EXTENSION postgres_fdw;

CREATE SERVER database_2
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'db-2', dbname 'database_2', port '5432');

CREATE USER MAPPING FOR database_1
SERVER database_2
OPTIONS (user 'database_2', password 'database_2');

CREATE FOREIGN TABLE external_managers_table
(
    manager_id integer,
    employee_id integer,
    experience_years integer,
)
SERVER database_2
OPTIONS (table_name 'managers');
```

4. Query the `managers` table in `database_1`:
```sql
SELECT * FROM external_managers_table;
```
# postgres_fdw_extension
