---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "Autenticacion P2"
date:   2021-03-11 21:50:01 +0100
categories: PePS
---

       Diego Camañ                       Autenticación P2                          PePS   

#                                                                                       Autenticación HTTP 2



Para aumentar la seguridad utilizaremos en un servidor Apache sus ficheros de configuración para obligar a que los usuarios que entren estén validados. Para ello modificaremos los ficheros `apache2.conf` y/o `.htaccess`para obligar a leer el fichero `.htpasswd` al que sólo tendrá acceso apache y en el que se encuentran los usuarios y contraseñas  a los que se les permitirá el acceso.



