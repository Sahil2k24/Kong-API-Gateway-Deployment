Here’s a detailed README file for your GitHub repository:

---

# Kong Gateway Installation, Configuration, and Setup

Kong Gateway is a lightweight, fast, and flexible cloud-native API gateway. This documentation provides step-by-step instructions for installing, configuring, and setting up Kong Gateway with PostgreSQL as the database.

## Prerequisites

- Ubuntu OS
- Admin access to the server
- Basic knowledge of PostgreSQL and Kong Gateway

---

## Table of Contents

1. [Install PostgreSQL](#step-1-install-postgresql)
2. [Create PostgreSQL User and Database](#step-2-create-postgresql-user-and-database)
3. [Install Kong Gateway](#step-3-install-kong-gateway)
4. [Configure PostgreSQL Authentication](#step-4-configure-postgresql-authentication)
5. [Configure Kong](#step-5-configure-kong)
6. [Run Database Migrations](#step-6-run-database-migrations)
7. [Start Kong Gateway](#step-7-start-kong-gateway)
8. [Kong Services Setup](#kong-services-setup)
   - Create Gateway Service
   - Create Route
   - Set Up Rate Limiting
   - Set Up API Key Authentication
   - Create Consumer and Set Up Credentials
   - Testing
   - Logging: File Log Plugin
   - API Versioning

---

## Step 1: Install PostgreSQL

1. **Update the package list:**
   ```bash
   sudo apt update
   ```

2. **Install PostgreSQL:**
   ```bash
   sudo apt install postgresql
   ```

3. **Check PostgreSQL service status:**
   ```bash
   sudo systemctl status postgresql
   ```

---

## Step 2: Create PostgreSQL User and Database

1. **Log into PostgreSQL shell:**
   ```bash
   sudo -u postgres psql
   ```

2. **Create a user:**
   ```sql
   CREATE USER kong WITH PASSWORD 'super_secret';
   ```

3. **Create a database:**
   ```sql
   CREATE DATABASE kong OWNER kong;
   ```

4. **Exit PostgreSQL:**
   ```sql
   \q
   ```

---

## Step 3: Install Kong Gateway

1. **Download the Kong package:**
   ```bash
   curl -Lo kong-enterprise-edition-3.8.0.0.deb "https://packages.konghq.com/public/gateway-38/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_3.8.0.0/kong-enterprise-edition_3.8.0.0_$(dpkg --print-architecture).deb"
   ```

2. **Install Kong:**
   ```bash
   sudo apt install -y ./kong-enterprise-edition-3.8.0.0.deb
   ```

3. **Optional: Fixing Permission Issues**
   - **Download to /tmp:**
     ```bash
     curl -Lo /tmp/kong-enterprise-edition-3.8.0.0.deb "https://packages.konghq.com/public/gateway-38/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_3.8.0.0/kong-enterprise-edition_3.8.0.0_$(dpkg --print-architecture).deb"
     ```
   - **Install the Package:**
     ```bash
     sudo apt install -y /tmp/kong-enterprise-edition-3.8.0.0.deb
     ```
   - **Remove the Temporary File:**
     ```bash
     sudo rm /tmp/kong-enterprise-edition-3.8.0.0.deb
     ```

---

## Step 4: Configure PostgreSQL Authentication

1. **Edit the `pg_hba.conf` file:**
   ```bash
   sudo nano /etc/postgresql/12/main/pg_hba.conf
   ```

   Update authentication to `md5`:
   ```
   local   all             all                                     md5
   host    all             all             127.0.0.1/32            md5
   host    all             all             ::1/128                 md5
   host    all             all             0.0.0.0/0               md5
   ```

2. **Set PostgreSQL to listen on all IPs:**
   - **Edit `postgresql.conf`:**
     ```bash
     sudo nano /etc/postgresql/12/main/postgresql.conf
     ```
   - **Modify the line:**
     ```conf
     listen_addresses = '*'
     ```

3. **Restart PostgreSQL:**
   ```bash
   sudo systemctl restart postgresql
   ```

---

## Step 5: Configure Kong

1. **Copy the default configuration file:**
   ```bash
   sudo cp /etc/kong/kong.conf.default /etc/kong/kong.conf
   ```

2. **Edit the configuration file:**
   ```bash
   sudo nano /etc/kong/kong.conf
   ```

   Modify the database settings:
   ```conf
   database = postgres
   pg_host = 127.0.0.1
   pg_port = 5432
   pg_user = kong
   pg_password = super_secret
   pg_database = kong
   ```

---

## Step 6: Run Database Migrations

Run migrations:
```bash
kong migrations bootstrap -c /etc/kong/kong.conf
```

---

## Step 7: Start Kong Gateway

1. **Start Kong:**
   ```bash
   kong start -c /etc/kong/kong.conf
   ```

2. **Check Kong status:**
   ```bash
   curl -i http://localhost:8001
   ```

---

## Kong Services Setup

### 1. Create Gateway Service
1. Log into Kong Manager and select the default workspace.
2. Navigate to **Services** and click **+ Add Service**.
3. Fill in the fields:
   - **Name:** Kafka-Health-Service
   - **Tags:** example
   - **Service Endpoint:** 
     - Full URL: `http://api-dev.catalystai.work/health/kafka`

4. Click **Create**.

### 2. Create Route
1. Navigate to **Routes** and click **+ Add Route**.
2. Fill in the details:
   - **Name:** Kafka-Health-Route
   - **Paths:** `/health/kafka`
   - **Service:** Kafka-Health-Service

3. Click **Create**.

### 3. Set Up Rate Limiting
1. Go to the **Plugins** tab of the route.
2. Install **Rate Limiting Plugin** with a limit of 10 requests per minute.

### 4. Set Up API Key Authentication
1. Install the **Key Authentication Plugin** on the route.
2. Use default settings and click **Create**.

### 5. Create Consumer and Set Up Credentials
1. Create a consumer:
   - **Username:** kafka-consumer
   - **Custom ID:** kafka-consumer-id
2. Add an API key (`my-api-key`) to the consumer.

### 6. Logging: File Log Plugin
1. Enable the **File Log Plugin** for the route.
2. Configure:
   - **File Path:** `/var/log/kong/kong.log`

### 7. API Versioning
1. Create versioned routes:
   - `/v1/health/kafka`
   - `/v2/health/kafka`

---

## Testing

**With API Key:**
```bash
curl -i -X GET http://<KONG_IP>:8000/health/kafka --header "apikey: my-api-key"
```

---

## Logs

View logs:
```bash
tail -f /var/log/kong/kong.log
```

---

## License

This documentation is provided under the MIT License.

--- 

You can copy and paste this into a `README.md` file for your GitHub repository!
