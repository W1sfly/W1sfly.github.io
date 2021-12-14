---
layout: single
title: HackmyVM  SERVE
excerpt: "Máquina HackmyVM - SERVE - Dificultad:Easy"
date: 14-12-2021
classes: wide
header:
  teaser: assets/images/post_hackmyvm_serve/fondo.png
  teaser_home_page: true
categories:
  - HackmyVM
  - CTF
tags:
  - Linux
  - Easy
  - Brute Force
  - Ssh
  
---

![](/assets/images/post_hackmyvm_serve/serve.png)


**Resumen:**

|Herramientas|
|------------|
|Nmap|
|Gobuster|
|John|
|Hydra|

|Escalada de privilegios|
|-----------------------|
|id_rsa|
|RSAcrack|
|sudo -l|

Vamos a probar la plataforma HackMyVM con la siguiente máquina "easy", una máquina sencilla pero viene bien para afianzar algunas cosas como por ejemplo el tema de las claves SSH.

# Nmap

Hacemos un nmap para ver que puertos tiene abierto la máquina, viendo el resultado ya sabemos que vamos a tener una web y una conexión ssh.

![](/assets/images/post_hackmyvm_serve/1.png)

Tiramos otra vez de nmap pero ahora veremos que versiones tienen esos puertos.  
Estamos ante un Linux y tenemos un apache 2.4.

![](/assets/images/post_hackmyvm_serve/2.png)

# WEB

Abrimos un navegador y metemos la IP para ver que nos muestra, pero solo vemos esto:

![](/assets/images/post_hackmyvm_serve/3.png)

# GoBuster

Como no vemos nada más vamos a ver si encontramos más directorios con esa IP.  
Encontramos dos que nos llaman la atención, un "/secrets" y un "/webdab"

![](/assets/images/post_hackmyvm_serve/4.png)

Miramos primero el "/secrets" pero no nos muestra nada.......

![](/assets/images/post_hackmyvm_serve/5.png)

Ahora miramos el de "/webdab" y como ya nos imaginábamos nos da un panel de login.  
Intentamos algunas "pass" por defecto pero no logramos acceso.

![](/assets/images/post_hackmyvm_serve/6.png)

Podemos mirar ahora si hay archivos html, txt ..... así que volvemos a lanzar "gobuster".  
Encontramos un txt que nos puede servir:

![](/assets/images/post_hackmyvm_serve/7.png)

Lo ponemos en la URL para ver si podemos ver su contenido.  
Nos encontramos ante un mensaje en el cual le está indicando a "teo" (ya tenemos un posible usuario) que sus credenciales están en el directorio "/secrets" en una base de datos, directorio que ya sabíamos que existía.

![](/assets/images/post_hackmyvm_serve/8.png)

Recordamos que poniéndolo en la URL no nos mostraba nada, así que, otra vez con gobuster vamos a buscar pero ahora en ese directorio en concreto, para la búsqueda, como sabemos que lo que queremos es una base de datos, buscamos en google que tipo de extensiones puede tener una y nos indica "KDBX", así que haremos una búsqueda con esa extensión y damos con una.

![](/assets/images/post_hackmyvm_serve/9.png)

Nos lo descargamos, para abrirlo tenemos que hacerlo con “KeePass” ya que son archivos que utiliza ese programa, pero claro, la base de datos descargada tiene que tener unas credenciales para abrirla, para intentar romperla vamos a hacer lo siguiente:

![](/assets/images/post_hackmyvm_serve/10.png)

Lo anterior lo guardamos en un fichero para intentar romperla con John.  
Conseguimos descifrar la "pass" de la base de datos:

![](/assets/images/post_hackmyvm_serve/11.png)

Con esa contraseña ya podremos abrir la base de datos y encontramos las credenciales para el login webdav.

![](/assets/images/post_hackmyvm_serve/12.png)

Pero claro en la pass “w3bd4vXXX” vemos “XXX”, la nota nos decía también que teníamos que cambiar esas X por el número de trabajador, así que tenemos que averiguar cuál es ese número. Con burpsuite no vamos a poder hacerlo ya que no tenemos el campo de la pass en texto plano así que tenemos que recurrir a “hydra” que nos servirá para hacer un ataque de fuerza bruta a un login, pero primero tenemos que crearnos un diccionario con todos los casos posibles, que tendría que ir desde el 001 al 999, para ello:

![](/assets/images/post_hackmyvm_serve/13.png)

Ejecutamos Hydra y nos da la contraseña completa:

![](/assets/images/post_hackmyvm_serve/14.png)

Ahora la probamos y obtenemos acceso. Vemos un directorio pero no contiene nada.

![](/assets/images/post_hackmyvm_serve/15.png)

Podemos probar si tenemos permisos de escritura en ese directorio, para ello hacemos lo siguiente donde vemos que si podemos subir archivos.

![](/assets/images/post_hackmyvm_serve/16.png)

Ahora la idea sería de obtener una shell. Nos creamos una Reverse shell en php, la subimos, nos ponemos a la escucha y la ejecutamos:

![](/assets/images/post_hackmyvm_serve/17.png)

Nos fijamos que no somos el usuario "Teo", si nos vamos a su directorio no tenemos permiso.

![](/assets/images/post_hackmyvm_serve/18.png)

# Escalada del usuario ww-data a teo

Buscamos las formas de convertirnos en el usuario "teo".  
Hacemos un “sudo -l” y vemos que podemos ejecutar el wget sin contraseña como el usuario teo, tenemos que pensar la manera de utilizar ese comando para convertirnos en ese usuario, si pensamos un poco teníamos un servicio ssh abierto.

![](/assets/images/post_hackmyvm_serve/35.png)

Lo podemos hacer de dos formas:  
# Opción 1:

Creandonos una id_rsa.pub propia y metiendola en "authorized_keys" de la víctima.  

Vamos a generarnos una clave con el comando “ssh-keygen”, esto nos dará dos claves, una privada(id_rsa) y una pública (id_rsa.pub), ahora la idea es meter nuestra clave publica en el “authorized_keys” del usuario teo, para que cuando nos conectemos con nuestra clave pública (que la hemos autorizado) nos coja nuestra clave privada que la tenemos en nuestro ordenador y consigamos la conexión.  
Nos montamos un servidor donde vamos a compartir las claves:

![](/assets/images/post_hackmyvm_serve/19.png)

Ahora en el equipo victima ejecutamos el “wget” para que coja nuestra clave pública y la meta en el fichero “authorized_keys”.

![](/assets/images/post_hackmyvm_serve/20.png)

Lo probamos y confirmamos que podemos conectarnos por ssh con nuestra clave.

![](/assets/images/post_hackmyvm_serve/21.png)

# Opción 2:

Podemos coger la clave privada de teo y conectarnos con esa clave.  
Hacemos lo siguiente para que nos muestre la clave privada.

![](/assets/images/post_hackmyvm_serve/22.png)

Al ponernos a la escucha nos mostrará esa clave privada por consola.

![](/assets/images/post_hackmyvm_serve/23.png)

Esta clave nos la copiamos, le damos permiso “600” y nos conectamos de la siguiente manera. 

(**para hacer esta prueba he reseteado la máquina porque anteriormente lo hicimos de otra manera**)  

Observamos que nos pide una pass. 

![](/assets/images/post_hackmyvm_serve/24.png)

Con la siguiente herramienta podemos obtenerla.

![](/assets/images/post_hackmyvm_serve/25.png)

y ahora lo volvemos a repetir introduciendo la contraseña que hemos conseguido.

![](/assets/images/post_hackmyvm_serve/26.png)

# Flag Usuario

![](/assets/images/post_hackmyvm_serve/27.png)

# Escalada de privilegios

Comprobamos que no tenemos acceso al directorio "root".

![](/assets/images/post_hackmyvm_serve/28.png)

Volvemos hacer un sudo -l y vemos ahora que podemos ejecutar otro comando pero esta vez como root sin proporcionar contraseña.

![](/assets/images/post_hackmyvm_serve/29.png)

Lo ejecutamos para ver que hace ya que no tenemos ni idea que es y nos da un ejemplo a ejecutar.

![](/assets/images/post_hackmyvm_serve/30.png)

Ejecutamos ese ejemplo a ver que sucede.

![](/assets/images/post_hackmyvm_serve/31.png)

Y nos fijamos que al final de la página se queda pendiente porque no se ha cargado todo.  
Esos ":" lo podemos aprovechar para introducir comandos.

![](/assets/images/post_hackmyvm_serve/32.png)

Lo que nos interesa es una bash como root, así que ponemos lo siguiente y lo conseguimos.

![](/assets/images/post_hackmyvm_serve/33.png)

# Flag ROOT

![](/assets/images/post_hackmyvm_serve/34.png)

![](/assets/images/post_hackmyvm_serve/gi.gif)









