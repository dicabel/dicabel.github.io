---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "Docker NGINX"
date:   2021-01-30 02:40:01 +0100
categories: PePS
---

       Diego Camañ                     Docker NGINX                            PePS   

#                                                DOCKER NGINX 



## Instalación NGINX

​                                  

Tras buscarlo en DockerHub lo instalamos, comprobamos que podemos acceder via localhost:8080                       ![Instalacion](/myassets/img/Docker/1 web nginx.jpg)     

​                          

## Agregar HTML personalizado

Enlazamos nuestro html dentro del contenedor utilizando un volumen montado

   ![Agregar](/myassets/img/Docker/2 volumen.jpg)  



Aquí se puede ver el resultado   

![Volumen](/myassets/img/Docker/2b volumen.jpg)

## Crear una imagen personalizada

En el directorio donde tenemos el Dockerfile realizamos la contruccion (build) de la imagen personalizada



![Dockerfile](/myassets/img/Docker/3 into docker web.jpg)

​                                                                        

Ejecutamos la instancia de nuestra imagen y vemos que accedemos al hmtl que incluimos anteriormente

![Dockerfile](/myassets/img/Docker/3b run docker web.jpg)




