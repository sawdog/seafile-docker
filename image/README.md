# Build image

1. modify `SEAFILE_VERSION=` in Dockerfile
2. copy dirs `scripts, cluster_scripts, services, templates` to the directory that contains Dockerfile
3. `docker build -t image_name:tag ./`
