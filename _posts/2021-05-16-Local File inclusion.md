---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "Webs vulnerables y SQLi"
date:   2021-05-19 19:09:01 +0200
categories: PePS
---

    Diego Camañ              	   Webs vulnerables y SQLi 	                       PePS   

#                                                                                       Webs vulnerables y SQLi



##                                                                                       **Nos basamos en File upload**





![file upload 2b prepeared connected](/myassets/img/file upload 2b prepeared connected-1620500674797.png)

​              Hemos subido este fichero que apunta a nuestra Ip con un un reverse shell





![file upload 3 connected](/myassets/img/file upload 3 connected.png)

y aquí ya hemos conectado



Ahora lo realizaremos subiendo el fichero en BASE64 para que no detecten que en un ejecutable PHP

![file upload base64 1](/myassets/img/file upload base64 1.png)

Conversión a Base64

![file upload base64 2 con tmp](/myassets/img/file upload base64 2 con tmp.png)

Subida como texto en Base64





![file upload base64 4 con tmp ok](/myassets/img/file upload base64 4 con tmp ok.png)

Renombramos el fichero a PHP



![file upload base64 5 ejecucion](/myassets/img/file upload base64 5 ejecucion.png)

Ejecución del PHP  y acceso remoto

![file upload base64 6 acceso remoto](/myassets/img/file upload base64 6 acceso remoto.png)

Ya hemos accedido

