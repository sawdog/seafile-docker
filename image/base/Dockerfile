# Lastet phusion baseimage as of 20180412, based on ubuntu 18.04
# See https://hub.docker.com/r/phusion/baseimage/tags/
FROM phusion/baseimage:0.11

RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get update --fix-missing && apt-get install -y \
    curl \
    git \
    htop \
    libmysqlclient-dev \
    libmemcached11 \
    libmemcached-dev \
    libreoffice \
    libreoffice-script-provider-python \
    libsm-dev \
    net-tools \
    openjdk-8-jre \
    openssl \
    poppler-utils \
    psmisc \
    python3 \
    python3-ldap \
    python3-mysqldb \
    python3-pip \
    python3-rados \
    python3-setuptools \
    pwgen \
    sqlite3 \
    tzdata \
    vim \
    wget \
    zlib1g-dev 

RUN python3 -m pip install --upgrade pip && \
    rm -rf /root/.cache/pip

ADD requirements.txt  /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

RUN mkdir -p /opt/seafile/ && mkdir -p /etc/my_init.d && rm -f /etc/my_init.d/00_regen_ssh_host_keys.sh

RUN rm -rf \
    /root/.cache \
    /root/.npm \
    /root/.pip \
    /usr/local/share/doc \
    /usr/share/doc \
    /usr/share/man \
    /usr/share/vim/vim74/doc \
    /usr/share/vim/vim74/lang \
    /usr/share/vim/vim74/spell/en* \
    /usr/share/vim/vim74/tutor \
    /var/lib/apt/lists/* \
    /tmp/*
