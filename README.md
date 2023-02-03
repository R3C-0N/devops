# TP 3

## Load balancing

We need to duplicate the backend

```yaml
# tasks file for roles/launch-app
- name: Run Backend-golden
  docker_container:
    name: devops-backend-golden-1
    image: matmed/tp01-backend:1.5
    networks:
      - name: my-network

- name: Run Backend-silver
  docker_container:
    name: devops-backend-silver-1
    image: matmed/tp01-backend:1.5
    networks:
      - name: my-network

- name: Run Backend-bronze
  docker_container:
    name: devops-backend-bronze-1
    image: matmed/tp01-backend:1.5
    networks:
      - name: my-network

- name: Run Backend-platinium
  docker_container:
    name: devops-backend-platinium-1
    image: matmed/tp01-backend:1.5
    networks:
      - name: my-network
```

Now we have 4 backends (golden, silver, bronze, platinium)

Then we need to create a load balancing cluster in apache config

```conf
<VirtualHost *:8080>
    ServerName devops-backend
    ProxyPreserveHost On

    <Proxy balancer://backendcluster>
        BalancerMember http://devops-backend-platinium-1:8080
        BalancerMember http://devops-backend-silver-1:8080
        BalancerMember http://devops-backend-bronze-1:8080
        BalancerMember http://devops-backend-golden-1:8080
        ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPass / balancer://backendcluster/
    ProxyPassReverse / balancer://backendcluster/
</VirtualHost>
```

And we also need new modules in apache

```conf
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
```

We can also update our docker-compose file

```yaml
            [...]
            
    backend-golden:
        build: ./backend-api
        networks:
            - my-network
        depends_on:
            - database
            
    backend-silver:
        build: ./backend-api
        networks:
            - my-network
        depends_on:
            - database
            
    backend-bronze:
        build: ./backend-api
        networks:
            - my-network
        depends_on:
            - database
            
    backend-platinium:
        build: ./backend-api
        networks:
            - my-network
        depends_on:
            - database
            
            [...]
            
    httpd:
        build: ./http-server
        ports:
            - "8080:8080"
        networks:
            - my-network
        depends_on:
            - backend-golden
            - backend-silver
            - backend-bronze
            - backend-platinium

            [...]
```