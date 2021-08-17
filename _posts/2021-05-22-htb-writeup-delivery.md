---
layout: single
title: Creando una antena amplificadora de wifi
excerpt: "Primera página, ni tu te la crees, waaaaaaaaaaaaahwawawah"
date: 2021-08-17
classes: wide
header:
  teaser: /assets/images/htb-writeup-delivery/delivery_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - osticket
  - mysql
  - mattermost
  - hashcat
  - rules
---

hola este es mi primer sitio, claro que si compañeros

```bash
#!/bin/bash
for port in $(seq 1 65535); do
	timeout 1 bash -c "echo "" > /dev/tcp/127.0.0.1/443" 2>/dev/null && echo -n "[+] PORT $port - OPEN"
done
```
