Kong Gateway is a lightweight, fast, and flexible cloud-native API gateway. An API gateway is a reverse proxy that lets you manage, configure, and route requests to your APIs.

Kong Gateway Installation, Configuration, and Setup Documentation

Step 1: Install PostgreSQL

Update the package list:

sudo apt update

Install PostgreSQL:

sudo apt install postgresql

Check PostgreSQL service status:

sudo systemctl status postgresql

Step 2: Create PostgreSQL User and Database

Log into PostgreSQL shell:

sudo -u postgres psql

Create a user:

CREATE USER kong WITH PASSWORD 'super_secret';

Create a database:

CREATE DATABASE kong OWNER kong;

Exit PostgreSQL:

\q

Step 3: Install Kong Gateway

Download the Kong package:

curl -Lo kong-enterprise-edition-3.8.0.0.deb "https://packages.konghq.com/public/gateway-38/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_3.8.0.0/kong-enterprise-edition_3.8.0.0_$(dpkg --print-architecture).deb"

Install Kong:

sudo apt install -y ./kong-enterprise-edition-3.8.0.0.deb

Optional: Fixing Permission Issues

If you encounter permission issues while accessing the downloaded .deb file, you can follow these steps:

Download to /tmp:

curl -Lo /tmp/kong-enterprise-edition-3.8.0.0.deb "https://packages.konghq.com/public/gateway-38/deb/ubuntu/pool/jammy/main/k/ko/kong-enterprise-edition_3.8.0.0/kong-enterprise-edition_3.8.0.0_$(dpkg --print-architecture).deb"

Install the Package:

sudo apt install -y /tmp/kong-enterprise-edition-3.8.0.0.deb

Remove the Temporary File:

sudo rm /tmp/kong-enterprise-edition-3.8.0.0.deb

Step 4: Configure PostgreSQL Authentication

Edit the pg_hba.conf file:

sudo nano /etc/postgresql/12/main/pg_hba.conf

Update authentication to md5:

local   all             all                                     md5
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
host    all             all             0.0.0.0/0               md5

Set PostgreSQL to listen on all IPs:

Open postgresql.conf:

sudo nano /etc/postgresql/12/main/postgresql.conf

Modify the line:

listen_addresses = '*'

Restart PostgreSQL:

sudo systemctl restart postgresql

Step 5: Configure Kong

Copy the default configuration file:

sudo cp /etc/kong/kong.conf.default /etc/kong/kong.conf

Edit the configuration file:

sudo nano /etc/kong/kong.conf

Modify the database settings:

database = postgres
pg_host = 127.0.0.1
pg_port = 5432
pg_user = kong
pg_password = super_secret
pg_database = kong

Step 6: Run Database Migrations

Run migrations:

kong migrations bootstrap -c /etc/kong/kong.conf

Step 7: Start Kong Gateway

Start Kong:

kong start -c /etc/kong/kong.conf

Check Kong status:

curl -i http://localhost:8001

Kong Services Setup

1. Create Gateway Service

Steps:

Log into Kong Manager and select the default workspace.

Navigate to Services.

Click on + Add Service.

Fill in the fields:

Name: Kafka-Health-Service (or any relevant name)

Tags: example (or any relevant tags)

Service Endpoint:

Choose one of the following options:

Full URL:

Enter the complete URL (e.g., http://api-dev.catalystai.work/health/kafka).

Protocol, Host, Port, and Path:

Protocol: http

Host: api-dev.catalystai.work

Path: /health/kafka

Port: 80

Click Create.

2. Create Route

Steps:

In Kong Manager, navigate to Routes.

Click on + Add Route.

Fill in the details:

Name: Kafka-Health-Route

Service: Select Kafka-Health-Service.

Paths: /health/kafka

Click Create.

3. Set Up Rate Limiting

Steps:

Navigate to the Plugins tab for the Kafka-Health-Route.

Click on Install Plugin.

Search for Rate Limiting and click Enable.

Configure:

Limit: 10 requests per minute.

Click Create.

4. Set Up API Key Authentication

Steps:

Navigate to the Plugins tab for the Kafka-Health-Route.

Click on Install Plugin.

Search for Key Authentication and click Enable.

Use the default settings.

Click Create.

5. Create Consumer and Set Up Credentials

Steps:

Navigate to Consumers.

Click on New Consumer.

Fill in:

Username: kafka-consumer

Custom ID: kafka-consumer-id

Click Create.

Open the consumer’s page, go to Credentials tab.

Click New Key Auth Credential.

Set Key to my-api-key.

Click Create.

6. Testing

To Test Route with API Key:

Browser:

http://<KONG_IP>:8000/health/kafka?apikey=my-api-key

curl:

curl -i -X GET http://<KONG_IP>:8000/health/kafka --header "apikey: my-api-key"

7. Logging: File Log Plugin

1. Enable the Plugin

Scope: Choose to enable the plugin globally or on a specific service/route.

Navigate to: Plugins tab of the chosen Service or Route.

2. Install the File Log Plugin

Click on Install Plugin.

Select File Log from the list.

3. Configure Plugin Settings

File Path: Specify where logs will be stored.

/var/log/kong/kong.log

Log Level: Set the desired log level (e.g., info, debug, error).

Log Format (optional): Define the format of the logs (e.g., json).

Viewing Logs

SSH into the Kong server and use:

tail -f /var/log/kong/kong.log

8. API Versioning

Overview

To implement API versioning for a single service (e.g., Kafka) in Kong, you need to create separate routes for each version. This documentation provides the configuration details for setting up versioned routes.

Configuration Steps

Create Routes for Each API Version

Route for v1

Name: Kafka-Health-Route-v1

Path: /v1/health/kafka

Service: Kafka (existing service)

Route for v2

Name: Kafka-Health-Route-v2

Path: /v2/health/kafka

Service: Kafka (existing service)

Testing the Versioned API

Access v1:

GET https://api.example.com/v1/health/kafka

Access v2:

GET https://api.example.com/v2/health/kafka

This documentation provides a comprehensive guide to configuring Kong Gateway, setting up services and routes, implementing API versioning, and testing the setup. 
