# Setup for Custom JMS Edge using Solace as JMS Provider

This repository contains a Docker Compose setup for running a custom JMS Edge environment with Solace PubSub+ Standard and webMethods Edge Runtime.

## Prerequisites

- Docker
- Docker Compose

### Build custom webMethods Edge Runtime base image for Solace JMS

Use the Dockerfile_base_jms to build a custom webMethods Edge Runtime image with all required JMS client libs for Solace.

```
docker build -f Dockerfile_base_jms . -t webmethods-edge-runtime-solace:11.0.10
```

##  Docker Compose Services

### solace

- **Container Name**: `solace-single-node`
- **Image**: `solace/solace-pubsub-standard:latest`
- **Ports**:
  - `8008:8008` (Web transport)
  - `1443:1443` (Web transport over TLS)
  - `1943:1943` (SEMP over TLS)
  - `1883:1883` (MQTT Default VPN)
- **Volumes**:
  - `storage-group:/var/lib/solace`
- **Restart Policy**: On failure, max attempts: 1

### is-edge

- **Container Name**: `is-edge`
- **Image**: `webmethods-edge-runtime-solace:11.0.10`
- **Environment Variables**:
  - `SAG_IS_CLOUD_REGISTER_URL=${SAG_IS_CLOUD_REGISTER_URL}`
  - `SAG_IS_EDGE_CLOUD_ALIAS=${SAG_IS_EDGE_CLOUD_ALIAS}`
  - `SAG_IS_CLOUD_REGISTER_TOKEN=${SAG_IS_CLOUD_REGISTER_TOKEN}`
- **Ports**:
  - `5558:5555`
- **Volumes**:
  - `./application.properties:/opt/softwareag/IntegrationServer/application.properties`
  - `./jms/git:/mnt/git`
  - `edge-msr-solace-packages:/opt/softwareag/IntegrationServer/packages`
- **Restart Policy**: On failure

## Usage

1. Clone this repository:
   ```sh
   git clone https://github.com/your-repo/custom-jms-edge.git
   cd custom-jms-edge````

2. Ensure the application.properties file is in the root directory of the repository.

3. Start the Docker Compose setup:
```
docker-compose --env-file .env up
```

where the .env file contains the following variables:

```
SAG_IS_CLOUD_REGISTER_URL=
SAG_IS_EDGE_CLOUD_ALIAS=
SAG_IS_CLOUD_REGISTER_TOKEN=
```

4. To stop the Docker Compose setup:

```
docker-compose  down
```

## Accessing Services
The primary service can be accessed via the following ports:

- Web transport: http://localhost:8008
- Web transport over TLS: https://localhost:1443
- SEMP over TLS: https://localhost:1943
- MQTT Default VPN: mqtt://localhost:1883
- The is-edge service can be accessed via the following port: http://localhost:5558

## Volumes
- storage-group: Persistent storage for the primary service.
- ./application.properties: Configuration file for the is-edge service.
- git: Mounted directory for the is-edge service.
- edge-msr-solace-packages: Packages directory for the is-edge service.

## Using Docker only (No Docker Compose)

### Solace

```
docker run -d \
  --name solace-single-node \
  -e username_admin_globalaccesslevel=admin \
  -e username_admin_password=admin \
  -e system_scaling_maxconnectioncount=100 \
  -v storage-group:/var/lib/solace \
  --shm-size=1g \
  --ulimit core=-1 \
  --ulimit nofile=2448:6592 \
  -p 8008:8008 \
  -p 1443:1443 \
  -p 1943:1943 \
  -p 1883:1883 \
  -p 8080:8080 \
  -p 55554:55555 \
  -p 8000:8000 \
  -p 5672:5672 \
  -p 9000:9000 \
  -p 2222:2222 \
  solace/solace-pubsub-standard:latest
```

### webMethods Edge Runtime

```
docker run -d \
  --name is-edge \
  -e SAG_IS_CLOUD_REGISTER_URL=<SAG_IS_CLOUD_REGISTER_URL> \
  -e SAG_IS_EDGE_CLOUD_ALIAS=<SAG_IS_EDGE_CLOUD_ALIAS> \
  -e SAG_IS_CLOUD_REGISTER_TOKEN=<SAG_IS_CLOUD_REGISTER_TOKEN> \
  -v ./application.properties:/opt/softwareag/IntegrationServer/application.properties \
  -v ./jms/git:/mnt/git \
  -v edge-msr-solace-packages:/opt/softwareag/IntegrationServer/packages \
  -p 5558:5555 \
  webmethods-edge-runtime-solace:11.0.10
```
