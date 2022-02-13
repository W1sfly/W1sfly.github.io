---
layout: single
title: TryHackMe  Gallery
excerpt: "Máquina THM - Gallery - Dificultad:Easy"
date: 13-02-2022
classes: wide
header:
  teaser: assets/images/post_THM_gallery/Screenshot_2.png
  teaser_home_page: true
categories:
  - TryHackMe
  - CTF
tags:
  - Linux
  - Easy
  - CMS simple image gallery
  - SQL & RCE
  - shell nano
  
---

![](/assets/images/post_THM_gallery/Screenshot_1.png)

URL: <https://tryhackme.com/room/gallery666>

# <ins>Resumen:</ins>

![](/assets/images/post_THM_gallery/Screenshot_3.png)

Una máquina donde nos encontraremos ante un CMS vulnerable utilizando técnicas diferentes para conseguir una shell, una vez dentro una búsqueda para pasar de usuario y una escalada en la que hay que mirar un script que se ejecuta como root.


# <ins>Reconocimiento</ins>

Empezamos con un escaneo "nmap" para ver que puertos nos encontramos abiertos en la máquina, vemos dos.

![](/assets/images/post_THM_gallery/Screenshot_4.png)

Ahora que ya sabemos los puertos vamos a ver qué información extra conseguimos de ellos.

![](/assets/images/post_THM_gallery/Screenshot_5.png)

# <ins>WEB</ins>

Introducimos la IP en la url.

![](/assets/images/post_THM_gallery/Screenshot_6.png)

Buscamos con gobuster y encontramos algo.

![](/assets/images/post_THM_gallery/Screenshot_7.png)

Damos con un login y el nombre del CMS, intentamos credenciales por defectos sin éxito.

![](/assets/images/post_THM_gallery/Screenshot_8.png)

Como tenemos el nombre del CMS vamos a buscar si hay vulnerabilidades, en este caso encontramos dos.

![](/assets/images/post_THM_gallery/Screenshot_9.png)

Miramos ambas a ver cómo funcionan y nos da una idea de cómo poder logearnos. Si introducimos en el "username" "admin'-- -" entraremos.

![](/assets/images/post_THM_gallery/Screenshot_10.png)

Primero, antes de ejecutar el RCE, vamos a ver la base de datos gracias a "Simple Image Gallery System 1.0 - 'id' SQL Injection", miramos como tenemos que hacerlo..............

![](/assets/images/post_THM_gallery/Screenshot_11.png)

y la obtenemos.

![](/assets/images/post_THM_gallery/Screenshot_12.png)

Vamos viendo el contenido de las tablas y obtenemos el hash del usuario "admin"

![](/assets/images/post_THM_gallery/Screenshot_13.png)

Llegado a este punto, lo que nos interesa es entablarnos una "reverseShell", esto lo podemos conseguir de dos formas diferentes.

- Método 1: Subida de fichero.  

Mirando un poco la web vemos como podemos subir fichero en los álbumes, en teoría parece ser que sólo se pueden subir imágenes.

![](/assets/images/post_THM_gallery/Screenshot_14.png)

Pero no, nos deja subir también nuestra "reverseShell" en php.

![](/assets/images/post_THM_gallery/Screenshot_15.png)

Nos ponemos a la escucha y una vez que subamos el archivo nos la entabla.

![](/assets/images/post_THM_gallery/Screenshot_16.png)

- Método 2: Simple Image Gallery 1.0 - Remote Code Execution (RCE).

![](/assets/images/post_THM_gallery/Screenshot_26.png)

Con la ejecución de ésta vulnerabilidad, con sólo pasarle la IP nos dará una URL en la que podremos ejecutar comandos.

![](/assets/images/post_THM_gallery/Screenshot_27.png)

Como podemos ejecutar comandos también podremos entablarnos una "reverseshell" utilizando netcat.

![](/assets/images/post_THM_gallery/Screenshot_29.png)

-------------------------------------------------

Una vez dentro, mirando un poco como está formado todo el tema de directorios de usuarios, nos encontramos que tenemos dos, tiene la pinta de que tenemos que pasarnos al usuario "mike".

![](/assets/images/post_THM_gallery/Screenshot_17.png)

Hacemos un pequeño reconocimiento a ver si vemos algo con lo que podamos convertirnos en ese usuario y podemos conseguir sus credeciales de dos formas:

- Reconocimiento manual:

Si miramos en su directorio vemos el siguiente archivo oculto el cual nos puede dar información relevante, pero tenemos un impedimento y es que no tenemos permisos para leerlo.

![](/assets/images/post_THM_gallery/Screenshot_31.png)

Buscamos y vemos en la carpeta "backups" que hay un backup del directorio "home" del usuario, en este caso si tenemos permisos para poder ver el archivo y al abrirlo vemos unas credenciales.

![](/assets/images/post_THM_gallery/Screenshot_30.png)

- Reconocimiento automático:

Utilizando la herramienta "LinPEAS" también logramos encontrar las credenciales del usuario.

![](/assets/images/post_THM_gallery/Screenshot_18.png)


# <ins> Flag USER </ins>  

![](/assets/images/post_THM_gallery/Screenshot_19.png)


# <ins> Escalar Privilegios </ins>

Miramos si hay algún archivo que podamos ejecutar como root sin proporcionar contraseña y vemos el siguiente script en bash.

![](/assets/images/post_THM_gallery/Screenshot_20.png)

Miramos que hace y vemos que hay varias opciones a elegir y cada una hace una cosa, la que nos interesa es la de "read" ya que gracias a "nano" podemos hacer algo que veremos a continuación.

![](/assets/images/post_THM_gallery/Screenshot_21.png)

Como vemos podemos conseguir una shell gracias a "nano" haciendo lo siguiente.

![](/assets/images/post_THM_gallery/Screenshot_22.png)

Así que ejecutamos el script anterior como el usuario "root" y eligiendo la opción de "read".

![](/assets/images/post_THM_gallery/Screenshot_23.png)

Ahora seguimos los pasos que nos indicaba anteriormente para conseguir una shell gracias al comando "nano" en el fichero "txt" que nos abre.

![](/assets/images/post_THM_gallery/Screenshot_24.png)

y listo....... ![](/assets/images/post_THM_gallery/giphy.gif)

# <ins> Flag ROOT </ins>

![](/assets/images/post_THM_gallery/Screenshot_25.png)








