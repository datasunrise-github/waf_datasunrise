# DataSunrise: Deploying a Salesforce and ChatGPT Connection

This guide covers the deployment architecture, component parameters, system requirements, and first-time login steps for setting up a connection between Salesforce and ChatGPT using DataSunrise.

## Architecture Overview

The deployment includes the following key components:

1. **Proxy Service**:
   - Handles connections and routing between Salesforce, ChatGPT, and other services.
   - Configurable via Docker environment variables.
   
2. **Redis**:
   - Used for session management and caching.
   
3. **Elasticsearch**:
   - Provides search and analytics capabilities.
   
4. **DataSunrise ECAP**:
   - Central component for security and monitoring.
   - Manages license keys and administrative access.

## Deployment Parameters

### Proxy Service

- **ADMIN_PASSWORD**: Password for DataSunrise admin access.
- **CONFIG_REPO_URL**: URL for the configuration repository (default: DataSunrise GitHub repository).
- **CONFIG_SOURCE**: You can change source from repository to local destination with option `LOCAL`. In this case,
                     will be used the configuration in `/opt/proxy/scripts/` directory inside the proxy-service container. Put `config.json` inside
                     the container during startup by using volumes.

### DataSunrise ECAP

- **DS_LICENSE_KEY**: DataSunrise license key.
- **DS_ADMIN_PASSWORD**: Admin password for DataSunrise.
  
### Optional Database Configurations

- **DICTIONARY_TYPE**: Type of the remote dictionary database.
- **DICTIONARY_HOST**: Hostname or IP of the dictionary database.
- **DICTIONARY_PORT**: Port for the dictionary database.
- **DICTIONARY_DB_NAME**: Name of the dictionary database.
- **DICTIONARY_LOGIN**: Username for the dictionary database.
- **DICTIONARY_PASS**: Password for the dictionary database.
- **AUDIT_TYPE**: Type of the remote audit database.
- **AUDIT_HOST**: Hostname or IP of the audit database.
- **AUDIT_PORT**: Port for the audit database.
- **AUDIT_DB_NAME**: Name of the audit database.
- **AUDIT_LOGIN**: Username for the audit database.
- **AUDIT_PASS**: Password for the audit database.

## System Requirements

- **Docker**: Ensure Docker is installed and running on your system.
- **Docker Compose**: Required for orchestrating the services.
- **DataSunrise License**: Valid license key for DataSunrise.
- **Salesforce Credentials**: Valid Salesforce account credentials for integration.

## First-Time Login and Usage

1. **Start the Deployment**:
   - Navigate to the directory containing the `docker-compose.yml`.
   - Run the following command to start the infrastructure:
     ```bash
     docker-compose up -d
     ```

2. **To audit the Salesforce or ChatGPT sessions through the squid-proxy, do the following**:
   - Indicate the Host `<yourdockerhost>` and the Port `3128` in your browser or Network System Settings.
   - The squid certificate is located in the `datasunrise_ecap-1` docker container, in the `/home/datasunrise/ssl/squidCA.pem` directory.
   - Copy a file from a docker to a host directory <destiny> with the following command:
      ```bash
      docker cp <project name>-datasunrise_ecap-1:/home/datasunrise/ssl/squidCA.pem <destiny>
      ```

3. **To configure the proxy-service, do the following**:

   **Important**: This is required only for Salesforce settings.

   - Open `http://<your docker host>:9999/ui/auth/login` in your browser.
   - Enter the admin password set via the `ADMIN_PASSWORD` environment variable.
   - In the Applications - Salesforce Connections add new connection. Specify username, password and TOTP Secret from your Salesforce account.

5. **Configure Salesforce instance**:

   In DataSunrise go to the Configuration#Databases#Add Database. Input the following information to connect Redis service:

      - Database Type (Salesforce);
      - Hostname or IP (Redis);
      - Port;
      - Database user name;
      - Password (the Redis password).
   
   **Important**: You need to choose one of the two Save Password methods: Save in DataSunrise or Retrieve.

6. **After configuring the connection**:
   
   Set up an audit rule to monitor Salesforce or ChatGPT sessions. 
