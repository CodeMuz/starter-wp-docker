# starter-wp-docker
A docker compose workflow for local & team wordpress development

1. Dependencies: docker, docker-compose
2. Add database dump (i.e init.sql) into db_data folder
3. Run `docker-compose up -d`
4. Creat /etc/hosts entry: `127.0.0.1 app.local`
5. Visit http://app.local

6. if using port 80 for docker container, make sure apache isnt running

Create the docker-compose.yaml in a new folder:

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
    db_data:

Running docker-compose down will reload the database

8a. Unfortunately the wordpress docker images are not always up to date. Without hard setting the wp version in docker-compose.yaml or version controlling the core files, one can rebuild containers everytime an update is released. However this instead can be handled by versioning wp in git and seeding the container using an up to date init.sql. (See 6)

An antipattern is to update application files from within the container, i.e using the admin panel update links. 
  Instead one can use the CLI such as: $wp core updates you may need to add the absolute url of your databse path: i.e to 127.0.0.1:3306 as the CLI doesn't instantiate docker environment variables and will report 'Error establishing a database connection'

8b. An alternative is to not use the official wordpress image but to generte a LAP stack
using a Dockerfile to enable php apache with mysql enabled. See docker-compose.yaml:

FROM php:7.2.2-apache
RUN docker-php-ext-install mysqli

docker-compose.yaml
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

Dockerfile:

FROM php:7.2.2-apache
RUN docker-php-ext-install mysqli% 

7. To export your database use: 
docker exec local_db_1 /usr/bin/mysqldump -u wordpress --password=wordpress wordpress > backup.sql
(Must target container running mysql, not apache server!)

8. To restore a database:
cat backup.sql | docker exec -i local_db_1 /usr/bin/mysql -u wordpress --password=wordpress wordpress
