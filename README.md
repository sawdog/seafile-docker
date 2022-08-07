[![Build Status](https://secure.travis-ci.org/haiwen/seafile-docker.png?branch=master)](http://travis-ci.org/haiwen/seafile-docker)

## Why Fork It?

I forked this for a few reasons:
1. I got tired of waiting and waiting for Docker to get pro updates. 
  When asking about them on the forums, it was crickets and radio silence.
  I found a few docker images out there, but I felt were either too dodgy 
  or  they plain didn't work.  No thanks.
2. This works.  I am using it.
3. I want to be able to solve my own problems and simplify the layout and steps.

### Seafile Pro 7.x
Note: I started with a fresh docker container based of this docker a Pro 7.x image.
  If you want to use your current container, you may just have to deal with 
 symlink in 1 case.  Try it and let me know.

### Existing DB w/ New Host Container
Run 7.x to 8.x upgrade script (i did this within my existing 7.1.17 container)
1. Turn off the 7.x container.
```shell
docker-compose down
```
This will clear the containers from local repo (be aware if you're persisting changes
 in the container, they will be lost.  Don't persist anything in these containers is my recommendation.)
2. Start new container from the new seafile-pro 8.x image in your docker-compose.
```shell
docker-compose up -d
```
3Connect to your new running container.
```shell
docker exec -it seafile /bin/bash
```
4. Rerun the setup script
```shell
./seafile-server-latest/setup-seafile.sh
```

To Support a single container and reuse of existing NGINX / Proxy these steps are required so that the Nginx container has access to the necessary media files.
1. Stay logged into the container or re-login
2. Move Media files:
```shell
mv seafile-server-latest/seahub/media /shared/seafile
```
8. To make things consistent symlink to /opt/seafile along with the other 
packages that you see there.
```shell
cd /opt/seafile; ln -s /shared/seafile/media .
```
9. I add another symlink media from shared location to the orinal location in seahub. 
```shell
  cd seafile-server-latest/seahub; ln -s /opt/seafile/media .
```
10. Not sure why this mess came about - will relook and see if it's the process; but you
 have to fix symlinks in the media directory: 
```shell
cd /share/seafile/media; rm media/avatars; rm media/custom;
cd media; ln -s /share/seafile/seahub-data/avatars .; ln -s /share/seafile/seahub-data/seahub-data/custom .
```

This seemed the best way for my setup that I could figure out. It it follows 
the existing setup patterns and works in and out of the container if you mount the
 volume from the host as I do. 

Finally I mount this media directory 
on in my Nginx proxy container where it can then serve those static media files.
*Note*
If you're running a different proxy setup, e.g. Nginx on a different host, this is not going to 
work. I'll be thinking about it when i move over to Traefik.


After the static files are mountable for your proxy, restart that and you  should be running a new, clean 8.x pro instance against your existing database and filestorage.

## Todo

It's still a bit messy, the scripts are hard to follow for what's needed.
Tracing errors and fixing then is still a pita, but think 9.0.5 and 9.0.6 images I've made 
are few less messy. They still work and continue towards simple and consistent.

### Traefik
Moving forward I plan to move proxy on to Traefik. I expect it to be as ore more simple to use as a proxy - but my goal is that whatever proxy container or mechanism will *just work*.

All of this should be simplified and moved into the container ]seafile container via the Dockerfile, which I plan to do. The above steps should be moot and the task will be -- mount a new volume to your proxy!

## Gotchas

### 9.0.6
I had to install redis page in my container, so it's missing from the base image and the python packages which are included in pro/...

#### Authentication for Elasticsearch (7.16)
You have to turn on security - and in doing so causes Seafile indexing to fail.  This is easy to fix, but currently it's a manual step:

1. Add to your elasticsearch docker-compose.yml environment for elasticsearch
        - xpack.security.enabled=true
2. Loging to the elasticsearch container:
```sh
docker exec -it elasticsearch /bin/sh
```
3. Setup passwords for the built-in elastic accounts:
```sh
./bin/elasticsearch-setup-passwords auto
```
4. Copy the output from the password setup into a secure location so that you
    can use them as necessary.  Take note of the 'elastic' user for using in seafile
5. Login to the seafile container:
```shell
docker exec -it seafile /bin/sh
```
6. Add Authentication support for the Elasticsearch in seafile.
  Edit the connection.py file here `root@2f6b9830d50a:/opt/seafile/seafile-server-latest/pro/python/seafes/connection.py` to the following:
```python
def es_get_conn():
    es = Elasticsearch(['{}:{}'.format(seafes_config.host, seafes_config.port)], http_auth=('elastic', ''YOUR_PASSWORD'), maxsize=50, timeout=30)
    return es
```
7. Restart the container.

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
