FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
ARG POSTGRES_PASSWORD
RUN apt-get update && apt-get install -y \
   gnupg \
   postgresql-common
RUN yes | /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
RUN apt-get update && apt-get install -y \
   postgresql-14 \
   postgresql-server-dev-14  \
   gcc \
   cmake \
   libssl-dev \
   libkrb5-dev \
   git \
   nano
RUN git clone https://github.com/timescale/timescaledb/
RUN cd timescaledb/ && ./bootstrap
RUN cd timescaledb/./build && make
RUN cd timescaledb/./build && make install -j
RUN cd ..
RUN cd ..
RUN su postgres
RUN pg_dropcluster 14 main --stop
RUN pg_createcluster 14 main -- --auth-host=scram-sha-256 --auth-local=peer --encoding=utf8
USER postgres
RUN service postgresql start && \
   psql -U postgres -d postgres -c "alter user postgres with password '${POSTGRES_PASSWORD}';" && \
   psql -U postgres -d postgres -c "alter system set listen_addresses to '*';" && \
   psql -U postgres -d postgres -c "alter system set shared_preload_libraries to 'timescaledb';"   
RUN sed -i "s|# host    .*|host all all all scram-sha-256|g" /etc/postgresql/14/main/pg_hba.conf
RUN service postgresql restart &&\
   psql -X -c "create extension timescaledb;"
