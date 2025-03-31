# DataSunrise: Creating a Web Application and Generative AI Connection

This guide covers the deployment architecture, component parameters, system requirements, and first-time login steps for setting up a connection between Salesforce, ChatGPT, Amazon Bedrock, and Azure OpenAI using DataSunrise.

## Table of Contents
1. [Architecture Overview](#architecture-overview)  
2. [Deployment Parameters](#deployment-parameters)  
      - [Proxy Service](#proxy-service)  
      - [DataSunrise ECAP](#datasunrise-ecap)  
      - [Optional Database Configurations](#optional-database-configurations)  
3. [Salesforce](#salesforce)  
4. [ChatGPT](#chatgpt)  
5. [Amazon Bedrock](#amazon-bedrock)  
6. [Azure OpenAI](#azure-openai)  

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

## Salesforce

### Prerequisites

- **Docker**: Ensure Docker is installed and running on your system.
- **Docker Compose**: Required for orchestrating the services.
- **DataSunrise License**: Valid license key for DataSunrise.
- **Salesforce Credentials**: Valid Salesforce account credentials for integration.

Start the infrastructure from the directory with the docker-compose.yml with the following command:

```bash
 docker-compose up -d
```
### To audit the Salesforce or ChatGPT sessions through the squid-proxy, do the following:
   1)  Indicate the Host <yourdockerhost> and the Port 3128 in your browser or Network System Settings.
   2) The squid certificate is located in the datasunrise_ecap-1 docker container, in the '/home/datasunrise/ssl/'
 squidCA.pem directory.

   3) Copy a file from a docker to a host directory <destiny> with the following command:
      ```bash
      docker cp <project name>-datasunrise_ecap-1:/home/datasunrise/ssl/squidCA.pem <destiny>
      ```
### To configure the proxy-service, do the following:

In DataSunrise go to the Configuration→Databases→Add Database. Input the following information to connect Redis service:
 Database Type (Salesforce);
 Hostname or IP (Redis);
 Port;
 Database user name;
 Password (the Redis password).

   4) Open http://<your docker host>:9999/ui/auth/login in your browser.
   5) In the Applications → Add Salesforce add a new connection. Specify the username, password, and TOTP Secret from your Salesforce account.

In DataSunrise go to the Configuration → Databases → Add Database. Input the following information to connect Redis service:
   - Database Type (Salesforce);
   - Hostname or IP (Redis);
   - Port;
   - Database user name;
   - Password (the Redis password).
   
## ChatGPT

### Prerequisites:
   - Docker: Ensure Docker is installed and running on your system.
 	
   - Docker Compose: Required for orchestrating the services.
 	
   - DataSunrise License: Valid license key for DataSunrise.

Start the infrastructure from the directory with the docker-compose.yml with the following command: 

```bash
docker-compose up -d 
```

### To audit the sessions through the squid-proxy, do the following:
   1) Indicate the Host and the Port 3128 in your browser or Network System Settings. 
   2) The squid certificate is located in the datasunrise_ecap-1 docker container, in the /home/datasunrise/ssl/ squidCA.pem directory. 
   3) Copy a file from a docker to a host directory with the following command: 
      ```bash
      docker cp -datasunrise_ecap-1:/home/datasunrise/ssl/squidCA.pem 
      ```

### To configure the proxy-service, do the following: 

   1) In DataSunrise go to the Configuration→Databases→Add Database.
   2) Input the following information to connect Redis service:
         - Database Type (Generative AI); 
         - Hostname or IP (Redis); 
         - Port;
         - Database user name; 
         - Password (the Redis password).

**Important:** You need to choose one of the two Save Password methods: Save in DataSunrise or Retrieve.

## Amazon Bedrock

### Prerequisites:
   - Docker: Ensure Docker is installed and running on your system.
 	
   - Docker Compose: Required for orchestrating the services.
 	
   - DataSunrise License: Valid license key for DataSunrise.

   - AWS CLI: Download AWS CLI from the official [website](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
 
 To configure DataSunrise to connect to Bedrock with AWS CLI, do the following:

   1) Add the Bedrock profile to the AWS configuration:

       [profile bedrock] aws_access_key_id = aws_secret_access_key = region = output = json 

   2) Create a folder to run AWS CLI. 
   3) In DataSunrise go to the Configuration→Databases→Add Database. Input the following information to connect Redis service:
   - Database Type (Generative AI);
   - Hostname or IP (Redis);
   - Port; 
   - Database user name; 
   - Password (the Redis password).

**Important:** You need to choose one of the two Save Password methods: Save in DataSunrise or Retrieve. After all the configuration is done, you will be able to create an Audit Rule.

## Azure OpenAI

### Prerequisites:
   - Docker: Ensure Docker is installed and running on your system.
 	
   - Docker Compose: Required for orchestrating the services.
 	
   - DataSunrise License: Valid license key for DataSunrise.


   1. Add the Azure OpenAI profile to the  Azure configuration:
         ```bash
         export AZURE_OPENAI_ENDPOINT="" export AZURE_OPENAI_API_KEY=""
         ```
   2. Execute the following command:
         ```bash
          curl $AZURE_OPENAI_ENDPOINT --proxy http://127.0.0.1:3128
          -H "Content-Type: application/json"
          -H "api-key: $AZURE_OPENAI_API_KEY"  
         -k -d "{\"messages\":[{\"role\": \"user\", \"content\": \"1+1\"}]}" 
         ```
**Note:** it is mandatory to indicate the --proxy http://127.0.0.1:3128 to process prompts through DataSunrise. 

   3. In DataSunrise go to the Configuration→Databases→Add Database. Input the following information to connect Redis service: 
   - Database Type (Generative AI);
   - Hostname or IP (Redis);
   - Port; 
   - Database user name; 
   - Password (the Redis password).

**Important:** You need to choose one of the two Save Password methods: Save in DataSunrise or Retrieve. After all the configuration is done, you will be able to create an Audit Rule.
