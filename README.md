# Traefik Proxy

A simple example of how to use the Traefik Proxy as a simple router that can connect to LetsEncrypt and provide free SSL certificate.

I used the following images and projects:
- Traefik v2: https://docs.traefik.io/

See [docker-compose.yml](docker-compose.yml) file:
```yaml
version: "3.3"

services:
  whoami:
    image: containous/whoami
    container_name: whoami
    restart: unless-stopped
    network_mode: "bridge"
    labels:
      - "traefik.http.routers.whoami.rule=Host(`whoami.joaogabriel.org`)"
      - "traefik.http.routers.whoami.tls=true"
      - "traefik.http.routers.whoami.tls.certresolver=myresolver"

  traefik:
    image: traefik:v2.2.1
    container_name: traefik
    restart: unless-stopped
    network_mode: "bridge"
    command:
      - "--api.debug=true"
      - "--api.insecure=false"
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.email=gabriel@joaogabriel.org"
      - "--certificatesresolvers.myresolver.acme.storage=acme.json"
      - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"
    labels:
      - "traefik.http.routers.api.rule=Host(`traefik.joaogabriel.org`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=test:$$apr1$$H6uskkkW$$IgXLP6ewTrSuBkTrqE8wj/,test2:$$apr1$$d9hr9HBB$$4HxwgUir3HP4EsggP/QNo0"
      - "traefik.http.routers.api.tls=true"
      - "traefik.http.routers.api.tls.certresolver=myresolver"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./acme.json:/acme.json
    ports:
      - 80:80
      - 443:443
``` 

The [acme.json](acme.json) file will be used to persist SSL certificates (do this because LetsEncrypt has a rate limit).

In this example, the dashboard is present in https://traefik.joaogabriel.org.

### Another option
Another option is to use the Docker Nginx Proxy, simpler but just as robust.
https://github.com/jgcl/docker-nginx-proxy