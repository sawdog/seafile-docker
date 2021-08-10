[![Build Status](https://secure.travis-ci.org/haiwen/seafile-docker.png?branch=master)](http://travis-ci.org/haiwen/seafile-docker)

### Why Fer?

Fork of the seafile docker images. I got tired of waiting and waiting with no pro updates and struggled
with finding any that actually work.  I plan to split things up and it a little cleaner behind traefik and update dependencies as much as possible. 

And there's some fscking dodgy stuff going on with some community member who think it's cool to take open source, put it behind their walled garden with locked up private git repos and ask for crypto currency to support him.
That right there buuuull.

## About
- [Docker](https://docker.com/) is an open source project to pack, ship and run any Linux application in a lighter weight, faster container than a traditional virtual machine.

- The base image configures Seafile with the Seafile team's recommended optimal defaults.

If you are not familiar with docker commands, please refer to [docker documentation](https://docs.docker.com/engine/reference/commandline/cli/).

### Getting Started

To login the seafile registry for images hosted on docher hub:

```sh
docker login
```

To run the seafile server container:

```sh
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  sawdog/seafile-pro:latest
```

Wait for a few minutes for the first time initialization, then visit `http://seafile.example.com` to open Seafile Web UI.

This command will mount folder `/opt/seafile-data` at the local server to the docker instance. You can find logs and other data under this folder.

### Put your licence file

If you have a `seafile-license.txt` licence file, simply put it in the folder `/opt/seafile-data/seafile/`. In your host machine:

```sh
mkdir -p /opt/seafile-data/seafile/
cp /path/to/seafile-license.txt /opt/seafile-data/seafile/
```

Then restart the container.

```sh
docker restart seafile
```

### More configuration Options

#### Custom Admin Username and Password

The default admin account is `me@example.com` and the password is `asecret`. You can use a different password  by setting the container's environment variables:
e.g.

```sh
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -e SEAFILE_ADMIN_EMAIL=me@example.com \
  -e SEAFILE_ADMIN_PASSWORD=a_very_secret_password \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  {pro-host}/sawdog/seafile-pro:latest
```

If you forget the admin password, you can add a new admin account and then go to the sysadmin panel to reset user password.

#### Let's encrypt SSL certificate
If you want to run Nginx with Let's Encrypt -- perfect -- see the example
docker-compose.yml.  Not in the seafile image though....

#### Modify Seafile Server Configurations

The config files are under `shared/seafile/conf`. You can modify the configurations according to [Seafile manual](https://manual.seafile.com/)

After modification, you need to restart the container:

```
docker restart seafile
```

#### Find logs

The seafile logs are under `/shared/logs/seafile` in the docker, or `/opt/seafile-data/logs/seafile` in the server that run the docker.

The system logs are under `/shared/logs/var-log`, or `/opt/seafile-data/logs/var-log` in the server that run the docker.

#### Add a new Admin

Ensure the container is running, then enter this command:

```
docker exec -it seafile /opt/seafile/seafile-server-latest/reset-admin.sh
```

Enter the username and password according to the prompts. You now have a new admin account.

### Directory Structure

#### `/shared`

Placeholder spot for shared volumes. You may elect to store certain persistent information outside of a container, in our case we keep various logfiles and upload directory outside. This allows you to rebuild containers easily without losing important information.

- /shared/seafile: This is the directory for seafile server configuration and data.
- /shared/logs: This is the directory for logs.
    - /shared/logs/var-log: This is the directory that would be mounted as `/var/log` inside the container. 
    - /shared/logs/seafile: This is the directory that would contain the log files of seafile server processes. For example, you can find seaf-server logs in `shared/logs/seafile/seafile.log`.

### Upgrading Seafile Server

TO upgrade to latest version of seafile server:

```sh
docker pull sawdog/seafile-pro:latest
docker rm -f seafile
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -e SEAFILE_ADMIN_EMAIL=me@example.com \
  -e SEAFILE_ADMIN_PASSWORD=a_very_secret_password \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  -p 443:443 \
  -p 8002:8002
  {pro-host}/sawdog/seafile-pro:latest
```

### Troubleshooting

You can run docker commands like "docker logs" or "docker exec" to find errors.

```sh
docker logs -f seafile
# or
docker exec -it seafile bash
```
rtificate, which does not exist by default.

### Upgrading Seafile Server

TO upgrade to latest version of seafile server:

```sh
docker pull {pro-host}/sawdog/seafile-pro:latest
docker rm -f seafile
docker run -d --name seafile \
  -e SEAFILE_SERVER_HOSTNAME=seafile.example.com \
  -e SEAFILE_ADMIN_EMAIL=me@example.com \
  -e SEAFILE_ADMIN_PASSWORD=a_very_secret_password \
  -v /opt/seafile-data:/shared \
  -p 80:80 \
  -p 443:443 \
  sawdog/seafile-pro:latest
```

### Troubleshooting

You can run docker commands like "docker logs" or "docker exec" to find errors.

```sh
docker logs -f seafile
# or
docker exec -it seafile bash
```
