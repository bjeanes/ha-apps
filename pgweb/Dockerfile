# https://developers.home-assistant.io/docs/apps/configuration#app-dockerfile
ARG BUILD_FROM=ghcr.io/hassio-addons/base:21.0.1
FROM ${BUILD_FROM}

# Copy root filesystem
COPY rootfs /

ARG PGWEB_VERSION=0.17.0
# Supplied by BuildKit. Upstream names its release assets with Go's
# architecture strings, which match neither Docker's nor Home Assistant's.
ARG TARGETARCH

RUN \
    if [ -z "${TARGETARCH}" ]; then \
        echo "TARGETARCH is not set, please use Docker BuildKit for the build." && exit 1; \
    fi \
    && case "${TARGETARCH}" in \
        amd64) pgweb_arch="amd64" ;; \
        arm64) pgweb_arch="arm64" ;; \
        *) echo "Unsupported TARGETARCH: ${TARGETARCH}" && exit 1 ;; \
    esac \
    && curl -sSLf -o /tmp/pgweb.zip \
        "https://github.com/sosedoff/pgweb/releases/download/v${PGWEB_VERSION}/pgweb_linux_${pgweb_arch}.zip" \
    && unzip -q /tmp/pgweb.zip -d /usr/bin \
    && mv /usr/bin/pgweb_* /usr/bin/pgweb \
    && rm -f /tmp/pgweb.zip

LABEL \
    org.opencontainers.image.title="Home Assistant App: pgweb" \
    org.opencontainers.image.description="Web-based PostgreSQL database browser" \
    org.opencontainers.image.source="https://github.com/bjeanes/ha-apps" \
    org.opencontainers.image.licenses="Apache License 2.0"
