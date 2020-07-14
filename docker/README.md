## Netxcloud passwordless configuration

The target here is a POC to test Passwordless feature of Nextcloud with different Fido2 keys.

This Docker-compose configuration sample has been pulled from [lfache/awesome-traefik](https://github.com/lfache/awesome-traefik) and modified to remove Traefik dependency. This allows to test it on Synology's Docker version which has a non removable reverse proxy.

## Prerequisites

You must have a SSL cert available (not self signed) in order to use FIDO2 / Passwordless feature.  

All secrets can be generated randomly by using `./gen-secrets.sh` 

### Nextcloud examples

Project structure:
```
.
├── db
├── html
├── secrets
│   ├── mysql-database.txt
│   ├── mysql-user.txt
│   ├── mysql-password.txt
├── docker-compose.yaml
├── gen-secrets.sh
└── README.md
```

[_docker-compose.yaml_](docker-compose.yaml)
```
version: '3.7'
services:
  database_nc:
    image: mariadb:10
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    restart: unless-stopped
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
      MYSQL_USER_FILE: '/run/secrets/mysql-user'
      MYSQL_DATABASE_FILE: '/run/secrets/mysql-database'
      MYSQL_PASSWORD_FILE: '/run/secrets/mysql-password'
    secrets:
      - mysql-user
      - mysql-database
      - mysql-password
    networks:
      - lan 
    volumes: 
      - ./db:/var/lib/mysql

  nextcloud:
    depends_on:
      - database_nc
    image: nextcloud:latest
    restart: unless-stopped
    environment:
      MYSQL_HOST: 'database_nc'
      MYSQL_DATABASE_FILE: '/run/secrets/mysql-database'
      MYSQL_USER_FILE: '/run/secrets/mysql-user'
      MYSQL_PASSWORD_FILE: '/run/secrets/mysql-password'
      NEXTCLOUD_ADMIN_PASSWORD: ${NEXTCLOUD_ADMIN_PASSWORD}
      NEXTCLOUD_ADMIN_USER: ${NEXTCLOUD_ADMIN_USER}
      NEXTCLOUD_TRUSTED_DOMAINS: ${NEXTCLOUD_URL}
    ports: 
      - 8081:80
    secrets:
      - mysql-user
      - mysql-database
      - mysql-password
    networks:
      - lan
    volumes:
      - ./html:/var/www/html

networks:
  lan:

secrets:
  mysql-user:
    file: ./secrets/mysql-user.txt

  mysql-database:
    file: ./secrets/mysql-database.txt

  mysql-password:
    file: ./secrets/mysql-password.txt

```

This docker-compose file decribes a Nextcloud instance with its database. Here we chose the latest version of Nextcloud based on Apache. The database is MariaDB.  
You can generate your secrets with `./gen-secrets.sh` 


## Deploy with docker-compose
This repository uses environment variables to pass `NEXTCLOUD_URL`, `NEXTCLOUD_ADMIN_USER`, `NEXTCLOUD_ADMIN_PASSWORD` to your stack.

```
$ sudo NEXTCLOUD_URL=nextcloud.mydomain.com NEXTCLOUD_ADMIN_USER=admin NEXTCLOUD_ADMIN_PASSWORD=mypassword docker-compose up 
````

## Expected result

Listing containers must show two containers running:
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                        NAMES
926a77ccc1e3        nextcloud:18        "/entrypoint.sh apac…"   46 seconds ago      Up 44 seconds       8081->80/tcp                                 nextcloud_nextcloud_1
79d80fddd62b        mariadb:10          "docker-entrypoint.s…"   48 seconds ago      Up 45 seconds       3306/tcp                                     nextcloud_database_nc_1
```

After the application starts, navigate to `https://nextcloud.mydomain.com` in your web browser to access Nextcloud installation:

Stop and remove the containers
```
$ docker-compose down
```

Stop and remove containers and volumes
```
$ docker-compose down -v
```
