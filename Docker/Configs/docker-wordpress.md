# Arquivo configuração para usar wordpress
- É necessário criar uma pasta `config` no local onde o arquivo docker-compose.yaml está
```docker
version: '3.7'

services:
  db:
    image: mariadb:10.5.8
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_DATABASE: "wordpress"
      MYSQL_USER: "wordpress"
      MYSQL_PASSWORD: "wordpress"
  
  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    restart: always
    ports: 
      - 8080:80
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_DATABASE: "wordpress"
      MYSQL_USER: "wordpress"
      MYSQL_PASSWORD: "wordpress"
  
  wordpress:
    depends_on:
     - db
    image: wordpress:latest
    ports:
      - 8000:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./config/uploads.ini:/usr/local/etc/php/conf.d/uploads.ini

volumes:
  db_data:
```