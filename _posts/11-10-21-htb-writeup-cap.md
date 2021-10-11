---
layout: single
title: HTB  CAP
date: 11-10-2021
header:
  teaser:
categories:
  - HTB
  - CTF
tags:
  - Linux
  - Easy
  - Capabilities
---

![imagen](https://user-images.githubusercontent.com/86053823/122618877-76628180-d08f-11eb-9752-f8db1e7fed7a.png)

**Resumen:**

|Herramientas|
|------------|
|Nmap|
|Dirb|
|Wireshark|

|Escalada de privilegios|
|-----------------------|
|getcap (capabilities)|

Nos encontramos ante una máquina de dificultad EASY, veremos si es tan EASY como dicen.

- **Nmap**

Lo primero que hacemos es un escaneo con nmap, en este caso con el “-A” nos detectará las versiones, servicios...... de todo un poco, éste parámetro no es muy recomendable ya que es bastante ruidoso.

![imagen](https://user-images.githubusercontent.com/86053823/122618798-47e4a680-d08f-11eb-83a3-71e9e082c9c0.png)

Encontramos 3 servicios en los que sus versiones no tienen ninguna vulnerabilidad explotable pero que nos hacemos una idea por donde empezar a investigar

![imagen](https://user-images.githubusercontent.com/86053823/122619519-ec1b1d00-d090-11eb-884c-0f88bb283ee8.png)
![imagen](https://user-images.githubusercontent.com/86053823/122619526-efaea400-d090-11eb-8bf3-8283907d81cd.png)

Observamos como hay un puerto 80 abierto

![imagen](https://user-images.githubusercontent.com/86053823/122619532-f210fe00-d090-11eb-8b79-4a498f9258b8.png)

El siguiente paso será introducir la IP en un navegador para ver que contiene esa web. Mirando un poco la estructura nos paramos en la siguiente página donde vemos que nos podemos descargar “algo”, es un archivo “.pcap” (este tipo de archivo es utilizado por aplicaciones como wireshark para monitorear trafico de red), pero observamos que no tiene nada.

![imagen](https://user-images.githubusercontent.com/86053823/122619988-18836900-d092-11eb-9449-5ec8734aef81.png)

Analizamos las cabeceras por si vemos algo interesante, en este caso no

![imagen](https://user-images.githubusercontent.com/86053823/122619962-04d80280-d092-11eb-840c-d714c9147b8c.png)

Buscamos vulnerabilidades con nmap para ver un poco si nos da más información sobre el objetivo.

![imagen](https://user-images.githubusercontent.com/86053823/122620122-68fac680-d092-11eb-9038-83252d2263cf.png)

Y algo interesante es el /data/8, si volvemos a la url anterior vemos como el “data” que estábamos viendo es el 9, en este caso nos ha descubierto que hay un 8 y podemos intuir que hay más.

![imagen](https://user-images.githubusercontent.com/86053823/122620131-6e581100-d092-11eb-8170-60c63633bf5e.png)

Probamos con data/8 y ya vemos como hay datos, así que vamos a descargarlo a ver que contiene

![imagen](https://user-images.githubusercontent.com/86053823/122620198-98113800-d092-11eb-98f3-c2e733ab6819.png)

- **Wireshark**

Como lo que descargamos es un .pcap lo abriremos con Wireshark y nos ha dado una PASS del FTP 

![imagen](https://user-images.githubusercontent.com/86053823/122620286-d870b600-d092-11eb-9a61-6ba1d558eaea.png)

Lo probamos, lo lógico sería que el usuario fuera la que vimos en la web, "Nathan", pero al aplicarle la PASS encontrada no logramos la conexión

![imagen](https://user-images.githubusercontent.com/86053823/122620376-166dda00-d093-11eb-9a9c-1c0ed5063e4d.png)

Con esto llegamos a la conclusión que la clave tiene que estar dentro de algún pcap, hay que averiguar en cual.

- **Dirb**

Podemos buscar si hay mas cosas en la carpeta "DATA" que es donde habíamos encontrado el archivo "8"

![imagen](https://user-images.githubusercontent.com/86053823/122620493-69e02800-d093-11eb-9140-86238f7c2cd5.png)

y BINGO, hay muchos más.

![imagen](https://user-images.githubusercontent.com/86053823/122620502-6f3d7280-d093-11eb-8def-7f9123a9efee.png)

Empezaremos a probar por el primero

- **En busca de la flag de USUARIO**

Encontramos el "USER" y otra "PASS", ésta si que parece ser la buena (nos lo han puesto fácil poniendolo en el primero y no en el último)

![imagen](https://user-images.githubusercontent.com/86053823/122620570-9300b880-d093-11eb-9e56-27fd39046ce8.png)

Ahora lo comprobamos y logramos conectarnos por FTP

![imagen](https://user-images.githubusercontent.com/86053823/122620712-eecb4180-d093-11eb-83ab-f3feadfb6ba9.png)

En este paso ya podremos visualizar la FLAG del usuario.

![imagen](https://user-images.githubusercontent.com/86053823/122620747-04406b80-d094-11eb-8dda-8256026bdace.png)

Vamos hacer una prueba y es que también teníamos un protocolo SSH, podemos probar si tiene las mismas credenciales

Y exactamente tiene las mismas

![imagen](https://user-images.githubusercontent.com/86053823/122620933-7c0e9600-d094-11eb-9a0f-0bbdb9a2f7e8.png)


***Lo anterior se ha hecho todo un poco paso por paso y utilizando herramientas para descubrir cosas, pero lo podríamos haber realizado de una forma mucho más sencilla, al ver que en la url nos estaba dando un “data/8” podríamos haber probado con más números y hubiéramos llegado a la misma conclusión.

- **En busca de la flag de ROOT**

Comprobamos a ver si podemos ver los dos archivos importantes para encontrar las contraseñas de los usuarios

"passwd" podemos verlo

![imagen](https://user-images.githubusercontent.com/86053823/122621101-d7408880-d094-11eb-98eb-7f5e80f882f7.png)

Pero "shadow" no

![imagen](https://user-images.githubusercontent.com/86053823/122621188-12db5280-d095-11eb-8651-312828a44ff4.png)

Por lo que tenemos que buscar como poder escalar privilegios para convertirnos en root

Las máquinas de HTB nos suelen dar algunas pistas, por cómo se llama, “CAP”, puede que vayan los tiros por la parte de las “capabilities”, probamos "getcap", con el siguiente comando podremos ver las capabilities que hay en el sistema asociadas a determinados ficheros. Observamos como “python” tiene una bastante interesante que es la “cap_setuid”

![imagen](https://user-images.githubusercontent.com/86053823/122621270-4e761c80-d095-11eb-8496-6e4e225f7c49.png)

```diff
Antes de continuar vamos a explicar que es eso de las capabilities y como usarlas para poder escalar privilegios. 
Las capabilities nos permiten gestionar que permisos tiene un proceso para acceder a las partes del kernel, por ejemplo con “setcap” podemos hacer que un usuario no administrativo use un determinado proceso sin necesidad de tener privilegios especiales, es decir, los procesos que tengan asignada capabilitie solo permitirá al usuario ejecutar ciertas acciones privilegiadas, pero no obtener el rol de superusuario. 
Hay muchas pero en este caso la que nos importa es la que tiene python que es la "cap_setuid". 
Ésta capabilities nos va a permitir asignarle el UID de root.
```

Vamos a poner “bash” como “setuid root” para intentar hacer la escalada de privilegios. Para ello ponemos el 0 que es el que pertenece a root en “os.setuid” y en “os.system” con “chmod +s” activaremos el modo suid (setuid).

![imagen](https://user-images.githubusercontent.com/86053823/122621289-6188ec80-d095-11eb-993d-b9517ad6faab.png)

Una web bastante interesante que nos facilitara bastante el trabajo es https://gtfobins.github.io/ , aquí podemos buscar un comando y nos dirá, en este caso, como explotar el tema de la capabilities.

Comprobamos que se ha cambiado

![imagen](https://user-images.githubusercontent.com/86053823/122621318-72d1f900-d095-11eb-9f3c-183ab1830f86.png)

Ahora con el “-p” preservaremos los permisos indicando la escalada de privilegios

![imagen](https://user-images.githubusercontent.com/86053823/122621336-7feee800-d095-11eb-93ea-de554f282702.png)

Ya solo nos queda buscar el fichero que contiene la flag del usuario root

![imagen](https://user-images.githubusercontent.com/86053823/122621371-9dbc4d00-d095-11eb-94e9-32fdf71d240a.png)


Una maquina que al final ha sido bastante sencilla pero que si nunca se ha tocado el tema de las “capabilities” es bastante buena para aprender un poco sobre el tema.


