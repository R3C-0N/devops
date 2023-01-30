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
```
docker build -t mathis/tp01/database .
```

**create network**
```
docker network create app-network
```

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

**Result**
```bash
docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                    NAMES
0201e27823d0   mathis/tp01/database   "docker-entrypoint.s…"   4 minutes ago    Up 4 minutes    3306/tcp, 5432/tcp       app-database
e30d9652ac37   adminer                "entrypoint.sh php -…"   36 minutes ago   Up 36 minutes   0.0.0.0:8090->8080/tcp   adminer
```