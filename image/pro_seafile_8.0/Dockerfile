FROM sawdog/seafile-pro-base-server-image:1.0.0
WORKDIR /opt/seafile

ENV SEAFILE_VERSION=8.0.8 SEAFILE_SERVER=seafile-pro-server

RUN curl -sSL -G -d "p=/pro/seafile-pro-server_${SEAFILE_VERSION}_x86-64_Ubuntu.tar.gz&dl=1" https://download.seafile.com/d/6e5297246c/files/ \
    | tar xzf - -C /opt/seafile/

ADD scripts/create_data_links.sh /etc/my_init.d/01_create_data_links.sh

COPY scripts /scripts
RUN chmod u+x /scripts/*

EXPOSE 8002 8080 8000

CMD ["/sbin/my_init", "--", "/scripts/start.py"]
