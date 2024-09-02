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

2. **Access the DataSunrise Interface**:
   - Open a browser and navigate to: `http://<your-docker-host>:9999/ui/auth/login`.
   - Enter the admin password set via the `ADMIN_PASSWORD` environment variable.

3. **Configure Salesforce Connection**:
   - In the DataSunrise interface, go to `Configuration > Databases > Add Database`.
   - Select `Salesforce` as the database type.
   - Provide the Redis connection details and save the configuration.

4. **Create an Audit Rule**:
   - After configuring the connection, set up an audit rule to monitor Salesforce or ChatGPT sessions.
