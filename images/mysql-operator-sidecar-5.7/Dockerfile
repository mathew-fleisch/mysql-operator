###############################################################################
#  Docker image for sidecar containers
###############################################################################

FROM debian:buster-slim as sidecar

RUN groupadd -g 999 mysql
RUN useradd -u 999 -r -g 999 -s /sbin/nologin \
    -c "Default Application User" mysql

COPY rootfs/ /

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-transport-https ca-certificates unzip pigz wget gnupg \
    && rm -rf /var/lib/apt/lists/*

RUN export RCLONE_VERSION=1.57.0 \
    && HOSTOS=$(uname | tr '[:upper:]' '[:lower:]') \
    && HOSTARCH=$(uname -m | sed -e 's/aarch/arm/g') \
    && gpg --import /etc/gpg-keys/rclone.gpg \
    && wget -nv https://github.com/ncw/rclone/releases/download/v${RCLONE_VERSION}/rclone-v${RCLONE_VERSION}-${HOSTOS}-${HOSTARCH}.zip \
    && wget -nv https://github.com/ncw/rclone/releases/download/v${RCLONE_VERSION}/SHA256SUMS \
    && gpg --verify --output=- SHA256SUMS > sums \
    && sha256sum -c --ignore-missing sums \
    && unzip rclone-*-${HOSTOS}-${HOSTARCH}.zip \
    && mv rclone-*-${HOSTOS}-${HOSTARCH}/rclone /usr/local/bin/ \
    && chmod 755 /usr/local/bin/rclone \
    && rm -r rclone-*-${HOSTOS}-${HOSTARCH} rclone-*-${HOSTOS}-${HOSTARCH}.zip

RUN echo 'deb https://repo.percona.com/apt buster main' > /etc/apt/sources.list.d/percona.list

ARG XTRABACKUP_PKG=percona-xtrabackup-24
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        percona-toolkit ${XTRABACKUP_PKG} unzip default-mysql-client \
    && rm -rf /var/lib/apt/lists/*

USER mysql
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
