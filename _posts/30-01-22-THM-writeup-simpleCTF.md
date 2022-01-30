---
layout: single
title: TryHackMe  simple CTF
excerpt: "Máquina THM - simple CTF - Dificultad:Easy"
date: 30-01-2022
classes: wide
header:
  teaser: assets/images/post_THM_simpleTCF/Screenshot_2.png
  teaser_home_page: true
categories:
  - TryHackMe
  - CTF
tags:
  - Linux
  - Easy
  - CMS Made Simple
  
---

![](/assets/images/post_THM_simpleTCF/Screenshot_1.png)


# <ins>Resumen:</ins>

Máquina bastante sencilla donde lo más interesante nos lo encontramos en la explotación del CMS.  

# <ins>Reconocimiento</ins>

Comenzamos lanzando un nmap para ver qué puertos tiene abiertos la máquina, hacemos un escaneo bastante ruidoso pero rápido.  
Nos encontramos ante un ftp, una web (que en ese momento nos lo está marcando como cerrado) y otro puerto un poco diferente.

![](/assets/images/post_THM_simpleTCF/Screenshot_3.png)

Una vez que tenemos los puertos vamos a ver que versiones e información extra podemos obtener.  
Empezamos a ver cosas interesantes, el servicio FTP tiene acceso a través de "anonymous", el HTTP ya nos aparece como "abierto", donde vemos un robots.txt con un directorio y el puerto que vimos anteriormente el cual era un poco diferente ya podemos ver como aloja un SSH.

 ![](/assets/images/post_THM_simpleTCF/Screenshot_4.png)
 
 Viendo que tenemos una web, vamos a ver ante que nos encontramos pero solo veremos info de apache, así que vamos a ver si encontramos algún directorio.  
 Ejecutamos "gobuster" y encontramos uno:
 
 ![](/assets/images/post_THM_simpleTCF/Screenshot_5.png)
 
Lo introducimos en la URL y ya vemos que nos encontramos ante el gestor de contenido "CMS Made SImple".

 ![](/assets/images/post_THM_simpleTCF/Screenshot_6.png)
 
 Si ya nos hemos topado ante este gestor de contenido sabremos que tiene un login y que hay versiones vulnerables. Primero vamos a ver donde se encuentra el login, para ello, volvemos a ejecutar "gobuster" y encontramos el login de la web.
 
 ![](/assets/images/post_THM_simpleTCF/Screenshot_7.png)
 
 Ahora nos faltaría saber que versión tiene este gestor de contenidos, miramos por la web y en la misma página inicial la encontramos.
 
![](/assets/images/post_THM_simpleTCF/Screenshot_8.png)

# <ins> Explotación </ins>

Buscamos vulnerabilidades asociadas a esa versión y encontramos una:

![](/assets/images/post_THM_simpleTCF/Screenshot_9.png)

Nos la descargamos, miramos un poco su funcionamiento y ejecución y la ejecutamos, al ejecutarla nos encontramos con errores de código.

![](/assets/images/post_THM_simpleTCF/Screenshot_10.png)

Su solución es bastante sencilla, podemos utilizar el programa "2to3" que nos facilitará el trabajo.  

Una vez ejecutada vamos a conseguir al usuario del panel de login.

![](/assets/images/post_THM_simpleTCF/Screenshot_11.png)

Ahora nos encontraremos ante un problema, el anterior exploit nos da la posibilidad de, agregando una wordlist, poder crackear la contraseña que anteriormente también ha encontrado, pero nos va a volver a dar problemas de código, tendremos que encontrar los fallos y arreglarlo pero.............podemos también utilizar otra alternativa que es HYDRA, para ello tendremos que conocer como se llaman los campos del login, una vez lanzada obtendremos la pass del usuario.

![](/assets/images/post_THM_simpleTCF/Screenshot_12.png)

Y nos logeamos

![](/assets/images/post_THM_simpleTCF/Screenshot_13.png)

La idea es conseguir una shell, podemos hacerlo de diferentes formas, aquí lo haremos a través de la subida de una reverseShell en el apartado "FileMager" donde nos permite subir ficheros pero no con extensión "php" ya que no nos dejará subir ese tipo de extensiones, pero si buscamos un poco por "google" veremos que hay un tipo de extensión que si nos la permite.

![](/assets/images/post_THM_simpleTCF/Screenshot_14.png)

Ahora nos ponemos a la escucha y ponemos en la URL la dirección donde se encuentra la "reverseShell" que subimos anteriormente logrando el fin buscando.

![](/assets/images/post_THM_simpleTCF/Screenshot_15.png)

Tenemos un usuario limitado que es "www-data" pero tenemos que tener en cuenta que tenemos una contraseña del usuario "mitch" por lo que vamos a migrar a ese usuario. (otra forma hubiera sido conectarnos a través de SSH ya que recordamos que tenemos el puerto 2222 con ese servicio)

![](/assets/images/post_THM_simpleTCF/Screenshot_16.png)

# <ins> Flag USER </ins>  


![](/assets/images/post_THM_simpleTCF/Screenshot_17.png)

# <ins> Escalar Privilegios </ins>

Vemos que tenemos otro usuario en el directorio "home" donde no tenemos permisos de acceso, así que vamos a empezar a buscar formas de escalar privilegios, ejecutamos un "sudo -l" a ver si hay algún tipo de privilegios donde poder aprovecharnos, y vemos uno, "vim", el cual podemos ejecutarlo como root sin proporcionar contraseña.

![](/assets/images/post_THM_simpleTCF/Screenshot_18.png)

Buscamos como poder utilizarlo

![](/assets/images/post_THM_simpleTCF/Screenshot_19.png)

Lo ejecutamos como nos indica la información anterior y conseguimos ser "root"

![](/assets/images/post_THM_simpleTCF/Screenshot_20.png)

# <ins> Flag ROOT </ins>

![](/assets/images/post_THM_simpleTCF/Screenshot_21.png)








