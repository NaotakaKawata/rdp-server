FROM ubuntu:18.04
LABEL maintainer="Naotaka Kawata"
LABEL version="1.0"

ARG host_name="Challenger"
ENV DEBIAN_FRONTEND=noninteractive \
    HOSTNAME=$host_name
    
# ARG http_proxy=http://proxy.example.com:8080
# ARG https_proxy=https://proxy.example.com:8080
ENV HTTP_PROXY=$http_proxy
ENV HTTPS_PROXY=$https_proxy

#Locale and Language setting
RUN apt-get update && \
    apt-get install -y ibus-mozc language-pack-ja-base language-pack-ja fonts-takao
ARG Use_Language="en_US.UTF-8"
ARG Use_TimeZone="Asia/Tokyo"
ENV LANG=$Use_Language \
    TZ=$Use_TimeZone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    locale-gen $LANG && \
    echo LANG=$LANG >> /etc/default/locale

#User setting
ARG root_password="super"
ARG user_name="user"
ARG user_password="root"
RUN echo root:$root_password | chpasswd && \
    apt-get update && apt-get install -y openssl sudo && \
    useradd -m -G sudo $user_name -p $(openssl passwd -1 $user_password) --shell /bin/bash

#RDP setting
RUN apt-get update && \
    apt-get install -y xfce4 xfce4-terminal xfce4-goodies xrdp && \
    adduser xrdp ssl-cert && \
    update-alternatives --set x-terminal-emulator /usr/bin/xfce4-terminal.wrapper
COPY ./config/xrdp/sesman.ini /etc/xrdp/sesman.ini
COPY ./config/xrdp/xrdp.ini /etc/xrdp/xrdp.ini
COPY ./config/xrdp/default.pa /etc/xrdp/pulse/default.pa
EXPOSE 3389

#SSH setting
RUN apt-get update && \
    apt-get install -y openssh-server && mkdir /var/run/sshd
COPY ./config/ssh/sshd_config /etc/ssh/sshd_config
COPY ./config/ssh/hosts.allow /etc/hosts.allow
EXPOSE 22

#Startup setting
RUN apt-get update && \
    apt-get install -y supervisor
ADD ./config/supervisord/* /etc/supervisor/conf.d/

#Install Preferred package
RUN apt-get update && \
    apt-get install -y \
    git gedit \
    wget curl net-tools \
    build-essential software-properties-common

# Clean up
RUN apt-get clean && apt-get autoremove && \
    rm -rf /var/cache/apt/archives/* /var/lib/apt/lists/*

CMD ["bash", "-c", "/usr/bin/supervisord -c /etc/supervisor/supervisord.conf"]
WORKDIR /