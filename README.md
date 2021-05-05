## Compose sample application
### Laravel application with Nginx, MySQL 8, Composer, Artisan, NPM, Mailhog

Project structure:
```
|── dockerfiiles
    |── nginx
        |── default.conf
        |── Dockerfile
    |── php
        |── crontab
        |── Dockerfile
    |── Dockerfile.composer
|── src (Laravel framework)
|── .editorconfig
|── .env
|── docker-compose.yml
|── README.md
```

[_docker-compose.yaml_](docker-compose.yaml)
```
version: '3.9'
services:

    # Webserver
    nginx:
        build:
            context: ./dockerfiles/nginx
            dockerfile: Dockerfile
        image: markmarilag27/nginx
        container_name: nginx
        restart: unless-stopped
        ports:
            - "80:80"
        volumes:
            - ./src:/var/www/html

    # PHP Interpreter
    php:
        build:
            context: ./dockerfiles/php
            dockerfile: Dockerfile
        image: markmarilag27/php8
        container_name: php
        restart: unless-stopped
        volumes:
            - ./src:/var/www/html

    # Database
    mysql:
        image: mysql:latest
        container_name: mysql
        restart: unless-stopped
        command: --default-authentication-plugin=mysql_native_password
        env_file:
            - .env
        volumes:
            - mysql-data:/var/lib/mysql/

    # Composer
    composer:
        build:
            context: ./dockerfiles
            dockerfile: Dockerfile.composer
        image: markmarilag27/composer
        container_name: composer
        volumes:
            - ./src:/var/www/html

    # Artisan
    artisan:
        image: markmarilag27/php8
        container_name: artisan
        working_dir: /var/www/html
        entrypoint: ["php", "/var/www/html/artisan"]
        volumes:
            - ./src:/var/www/html

    # Node package manager
    npm:
        image: node:lts-alpine
        container_name: npm
        working_dir: /var/www/html
        entrypoint: ["npm"]
        volumes:
            - ./src:/var/www/html

    # Mail testing
    mailhog:
        image: mailhog/mailhog
        container_name: mailhog
        ports:
            - "1025:1025"
            - "8025:8025"

# Volumes
volumes:
  mysql-data:
```

## Build images and with docker-compose
Building our images before running the containers
```
$ sudo docker-compose build
mysql uses an image, skipping
composer uses an image, skipping
artisan uses an image, skipping
npm uses an image, skipping
mailhog uses an image, skipping
Building nginx
Sending build context to Docker daemon  4.096kB

Step 1/4 : FROM nginx:stable-alpine
 ---> d0d03989cd8e
Step 2/4 : WORKDIR /etc/nginx/conf.d
 ---> Running in 7c305dda47f5
Removing intermediate container 7c305dda47f5
 ---> 71d4f2ad0cc9
Step 3/4 : COPY default.conf .
 ---> d62a76386f42
Step 4/4 : WORKDIR /var/www/html
 ---> Running in 72956ca037e7
Removing intermediate container 72956ca037e7
 ---> 70eaf7a2ff0c
Successfully built 70eaf7a2ff0c
Successfully tagged markmarilag27/nginx:latest
Building php
Sending build context to Docker daemon  4.608kB

Step 1/13 : FROM php:8-fpm-alpine
 ---> 4736bdf691d0
Step 2/13 : WORKDIR /var/www/html
 ---> Running in f1c33b3e4e5e
...
...
(1/48) Installing m4 (1.4.18-r2)
...

```
### Run composer with docker-compose and installing laravel framework
We will install laravel using [Composer](https://laravel.com/docs/8.x#installation-via-composer)
```
$ sudo docker-compose run composer create-project laravel/laravel .
Creating laravel-docker-example_composer_run ... done
Creating a "laravel/laravel" project at "./"
Installing laravel/laravel (v8.5.16)
  - Downloading laravel/laravel (v8.5.16)
  - Installing laravel/laravel (v8.5.16): Extracting archive
Created project in /var/www/html/.
> @php -r "file_exists('.env') || copy('.env.example', '.env');"
Loading composer repositories with package information
Updating dependencies
Lock file operations: 104 installs, 0 updates, 0 removals
  - Locking asm89/stack-cors (v2.0.3)
...
...
...
78 package suggestions were added by new dependencies, use `composer suggest` to see details.
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: facade/ignition
Discovered Package: fideloper/proxy
Discovered Package: fruitcake/laravel-cors
Discovered Package: laravel/sail
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
74 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan key:generate --ansi
Application key set successfully.

```
Run `sudo chown -R $USER:$USER src` to give permission to the created src directory

### Run with docker-compose
Running the containers
```
$ sudo docker-compose up -d
Creating network "laravel-docker-example_default" with the default driver
Creating php      ... done
Creating npm      ... done
Creating composer ... done
Creating nginx    ... done
Creating artisan  ... done
Creating mailhog  ... done
Creating mysql    ... done
```

### Expected result
Listing containers must show one container running and the port mapping as below:
```
$ sudo docker container ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                                                  NAMES
d130ac2d40d1   mysql:latest          "docker-entrypoint.s…"   39 seconds ago   Up 35 seconds   3306/tcp, 33060/tcp                                                                    mysql
07e7f6d61378   mailhog/mailhog       "MailHog"                39 seconds ago   Up 35 seconds   0.0.0.0:1025->1025/tcp, :::1025->1025/tcp, 0.0.0.0:8025->8025/tcp, :::8025->8025/tcp   mailhog
cdef24dfdc10   markmarilag27/nginx   "/docker-entrypoint.…"   39 seconds ago   Up 35 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp                                                      nginx
9325a935cf86   markmarilag27/php8    "docker-php-entrypoi…"   39 seconds ago   Up 35 seconds   9000/tcp                                                                               php
```
Open your browser and visit http://localhost

### Remove containers with docker-compose
Stop and remove the containers
```
$ sudo docker-compose down
Stopping mysql   ... done
Stopping mailhog ... done
Stopping nginx   ... done
Stopping php     ... done
Removing mysql    ... done
Removing mailhog  ... done
Removing artisan  ... done
Removing composer ... done
Removing nginx    ... done
Removing npm      ... done
Removing php      ... done
Removing network laravel-docker-example_default
```
### Using other containers with docker-compose
Install node packages
`
sudo docker-compose run npm install
`
Run migration files [Artisan](https://laravel.com/docs/8.x/artisan)
`
sudo docker-compose run artisan migrate
`
You can also access [Mailhog](https://github.com/mailhog/MailHog) with your browser http://localhost:8025
