
![image](https://github.com/user-attachments/assets/e9a6fc90-c2b2-4bcd-9767-c757c893a4a9)

# Kong Gateway Installation and Configuration Guide

Kong Gateway is a cloud-native, lightweight, and fast API gateway that enables reverse proxying, routing, and managing APIs. It provides essential features such as API versioning, authentication, rate limiting, and plugin support.

This guide walks you through the installation, configuration, and setup of Kong Gateway (OSS and Enterprise). Additionally, it covers detailed steps for managing services, routes, plugins, versioning, and logging.

---

## Prerequisites

Before starting the installation process, ensure you have:

- A machine running Ubuntu (or a similar Linux-based OS)
- Access to root or sudo privileges
- A PostgreSQL instance for Kong’s data store

---

## Step 1: Install PostgreSQL

1. Update the package list:

   ```bash
   sudo apt update
   ```

2. Install PostgreSQL:

   ```bash
   sudo apt install postgresql
   ```

3. Check the status of the PostgreSQL service:

   ```bash
   sudo systemctl status postgresql
   ```

---

## Step 2: Create PostgreSQL User and Database

1. Log into PostgreSQL shell:

   ```bash
   sudo -u postgres psql
   ```

2. Create a PostgreSQL user:

   ```sql
   CREATE USER kong WITH PASSWORD 'super_secret';
   ```

3. Create a database for Kong:

   ```sql
   CREATE DATABASE kong OWNER kong;
   ```

4. Exit PostgreSQL:

   ```sql
   \q
   ```

---

## Step 3: Install Kong Gateway

1. Download the Kong package:

   ```bash
   curl -Lo kong-enterprise-edition-3.8.0.0.deb "https://packages.konghq.com/public/gateway-38/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_3.8.0.0/kong-enterprise-edition_3.8.0.0_$(dpkg --print-architecture).deb"
   ```

2. Install Kong:

   ```bash
   sudo apt install -y ./kong-enterprise-edition-3.8.0.0.deb
   ```

3. If you encounter permission issues, use the following steps:

   - Download to `/tmp`:

     ```bash
     curl -Lo /tmp/kong-enterprise-edition-3.8.0.0.deb "https://packages.konghq.com/public/gateway-38/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_3.8.0.0/kong-enterprise-edition_3.8.0.0_$(dpkg --print-architecture).deb"
     ```

   - Install the package:

     ```bash
     sudo apt install -y /tmp/kong-enterprise-edition-3.8.0.0.deb
     ```

   - Remove the temporary file:

     ```bash
     sudo rm /tmp/kong-enterprise-edition-3.8.0.0.deb
     ```

---

## Step 4: Configure PostgreSQL Authentication

1. Edit the `pg_hba.conf` file:

   ```bash
   sudo nano /etc/postgresql/12/main/pg_hba.conf
   ```

2. Update the authentication to `md5`:

   ```txt
   local   all             all                                     md5
   host    all             all             127.0.0.1/32            md5
   host    all             all             ::1/128                 md5
   host    all             all             0.0.0.0/0               md5
   ```

3. Set PostgreSQL to listen on all IPs:

   - Open `postgresql.conf`:

     ```bash
     sudo nano /etc/postgresql/12/main/postgresql.conf
     ```

   - Modify the line:

     ```txt
     listen_addresses = '*'
     ```

4. Restart PostgreSQL:

   ```bash
   sudo systemctl restart postgresql
   ```

---

## Step 5: Configure Kong

1. Copy the default configuration file:

   ```bash
   sudo cp /etc/kong/kong.conf.default /etc/kong/kong.conf
   ```

2. Edit the configuration file:

   ```bash
   sudo nano /etc/kong/kong.conf
   ```

3. Modify the database settings:

   ```txt
   database = postgres
   pg_host = 127.0.0.1
   pg_port = 5432
   pg_user = kong
   pg_password = super_secret
   pg_database = kong
   ```

---

## Step 6: Run Database Migrations

Run the migrations to set up the database schema:

```bash
kong migrations bootstrap -c /etc/kong/kong.conf
```

---

## Step 7: Start Kong Gateway

1. Start Kong:

   ```bash
   kong start -c /etc/kong/kong.conf
   ```

2. Check the Kong status:

   ```bash
   curl -i http://localhost:8001
   ```

---

## Kong Services Setup

### 1. Create Gateway Service

1. Log into Kong Manager and select the default workspace.
2. Navigate to **Services**.
3. Click on **+ Add Service** and fill in the fields:
   - Name: `Kafka-Health-Service` (or relevant name)
   - Tags: `example` (or relevant tags)
   - Service Endpoint:
     - Full URL: `http://api-dev.catalystai.work/health/kafka`
     - Or specify **Protocol**, **Host**, **Port**, and **Path**.

4. Click **Create**.

### 2. Create Route

1. Navigate to **Routes**.
2. Click on **+ Add Route** and fill in the details:
   - Name: `Kafka-Health-Route`
   - Service: Select `Kafka-Health-Service`.
   - Paths: `/health/kafka`

3. Click **Create**.

### 3. Set Up Rate Limiting

1. Navigate to the **Plugins** tab for the `Kafka-Health-Route`.
2. Click **Install Plugin** and search for **Rate Limiting**.
3. Configure:
   - Limit: `10 requests per minute`.
4. Click **Create**.

### 4. Set Up API Key Authentication

1. Navigate to the **Plugins** tab for the `Kafka-Health-Route`.
2. Click **Install Plugin** and search for **Key Authentication**.
3. Use the default settings.
4. Click **Create**.

### 5. Create Consumer and Set Up Credentials

1. Navigate to **Consumers**.
2. Click **New Consumer** and fill in:
   - Username: `kafka-consumer`
   - Custom ID: `kafka-consumer-id`

3. Create credentials:
   - Open the consumer’s page.
   - Go to the **Credentials** tab.
   - Click **New Key Auth Credential**.
   - Set **Key** to `my-api-key`.
4. Click **Create**.

### 6. Testing

Test the route with the API key:

- **Browser**: `http://<KONG_IP>:8000/health/kafka?apikey=my-api-key`
- **Curl**:

  ```bash
  curl -i -X GET http://<KONG_IP>:8000/health/kafka --header "apikey: my-api-key"
  ```

### 7. Logging: File Log Plugin

1. Enable the plugin globally or on a specific service/route.
2. Install the **File Log Plugin** and configure the following:
   - **File Path**: `/var/log/kong/kong.log`
   - **Log Level**: `info` (or `debug`, `error`)
   - **Log Format**: `json` (optional)

3. To view logs:

   ```bash
   tail -f /var/log/kong/kong.log
   ```

---

## API Versioning

### Overview

Implementing API versioning allows you to manage different versions of your APIs for backward compatibility.

### Configuration Steps

1. **Route for v1**:
   - Name: `Kafka-Health-Route-v1`
   - Path: `/v1/health/kafka`
   - Service: `Kafka` (existing service)

2. **Route for v2**:
   - Name: `Kafka-Health-Route-v2`
   - Path: `/v2/health/kafka`
   - Service: `Kafka` (existing service)

### Testing the Versioned API

- Access v1: `GET https://api.example.com/v1/health/kafka`
- Access v2: `GET https://api.example.com/v2/health/kafka`

---

## Kong Gateway: OSS vs. Enterprise

Kong Gateway is available in two packages: **Open Source (OSS)** and **Enterprise**.

### Kong Gateway (OSS)

- **Key Features**:
  - Open-source package with basic API gateway functionalities.
  - Can be managed via Kong’s Admin API, Kong Manager OSS, or declarative configuration.

### Kong Gateway (Enterprise)

- **Key Features**:
  - Includes Kong Manager with advanced features.
  - **Free mode**: Adds Kong Manager to the OSS functionality.
  - **Enterprise mode**: Includes RBAC, enterprise plugins, and more.

### Feature Comparison

| Capability                               | Kong Manager Enterprise | Kong Manager OSS |
|------------------------------------------|-------------------------|------------------|
| Manage all workspaces in one place       | ✅                      | ❌               |
| Create and manage routes and services    | ✅                      | ✅               |
| Activate or deactivate plugins           | ✅                      | ✅               |
| Manage certificates                      | ✅                      | ✅               |
| Group services, plugins, consumers       | ✅                      | ❌               |
| Manage teams                             | ✅                      | ❌               |
| Centrally store and access key sets      | ✅                      | ✅               |

### Limitations of Kong OSS

- **Limited Access**: Free mode of Kong OSS limits functionality in Kong Manager.
- **No Multi-Workspace Management**: Only available in Enterprise.
- **No Advanced Grouping for Resources**: Kong OSS lacks grouping capabilities.
- **No Dedicated API Versioning**: Use flexible routing for versioning in Kong OSS.
- **No Team Collaboration Features**: Enterprise includes role-based access controls.
- **No Advanced Monitoring**: OSS lacks analytics tools available in Enterprise.
- **RBAC Restrictions**: Kong OSS lacks granular user permissions.
- **No Centralized Sensitive Data Management**: Enterprise provides enhanced data security.

---

## Conclusion

This guide provides a comprehensive installation and setup process for both Kong Gateway OSS and Enterprise. It covers database configuration, Kong service setup, API versioning, logging, and a detailed comparison between Kong OSS and Enterprise features.

