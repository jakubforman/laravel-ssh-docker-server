# Laravel for Docker

Contains Dockerfiles for Laravel applications, optimized for various PHP versions. Tools used in Dockerfile:
`Docker` `Synology NAS` `Portainer` `Laravel` `Redis` `PostgreSQL` `Pulse` `Nginx` `Supervisor` `PHP` `Composer` `SSH`

## Create a new stack in portainer

- Name it
- Select **Repository** and set https://github.com/jakubforman/laravel-ssh-docker-server
- Set repository reference `refs/heads/main`
- Add environment variables (e.g., `APP_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` ...)
- Add environment variable `APP_PATH` - in NAS create new folder for your project (domain name pattern) and set it as `APP_PATH` (e.g., `/volume1/docker/example.com`)
- Add environment variable `SSH_PUBLIC_KEY` - set your own key path (e.g., `ssh-ed25519 AAAAC3NzaC1l...... deployer-example.com`)
- Set up all other environment variables (some of the examples are in [.env.example](.env.example))
- Click `Deploy the stack`

## Domain DNS setup

- Set `CNAME` record for your domain to point to your NAS IP address or Synology NAS domain (e.g., `your-nas.synology.me`)

## In NAS

- Set a reverse proxy domain and port for your real domain (e.g., `https://example.com:443` > `https://localhost:<APP_PORT>`)
- Set a new certificate for a domain in Synology DSM > Security > Certificates
- Set the certificate for a domain service in Synology DSM > Security > Settings

## In router

- Static IP address
- Port forwarding (same port as in reverse proxy)
