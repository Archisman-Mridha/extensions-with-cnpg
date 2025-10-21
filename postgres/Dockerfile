FROM ghcr.io/cloudnative-pg/postgresql:17.6-system-bookworm AS packager

USER root

RUN apt-get -y update && \
  apt-get -y install curl

# Install the Citus extension
RUN curl https://install.citusdata.com/community/deb.sh | bash && \
  apt-get -y install postgresql-17-citus-13.0 --no-install-recommends
