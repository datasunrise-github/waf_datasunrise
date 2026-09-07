# DataSunrise WAF for Web Applications and Generative AI

This repository contains a Docker Compose deployment for monitoring Salesforce and Generative AI
traffic with DataSunrise. It runs DataSunrise eCAP, an HTTP proxy, Redis, Elasticsearch, and the
proxy-service UI on one Docker host.

Clients send HTTP and HTTPS traffic through the proxy exposed by the `datasunrise_ecap` service.
DataSunrise can then audit, secure, or mask supported web application traffic.

## Deployment Architecture

| Service | Purpose | Port |
| --- | --- | --- |
| `datasunrise_ecap` | DataSunrise and the HTTP proxy used for monitored traffic | `3128`, `11000` |
| `proxy-service` | Configuration service and UI for web application monitoring | `9999` |
| `redis` | Internal session and service data storage | `6379` on the Compose network |
| `elasticsearch` | Internal search and analytics storage | `9200` and `9300` on the Compose network |

The supplied Compose file is a self-contained deployment. DataSunrise eCAP is already included, so
do not deploy a separate eCAP service alongside it.

The Compose file does not connect eCAP to an existing DataSunrise management server and does not
configure shared Dictionary or Audit databases. If an existing DataSunrise deployment must be
reused, contact DataSunrise Support to confirm a supported architecture before changing the
Compose file.

## Prerequisites

- Docker Engine
- Docker Compose
- A DataSunrise license key, if required for the deployment
- Credentials for the web application or API that will be monitored

## Configure the Deployment

Open `docker-compose.yml` and replace the placeholder values before starting the services.

| Placeholder | Where it is used | Description |
| --- | --- | --- |
| `<REDIS_PASSWORD>` | `redis`, `proxy-service`, `datasunrise_ecap` | Use the same Redis password in all three services. |
| `<ELASTICSEARCH_PASSWORD>` | `elasticsearch`, its health check, `proxy-service` | Use the same password in all three locations. |
| `<ADMIN_PASSWORD>` | `proxy-service` | Password for the proxy-service UI. |
| `<LICENSE_KEY_OPTIONAL>` | `datasunrise_ecap` | DataSunrise license key. |
| `<DS_ADMIN_PASSWORD_OPTIONAL>` | `datasunrise_ecap` | Password for the DataSunrise administrator account. |

By default, `proxy-service` loads its configuration from this repository:

```yaml
CONFIG_SOURCE=GIT
CONFIG_REPO_URL=https://github.com/datasunrise-github/waf_datasunrise
```

To use a local configuration, set `CONFIG_SOURCE=LOCAL` and mount `config.json` and the required
scripts into `/opt/proxy/scripts/` in the `proxy-service` container.

## Start the Deployment

From the directory containing `docker-compose.yml`, run:

```bash
docker compose up -d
```

Check the service status:

```bash
docker compose ps
```

### Docker Compose Project Name

Docker Compose uses a project name to identify the resources created for a deployment. By default,
the project name is based on the directory that contains the Compose file.

To set it explicitly, use either of these methods:

```bash
docker compose -p datasunrise-waf up -d
```

or create a `.env` file next to `docker-compose.yml`:

```dotenv
COMPOSE_PROJECT_NAME=datasunrise-waf
```

Use the same project name in later Compose commands. The commands in this guide refer to Compose
service names, so you do not need to determine generated container names.

## Configure the HTTP Proxy and CA Certificate

Complete these steps once for each client that sends monitored traffic through DataSunrise.

1. Configure the client to use the following HTTP and HTTPS proxy:

   ```text
   Host: <docker-host>
   Port: 3128
   ```

   Replace `<docker-host>` with the hostname or IP address of the Docker host. Use `127.0.0.1` when
   the client runs on that host.

2. Copy the proxy CA certificate to the current directory:

   ```bash
   docker compose cp datasunrise_ecap:/home/datasunrise/ssl/squidCA.pem ./squidCA.pem
   ```

3. Trust `squidCA.pem` on the client:

   - For a browser, import it into the browser or operating system trust store.
   - For a command-line client, specify it with the client's CA certificate option or environment
     variable.

The proxy decrypts HTTPS traffic for inspection. The client will reject the proxy certificate
unless its CA certificate is trusted.

## Configure DataSunrise

1. Open the DataSunrise Web Console:

   ```text
   https://<docker-host>:11000
   ```

2. Go to **Configuration** -> **Databases** and click **Add Database**.

3. Configure the connection:

   | Field | Value |
   | --- | --- |
   | Database Type | `Salesforce` for Salesforce, or `Generative AI` for ChatGPT, Claude, Amazon Bedrock, and Azure OpenAI |
   | Hostname or IP | `redis` |
   | Port | `6379` |
   | Database user name | `default` |
   | Password | The value configured for `<REDIS_PASSWORD>` |

4. Select **Save in DataSunrise** or **Retrieve** as the password storage method, then save the
   connection.

These settings connect DataSunrise to the Redis service used by the proxy. They are not the
credentials for Salesforce or a Generative AI provider.

5. Create the required Audit, Security, or Dynamic Masking Rules for the connection.

## Monitor Salesforce

1. Open the proxy-service UI:

   ```text
   http://<docker-host>:9999/ui/auth/login
   ```

2. Sign in with the password configured for `<ADMIN_PASSWORD>`.
3. Go to **Applications** -> **Add Salesforce** and enter the Salesforce username, password, and
   TOTP secret.
4. Open Salesforce from a browser configured to use the DataSunrise proxy.
5. Verify the activity under **Audit** -> **Transactional Trails** in DataSunrise.

## Monitor ChatGPT or Claude

1. Configure the browser to use the DataSunrise proxy and trust `squidCA.pem`.
2. Open ChatGPT or Claude and send a prompt.
3. Verify the activity under **Audit** -> **Transactional Trails** in DataSunrise.

## Monitor Amazon Bedrock

Configure AWS credentials and a Region that can access Amazon Bedrock. Then configure the AWS
client to use the proxy and its CA certificate:

```bash
export HTTP_PROXY="http://<docker-host>:3128"
export HTTPS_PROXY="http://<docker-host>:3128"
export AWS_CA_BUNDLE="/path/to/squidCA.pem"
```

Run an Amazon Bedrock Runtime request and verify it under **Audit** -> **Transactional Trails** in
DataSunrise. AWS credentials remain in the AWS client configuration and are not entered in the
DataSunrise connection.

See the AWS CLI [`invoke-model` reference](https://docs.aws.amazon.com/cli/latest/reference/bedrock-runtime/invoke-model.html)
for request syntax.

## Monitor Azure OpenAI

Set the full Azure OpenAI request endpoint and API key:

```bash
export AZURE_OPENAI_ENDPOINT="<Azure-OpenAI-request-endpoint>"
export AZURE_OPENAI_API_KEY="<Azure-OpenAI-API-key>"
```

Send a request through the proxy and verify the connection with `squidCA.pem`:

```bash
curl "$AZURE_OPENAI_ENDPOINT" \
  --proxy "http://<docker-host>:3128" \
  --cacert "/path/to/squidCA.pem" \
  --header "Content-Type: application/json" \
  --header "api-key: $AZURE_OPENAI_API_KEY" \
  --data '{"messages":[{"role":"user","content":"1+1"}]}'
```

Verify the request under **Audit** -> **Transactional Trails** in DataSunrise. The endpoint and API
key remain in the Azure OpenAI client configuration.

## Operations

Restart the DataSunrise eCAP service after changing its configuration:

```bash
docker compose restart datasunrise_ecap
```

Stop and remove the deployment:

```bash
docker compose down
```

If you set an explicit project name with `-p`, include it in these commands as well.

## Troubleshooting

If monitored traffic does not appear in DataSunrise, check the following:

- All services are running in `docker compose ps`.
- The client uses `<docker-host>:3128` as its HTTP and HTTPS proxy.
- The client trusts `squidCA.pem`.
- The DataSunrise connection uses `redis` on port `6379` and the configured Redis password.
- The required Audit, Security, or Dynamic Masking Rule is enabled for the connection.

View service logs with:

```bash
docker compose logs <service-name>
```

## References

- [Docker Compose project names](https://docs.docker.com/compose/how-tos/project-name/)
- [`docker compose cp`](https://docs.docker.com/reference/cli/docker/compose/cp/)
- [Using an HTTP proxy with AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-proxy.html)
- [AWS CLI environment variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)
