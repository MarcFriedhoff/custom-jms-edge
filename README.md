# Setup for Custom JMS Edge using Solcace as JMS Provider

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
  - `SAG_IS_CLOUD_REGISTER_URL=https://psdev.int-aws-de.webmethods.io`
  - `SAG_IS_EDGE_CLOUD_ALIAS=EdgeRuntime_mfri_edge`
  - `SAG_IS_CLOUD_REGISTER_TOKEN=bebda863776845bbb0e97f0bec614c14298c2469169a443d98f1c96117a9774d`
- **Ports**:
  - `5558:5555`
- **Volumes**:
  - `./application.properties:/opt/softwareag/IntegrationServer/application.properties`
  - `/Users/mfri/Work/customer/enexis/training/jms/git:/mnt/git`
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
docker-compose up
```

4. To stop the Docker Compose setup:

```
docker-compose down
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