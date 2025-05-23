FROM nats:2.10.1-scratch as nats
FROM eturnal/eturnal:1.12.0 AS eturnal
FROM strukturag/nextcloud-spreed-signaling:1.1.3 as signaling
FROM alpine:3.18.4 as janus

ARG JANUS_VERSION=v0.14.0
WORKDIR /src
RUN set -ex; \
    apk add --no-cache \
        ca-certificates \
        git \
        autoconf \
        automake \
        build-base \
        pkgconfig \
        libtool \
        util-linux \
        glib-dev \
        zlib-dev \
        openssl-dev \
        jansson-dev \
        libnice-dev \
        libconfig-dev \
        libsrtp-dev \
        libusrsctp-dev \
        gengetopt-dev \
        libwebsockets-dev; \
    git clone --recursive https://github.com/meetecho/janus-gateway --depth=1 --single-branch --branch "$JANUS_VERSION" /src; \
    /src/autogen.sh; \
    /src/configure --disable-rabbitmq --disable-mqtt --disable-boringssl; \
    make; \
    make install; \
    make configs; \
    rename -v ".jcfg.sample" ".jcfg" /usr/local/etc/janus/*.jcfg.sample

FROM alpine:3.18.4
ENV ETURNAL_ETC_DIR="/conf"
COPY --from=janus     /usr/local                          /usr/local
COPY --from=eturnal   /opt/eturnal                        /opt/eturnal
COPY --from=nats      /nats-server                        /usr/local/bin/nats-server
COPY --from=signaling /usr/bin/nextcloud-spreed-signaling /usr/local/bin/nextcloud-spreed-signaling

COPY --chmod=775 start.sh /start.sh
COPY --chmod=775 healthcheck.sh /healthcheck.sh
COPY --chmod=664 supervisord.conf /supervisord.conf

RUN set -ex; \
    apk add --no-cache \
        ca-certificates \
        tzdata \
        bash \
        openssl \
        supervisor \
        bind-tools \
        netcat-openbsd \
        \
        glib \
        zlib \
        libssl3 \
        libcrypto3 \
        jansson \
        libnice \
        libconfig \
        libsrtp \
        libusrsctp \
        libwebsockets \
        \
        shadow; \
    useradd --system eturnal; \
    apk del --no-cache \
        shadow; \
    \
# Give root a random password
    echo "root:$(openssl rand -base64 12)" | chpasswd; \
    \
    touch \
        /etc/nats.conf \
        /etc/eturnal.yml; \
    echo "listen: 127.0.0.1:4222" | tee /etc/nats.conf; \
    mkdir -p \
        /var/tmp \
        /conf \
        /var/lib/turn \
        /var/log/supervisord \
        /var/run/supervisord \
        /usr/local/lib/janus/loggers; \
    chown eturnal:eturnal -R \
        /usr \
        /opt/eturnal \
        /etc/nats.conf \
        /var/log/supervisord \
        /var/run/supervisord; \
    chmod 777 -R \
        /tmp \
        /conf \
        /opt/eturnal \
        /var/run/supervisord \
        /var/log/supervisord; \
    ln -s /opt/eturnal/bin/stun /usr/local/bin/stun; \
    ln -s /opt/eturnal/bin/eturnalctl /usr/local/bin/eturnalctl

USER eturnal
ENTRYPOINT ["/start.sh"]
CMD ["supervisord", "-c", "/supervisord.conf"]

HEALTHCHECK CMD /healthcheck.sh
LABEL com.centurylinklabs.watchtower.enable="false"
