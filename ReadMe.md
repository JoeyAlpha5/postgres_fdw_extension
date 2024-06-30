# **postgres_fdw tutorial**

### **YouTube video:**
[![PostgreSQL Foreign Data Wrapper (postgres_fdw) Tutorial](https://img.youtube.com/vi/G6gVuVms0tA/0.jpg)](https://youtu.be/G6gVuVms0tA)

The Postgres foreign data wrapper (`postgres_fdw`) extension can be used to retrieve or access data
on other PostgreSQL servers.

The docker-compose file in this directory sets up two PostgreSQL servers, `database_1` and `database_2`.
To demonstrate the use of the `postgres_fdw` extension, we will create a foreign data wrapper in `database_1` 
that connects to `database_2` and accesses the `managers` table. Please see the diagrams below:

#### Two Postgres servers without the fdw connection

```plaintext
+----------------+                  |                  +----------------+
|                |                  |                  |                |
|  Postgres      |                  |                  |  Postgres      |
|  Server 1      |                  |                  |  Server 2      |
|                |                  |                  |                |
|  database_1    |                  |                  |  database_2    |
|                |                  |                  |                |
|  +-----------+ |                  |                  |  +-----------+ |
|  | employees | |                  |                  |  | managers  | |
|  +-----------+ |                  |                  |  +-----------+ |
|                |                  |                  |                |
+----------------+                  |                  +----------------+
          |                         |                          |
          |                         |                          |
          |                         |                          |
          |                         |                          |
          |                         |                          |
          |                         |                          |
+-------------------+               |               +-------------------------+
|   employees       |               |               |     managers            |
+-------------------+               |               +-------------------------+
| employee_id       |               |               | manager_id              |
| first_name        |               |               | employee_id             |
| last_name         |               |               | experience_years        |
| email             |               |               +-------------------------+
| department        |               |
+-------------------+               |

```


#### Two Postgres servers with the fdw connection

```plaintext
+----------------+                  |                  +----------------+
|                |                  |                  |                |
|  Postgres      |                  |                  |  Postgres      |
|  Server 1      |                  |                  |  Server 2      |
|                |                  |                  |                |
|  database_1    |                  |                  |  database_2    |
|                |                  |                  |                |
|  +-----------+ |                  |                  |  +-----------+ |
|  | employees | |                  |                  |  | managers  | |
|  +-----------+ |                  |                  |  +-----------+ |
|  | managers  | |  ----------------|----------------> |                |
|  | (foreign  | |  FDW connection  |                  |                |
|  |  table)   | |                  |                  |                |
+----------------+                  |                  +----------------+
          |                         |                          |
          |                         |                          |
          |                         |                          |
          |                         |                          |
          |                         |                          |
          |                         |                          |
+-------------------+               |               +-------------------------+
|   employees       |               |               |     managers            |
+-------------------+               |               +-------------------------+
| employee_id       |               |               | manager_id              |
| first_name        |               |               | employee_id             |
| last_name         |               |               | experience_years        |
| email             |               |               +-------------------------+
| department        |               |
+-------------------+               |
```

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
     - Hostname: `db-2`
     - Username: `database_2`
     - Password: `database_2`

3 Create the employees table in `database_1`:
```sql
CREATE TABLE IF NOT EXISTS public.employees
(
    employee_id integer NOT NULL,
    first_name character varying(50),
    last_name character varying(50),
    email character varying(100),
    department character varying(50),
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

5. Insert some data into the `employees` table in `database_1`:
```sql
INSERT INTO public.employees (employee_id, first_name, last_name, email, department) VALUES
(1, 'John', 'Doe', 'john.doe@example.com', 'Sales'),
(2, 'Jane', 'Smith', 'jane.smith@example.com', 'HR'),
(3, 'Michael', 'Johnson', 'michael.johnson@example.com', 'IT'),
(4, 'Patricia', 'Brown', 'patricia.brown@example.com', 'Finance'),
(5, 'Robert', 'Jones', 'robert.jones@example.com', 'Marketing'),
(6, 'Linda', 'Miller', 'linda.miller@example.com', 'Sales'),
(7, 'William', 'Davis', 'william.davis@example.com', 'HR'),
(8, 'Elizabeth', 'Garcia', 'elizabeth.garcia@example.com', 'IT'),
(9, 'James', 'Martinez', 'james.martinez@example.com', 'Finance'),
(10, 'Mary', 'Hernandez', 'mary.hernandez@example.com', 'Marketing');
```

6. Insert some data into the `managers` table in `database_2`:
```sql
INSERT INTO public.managers (manager_id, employee_id, experience_years) VALUES
(1, 1, 5),
(2, 2, 7),
(3, 3, 10),
(4, 4, 8);
```

7. Connect to `database_1` and create the foreign data wrapper:
```sql

--- 0. Install the extension
CREATE EXTENSION postgres_fdw;

--- 1. Create the foreign server
CREATE SERVER database_2
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'db-2', dbname 'database_2', port '5432');

--- 2. Create the user mapping
CREATE USER MAPPING FOR database_1
SERVER database_2
OPTIONS (user 'database_2', password 'database_2');

--- 3. Create the foreign table
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

