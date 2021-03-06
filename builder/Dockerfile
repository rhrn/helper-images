# SPDX-License-Identifier: GPL-3.0-or-later
# author  : paulfantom

# This image is used to speed up build process for official netdata images.

ARG ARCH=amd64-v3.12
FROM multiarch/alpine:${ARCH} as builder

# Install prerequisites
RUN apk --no-cache add alpine-sdk \
                       autoconf \
                       automake \
                       bash \
                       build-base \
                       curl \
                       jq \
                       json-c-dev \
                       libmnl-dev \
                       libuuid \
                       lm_sensors \
                       netcat-openbsd \
                       nodejs \
                       pkgconfig \
                       py-pip \
                       python3 \
                       python3-dev \
                       util-linux-dev \
                       zlib-dev \
                       libuv-dev \
                       lz4-dev \
                       openssl-dev \
                       libgcrypt-dev \
                       mariadb-dev \
                       cmake \
                       mongo-c-driver-dev

# Add Python dependencies from PyPi using `pip` we have not APK(s) for:
RUN pip install mysqlclient

# Instaling psycopg2 requires additional alpine packages
# Removing dependencies again after psycopg2 has been installed
RUN apk add --no-cache --virtual .build-deps \
    gcc \
    musl-dev \
    postgresql-dev \
    && pip install --no-cache-dir psycopg2 \
    && apk del --no-cache .build-deps

# External dependencies (bundled to avoid network access)
COPY deps /deps

# Judy doesnt seem to be available on the repositories, download manually and install it
ENV JUDY_VER 1.0.5
RUN if [ "${ARCH}" == "aarch64" ]; then \
    echo "Executing ${ARCH} build, adding exceptional option --build for judy" \
    && cp /deps/Judy-${JUDY_VER}.tar.gz /judy.tar.gz \
    && cd / && tar -xf judy.tar.gz && rm judy.tar.gz \
    && cd /judy-${JUDY_VER} \
    && CFLAGS="-O2 -s" CXXFLAGS="-O2 -s" ./configure --build=aarch64-unknown-linux-gnu --prefix=/deps \
    && make \
    && make install; \
    # non aarch64 branch
    else \
    cp /deps/Judy-${JUDY_VER}.tar.gz /judy.tar.gz \
    && cd / && tar -xf judy.tar.gz && rm judy.tar.gz \
    && cd /judy-${JUDY_VER} \
    && CFLAGS="-O2 -s" CXXFLAGS="-O2 -s" ./configure --prefix=/deps \
    && make \
    && make install; \
    fi;

# freeipmi
ENV FREEIPMI_VER 1.6.4
COPY builder/patches/freeipmi-argp-redefine.patch /freeipmi-${FREEIPMI_VER}/
RUN cp /deps/freeipmi-${FREEIPMI_VER}.tar.gz /freeipmi.tar.gz \
    && cd / \
    && tar -xf freeipmi.tar.gz \
    && rm freeipmi.tar.gz \
    && cd /freeipmi-${FREEIPMI_VER} \
    && patch -p 0 < freeipmi-argp-redefine.patch \
    && rm freeipmi-argp-redefine.patch \
    && CPPFLAGS="-Dgetmsg\(a,b,c,d\)=errno=-1,-1 -Dputmsg\(a,b,c,d\)=errno=-1,-1" ./configure --prefix=/deps \
    && make \
    && make install \
    && cd / \
    && rm -rf /freeipmi-${FREEIPMI_VER}
