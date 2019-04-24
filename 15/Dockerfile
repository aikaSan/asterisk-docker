FROM centos:centos7
ENV build_date 2019-04-22

RUN yum update -y && \
    yum install -y epel-release deltarpm patch && \
    yum install -y \
        git \
        kernel-headers \
        gcc \
        gcc-c++ \
        cpp \
        ncurses \
        ncurses-devel \
        libxml2 \
        libxml2-devel \
        sqlite \
        sqlite-devel \
        openssl-devel \
        newt-devel \
        kernel-devel \
        libuuid-devel \
        net-snmp-devel \
        xinetd \
        tar \
        jansson-devel \
        make \
        bzip2 \
        pjproject-devel \
        libsrtp-devel \
        gsm-devel \
        speex-devel \
        gettext \
      && yum clean all \
      && rm -rf /var/cache/yum/*

# Download asterisk.
WORKDIR /usr/src

COPY build-asterisk.sh .
RUN ./build-asterisk.sh

WORKDIR /

# Update max number of open files.
RUN sed -i -e 's/# MAXFILES=/MAXFILES=/' /usr/sbin/safe_asterisk

# Copy in default configs
COPY http.conf /etc/asterisk/http.conf

RUN localedef -i en_US -f UTF-8 en_US.UTF-8

# And run asterisk in the foreground.
CMD asterisk -f