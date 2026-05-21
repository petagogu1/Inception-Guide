*This activity has been created as part of the 42 curriculum by P3T4G0GU.*

# Inception

## Description

Inception is a system administration project from the 42 curriculum. The goal is to deploy a fully working WordPress website running entirely inside Docker containers on a Virtual Machine, accessible via HTTPS.

The infrastructure is made up of three services, each running in its own container:

- **NGINX** — the only entry point into the infrastructure, handles all HTTPS traffic on port 443 using TLSv1.2/TLSv1.3, and forwards PHP requests to WordPress
- **WordPress + php-fpm** — runs the actual website
- **MariaDB** — stores all website data (users, posts, settings)

All containers are connected through a private Docker network, built from custom Dockerfiles based on Debian, and orchestrated with Docker Compose. Sensitive data like passwords is handled via Docker secrets — never hardcoded. Data persists across restarts using Docker volumes.

## Instructions

### Prerequisites

- A Virtual Machine running Debian
- Docker and Docker Compose installed
- `make` installed (`sudo apt-get install -y make`)


### Makefile commands you will use

| Command | Effect |
|---|---|
| `make` | Build images and start all containers |
| `make down` | Stop all containers |
| `make clean` | Stop containers and remove images and volumes |
| `make fclean` | Full reset — removes everything including host data |
| `make re` | Full reset and rebuild |

## Resources

- [Docker documentation](https://docs.docker.com/)
- [Docker Compose reference](https://docs.docker.com/compose/)
- [NGINX beginner's guide](https://nginx.org/en/docs/beginners_guide.html)
- [MariaDB documentation](https://mariadb.com/kb/en/documentation/)
- [WP-CLI handbook](https://make.wordpress.org/cli/handbook/)

