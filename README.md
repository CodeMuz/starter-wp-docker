# Docker + Wordpress
A docker compose workflow for local & team wordpress development

1. Dependencies: docker, docker-compose
2. (*Optional) Create a db_data folder and seed database with init.sql
3. Run `docker-compose up -d` 
4. Create /etc/hosts entry: `127.0.0.1 app.local`
5. Visit http://app.local (Success!)

6. (*Optional) Edit the docker-compose.yaml to pull latest Wordpress Image:
```
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - ./db_data:/docker-entrypoint-initdb.d
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
     ports:
       - "127.0.0.1:3306:3306"
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "127.0.0.1:80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
     volumes:
       - ./:/var/www/html
volumes:
    db_data:`
```

*N.B Running docker-compose down will reload the database

7. Unfortunately the wordpress docker images are not always up to date. Without hard setting the wp version in docker-compose.yaml or version controlling the core files, one can rebuild containers everytime an update is released. However this instead can be handled by versioning wp in git and seeding the container using an up to date init.sql. (See 8)

An antipattern is to update application files from within the container, i.e using the admin panel update links. 
  Instead one can use the CLI such as: $wp core updates you may need to add the absolute url of your databse path: i.e to 127.0.0.1:3306 as the CLI doesn't instantiate docker environment variables and will report 'Error establishing a database connection'

+ An alternative is to not use the official wordpress image but to generte a LAP stack
using a Dockerfile to enable php apache with mysql enabled. See docker-compose.yaml:

docker-compose.yaml
```
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - ./db_data:/docker-entrypoint-initdb.d
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
     ports:
       - "127.0.0.1:3306:3306"
   php-apache:
     build: .
     ports:
       - 80:80
     volumes:
       - ./:/var/www/html
     depends_on:
       - db
     links:
       - 'db'
     stdin_open: true
     tty: true
volumes:
    db_data:
```

Dockerfile:
```
FROM php:7.2.2-apache
RUN docker-php-ext-install mysqli% 
```
8. To export your database use: 
```
docker exec local_db_1 /usr/bin/mysqldump -u wordpress --password=wordpress wordpress > backup.sql
(Must target container running mysql, not apache server!)
```
9. To restore a database:
```
cat backup.sql | docker exec -i local_db_1 /usr/bin/mysql -u wordpress --password=wordpress wordpress
```
