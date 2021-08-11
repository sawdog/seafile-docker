server_version=8.0.8

base_image=sawdog/seafile-pro-base-server-image:1.0.0
base_image_squashed=sawdog/seafile-pro-base-server-image:1.0.0-squashed
server_image=sawdog/seafile-pro:$(server_version)
server_image_squashed=sawdog/seafile-pro:$(server_version)-squashed
latest_server_image=sawdog/seafile-pro:latest

all:
	@echo
	@echo Pleaes use '"make base"' or '"make server"' or '"make push"'.
	@echo

base:
	docker pull phusion/baseimage:focal-1.0.0
	docker-squash --tag phusion/baseimage:focal-latest phusion/baseimage:focal-1.0.0
	docker tag phusion/baseimage:focal-latest phusion/baseimage:focal-1.0.0
	cd base && docker build -t $(base_image) .
	docker-squash --tag $(base_image_squashed) $(base_image)
	docker tag $(base_image_squashed) $(base_image)
	docker rmi `docker images --filter "dangling=true" -q --no-trunc`

squash-server:
	cd pro_seafile && cp -rf ../../scripts ./ && docker build -t $(server_image) .
	rm -rf ./scripts
	docker-squash --tag $(server_image_squashed) $(server_image) --from-layer=$(base_image)
	docker tag $(server_image_squashed) $(server_image)
	docker tag $(server_image) $(latest_server_image)
	docker rmi `docker images --filter "dangling=true" -q --no-trunc`

server:
	cd pro_seafile && cp -rf ../../scripts ./ && docker build -t $(server_image) .
	rm -rf ./scripts
	docker tag $(server_image) $(latest_server_image)

push-base:
	docker tag $(base_image) sawdog/$(base_image)
	docker push sawdog/$(base_image)

push-server:
	docker tag $(pro_server_image) $(pro_server_image)
	docker tag $(pro_server_image) $(latest_pro_server_image)
	docker push $(pro_server_image)
	docker push $(latest_pro_server_image)

push: push-base push-server

.PHONY: base server push push-base push-server
