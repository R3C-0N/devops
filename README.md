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
docker network create --subnet=172.18.0.0/16 app-network
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

## Backend-API

**Dockerfile**
```Dockerfile
FROM openjdk:17

COPY ./class /usr/src/backend-app

WORKDIR /usr/src/backend-app
RUN javac Main.java

CMD ["java", "Main"]
```

**build**
```
docker build -t mathis/tp01/backend-api .
```

**start backend-api**
```
docker run --network app-network --name backend-api --ip 172.19.0.4 -d mathis/tp01/backend-api
```

**destruct containers**
```
docker rm -f app-database
```

### 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

The multistage building is used to separate tasks. First we need to compile the project and then we execute it.

It is interesting because in the end, there is only compiled files for the execution and if a malicious user try to read them,
he will note have source code

## HTTP SERVER

**Dockerfile**
```Dockerfile
FROM httpd

COPY ./public/ /usr/local/apache2/htdocs/
```


**build**
```
docker build -t mathis/tp01/http-server .
```

**start http-server**
```
docker run --network app-network --name http-server -p 80:80 -d mathis/tp01/http-server
```

**destruct containers**
```
docker rm -f http-server
```

**Get configuration php**
```bash
$ /usr/local/apache2/conf/httpd.conf
...
```

**We need to add:**
```bash
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://<container_ip>:<container_port>/
    ProxyPassReverse / http://<container_ip>:<container_port>/
</VirtualHost>
```

container_ip must be the ip from backend-api docker in network

**It can be get with:**
```bash
docker inspect backend-api
```

**And uncomment these lines:**
```bash
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

