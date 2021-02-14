---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "WordPress Docker"
date:   2021-02-14 15:40:01 +0100
categories: PePS
---

       Diego Camañ                         WordPress DOCKER                            PePS   

#                                                                                       WORDPRESS



## Instalación WordPress con DOCKER

​                                  

Vamos a utilizar `Docker-composer` ya que necesitamos con contenedores, uno web para el propio Wordpress y una base de datos, en este caso MySQL, para separar y proteger ambos servicios.

El primer paso es preparar un fichero de configuración para los dockers llamado `docker-compse.yml` en la carpeta que hemos creado para tal efecto

![docker-compose yml](/myassets/img/docker-compose yml.jpg)





```
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

Aquí se puede descargar:  [docker-compose.yml](../YAMaLo/WordPress/docker-compose.yml) 

A continuación ejecutaremos `docker-compose up -d` para descargar las imágenes WordPress y MySQL necesarias y crear sus instancias relacionadas. La información creada en WordPress se almacenará en MySQL en el volumen `db_data` y ahí dentro en la base de datos que hemos llamado `wordpress`

![wordpress2](/myassets/img/wordpress2.png)



Si entramos en http://localhost:8000 (pues hemos habilitado el puerto externo 8000 para que entre en WordPress por el puerto interno 80) podemos configurar WordPress y dejarlo así



![wordpress3](/myassets/img/wordpress3.png)