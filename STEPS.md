# 🚀 Project Setup: Dockerized React + Spring Boot + AWS RDS (MariaDB)

This document records all steps taken to deploy a full-stack application (React frontend + Spring Boot backend) on AWS EC2 using Docker and Docker Compose with a secure MariaDB RDS connection.

---

## ✅ **Architecture**

```
React (Docker + Nginx)  -->  Spring Boot (Docker)  -->  AWS RDS (MariaDB)
Port 80                      Port 8080
```

---

# ==========================

# 1️⃣ INSTALL DOCKER ON EC2

# ==========================

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
 -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin -y

docker --version
```

---

# ==========================

# 2️⃣ INSTALL MYSQL CLIENT

# ==========================

```bash
sudo apt install mysql-client -y
```

Test RDS connectivity:

```bash
mysql -h <RDS-ENDPOINT> \
-u <USERNAME> -p 

CREATE DATABASE student_db;
GRANT ALL PRIVILEGES ON springbackend.* TO 'username'@'localhost' IDENTIFIED BY 'your_password';

```

---

# ==========================

# 3️⃣ CLONE PROJECT

# ==========================

```bash
git clone <REPO-URL>

```

---

# ==========================

# 4️⃣ PREPARE BACKEND CONFIG (IMPORTANT STEP YOU ADDED)

# ==========================

Go into backend directory and copy the original config file (application.properties)  to the project root for editing:

```bash
cd backend
cp src/main/resources/application.properties .
```

Edit it in the current directory:

```bash
vim application.properties
```

> Later, this file will be copied back into the source location during Docker build.

---

# ==========================

# 5️⃣ BACKEND CONFIGURATION

# ==========================

## `backend/application.properties`

```properties
server.port=8080

spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.database-platform=org.hibernate.dialect.MariaDBDialect
```

Return to project root:

```bash
cd ..
```

---

# ==========================

# 6️⃣ FRONTEND CONFIGURATION

# ==========================

## `frontend/.env`

```env
VITE_API_URL="http://<PUBLIC-IP>:8080/api"
```

---

# ==========================

# 7️⃣ DOCKER COMPOSE SETUP

# ==========================

## `docker-compose.yml`

```yaml
version: "3.8"

services:

  backend:
    build: ./backend
    container_name: backend
    ports:
      - "8080:8080"
    environment:
      DB_URL: "jdbc:mariadb://<RDS-ENDPOINT>:3306/<DATABASE-NAME>?useSSL=true&requireSSL=true&trustServerCertificate=true&serverTimezone=UTC"
      DB_USER: "DATABASE-USERNAME"
      DB_PASSWORD: "DATABASE-PASSWORD"
    restart: always

  frontend:
    build: ./frontend
    container_name: frontend
    ports:
      - "80:80"
    environment:
      VITE_API_URL: "http://<PUBLIC-IP>:8080/api"
    depends_on:
      - backend
    restart: always
```

---

# ==========================

# 8️⃣ CLEAN BUILD & RUN

# ==========================

```bash
docker compose down -v 
```
```
docker compose build --no-cache 
```
```
docker compose up -d
```

Check status:

```bash
docker compose ps
```

---

# ==========================

# 9️⃣ VALIDATE BACKEND

# ==========================

```bash
docker logs backend | grep -i hikari
```

Expected output:

```
HikariPool-1 - Added connection ...
HikariPool-1 - Start completed.
```

Test in browser:

```
http://<PUBLIC-IP>:8080/actuator/health
```

---

# ==========================

# 🔟 VALIDATE FRONTEND

# ==========================

Open in browser:

```
http://<PUBIC-IP>/
```

---

# ==========================

# 🔁 11️⃣ TEST RESTART POLICY

# ==========================

Kill backend:

```bash
docker kill backend
```

Check:

```bash
docker compose ps
```

Backend should show **Restarting**, then **Up**.

Restart Docker daemon:

```bash
sudo systemctl restart docker
```

Then:

```bash
docker compose ps
```

Both containers should be **Up**.

---

# ✅ FINAL STATE

You now have:

* ✔ Dockerized Spring Boot backend
* ✔ Dockerized React frontend (Nginx)
* ✔ Secure TLS connection to AWS RDS
* ✔ Docker Compose orchestration
* ✔ Auto-restart on failure
* ✔ Cloud deployment on EC2

---

### Suggested Git Commit Message

```
feat: deploy full-stack app with docker compose and secure RDS connection
```

