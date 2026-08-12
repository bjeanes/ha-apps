# https://developers.home-assistant.io/docs/apps/configuration#app-dockerfile
ARG BUILD_FROM=ghcr.io/hassio-addons/base:21.0.1
FROM ${BUILD_FROM}

ARG NOVNC_VERSION=1.7.0

ENV LANG=C.UTF-8

# Needed so the noVNC download below fails the build if curl fails mid-pipe.
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3018
RUN \
    apk add --no-cache \
        chromium \
        dbus \
        font-noto \
        font-noto-cjk \
        font-noto-emoji \
        jq \
        openbox \
        py3-numpy \
        supervisor \
        tigervnc \
        ttf-dejavu \
        websockify \
        xset \
    \
    && mkdir -p /opt/novnc \
    && curl -fsSL "https://github.com/novnc/noVNC/archive/refs/tags/v${NOVNC_VERSION}.tar.gz" \
        | tar -xz -C /opt/novnc --strip-components=1 \
    && rm -rf \
        /opt/novnc/.github \
        /opt/novnc/docs \
        /opt/novnc/snap \
        /opt/novnc/tests \
        /opt/novnc/utils \
        /var/cache/apk/*

# Copy root filesystem
COPY rootfs /

LABEL \
    org.opencontainers.image.title="Home Assistant App: VNC Browser" \
    org.opencontainers.image.description="Kiosk browser screens viewable over noVNC or any VNC client" \
    org.opencontainers.image.source="https://github.com/bjeanes/ha-apps" \
    org.opencontainers.image.licenses="Apache License 2.0"
