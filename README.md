# TP 1

## Database

### 1-1 Document your database container essentials: commands and Dockerfile.

**Dockerfile**
```Dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd

# install scheme
COPY ./scripts/CreateScheme.sql /docker-entrypoint-initdb.d
COPY ./scripts/InsertData.sql /docker-entrypoint-initdb.d
```

**build**
``docker build -t mathis/tp01/database .``

**create network**
``docker network create app-network``

**start database**
```
docker run --network app-network --name app-database -v /tmp/pgsql:/var/lib/postgresql/data -d mathis/tp01/database
```

**start adminer**
```
docker run -p "8090:8080" --network app-network --name adminer -d adminer
```

**destruct containers**
```
docker rm -f adminer
docker rm -f app-database
```
