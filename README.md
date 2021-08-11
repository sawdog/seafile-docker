[![Build Status](https://secure.travis-ci.org/haiwen/seafile-docker.png?branch=master)](http://travis-ci.org/haiwen/seafile-docker)

## Why Fork It?

I forked this for a few reasons. Firstly, I got tired of waiting and waiting with no pro updates.  When asking about them on the forums, nothing.  Rafio silence. Then I found a few docker images out there -- one is dodgy as fsck and he's asking for support. But his repacking of an open source project which he tries to get money from is.... closed.  No thanks.

Finally, This works.  I am using it.  
 

### How to use with Seafile Pro 7.x
Note: I started with a fresh docker container based of this docker imagbes.  If you want to use your current container, you may just have to deal with symlink in 1 case.  Try it and let me know.


### Existing DB w/ New Host Container
1. Run 7.x to 8.x upgrade script (i did this within my existing 7.1.17 container)
2. Turn off existing seafile-pro 7 container.
3. Clear old containers from locakl repo (be aware if you're pesisting changes
 in the container, they will be lost.  Don't persist anything in these containers is my recommendation.)
4. Start new container from the new seafile-pro 8.x images.
5. Connect to your new container (you want a bash shell).
6. Run seafile-server-latest/setup-seafile.sh

To Support a single container and reuse of existing NGINX / Proxy these steps are required so that the Nginx container has access to the necessary media files.
1. Stay in container or get a bash shell in the container.
2. Move mv seafile-server-latest/seahub/media /shared/seafile
8. To make things consistent, symlink to /opt/seafile along with the other 
packages that you see there.  cd /opt/seafile; ln -s /shared/seafile/media .
9. Symlink media from shared back into the pseahub package. 
  cd seafile-server-latest/seahub; ln -s /opt/seafile/media .
10. Fix symlinks from this mess in media rm media/avatars; rm media/custom;
cd media; ln -s ../seahub-data/avatars .; ln -s ../seahub-data/custom .

This is the most right I could figure out; it follows existing patterns and
works in and out of the container. I then mount this media directory 
on my Nginx proxy which also handles proxy responsibility for a dozen other containers as well.


After the static files are mountable for your proxy, restart that and you  should be running a new, clean 8.x pro instance against your existing database and filestorage.

### Todo
It's still a bit messy, the scrips are over engineered for what's needed -- and then trying to trace it through is a pita.  For how this works. so they do one job, and that's simple and consistent.

Moving forward I plan to move proxy on to Traefik. I expect it to be as ore more simple to use as a proxy - but my goal is that whatever proxy container or mechanism will *just work*.

All of this should be simplified and moved into the container ]seafile container via the Dockerfile, which I plan to do. The above steps should be moot and the task will be -- mount a new volume to your proxy!

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
