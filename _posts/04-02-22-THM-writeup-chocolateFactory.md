---
layout: single
title: TryHackMe  Chocolate Factory
excerpt: "Máquina THM - Chocolate Factory - Dificultad:Easy"
date: 04-02-2022
classes: wide
header:
  teaser: assets/images/post_THM_ChocolateFactory/portada2.png
  teaser_home_page: true
categories:
  - TryHackMe
  - CTF
tags:
  - Linux
  - Easy
  - Esteganografía
  - john
  - RCE
  
---

![](/assets/images/post_THM_ChocolateFactory/portada3.png)


# <ins>Resumen:</ins>

![](/assets/images/post_THM_ChocolateFactory/portada.png)

Muchos puertos en los que hay que fijarse bien, un ftp sencillo, esteganografía a jpg sin complicaciones, fuerza bruta rápida, RCE, sudo -l(vi) y un python para conseguir la flag root.

# <ins>Reconocimiento</ins>

Descubrimos los puertos que tiene la máquina víctima, entre ellos vemos un ftp, ssh y http.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_1.png)

Lanzamos los script básicos de reconocimiento y obtenemos las versiones de los servicios anteriores.
Cosas interesante que vemos es un ftp con anonymous.

 ![](/assets/images/post_THM_ChocolateFactory/Screenshot_2.png)
 
 A demás, en lo otros puertos vemos un mismo mensaje.
 
 ![](/assets/images/post_THM_ChocolateFactory/Screenshot_3.png)
 
 Pero si miramos uno por uno nos damos cuenta que hay uno que difiere de los demás, donde nos muestra un fichero que contiene una key.
 
![](/assets/images/post_THM_ChocolateFactory/Screenshot_4.png)
 
# <ins>FTP</ins>

Nos conectamos al servicio FTP con "anonymous" y vemos que contiene una imagen jpg, nos la descargamos.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_5.png)

# <ins>Steganografía</ins>

Con la imagen ejecutamos steghide para ver si contiene algo y vemos como tiene un txt.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_6.png)

Lo extraemos....

![](/assets/images/post_THM_ChocolateFactory/Screenshot_7.png)

y nos damos cuenta que estamos ante un "base64"

![](/assets/images/post_THM_ChocolateFactory/Screenshot_8.png)
 
Le hacemos un decode y obtenemos usuario y pass.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_9.png)

Nos la guardamos en un fichero y aplicamos fuerza bruta con john.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_10.png)

# <ins>WEB</ins>

La probamos por si fueran las de SSH pero no lo son, así que al abrir la WEB nos encontramos ante un login.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_11.png)


Metemos las credenciales que obtuvimos anteriormente y entramos.
Vemos como podemos ejecutar comandos.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_12.png)

Y ahora podemos hacer dos cosas:

- Entablarnos una reverseShell con python3, pero nos daría una shell con un usuario con pocos privilegios.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_24.png)

- Si pensamos un poco vemos como podemos ejecutar comandos y movernos entre directorios, como tenemos el puerto SSH abierto lo lógico es que el usuario "charlie" tenga una clave "id_rsa", así que nos vamos a su directorio y la visualizamos, pero tiene una trampa, que si vamos directo no la veremos ya que no se encuentra en "/.ssh/id_rsa", si no que se encuentra en el mismo directorio del usuario y con diferente nombre.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_14.png)

Ya solo faltaría guardárnosla, darle un formato correcto y darle los permisos adecuados.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_15.png)

# <ins> Flag USER </ins>  

![](/assets/images/post_THM_ChocolateFactory/Screenshot_16.png)

# <ins> Escalar Privilegios </ins>

Miramos y vemos el comando "vi"

![](/assets/images/post_THM_ChocolateFactory/Screenshot_17.png)

Buscamos y encontramos como explotarlo,

![](/assets/images/post_THM_ChocolateFactory/Screenshot_18.png)

Ahora siendo root buscamos la flag pero nos encontramos ante un script en python el cual nos pide una key para darnos la FLAG.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_19.png)

La key sabemos dónde está, ya que nos la proporciono con nmap, así que nos dirigimos al directorio y vemos su contenido el cual es la key que tenemos que utilizar.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_20.png)

Al ejecutar el script en python e introducirle la key que nos ha dado nos da problemas de código

![](/assets/images/post_THM_ChocolateFactory/Screenshot_21.png)

Tenemos que cambiar el código quitándole lo innecesario e introducimos directamente la key.

# <ins> Flag ROOT </ins>

Y al ejecutarlo si nos da ahora la FLAG.

![](/assets/images/post_THM_ChocolateFactory/Screenshot_23.png)








