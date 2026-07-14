# dockerize-web-app
# Cloud Native Migration - vProfile Application

## Project Overview

This project demonstrates the migration of the **vProfile** Java application from a traditional VM-based deployment to a **containerized architecture using Docker and Docker Compose**.

The application consists of multiple services:

- Tomcat (Java Application)
- MariaDB
- RabbitMQ
- Memcached

Each service runs in its own container and communicates through a Docker bridge network.

---

# Architecture

```
                +----------------------+
                |      Client          |
                +----------+-----------+
                           |
                        Port 8080
                           |
                    +------+------+
                    |   Tomcat    |
                    |  Java App   |
                    +------+------+
                           |
        ----------------------------------------
        |                  |                  |
        |                  |                  |
+-------+------+   +-------+------+   +-------+------+
|   MariaDB    |   | RabbitMQ     |   | Memcached    |
|    3306      |   |5672 /15672   |   |   11211      |
+--------------+   +--------------+   +--------------+
```

---

# Project Structure

```
docker-files/
│
├── app/
│   └── Dockerfile
│
├── db/
│   ├── Dockerfile
│   └── db_backup.sql
│
├── docker-compose.yml
│
└── README.md
```

---

# Technologies

- Docker
- Docker Compose
- Apache Tomcat 9
- Java 11
- Maven
- MariaDB 10.6
- RabbitMQ
- Memcached

---

# Docker Images

## Application

Built using a Multi-stage Dockerfile.

### Builder Stage

- Maven
- Java 11
- Clone source code
- Build WAR file

### Runtime Stage

- Tomcat 9
- Deploy ROOT.war

---

## Database

MariaDB official image.

Database initialization is performed automatically using:

```
/docker-entrypoint-initdb.d/
```

---

## RabbitMQ

Official RabbitMQ Management image.

Ports

- 5672
- 15672

---

## Memcached

Official Memcached image.

Port

- 11211

---

# Build Images

```bash
docker compose build
```

---

# Run Application

```bash
docker compose up -d
```

---

# Verify Containers

```bash
docker ps
```

---

# Verify Application

```
http://localhost:8080
```

---

# Verify RabbitMQ

```
http://localhost:15672
```

Username

```
test
```

Password

```
test
```

---

# Docker Compose Services

- app
- db01
- rmq01
- mc01

---

# Environment Variables

## MariaDB

| Variable | Value |
|----------|-------|
| MYSQL_ROOT_PASSWORD | admin123 |
| MYSQL_DATABASE | accounts |
| MYSQL_USER | admin |
| MYSQL_PASSWORD | admin123 |

---

## RabbitMQ

| Variable | Value |
|----------|-------|
| RABBITMQ_DEFAULT_USER | test |
| RABBITMQ_DEFAULT_PASS | test |

---

# Volumes

```
vprodbdata
```

Used for persistent MariaDB data.

---

# Ports

| Service | Port |
|----------|------|
| Tomcat | 8080 |
| MariaDB | 3306 |
| RabbitMQ | 5672 |
| RabbitMQ UI | 15672 |
| Memcached | 11211 |

---

# Troubleshooting

## 1. Application returns 404

### Cause

Application failed to deploy.

### Check

```bash
docker logs app
```

---

## 2. RabbitMQ AuthenticationFailureException

```
ACCESS_REFUSED
```

### Cause

Incorrect RabbitMQ username/password.

### Solution

Verify

```
RABBITMQ_DEFAULT_USER
RABBITMQ_DEFAULT_PASS
```

and

```
application.properties
```

contain matching credentials.

---

## 3. Database Connection Failed

### Cause

MariaDB is unavailable.

### Verify

```bash
docker logs db01
```

Check

```
MYSQL_DATABASE
MYSQL_USER
MYSQL_PASSWORD
```

---

## 4. Container Cannot Resolve Hostname

Example

```
UnknownHostException: db01
```

### Cause

Incorrect Docker Compose service name.

### Solution

Use Docker Compose service names

```
db01
rmq01
mc01
```

instead of IP addresses.

---

## 5. ROOT.war Missing

Verify

```bash
docker exec app ls /usr/local/tomcat/webapps
```

Expected

```
ROOT
ROOT.war
```

---

## Useful Commands

Build

```bash
docker compose build
```

Run

```bash
docker compose up -d
```

Stop

```bash
docker compose down
```

Rebuild

```bash
docker compose up --build
```

Logs

```bash
docker logs app
```

Shell

```bash
docker exec -it app bash
```

---

# Lessons Learned

- Multi-stage Docker builds significantly reduce image size.
- Official Docker images simplify deployment.
- Docker Compose provides service discovery using service names.
- Volumes preserve database data across container recreation.
- Environment variables are preferred over hardcoded credentials.
- Troubleshooting should start with container logs before checking the application.

---

