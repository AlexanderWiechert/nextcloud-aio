FROM haproxy:2.8.3-alpine3.18

# hadolint ignore=DL3002
USER root
ENV NEXTCLOUD_HOST nextcloud-aio-nextcloud
RUN set -ex; \
    apk add --no-cache \
        ca-certificates \
        tzdata \
        bash \
        bind-tools; \
    chmod -R 777 /tmp

COPY --chmod=775 *.sh /
COPY --chmod=664 haproxy.cfg /haproxy.cfg

ENTRYPOINT ["/start.sh"]
HEALTHCHECK CMD /healthcheck.sh
LABEL com.centurylinklabs.watchtower.enable="false"
