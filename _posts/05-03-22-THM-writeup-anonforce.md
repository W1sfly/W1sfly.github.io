---
layout: single
title: TryHackMe  AnonForce
excerpt: "Máquina THM - Anonforce - Dificultad:Easy"
date: 05-03-2022
classes: wide
header:
  teaser: assets/images/post_THM_anonforce/Screenshot_1.png
  teaser_home_page: true
categories:
  - TryHackMe
  - CTF
tags:
  - Linux
  - Easy
  - ftp
  - Claves asc/pgp
  
---

![](/assets/images/post_THM_anonforce/Screenshot_2.png)

URL: <https://tryhackme.com/room/bsidesgtanonforce>


# <ins>Resumen:</ins>

Nos encontramos ante un máquina bastante sencilla, no muestra ningún problema pero tiene un punto bueno en el caso de que nunca se haya tocado y son las claves asc y pgp, veremos qué hacer con ellas.

# <ins>Comenzamos.......</ins>

Lo primero lanzamos un nmap para ver que puertos nos encontramos abiertos, en este caso tenemos solo dos y ya nos hace ver que vamos a tener que sacar la información del servicio FTP.

![](/assets/images/post_THM_anonforce/Screenshot_3.png)

Ahora vemos como el servicio FTP vamos a poder entrar con "Anonymous" y a demás nos encontramos que podemos ver todos los directorios de la máquina víctima.

![](/assets/images/post_THM_anonforce/Screenshot_4.png)

Nos metemos en el FTP y empezamos a investigar un poco las carpetas aunque a primera vista ya hay una que nos llama la atención, "notread", nos metemos y vemos dos archivos interesantes (si no sabéis que es os recomiendo un poco de búsqueda por google), procedemos a su descarga.

![](/assets/images/post_THM_anonforce/Screenshot_5.png)

Seguimos mirando y en el directorio "home" nos encontramos con el nombre del usuario y con su FLAG.

![](/assets/images/post_THM_anonforce/Screenshot_6.png)


# <ins> Flag USER </ins>  

La flag del usuario ha sido regalada totalmente.

![](/assets/images/post_THM_anonforce/Screenshot_7.png)


# <ins> Escalar Privilegios </ins>

Ahora nos centraremos en los dos ficheros que vimos al principio, si abrimos el .asc ya sabemos ante que estamos (clave pgp).

![](/assets/images/post_THM_anonforce/Screenshot_8.png)

Para tratar con estos ficheros vamos a seguir los siguientes pasos:

- Extraemos el hash con "gpg2john" y lo desencriptamos con "john":

![](/assets/images/post_THM_anonforce/Screenshot_10.png)

- Importamos la clave pgp (la pass que solicita es la que hemos desencriptado anteriormente):

![](/assets/images/post_THM_anonforce/Screenshot_16.png)

- Desencriptamos el .pgp:

![](/assets/images/post_THM_anonforce/Screenshot_11.png)

y sorpresa, nos da el hash de "root".

![](/assets/images/post_THM_anonforce/Screenshot_12.png)

Lo guardamos en un archivo y le pasamos john para que nos la descifre.

![](/assets/images/post_THM_anonforce/Screenshot_13.png)

Ahora solo nos queda conectarnos por ssh.

![](/assets/images/post_THM_anonforce/Screenshot_14.png)

# <ins> Flag ROOT </ins>

Y aquí tenemos la flag.

![](/assets/images/post_THM_anonforce/Screenshot_15.png)


![](/assets/images/post_THM_anonforce/giphy.gif)





