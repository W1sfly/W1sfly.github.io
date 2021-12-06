---
layout: single
title: HTB  INTELLIGENCE
excerpt: "Máquina HTB - Intelligence - Dificultad:Medium"
date: 06-12-2021
classes: wide
header:
  teaser: assets/images/post_HTB_intelligence/icono.png
  teaser_home_page: true
categories:
  - HTB
  - CTF
tags:
  - Windows
  - Medium
  - AD
---

![](/assets/images/post_HTB_intelligence/1.png)


**Resumen:**

|Cosas que veremos|
|------------|
|SMB|
|Ldap|
|Kerbero|
|Python|

|Escalada de privilegios|
|-----------------------|
|Bloodhound|

Estamos ante una máquina Windows en el que se toca un Active Directory, una máquina bastante chula en la que se han tenido que probar muchas cosas para llegar a completarla. 

# Nmap

Primero vemos que puertos tiene abiertos y ya vamos viendo que estamos ante un AD (puerto 445, 389, 5985,......).

![](/assets/images/post_HTB_intelligence/2.png)

Ahora, con todos los puertos que ya hemos descubierto vamos a ir viendo que versiones y que información extra nos da a través de los script básicos de nmap.

![](/assets/images/post_HTB_intelligence/3.png)
  
# Web

Como tenemos un puerto 80 abierto vamos a ver que contiene, inspeccionamos un poco la web y vemos que hay algunos pdf que nos podemos descargar.

![](/assets/images/post_HTB_intelligence/5.png)

Si probamos a meternos en el directorio donde se alojan esos documentos no nos deja.

![](/assets/images/post_HTB_intelligence/17.png)

Le hacemos un Whatweb por si obtuviéramos información extra.

![](/assets/images/post_HTB_intelligence/7.png)

# Le metemos mano al AD

Vamos hacer una serie de comprobaciones para conocer un poco como está montado todo.  
Estamos ante un Windows10, en el que vemos su dominio (ya lo sabíamos) y en el que vemos que el SMB está firmado (esta vez lo han firmado grrrrrrrrrr)

![](/assets/images/post_HTB_intelligence/8.png)

Nos conectamos utilizando una "null session" pero no tenemos acceso.

![](/assets/images/post_HTB_intelligence/9.png)

Probamos a ver si hay algún recurso compartido con algún permiso.......pero nada.

![](/assets/images/post_HTB_intelligence/10.png)

Como tenemos un servidor LDAP vamos a enumerarlo y buscar si los "namingcontexts" (Son los DN --nombre de dominio-- base disponibles) nos da algo interesante.

![](/assets/images/post_HTB_intelligence/11.png)

# Volvemos a la parte Web

Parece que lo tenemos difícil…………………pensando un poco y recapitulando lo que llevamos hasta el momento se nos ocurre una cosa, teníamos dos archivos pdf que podíamos descargar, vamos a mirar sus metadatos a ver si nos da algo interesante.

Damos con un posible usuario.

![](/assets/images/post_HTB_intelligence/13.png)

Si miramos el otro vemos otro nombre --> William.Lee

Así que ya tenemos dos posibles usuarios, vamos a probar si son válidos o no. Para ello vamos a utilizar “kerbrute” para que con fuerza bruta descubra cuentas de usuarios que no requieran pre-autenticación. Al fichero que ya tengo para probar usuarios le vamos a agregar los dos que hemos encontrados y observamos cómo nos valida que existen y vemos también el de “administrator”.

![](/assets/images/post_HTB_intelligence/16.png)

Como tenemos usuarios potenciales pero no tenemos "pass" vamos hacer un "getNPUsers".

![](/assets/images/post_HTB_intelligence/15.png)

Tenemos que pensar un poco ya que parece que no podemos acceder por ningún lado................  
 
Viendo el nombre de los "pdf" nos hace pensar que pueden existir más ya que están fechados, la idea sería probar con más fechas, pero eso hacerlo a mano nos llevaría un tiempo bastante curioso así que nos montamos un pequeño script en python para, dada unas fechas, pruebe si existen en el servidor y nos lo descargue.

Versión python:

![](/assets/images/post_HTB_intelligence/19.png)

Versión Bash:

![](/assets/images/post_HTB_intelligence/20.png)

Comprobamos como había muchos más en el servidor y probamos uno a ver si nos sigue dando más usuarios.

![](/assets/images/post_HTB_intelligence/22.png)

Ahora tendríamos que verlos todos ya que seguramente nos darán más, para ello:

![](/assets/images/post_HTB_intelligence/23.png)

Probamos si esos usuarios son válidos.

![](/assets/images/post_HTB_intelligence/24.png)

Pero si lo utilizamos con "getNPUsers" nos da lo mismo que antes.

Ya lo único que nos quedaría (ya que necesitamos una "pass") es ver el contenido de esos "pdf" a ver si en alguno hay algo que nos pueda servir.

Para tener el contenido de todos y después poder filtrarlo en busca de palabras claves vamos hacerlo de la siguiente manera.

Utilizaremos "textract" para extraer el contenido, pero para que nos lo haga bien tenemos que ponerlos todos de la siguiente manera:

![](/assets/images/post_HTB_intelligence/25.png)

Ahora con python vamos a utilizar lo que hemos comentado antes el cual ira fichero por fichero extrayendo el contenido.

![](/assets/images/post_HTB_intelligence/26.png)

Al ejecutarlo guardaremos la salida en un archivo.

![](/assets/images/post_HTB_intelligence/27.png)

Con el contenido de todos los pdf filtraremos por palabras claves y vemos que con "pass" encontramos una posible password.

![](/assets/images/post_HTB_intelligence/28.png)

Ahora falta saber a qué usuario pertenece, como teníamos una lista de usuarios válidos lo probamos y damos con el usuario.

![](/assets/images/post_HTB_intelligence/29.png)

Validamos la pass, que es válida por smb pero no conseguimos un "pwn3d"

![](/assets/images/post_HTB_intelligence/30.png)

Como tenemos el puerto 5985 vamos a probar si podemos conectarnos con "evil-wnrm" pero no lo conseguimos.

![](/assets/images/post_HTB_intelligence/31.png)

Al tener usuario y contraseña podemos probar con "getUserSPNs" pero vemos que el usuario no es Kerberosteable.

![](/assets/images/post_HTB_intelligence/32.png)

Se nos van acabando otra vez las opciones............................. 


![](/assets/images/post_HTB_intelligence/bird.gif)


Probamos a mirar otra vez los recursos pero ahora con el usuario y pass encontrado y vemos que podemos leer algunos.

![](/assets/images/post_HTB_intelligence/34.png)

Nos conectamos al recurso “Users” 

![](/assets/images/post_HTB_intelligence/35.png)

# Flag User

![](/assets/images/post_HTB_intelligence/36.png)

A pesar de ya tener por lo menos la flag del usuario no tenemos todavía acceso a la máquina, así que vamos a ir mirando los demás directorios encontrando en uno de ellos un ".ps1" (inspeccionamos el código para ver un poco que es lo que hace)

![](/assets/images/post_HTB_intelligence/37.png)

Vamos a seguir obteniendo un poco más de información, ahora veremos cómo está estructurado el directorio, para ello:

![](/assets/images/post_HTB_intelligence/38.png)

y vemos los archivos que nos ha creado, entre ellos vemos a un usuario el cual vemos un “trusted to auth” con el que podemos impersonalizar a un usuario como por ejemplo el "administrador", así que tenemos que ir a por el usuario svc-int.

![](/assets/images/post_HTB_intelligence/39.png)

Esto nos indica que usuarios o grupos pueden obtener la pass de svc_int, cabe recordar que GMSA es “cuenta de servicio administrada de grupo”.

![](/assets/images/post_HTB_intelligence/41.png)

Con los ficheros obtenidos anteriormente buscamos que usuarios hay en el grupo "IT Support" y damos con dos:

![](/assets/images/post_HTB_intelligence/42.png)

Para llegar a "svc_int" vamos a tener que ir ahora por "TEd.Graves", ya que si observamos cuando nos metimos en el recurso "Users" a parte de "TIffany" aparecía "Ted".

Vamos a lanzar "bloodhound" para ver un poco si nos aclara las ideas.

![](/assets/images/post_HTB_intelligence/bloodhountifan.png)

El recorrido que tenemos que hacer es el siguiente:

![](/assets/images/post_HTB_intelligence/58.png)

Vemos la manera de poder llegar a "svc" pero en este caso no podemos utilizar esa herramienta, utilizariamos en su defecto "gMSADumper" que ya fue la que utilizamos con "Tiffany" para saber que usuarios llegaban a "svc-int".

![](/assets/images/post_HTB_intelligence/57.png)

Pero claro lo tenemos que hacer con "Ted"...........

# Conseguir las credenciales de Ted

El script que vimos en uno de los directorios vemos que se ejecuta cada 5 minutos, va mirando los registros DNS y si alguna empieza por “web” envía una solicitud con el dominio encontrado utilizando las credenciales de Ted.

![](/assets/images/post_HTB_intelligence/43.png)

Investigando un poco a ver que podemos utilizar para explotar el código anterior damos con la siguiente opción, que es la de crearnos un registro DNS que apunte a nuestra IP, para ello utilizamos la siguiente herramienta.

![](/assets/images/post_HTB_intelligence/45.png)

Comprobamos que se ha añadido el nuevo record que además apunta a nuestra ip.

![](/assets/images/post_HTB_intelligence/46.png)

Ahora como lo anterior ponía que se ejecutaba cada 5 minutos lo que hacemos es poner el “responder” y dejarlo en espera hasta que se active el script, el cual nos dará el hash de Ted:

![](/assets/images/post_HTB_intelligence/47.png)

Esperamos un poco y ...................

![](/assets/images/post_HTB_intelligence/48.png)

Nos lo copiamos y utilizamos john para romperla.

![](/assets/images/post_HTB_intelligence/49.png)

Validamos las credeciales.

![](/assets/images/post_HTB_intelligence/50.png)

Ahora que ya tenemos las credenciales de "Ted" ya podemos ir a por "svc_int" y conseguir su hash.

![](/assets/images/post_HTB_intelligence/52.png)

Intentamos crackearlo con "John" pero no es posible.

![](/assets/images/post_HTB_intelligence/53.png)

Bueno lo importante es que tenemos ya también al usuario "svc_int", ahora tenemos que ver cómo llegar al dominio.

Ahora entra en la acción lo que vimos con "ldapdomaindump" sobre el tema de poder impersonalizar a un usuario, para ello vemos el "esquemita" de "bloodhound" para hacernos una idea.  
Utilizaremos la siguiente herramienta, getST, para que nos proporcione un ticket con el que suplantaremos en este caso al usuario “administrator”, pero antes de lanzarla nos hace falta un dato, que es:

![](/assets/images/post_HTB_intelligence/59.png)

Nos da un error de sincronización de reloj entre servidor y máquina.

![](/assets/images/post_HTB_intelligence/60.png)

Lo solucionamos con:

![](/assets/images/post_HTB_intelligence/61.png)

Y ahora si nos crea el ticket.

![](/assets/images/post_HTB_intelligence/62.png)

Una vez que tenemos el ticket, “administrator.ccache”, lo utilizamos de la siguiente manera, donde con la variable de entorno “KRB5CCNAME” se lo agregamos.

![](/assets/images/post_HTB_intelligence/64.png)

# Flag Root

![](/assets/images/post_HTB_intelligence/66.png)![](/assets/images/post_HTB_intelligence/cat.gif)

Otra forma hubiera sido conectándonos al recurso compartido pero ahora con privilegios máximos.

![](/assets/images/post_HTB_intelligence/67.png)

Descargamos la flag y la visualizamos.

![](/assets/images/post_HTB_intelligence/68.png)

![](/assets/images/post_HTB_intelligence/69.png)  




