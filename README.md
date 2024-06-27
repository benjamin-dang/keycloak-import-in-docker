# Simple Keycloak Import into a Container

This repository provides a comprehensive guide on how to start Keycloak with import files using Docker or Podman. Follow the detailed steps below to ensure successful execution of this guide.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Import Files in the Same Directory](#import-files-in-the-same-directory)
- [Docker Compose File](#docker-compose-file)
- [Start Container](#start-container)
- [Open Keycloak in Browser](#open-keycloak-in-browser)
- [Stopping the Container](#stopping-the-container)
- [Summary](#summary)
- [Appendix: Export Your Keycloak from a Container to the Host](#appendix-export-your-keycloak-from-a-container-to-the-host)
- [Topic Related Info](#topic-related-info)

## Prerequisites

- image of Keycloak, preferably matching the version of the import files.
- Import files of a Keycloak instance.
- Docker or Podman installed on your machine.

**Note:** in this guide I used the **quay.io/keycloak/keycloak:19.0.3** image for the container

## Getting Started

First, clone this repository:

```bash
git clone https://github.com/benjamin-dang/keycloak-import-in-docker.git
```

Then open the directory in your terminal or your preferred IDE, and launch a terminal within that directory.

## Import Files in the Same Directory

Create a folder for the import files in the same directory as the `docker-compose.yml` file. Your directory structure should resemble the following:

```
|--import
|   |--import-files.json
|   |--...(more-import-files)
|--docker-compose.yml
|--README.md
```

## Docker Compose File

Your `docker-compose.yml` file should be configured as follows:

```yaml
version: '3.8'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:19.0.3
    container_name: test-keycloak
    ports:
      - "8024:8080"
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      DB_VENDOR: h2
    volumes:
      - {PATH_TO_FOLDER_THAT_CONTAINS_IMPORT_FILES}:/opt/keycloak/data/import
      - ./h2:/opt/keycloak/data/h2
    command: -v start-dev ${IMPORT_COMMAND}
```

Replace the placeholder **{PATH_TO_FOLDER_THAT_CONTAINS_IMPORT_FILES}** with the actual path. For instance, if you created a folder named **import** as indicated previously, it should be:

```yaml
volumes:
      - ./import:/opt/keycloak/data/import
```  

## Start Container

To execute the import, start the container with the following command:

```bash
IMPORT_COMMAND='--import-realm' podman compose up
```

or, if using Docker:

```bash
IMPORT_COMMAND='--import-realm' docker-compose up
```

This command will also create an **h2** folder in your directory, which is used to persist Keycloak data between container restarts.

**Note:** Execute this command only during the initial startup (once). For any following start ups you should use the normal:
```
podman compose up
```
or, if using Docker

```
docker-composu up
```

Ignore the warning that IMPORT_COMMAND is not set and defaults to empty string, because this is what we want on the following start ups.



## Open Keycloak in Browser

Open your browser and navigate to [http://localhost:8024](http://localhost:8024) to verify if the import was successful.

## Stopping the Container

To stop the container, run:

```bash
podman compose down
```

or, if using Docker:

```bash
docker-compose down
```

This will halt the container, retaining any changes made, including the import, thanks to the mounted h2 database. This setup is intended for testing purposes and not for production use.

**Note:** To stop the container and start afresh, execute:

```bash
podman compose down --volumes
```

or, if using Docker:

```bash
docker-compose down --volumes
```

This will also delete the mounted h2 folder.

## Summary

Importing Keycloak data into a container is straightforward with the correct `docker-compose.yml` configuration and a few commands. For information on exporting your Keycloak data, see the section below.

## Appendix: Export Your Keycloak from a Container to the Host

To export your Keycloak data, ensure the container is running:

```bash
podman ps
```

or, if using Docker:

```bash
docker ps
```

Use the container ID to execute the following command:

```bash
podman exec {CONTAINER_ID} bash -c 'cd /opt/keycloak/bin/ && ./kc.sh export --dir=/export'
```

or, if using Docker:

```bash
docker exec {CONTAINER_ID} bash -c 'cd /opt/keycloak/bin/ && ./kc.sh export --dir=/export'
```

Then, copy the export files to your host:

```bash
podman cp {CONTAINER_ID}:/export/. ./kc-export-files/
```

or, if using Docker:

```bash
docker cp {CONTAINER_ID}:/export/. ./kc-export-files/
```

This command will copy the export files to your current directory, placing them in a folder named `kc-export-files`.

**Note:** Ensure the folder contains only `.json` files. Remove any other files or folders.

## Topic Related Info

For further reading and detailed documentation, refer to the following resources:

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Podman Documentation](https://podman.io/getting-started/)
- [Docker Documentation](https://docs.docker.com/get-started/)
