# See https://hub.docker.com/r/phusion/baseimage/tags/
FROM phusion/baseimage:focal-1.0.0
ENV SEAFILE_SERVER=seafile-server
ENV SEAFILE_VERSION=8.0.7

RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get update --fix-missing && apt-get install -y \
    curl \
    git \
    htop \
    libmysqlclient-dev \
    libmemcached11 \
    libmemcached-dev \
    net-tools \
    nginx \
    smisc \
    python3 \
    python3-pip \
    python3-setuptools \
    tzdata \
    vim \
    wget

RUN python3 -m pip install --upgrade pip && \
    pip3 install --timeout=3600 \
    captcha  \
    click \
    colorlog \
    django==2.2.* \
    django-pylibmc \
    django-simple-captcha \
    future \
    jinja2 \
    mysqlclient \
    pyjwt \
    Pillow \
    pylibmc \
    pymysql \
    sqlalchemy \
    termcolor && rm -r /root/.cache/pip

# Scripts
COPY scripts /scripts
COPY templates /templates
COPY services /services
RUN chmod u+x /scripts/*

RUN mkdir -p /etc/my_init.d && \
    rm -f /etc/my_init.d/* && \
    cp /scripts/create_data_links.sh /etc/my_init.d/01_create_data_links.sh

RUN mkdir -p /etc/service/nginx && \
    rm -f /etc/nginx/sites-enabled/* /etc/nginx/conf.d/* && \
    mv /services/nginx.conf /etc/nginx/nginx.conf && \
    mv /services/nginx.sh /etc/service/nginx/run

# Seafile
WORKDIR /opt/seafile

RUN mkdir -p /opt/seafile/ && cd /opt/seafile/ && \
    wget https://download.seadrive.org/seafile-server_${SEAFILE_VERSION}_x86-64.tar.gz && \
    tar -zxvf seafile-server_${SEAFILE_VERSION}_x86-64.tar.gz && \
    rm -f seafile-server_${SEAFILE_VERSION}_x86-64.tar.gz

# For using TLS connection to LDAP/AD server with docker-ce.
RUN find /opt/seafile/ \( -name "liblber-*" -o -name "libldap-*" -o -name "libldap_r*" -o -name "libsasl2.so*" \) -delete


EXPOSE 80


CMD ["/sbin/my_init", "--", "/scripts/start.py"]
