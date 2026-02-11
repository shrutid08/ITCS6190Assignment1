## Docker Multi-Container Assignment – PostgreSQL + Python App

### Overview

This project demonstrates a simple two-container Docker stack using Docker Compose.
A PostgreSQL container hosts a seeded database of trip records, while a Python application container connects to the database, performs queries, computes basic statistics, and outputs a JSON summary file.

---

### Tech Stack

* Docker & Docker Compose
* PostgreSQL 16
* Python 3.11 with psycopg driver

---

### Project Structure

```
.
├── db/
│   ├── Dockerfile
│   └── init.sql
├── app/
│   ├── Dockerfile
│   └── main.py
├── compose.yml
├── Makefile
├── README.md
└── out/
```

---

### How to Run the Project

#### Build and start containers

```bash
docker compose up --build
```

Or using Makefile:

```bash
make
```

---

### Stopping the Containers

```bash
docker compose down -v
```

---

### Output Location and How to View It

The Python application generates a summary JSON file containing query results and statistics.

**Output file location:**

```
out/summary.json
```

This folder is mounted as a Docker volume so data persists on your host machine.

**To view the output:**

Option 1 (terminal):

```bash
cat out/summary.json
```

Option 2:

* Open the `out` folder in your IDE or file explorer
* Open `summary.json` directly.

The results are also printed to the terminal when the container runs.

---

### Environment Variables Used

The application connects to PostgreSQL using environment variables defined in `compose.yml`.

#### Database Container

```
POSTGRES_USER=appuser
POSTGRES_PASSWORD=secretpw
POSTGRES_DB=appdb
```

These create the database, user, and password automatically on first startup.

#### Python Application Container

```
DB_HOST=db
DB_PORT=5432
DB_USER=appuser
DB_PASS=secretpw
DB_NAME=appdb
APP_TOP_N=10
```

**Purpose:**

* `DB_HOST` → Docker service name of database container
* `DB_PORT` → PostgreSQL port
* `DB_USER/DB_PASS/DB_NAME` → DB login credentials
* `APP_TOP_N` → Number of longest trips returned in output

Environment variables make the application portable and configurable without modifying code.

---

### Example Output

```
=== Summary ===
{
  "total_trips": 6,
  "avg_fare_by_city": [
    {"city": "Charlotte", "avg_fare": 16.25},
    {"city": "New York", "avg_fare": 19.0},
    {"city": "San Francisco", "avg_fare": 20.25}
  ],
  "top_by_minutes": [
    {"city": "San Francisco", "minutes": 28, "fare": 29.3},
    {"city": "New York", "minutes": 26, "fare": 27.1},
    {"city": "Charlotte", "minutes": 21, "fare": 20.0},
    {"city": "Charlotte", "minutes": 12, "fare": 12.5},
    {"city": "San Francisco", "minutes": 11, "fare": 11.2},
    {"city": "New York", "minutes": 9, "fare": 10.9}
  ]
}
```

---

### Troubleshooting

**Database not ready**

* The Python app retries automatically until Postgres is ready.

**Permission issues with output folder**

```bash
mkdir -p out
chmod 755 out
```

**Docker file-sharing problems (Mac/Windows)**

* Ensure the project folder is added in Docker Desktop file sharing settings.



### Technical Details

```
Database Schema
CREATE TABLE trips (
    id SERIAL PRIMARY KEY,
    city TEXT NOT NULL,
    minutes INT NOT NULL,
    fare NUMERIC(6,2) NOT NULL
);
```

### Sample Data

6 trip records across 3 cities (Charlotte, New York, San Francisco)

Trip durations from 9 to 28 minutes

Fares ranging from $10.90 to $29.30



## Application Logic

1. Connection Retry: Waits up to 20 seconds for database availability
2. Query Execution: Runs three analytical queries
3. Data Processing: Computes statistics and formats results
4. Output Generation: Writes JSON to file and prints to console



### What This Project Demonstrates

* Multi-container Docker Compose setup
* Database initialization using SQL scripts
* Container networking via service names
* Environment variable configuration
* Persistent output using Docker volumes
* Automated reproducible workflow with one command.
