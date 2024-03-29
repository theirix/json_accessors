FROM debian:bullseye

# PostgreSQL version (like 15)
ARG PGVERSION
ARG PGCHANNEL=bullseye-pgdg

ENV DEBIAN_FRONTEND=noninteractive

# Install core packages
RUN apt-get update && \
    apt-get install --no-install-recommends -y sudo curl gnupg tzdata locales lsb-release ca-certificates \
    make gcc libssl-dev libkrb5-dev libicu-dev libdpkg-perl

# Configure repository
RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
    gpg --dearmor -o /etc/apt/keyrings/postgres.gpg && \
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/postgres.gpg] http://apt.postgresql.org/pub/repos/apt/ ${PGCHANNEL} main ${PGVERSION}" | \
    tee -a /etc/apt/sources.list.d/postgres.list

# Prefer packages from Postgres repository
RUN echo "Package: *\nPin: release o=apt.postgresql.org\nPin-Priority: 900" | \
    tee -a /etc/apt/preferences.d/pgdg.pref

# Install PostgreSQL
RUN apt-get update && \
    apt-get install --no-install-recommends -y postgresql-${PGVERSION?} postgresql-server-dev-${PGVERSION?} && \
    rm -rf /var/lib/apt/lists/*

# Create cluster
ENV PGBIN=/usr/lib/postgresql/${PGVERSION}/bin
ENV PGDATA="/var/lib/postgresql/${PGVERSION}/test"
ENV PATH="${PATH}:${PGBIN}"

RUN echo "postgres ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

COPY docker/entrypoint.sh /entrypoint.sh
COPY docker/test.sh /test.sh
RUN chmod a+x /entrypoint.sh /test.sh

RUN mkdir /src && chown postgres /src

USER postgres

RUN initdb

WORKDIR /src

ENTRYPOINT [ "/entrypoint.sh" ]
