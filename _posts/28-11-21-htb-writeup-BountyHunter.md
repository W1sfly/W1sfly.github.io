---
layout: single
title: HTB  BOUNTYHUNTER
excerpt: "Máquina HTB - BountyHunter - Dificultad:Easy"
date: 28-11-2021
classes: wide
header:
  teaser: assets/images/post_HTB_bountyhunter/icono.png
  teaser_home_page: true
categories:
  - HTB
  - CTF
tags:
  - Linux
  - Easy
  - python
  - XXE
---

![](/assets/images/post_HTB_bountyhunter/portada.png)


**Resumen:**

|Herramientas|
|------------|
|Nmap|
|Gobuster|
|Burp Suite|
|XXE|

|Escalada de privilegios|
|-----------------------|
|archivo.py|

En esta máquina veremos una vulnerabilidad XXE y un poco de interpretación con python, fundamentales para poder resolverla.

# Nmap

Lanzamos un NMAP con los parámetros necesarios para ver los puertos que tenemos abiertos y vemos como tenemos un http, por lo tanto ya sabemos que hay una página web por detrás. a demás tenemos un ssh que nos permitirá un acceso remoto.

![](/assets/images/post_HTB_bountyhunter/nmap1.png)

Volvemos a lanzar NMAP pero ahora, en base de los puertos que descubrimos anteriormente, ver que versiones e información extra obtenemos.

![](/assets/images/post_HTB_bountyhunter/nmap2.png)
    
# Whatweb

Vemos si a través de “whatweb” no ofrece más información pero no obtenemos mucho más.

![](/assets/images/post_HTB_bountyhunter/whatweb.png)

# Web

Como tenemos el puerto HTTP abierto vamos a ver que contiene, haciendo una pequeña inspección visual a la web damos con la siguiente pestaña en la que podemos introducir los datos que nos solicita y cargarlos en la web: 

![](/assets/images/post_HTB_bountyhunter/web1.png)

# Gobuster

Antes de seguir viendo que podemos hacer con lo anterior vamos a buscar directorios por si hubiera algunos más interesantes.

![](/assets/images/post_HTB_bountyhunter/gobuster.png)

Si nos metemos en el directorio “resources” vemos un "directoy Lister", ojeamos que hay:

![](/assets/images/post_HTB_bountyhunter/web2.png)

Encontramos el siguiente que se refiere al portal donde teníamos que ingresar cve y demás y vemos como en la “data” aparece xml, ya nos podemos ir haciendo una idea por donde van los tiros.

![](/assets/images/post_HTB_bountyhunter/web3.png)

Una cosa que debíamos a ver mirado antes con gobuster es si hay extensiones potenciales como por ejemplo .php o .txt y vemos que hay una que llama la atención "db.php".

![](/assets/images/post_HTB_bountyhunter/gobuster2.png)

Miramos pero no muestra nada………………¿que habrá ahí?

![](/assets/images/post_HTB_bountyhunter/web4.png)

# Burp Suite

Utilizamos burp suite e interceptamos para ver como esta transfiriendo los datos que le introducimos y vemos un valor en la data que se supone que serán los valores que estamos metiendo.

![](/assets/images/post_HTB_bountyhunter/burp1.png)

Lo metemos en el "repeater" y vemos como la data es interpretada. El valor de la data está en base64 pero una cosa importante a tener en cuenta es que también está en URL Encode.

![](/assets/images/post_HTB_bountyhunter/burp2.png)

Decodificamos la data desde el propio burp suite, para ello la seleccionamos y vemos en la parte derecha la decodificación, nos encontramos ante un XML y ya lo primero que se nos viene a la cabeza es XXE.

![](/assets/images/post_HTB_bountyhunter/burp3.png)

Probamos el siguiente código en XML el cual me devolvería el contenido de un archivo interno, en este caso el “passwd”.

~~~
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [<!ENTITY file SYSTEM "file:///etc/passwd"> ]>
		<bugreport>
		<title>&file;</title>
		<cwe>prueba</cwe>
		<cvss>prueba</cvss>
		<reward>prueba</reward>
		</bugreport>
~~~

Ahora para meter el código anterior hacemos lo siguiente:

Decode URL:

![](/assets/images/post_HTB_bountyhunter/burp4.png)

Decode base64 y vemos el contenido xml de los datos que introdujimos primero.

![](/assets/images/post_HTB_bountyhunter/burp5.png)

Cambiamos el código y hacemos “encode base64”

![](/assets/images/post_HTB_bountyhunter/burp6.png)

Lo copiamos.

![](/assets/images/post_HTB_bountyhunter/burp7.png)

Pero antes de enviarlo tenemos que recordar que utilizaba el URL Encode.

![](/assets/images/post_HTB_bountyhunter/burp8.png)

Y logramos ver el archivo "passwd".

![](/assets/images/post_HTB_bountyhunter/burp9.png)

__Nota__: lo anterior lo hemos hecho con la pestaña "decoder" del propio burp pero hay una manera más sencilla y directa que es utilizando un conjunto de teclas:  

*ctrl+shift+u --> Pasa de URLencode a Base64  

*ctrl+shift+b --> Decodifica

Como anteriormente descubrimos un db.php que “no hacía nada” y tiene pinta de ser de una base de datos o algo, podemos intentar verlo de la siguiente manera la cual nos dará el contenido de la información en base64:
~~~
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [<!ENTITY file SYSTEM "php://filter/read=convert.base64-encode/resource=/var/www/html/db.php"> ]>
		<bugreport>
		<title>&file;</title>
		<cwe>prueba</cwe>
		<cvss>prueba</cvss>
		<reward>prueba</reward>
		</bugreport>
~~~

Repetimos el mismo proceso de antes y vemos como nos devuelve el deb.php pero en base64.

![](/assets/images/post_HTB_bountyhunter/burp10.png)

Lo decodificamos y damos con una password.

![](/assets/images/post_HTB_bountyhunter/burp11.png)

Como teníamos un ssh abierto vamos a probarlo, probamos con el user admin pero no vale (ya que en el listado que vimos de usuarios no estaba) asi que probamos con el usuario:

![](/assets/images/post_HTB_bountyhunter/passwd.png)

Probamos y ya estamos dentro.

![](/assets/images/post_HTB_bountyhunter/ssh.png)

# Flag User

![](/assets/images/post_HTB_bountyhunter/ssh2.png)


# Escalar Privilegios

Hacemos un poco de reconocimiento en el sistema y nos encontramos con el siguiente permiso que tenemos en el que podemos ejecutar lo siguiente como "root" sin proporcionar contraseña.

![](/assets/images/post_HTB_bountyhunter/sudol.png)

Inspeccionamos primero el archivo python y conseguimos ver como tenemos que ejecutarlo.

![](/assets/images/post_HTB_bountyhunter/python.png)

Así que nos creamos un arhivo ".md".  

La idea es que nos devuelva una bash y como se ejecuta como “root” tendríamos una shell con privilegios.

![](/assets/images/post_HTB_bountyhunter/md.png)

Lo ejecutamos y......................CONSEGUIDO!!!!!!!!!!!!

![](/assets/images/post_HTB_bountyhunter/root.png)

# Flag root

![](/assets/images/post_HTB_bountyhunter/passroot.png)  
  
-----------------------------------------------------------------
![](/assets/images/post_HTB_bountyhunter/giphy.gif)  




