---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "Google Cloud"
date:   2021-02-18 22:50:01 +0100
categories: PePS
---

       Diego Camañ                         Google Cloud                            PePS   

#                                                                                       GOOGLE CLOUD



## Instalación APP en Google Cloud

##          

##                          

En primer lugar nos daremos de alta en https://cloud.google.com/ y luego creamos un proyecto, en mi caso llamado alu7369a.

![GoogleCloud APPEngine1](/myassets/img/GoogleCloud APPEngine1-1613834906655.png)



![GoogleCloud APPEngine2](/myassets/img/GoogleCloud APPEngine2.png)

En nuestro ordenador bajamos e instalamos SDK según las instrucciones en https://cloud.google.com/sdk/docs/quickstart#deb (estamos utilizando Kali) y donde acabamos con

```
sudo apt-get update && sudo apt-get install google-cloud-sdk
```



Como hemos creado la cuenta ya estamos validados en google cloud así que vamos a iniciar el servicio

 

```
gcloud init
```

Como el proyecto es en Node.js vamos a instalarlo, junto a las herramientas de gestion de paquetes de Node

```
sudo apt-get install nodejs
sudo apt-get install npm
```

Creamos nuestra carpeta carpeta para el proyecto y en ella iniciamos el gestor de paquetes de Node

```
mkdir googlecloud
npm init
```

Esta carpeta googlecloud la tenemos configurada en GitHub como repositorio para actualizarla cuando hagamos cambios en la configuración. El directorio node_modules lo añadiremos al .gitignore para que no se suba al repositorio.

Ya tenemos preparado el motor de programación. Vamos a crear la aplicación.

utilizamos 

```
npm install express
```

que crea el archivo [`package.json`](https://github.com/dicabel/GoogleCloud/blob/main/package.json) donde se guardan las dependencias (como express) y secuencias de comandos (scripts como  "start": "node server.js")

![GoogleCloud3 package.json](/myassets/img/GoogleCloud3 package.json.png)

Así que creamos el script de inicio `server.js`



![GoogleCloud4 server](/myassets/img/GoogleCloud4 server.png)

Luego cambiaremos el mensaje que enviamos por 'Hello from Ciberseguridad'

Para comprobarlo ejecutaremos `npm start`que ejecutará `server.js`. 



![GoogleCloud6 localhost8080](/myassets/img/GoogleCloud6 localhost8080.png)



Comprobamos que podemos verlo en el navegador en `http://localhost:8080`



![GoogleCloud5 npm start](/myassets/img/GoogleCloud5 npm start.png)



Vamos a subirla a Google Cloud. Para ello necesitamos un archivo  [`app.yaml`](https://github.com/dicabel/GoogleCloud/blob/main/app.yaml) especifique la configuración en el entorno del servicio App Engine de Google.

Lo único que ha de contener en nuestro caso es

```
runtime: nodejs14
```

 

Todo listo, ejecutamos `gloud app deploy`



![GoogleCloud7 subimos a google a la segunda funciona](/myassets/img/GoogleCloud7 subimos a google a la segunda funciona.png)



Si accedemos a la página asignada 

`https://alu7369a.oa.r.appspot.com`

la podremos ver

![GoogleCloud8 lo vemos en google](/myassets/img/GoogleCloud8 lo vemos en google.png)

Editamos el fichero [`server.js`](https://github.com/dicabel/GoogleCloud/blob/main/package.json) para que muestre "Hello from Ciberseguridad!!" (actualizamos repositorio GitHub como en ocasiones antreriores) y lo subimos de nuevo con `gloud app deploy`



Vemos el cambio

![GoogleCloud9 lo vemos en google Ciberseguridad](/myassets/img/GoogleCloud9 lo vemos en google Ciberseguridad.png)