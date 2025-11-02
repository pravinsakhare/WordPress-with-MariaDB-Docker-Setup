# WordPress with MariaDB Using Docker

A complete guide to setting up WordPress with MariaDB using Docker containers on a custom network.

## 📋 Prerequisites

- Docker installed and running
- Basic familiarity with Docker commands

## 🚀 Quick Start

### Step 1: Create a Docker Network

Create a custom bridge network so your containers can communicate by name:

```bash
docker network create mynetwork
```

### Step 2: Create a Docker Volume (Recommended)

Create a volume to persist your database data:

```bash
docker volume create mariadb-data
```

### Step 3: Run the MariaDB Container

Launch MariaDB with your database credentials:

```bash
docker run -d \
  --name my-mariadb-container \
  --network mynetwork \
  -e MARIADB_ROOT_PASSWORD='pass@123' \
  -e MARIADB_USER=pravin \
  -e MARIADB_PASSWORD='pass123' \
  -e MARIADB_DATABASE=mydb \
  -v mariadb-data:/var/lib/mysql \
  mariadb:latest
```

**What this does:**
- Container name: `my-mariadb-container`
- DB user: `pravin` / password: `pass123`
- Database: `mydb`
- Root password: `pass@123`
- Data persisted in Docker volume `mariadb-data`
- Attached to `mynetwork` so other containers can reach it by name

> **Note:** If your password contains special characters (like `@`), make sure to quote the value.

### Step 4: Run the WordPress Container

Launch WordPress on the same network:

```bash
docker run -d \
  --name my-wordpress-container \
  --network mynetwork \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=my-mariadb-container \
  -e WORDPRESS_DB_USER=pravin \
  -e WORDPRESS_DB_PASSWORD='pass123' \
  -e WORDPRESS_DB_NAME=mydb \
  wordpress:latest
```

**What this does:**
- Container name: `my-wordpress-container`
- Port mapping: `8080:80` (access WordPress on port 8080)
- Connected to `mynetwork`
- Database host: `my-mariadb-container` (uses container name resolution)
- Uses the same credentials from Step 3

### Step 5: Verify Containers Are Running

Check that both containers are up:

```bash
docker ps
```

Check logs if needed:

```bash
docker logs -f my-mariadb-container
docker logs -f my-wordpress-container
```

**Expected output:** MariaDB initialization messages, WordPress starting logs. No "Error establishing database connection".

### Step 6: Access Your WordPress Site

Open your browser and go to:

```
http://localhost:8080
```

You should see the WordPress installation page where you can complete the setup.

## 🔍 Troubleshooting

If you encounter issues, try these steps:

### 1. Verify both containers are on the same network

```bash
docker inspect my-wordpress-container --format '{{json .NetworkSettings.Networks}}'
```

### 2. Check container logs for errors

```bash
docker logs my-mariadb-container
docker logs my-wordpress-container
```

### 3. Verify database credentials match

Make sure these environment variables are identical:
- `WORDPRESS_DB_USER` matches `MARIADB_USER`
- `WORDPRESS_DB_PASSWORD` matches `MARIADB_PASSWORD`
- `WORDPRESS_DB_NAME` matches `MARIADB_DATABASE`

### 4. Handle special characters in passwords

If your password has special characters, ensure it's properly quoted in your `docker run` command.

### 5. Remove and recreate containers (if needed)

```bash
docker rm -f my-wordpress-container my-mariadb-container
docker volume rm mariadb-data  # only if you want to delete persisted data
```

Then repeat Steps 3-4.

## ⚙️ Configuration Reference

| Component | Value |
|-----------|-------|
| Network | `mynetwork` |
| MariaDB Container | `my-mariadb-container` |
| WordPress Container | `my-wordpress-container` |
| Database Name | `mydb` |
| Database User | `pravin` |
| Database Password | `pass123` |
| Root Password | `pass@123` |
| WordPress Port | `8080` |
| Access URL | http://localhost:8080 |

## 🛠️ Using Docker Compose (Alternative)

For easier management, you can use Docker Compose. Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:latest
    container_name: my-mariadb-container
    environment:
      MARIADB_ROOT_PASSWORD: 'pass@123'
      MARIADB_USER: pravin
      MARIADB_PASSWORD: 'pass123'
      MARIADB_DATABASE: mydb
    volumes:
      - mariadb-data:/var/lib/mysql
    networks:
      - mynetwork

  wordpress:
    image: wordpress:latest
    container_name: my-wordpress-container
    depends_on:
      - mariadb
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: pravin
      WORDPRESS_DB_PASSWORD: 'pass123'
      WORDPRESS_DB_NAME: mydb
    networks:
      - mynetwork

volumes:
  mariadb-data:

networks:
  mynetwork:
```

Then run:

```bash
docker-compose up -d
```

To stop:

```bash
docker-compose down
```

## 📝 Notes

- This setup is intended for **development and educational purposes**
- For production environments, consider:
  - Using environment files for sensitive data
  - Implementing proper security measures
  - Setting up SSL/TLS certificates
  - Regular backups of your database
  - Using specific version tags instead of `latest`

## 📄 License

This guide is provided for educational purposes. Feel free to use and modify as needed.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page.

---

**Happy Dockering! 🐳**