FROM debian:bookworm-slim AS installer

RUN apt-get -y update && \
  apt-get -y install curl

RUN curl https://install.citusdata.com/community/deb.sh | bash && \
  apt-get -y install postgresql-17-citus-13.0 --no-install-recommends

FROM scratch AS packager

COPY --from=installer /usr/lib/postgresql/17/lib/citus* /lib/
COPY --from=installer /usr/share/postgresql/17/extension/citus* /share/
