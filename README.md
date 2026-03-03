# Nextcloud Docker stack
A performance-optimized Nextcloud deployment using Docker Compose. This stack includes a PostgreSQL database, Redis for memory caching, and a dedicated cron container for background tasks. It is pre-configured for use with a Traefik reverse proxy, also running in Docker.

## Features
- Database: PostgreSQL 16 (Alpine-based)
- Caching: Redis for improved file locking and performance
- Background jobs: Dedicated cron container for reliable system maintenance
- Security: Pre-configured Traefik labels for HSTS and .well-known DAV redirects
- Utilities: Custom scripts for log management and database backups

## Prerequisites
Before starting, ensure you have:
- Docker
- An existing Docker network named proxy-net (used by Traefik)
- Traefik running in Docker - use my [trafik-docker repo](https://github.com/grimdork/traefik-docker), which is made for this
- A .env file in the root directory

### Environment Variables (.env)
The included `.env` file needs to be edited for your instance of Nextcloud.

```ini
BASENAME=nextcloud
POSTGRES_USER=nextcloud
POSTGRES_PASSWORD=your_secure_password
POSTGRES_DB=nextcloud
DOMAIN=cloud.yourdomain.com
NEXTCLOUD_DATA=./nextcloud_data
RESOLVER=acmeweb # Default in my Traefik container
```

## Installation & Setup
### Make sure Traefik is running
It should be running on the same network as Nextcloud is configured to.

### Copy the source directory
```bash
cp -r nextcloud-docker /home/docker
cd /home/docker/nextcloud-docker
rm -rf .git .github
```

### Point DNS
Make one or more DNS records pointing to your server/VPS (most likely A and AAAA).

### Spin up the containers
```bash
docker compose up -d
```

### Initial Configuration
- Navigate to your ${DOMAIN} in a browser
- Create your admin Account
- Since the database variables are passed via the environment, Nextcloud should detect the PostgreSQL setup automatically

### Run the `post-setup` script
Once the admin user is created and you can log in, run the included post-setup script to enable Redis, internal cron, and apply various system fixes:
```bash
chmod +x post-setup
./post-setup
```

## Maintenance Scripts
This repository includes several utility scripts to simplify management:

|Script|Description|
|------|-----------|
|./clearlog|Clears the nextcloud.log from the main container to save space|
|./dumpdb|Dumps the Postgres database into the ./backups directory. Run this before upgrades!|
|./restoredb|Restores a database dump from the ./backups directory|
|./post-setup|Configures Redis, cron, and performance tweaks after initial install|

## Backup & Recovery
Before performing major upgrades (e.g., changing the Nextcloud image version), always run a database dump:

```bash
./dumpdb
```

The backup will be stored in the ./backups folder. To restore, ensure the containers are running and execute `./restoredb`.

## Technical Notes
- PHP Memory Limit: Set to 2G to support the Nextcloud Office suite
- Upload limit: Set to 10G. Note that you may also need to adjust your Traefik/Nginx proxy "max body size" to match
- Log rotation: All containers are limited to 3 files of 10MB each to prevent disk exhaustion
