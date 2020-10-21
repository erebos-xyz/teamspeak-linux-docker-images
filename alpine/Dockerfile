FROM alpine:3.11

LABEL description "TeamSpeak is a full-featured proprietary VoIP software"

# this fork is maintained by kleberbaum
MAINTAINER Florian Kleber <kleberbaum@erebos.xyz>

# change here to desired version
ARG TEAMSPEAK_CHECKSUM=b1d5876854992bf9f5d7bc6b12be71bee9bfe90185b78c74bc50ed5a02f360a2
ARG TEAMSPEAK_URL=https://files.teamspeak-services.com/releases/server/3.12.1/teamspeak3-server_linux_alpine-3.12.1.tar.bz2

ENV PATH "${PATH}:/opt/ts3server"

# update, install and cleaning
RUN echo "## Installing base ##" && \
    echo "@community http://dl-cdn.alpinelinux.org/alpine/v3.10/community/" >> /etc/apk/repositories && \
    apk upgrade --update-cache --available && \
    \
    apk add --no-cache \
        bash \
        ca-certificates \
        libstdc++ \
        tini@community \
    \
    && echo "## Setup permissions ##" \
    && addgroup -g 9987 ts3server \
    && adduser -u 9987 -Hh /var/ts3server -G ts3server -s /sbin/nologin -D ts3server \
    && install -d -o ts3server -g ts3server -m 775 /var/ts3server /var/run/ts3server /opt/ts3server \
    \
    && echo "## set a very secure root password and enable su ##" \
    && echo "root:ciscocisco" | chpasswd \
    && chmod u+s /bin/su \
    \
    && echo "## Download and install TeamSpeak3 ##" \
    && apk add --no-cache --virtual .fetch-deps tar \
    && wget "${TEAMSPEAK_URL}" -O server.tar.bz2 \
    && echo "Verifying integrity of TeamSpeak..." \
    && echo "${TEAMSPEAK_CHECKSUM} *server.tar.bz2" | sha256sum -c - \
    && echo "All seems good, hash is valid." \
    && mkdir -p /opt/ts3server \
    && tar -xf server.tar.bz2 --strip-components=1 -C /opt/ts3server \
    && rm server.tar.bz2 \
    && apk del .fetch-deps \
    && mv /opt/ts3server/*.so /opt/ts3server/redist/* /usr/local/lib \
    && ldconfig /usr/local/lib \
    \
    && rm -rf /tmp/* /var/cache/apk/* /var/cache/distfiles/*

# setup directory where user data is stored
VOLUME /var/ts3server/
WORKDIR /var/ts3server/

#  9987 default voice
# 10011 server query
# 30033 file transport
EXPOSE 9987/udp 10011 30033 

# deploy init script
COPY entrypoint.sh /opt/ts3server

# starting via tini as init
ENTRYPOINT [ "/sbin/tini", "--", "./entrypoint.sh" ]

# change to none root user
USER ts3server

# additional post-installation configurations
CMD [ "./ts3server", "inifile=/var/run/ts3server/ts3server.ini" ]
