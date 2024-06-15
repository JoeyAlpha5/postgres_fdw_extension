# **postgres_fdw tutorial**

The `postgres_fdw` foreign data wrapper extension
allows you to access data stored in a PostgreSQL database on a remote server.

The docker-compose file in this directory sets up two PostgreSQL servers, `database_1` and `database_2`.
To demonstrate the use of the `postgres_fdw` extension, we will create a foreign data wrapper in `database_1` 
that connects to `database_2` and accesses the `managers` table.

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

3 Create the employees table in `database_1`:
```sql
CREATE TABLE IF NOT EXISTS public.employees
(
    employee_id integer NOT NULL,
    first_name character varying(50) COLLATE pg_catalog."default",
    last_name character varying(50) COLLATE pg_catalog."default",
    email character varying(100) COLLATE pg_catalog."default",
    department character varying(50) COLLATE pg_catalog."default",
    CONSTRAINT employees_pkey PRIMARY KEY (employee_id)
)
```

4. Create the managers table in `database_2`:
```sql
CREATE TABLE IF NOT EXISTS public.managers
(
    manager_id integer NOT NULL,
    employee_id integer,
    experience_years integer,
    CONSTRAINT managers_pkey PRIMARY KEY (manager_id)
)
``` 

5. Connect to `database_1` and create the foreign data wrapper:
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

6. Query the `managers` table in `database_1`:
```sql
SELECT * FROM external_managers_table;
```
# postgres_fdw_extension