FROM postgres:14.1-alpine

# settings env variables
ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

# install schemes
COPY ./scripts/CreateScheme.sql /docker-entrypoint-initdb.d
COPY ./scripts/InsertData.sql /docker-entrypoint-initdb.d

EXPOSE 5432