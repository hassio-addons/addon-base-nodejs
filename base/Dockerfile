ARG BUILD_FROM=ghcr.io/hassio-addons/base:15.0.3
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# NodeJS version to pack
ARG NODEJS_VERSION="18.19.0"

# hadolint ignore=DL3003,DL4006,SC2155,DL3018,SC2046
RUN \
    apk add --no-cache \
      libstdc++=13.2.1_git20231014-r0 \
    && ARCH= OPENSSL_ARCH='linux*' \
    && alpineArch="$(apk --print-arch)" \
    && case "${alpineArch##*-}" in \
      x86_64) OPENSSL_ARCH=linux-x86_64;; \
      x86) OPENSSL_ARCH=linux-elf;; \
      aarch64) OPENSSL_ARCH=linux-aarch64;; \
      arm*) OPENSSL_ARCH=linux-armv4;; \
      *) ;; \
    esac \
  && apk add --no-cache --virtual .build-deps-full \
    binutils-gold=2.41-r0 \
    g++=13.2.1_git20231014-r0 \
    gcc=13.2.1_git20231014-r0 \
    gnupg=2.4.3-r1 \
    libgcc=13.2.1_git20231014-r0 \
    linux-headers=6.5-r0 \
    make=4.4.1-r2 \
    python3=3.11.6-r1 \
  && export GNUPGHOME="$(mktemp -d)" \
  # gpg keys listed at https://github.com/nodejs/node#release-keys
  && for key in \
    4ED778F539E3634C779C87C6D7062848A1AB005C \
    141F07595B7B3FFE74309A937405533BE57C7D57 \
    74F12602B6F1C4E913FAA37AD3A89613643B6201 \
    DD792F5973C6DE52C432CBDAC77ABFA00DDBF2B7 \
    61FC681DFB92A079F1685E77973F295594EC4689 \
    8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    890C08DB8579162FEE0DF9DB8BEAB4DFCF555EF4 \
    C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
    108F52B48DB57BB0CC439B2997B01419BD92F80A \
    A363A499291CBBC940DD62E41F10027AF002F8B0 \
  ; do \
    gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$key" || \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODEJS_VERSION/node-v$NODEJS_VERSION.tar.xz" \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODEJS_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && gpgconf --kill all \
  && rm -rf "$GNUPGHOME" \
  && grep " node-v$NODEJS_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xf "node-v$NODEJS_VERSION.tar.xz" \
  && cd "node-v$NODEJS_VERSION" \
  && ./configure \
  && make -j$(getconf _NPROCESSORS_ONLN) V= \
  && make install \
  && apk del .build-deps-full \
  && cd .. \
  && rm -Rf "node-v$NODEJS_VERSION" \
  && rm "node-v$NODEJS_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  # Remove unused OpenSSL headers to save ~34MB. See this NodeJS issue: https://github.com/nodejs/node/issues/46451
  && find /usr/local/include/node/openssl/archs -mindepth 1 -maxdepth 1 ! -name "$OPENSSL_ARCH" -exec rm -rf {} \; \
  # smoke tests
  && node --version \
  && npm --version

# Entrypoint & CMD
ENTRYPOINT ["/init"]

# Build arugments
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION
ARG BUILD_REPOSITORY

# Labels
LABEL \
    io.hass.name="Addon NodeJS base for ${BUILD_ARCH}" \
    io.hass.description="Home Assistant Community Add-on: ${BUILD_ARCH} NodeJS base image" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="base" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="Addon NodeJS base for ${BUILD_ARCH}" \
    org.opencontainers.image.description="Home Assistant Community Add-on: ${BUILD_ARCH} NodeJS base image" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/master/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
