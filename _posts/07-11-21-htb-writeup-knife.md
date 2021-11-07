---
layout: single
title: HTB  CAP
excerpt: "Máquina HTB - Knife - Dificultad:Easy"
date: 07-11-2021
classes: wide
header:
  teaser: assets/images/post_HTB_knife/knife.png
  teaser_home_page: true
categories:
  - HTB
  - CTF
tags:
  - Linux
  - Easy
  - knife
---

![](/assets/images/post_HTB_knife/kniferesum.png)


**Resumen:**

|Herramientas|
|------------|
|Nmap|
|Nikto|
|Exploit PHP 8.1.0-dev|

|Escalada de privilegios|
|-----------------------|
|knife|

Vamos a seguir subiendo "writeup" de maquinas "easy" para ir avanzando poco a poco.

Comencemos......................

# Nmap

Hacemos un escaneo con nmap, en este caso con el “-A” nos detectará las versiones, servicios...... de todo un poco, éste parámetro no es muy recomendable ya que es bastante ruidoso.

En "writeup" posteriores iremos probando con otros parámetros de escaneo más acordes..

* Encontramos dos puertos abiertos:
    * Puerto 22 --> Vemos un servicio SSH.
    * Puerto 80 --> Sabemos que hay una web con un servidor "apache". 

![](/assets/images/post_HTB_knife/nmap.png)

Abrimos nuestro navegador y metemos la IP para ver que nos muestra a través del puerto 80 descubierto.

Aparentemente no vemos gran cosa.

![](/assets/images/post_HTB_knife/web.png)

# Nikto

Lanzaremos la herramienta "Nikto" para ver si encuentra algunas vulnerabilidades.  
Examinando lo que nos ha encontrado, damos con una versión PHP un poco sospechosa.

![](/assets/images/post_HTB_knife/nikto.png)

# Exploit

Investigamos un poco esa versión (PHP 8.1.0 dev) y nos encontraremos que es vulnerable.  
Luego nos informamos sobre esta vulnerabilidad para ver de que trata y vemos como se puede ejecutar código enviado en el "User-Agent".

Buscando exploit vemos el siguiente que ataca esa vulnerabilidad:  

![](/assets/images/post_HTB_knife/exploit.png)

Antes, primero miramos como se utiliza y vemos que es bastante sencillo, sólo tenemos que pasarle con el parámetro "-l" la URL de nuestra víctima.

![](/assets/images/post_HTB_knife/exploituso.png)

Así que nos lo descargamos y le damos permiso de ejecución.

![](/assets/images/post_HTB_knife/exploitperm.png)

Lo ejecutamos como indicamos anteriormente y observamos como conseguimos ser el usuario "james".

![](/assets/images/post_HTB_knife/exploitus.png)

Intentamos ejecutar algunos comandos pero la "shell" obtenida está muy limitada, aún así, podemos visualizar la flag del usuario.

![](/assets/images/post_HTB_knife/Flaguser.png)

# Escalar Privilegios

Buscamos a ver como podemos convertirnos en "root".  
Ejecutamos lo siguiente para ver si hay algún comando que podamos ejecutar con algún privilegio y vemos como podemos ejecutar el comando "knife" como "root" sin proporcionar contraseña.

![](/assets/images/post_HTB_knife/sudo-l.png)

Pero antes de ver como aprovecharnos de este comando, tenemos que solucionar el problema anterior con el exploit en el tema de ejecución de comandos, como comentábamos, tenemos muchos problemas para ejecutar ciertos comandos ya que la "Shell" es muy limitada, a la hora de intentar escalar privilegios nos va a ser imposible, así que tenemos que buscar otra alternativa.
  
Buscando otro exploit encontramos el siguiente:  

![](/assets/images/post_HTB_knife/exploit2.png)

Miramos como funciona y vemos en el código (en el anterior exploit no lo puse)como utiliza el "User-Agent" para intrducirle el payload que está configurado y el cual indicandole el "lhost" y el "lport" nos consigue una shell.

![](/assets/images/post_HTB_knife/CONFEXPL.png)


Lo ejecutamos como nos indica y.............conseguimos, nuevamente, ser el usuario "james" pero ésta vez a través de una bash.

![](/assets/images/post_HTB_knife/exploit2ex.png)

Ahora ya procederemos a escalar privilegios gracias al comando que podemos ejecutar como root sin proporcionar contraseña.  
Investigando un poco por la web, encontramos la siguiente página, la cual nos indica como podemos aprovecharnos de ese privilegios para conseguir una "sh" con privilegios "sudo".  

![](/assets/images/post_HTB_knife/sudoknife.png)

Ejecutando el anterior comando conseguimos convertirnos en "root".

![](/assets/images/post_HTB_knife/sudoknife2.png)

Ya solo nos queda meternos en el directorio "root" y visualizar la Flag.  
Otra cosa que podemos hacer al ejecutar "knife" es que, en vez de que nos de una "shell" con privilegios, poder visualizar directamente la Flag.

![](/assets/images/post_HTB_knife/flagroot.png)

# Otra forma de explotar PHP 8.1.0-dev

Vamos a explotar la vulnerabilidad de otra forma, ahora lo haremos de forma manual, es decir, sin tirar ningún exploit.  
Para ello utilizaremos el anterior exploit:

![](/assets/images/post_HTB_knife/exploit3.png)

Como vimos anteriormente se aprovecha del "User-Agent" para meterle un payload.

![](/assets/images/post_HTB_knife/exploit3com.png)

Pero ahora en vez de lanzar el exploit y darnos una shell automáticamente como antes, vamos a coger solo lo que nos interesa, que es la parte del "User-Agent" y vamos a modificarlo para que por ejemplo, como vamos a poder ejecutar comandos nos ejecute un "id".

![](/assets/images/post_HTB_knife/exploit3ex.png)

Y ahora probamos como entablarse una shell.

![](/assets/images/post_HTB_knife/exploit3us.png)

Es bueno conocer como funciona un exploit por dentro y ver que hace.  
Aquí he mostrado como se ha llegado a la misma conclusión utilizando la forma "automática" y la forma "manual".

A seguir aprendiendo....................

![](/assets/images/post_HTB_knife/hack.gif)






