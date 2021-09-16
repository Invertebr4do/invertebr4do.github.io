---
layout: single
title: Tratamiento de la TTY
excerpt: "Cuando logramos obtener RCE en algún sistema uno  de nuestros principales objetivos es mantener la conección, ya que sería un problema que, por ejemplo, al ejecutar ctrl_c para cancelar un proceso cerremos la conección sin querer, esto, a demás de obtener otros beneficios, lo solucionamos haciendo un tratamiento de la TTY"
date: 2021-08-27
classes: wide
header:
  teaser: /assets/images/Shell.png
  teaser_home_page: true
  icon: /assets/images/tratamiento-de-tty/Bash.png
categories:
  - tutorial
  - bash
tags:
  - linux
  - tty
  - shell
  - bash
---

<p align="center">
<img src="/assets/images/tratamiento-de-tty/img_header.png">
</p>

Una vez obtenemos una reverse shell necesitamos estar comodos con una **TTY** interactiva para evitar problemas, como **cerrar la conección** sin querer o simplemente para poder **desplazarnos** con las flechas, poder usar el **tab** para el auto completado de rutas, etc.
Esto es muy sencillo de lograr, simplemente debemos de hacer un tratamiento de la tty siguiendo estos pasos:

Empezamos en la reverse shell obtenida

```bash
$ script /dev/null -c bash
Script started, file is /dev/null
```
Luego de esto presionamos **ctrl_z** para suspender la shell

```bash
www-data@vulnnet:/$ ^Z
zsh: suspended  nc -nlvp 443
```

Ahora resetearemos la configuración de la shell que dejamos en segundo plano indicando **reset** y **xterm**

```bash
~$> stty raw -echo; fg
```
```bash
[1]  + continued  nc -nlvp 443
                              reset
reset: unknown terminal type unknown
Terminal type? xterm
```
Exportamos las variables de entorno **TERM** y **SHELL**

  - ```export TERM=xterm``` -> Debemos hacer esto ya que a pesar de haberle indicado que queríamos una **xterm** al momento de reiniciarlo la variable de entorno **TERM** vale **dump**
  - ```export SHELL=bash``` -> Para que nuestra shell sea una bash

```bash
www-data@vulnnet:/$ export TERM=xterm
www-data@vulnnet:/$ export SHELL=bash
```
Ya con esto hecho tendríamos una **TTY** full interactiva, pero falta una cosa para que sea lo más comoda posible, setear las filas y columnas, esto para poder ocupar toda nuestra pantalla ya que en este momento solo podemos usar una porción de esta
Vemos el tamaño de nuestra shell en una consola normal

```bash
~$> ssty size
40 123
```
Y las seteamos en la reverse shell **(en mi caso 40 filas y 123 columnas)**

```bash
www-data@vulnnet:/$ stty rows 40 columns 123
```
Y listo!, ya podemos disfrutar de una shell totalmente interactiva y muy comoda para continuar rompiendo
