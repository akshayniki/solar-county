# CI4 Docker Setup

This project sets up a CodeIgniter 4 (CI4) environment using Docker with Nginx, PHP, and MySQL.

## Prerequisites

Before you begin, ensure you have the following installed:

- Docker
- Docker Compose

## Project Setup

1. mkdir ci4-docker
2. cd ci4-docker
3. git clone https://github.com/codeigniter4/CodeIgniter4.git solar
4. cd solar
5. composer install --no-dev
6. Create "Dockerfile" file

## Dockerfile start
# Use PHP 8.1 with FPM (FastCGI Process Manager)
FROM php:8.1-fpm

# Install necessary PHP extensions
RUN apt-get update && apt-get install -y \
    libicu-dev \
    && docker-php-ext-install intl

# Set working directory
WORKDIR /var/www/html

# Set correct permissions
RUN chown -R www-data:www-data /var/www/html && chmod -R 755 /var/www/html

# Start PHP-FPM
CMD ["php-fpm"]
## Dockerfile close

7. Create "docker-compose.yml" file

## docker-compose.yml start
version: '3.8'

services:
  php:
    build:
      context: ./php
    container_name: solar-php
    working_dir: /var/www/html
    volumes:
      - .:/var/www/html
    networks:
      - ci4-network
    restart: always

  nginx:
    image: nginx:latest
    container_name: solar-nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - .:/var/www/html
    depends_on:
      - php
    restart: always
    networks:
      - ci4-network

  db:
    image: mysql:5.7
    container_name: ci4-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: solar
      MYSQL_USER: solar
      MYSQL_PASSWORD: solar
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - ci4-network

networks:
  ci4-network:

volumes:
  db_data:
## docker-compose.yml end

8. Create "php/Dockerfile" File
## php/Dockerfile start

FROM php:8.1-fpm

RUN apt-get update && apt-get install -y \
    libicu-dev \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl

CMD ["php-fpm"]

## php/Dockerfile end

9.docker-compose build php

10. Create "nginx/default.conf" File
## nginx/default.conf start

server {
    listen 80;

    server_name localhost;

    root /var/www/html/solar/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass solar-php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }
}

11. Create solar/.env file

## .env starts
CI_ENVIRONMENT = development

# Database Configuration
database.default.hostname = ci4-mysql
database.default.database = solar
database.default.username = solar
database.default.password = solar
database.default.DBDriver = MySQLi
database.default.DBPrefix = 
database.default.charset = utf8
database.default.DBDebug = true
database.default.cacheOn = false
database.default.padConnections = false

# Base URL (adjust based on your environment)
app.baseURL = 'http://localhost:8080/'

# App Settings
app.indexPage = ''
app.charset = UTF-8
app.timezone = UTC
app.language = en

# Session Settings
session.driver = CodeIgniter\Session\Handlers\FileHandler
session.cookieName = ci_session
session.expiration = 7200
session.savePath = 'writeable/session'

# Logging settings
log.threshold = 4  # 4 for Error logs (1 = Debug, 2 = Info, 3 = Warning, 4 = Error, 5 = Critical, 6 = Alert)

# Other settings
app.logging = true
## .env file end

13. docker-compose down
14. docker-compose build --no-cache
15. docker-compose up -d
16. http://localhost:8080
