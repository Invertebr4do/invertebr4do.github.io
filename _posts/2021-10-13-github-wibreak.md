---
layout: single
title: WiBreak - Tool
excerpt: "Herramienta hecha en bash para automatizar distintos ataques a redes wifi, tanto para optención de hanshake, para posteriormente sea crackeado, clientless PKMID attack, para la obtención de hashes de redes sin clientes conectados, e incluso para causar un poco de caos en la red con el parametro DESTROY, incluso logrando causar un DOS (USAR CON PRECAUCIÓN)"
date: 2021-10-13
classes: wide
header:
  teaser: /assets/images/github-tool-wibreak/WiBreak.png
  teaser_home_page: true
  icon: /assets/images/github.png
categories:
  - github
  - bash
  - tool
tags:
  - bash
  - scripting
  - aircrack-ng
  - PKMID
  - handshake
---

<p align="center">
<img src="/assets/images/github-tool-wibreak/img_header.png">
</p>

<h1 align="center">WiBreak</h1>
<hr>

[WiBreak](https://github.com/Invertebr4do/WiBreak) es una herramienta hecha en bash para automatizar distintos ataques a redes wifi, tanto para optención de hanshake, para posteriormente sea crackeado, clientless PKMID attack, para la obtención de hashes de redes sin clientes conectados, e incluso para causar un poco de caos en la red con el parametro DESTROY, incluso logrando causar un DOS (USAR CON PRECAUCIÓN)

Esta tool está pensada para que sea fácil de usar y efectiva, desde su parametro ```-l```, para listar las interfaces de red disponibles

<p align="center">
<img style="width: 50%; height: 100%; margin-right: 10px; margin-bottom: 10px;" src="/videos/wibreak/l_parameter.gif"/>
</p>

Como todos sus otros parametros (```-a```, ```-i```, ```-m```, ```-t```, ```-h```), los cuales iremos viendo, uno por uno

 - ```-h``` **~>** Parametro para desplegar el panel de ayuda

<p align="center">
<img style="width: 70%; height: 100%; margin-right: 10px; margin-bottom: 10px;" src="/videos/wibreak/h_parameter.gif"/>
</p>

- ```-a``` **~>** Este parametro nos permite seleccionar entre diferentes modos de ataque
	> **```handshake```** **~>** Esta opción afectua un ataque de **deautenticación**, para capturar un **handshake** y posteriormente lo **crackea**.
	>
	> **```PKMID```** **~>** Con esta opción podemos efectuar un ataque tipo **clientless PKMID**, opara obtener hashes de redes sin clientes conectados y posteriormente los rompe aplicando fuerza bruta.
	>
	> **```ALL```** **~>** Esta opción automatiza la ejecución de los **dos anteriores ataques** (handshake y PKMID), uno detras de otro.
	>
	> **```DESTROY```** **~>** Esto nos permitirá realizar un ataque a una red durante el tiempo que nosotros querramos, pudiendo incluso dejarla **inoperativa**, hasta que se reinicie el router.

- ```-i``` **~>** Para poder realizar cualquiera de los ataques disponibles se necesita agregar este parametro, indicandole el nombre de nuestra tarjeta de red en modo monitor.

- ```-m``` **~>** Con este parametro podemos indicar la dicrección **MAC** con la que querramos que nuestra tarjeta de red trabaje (*Este parametro es pocional*)

- ```-t``` **~>** Usando este parametro podremos inicar el tiempo en segundos, que querramos que dure el ataque (**Solo funciona con el ataque DESTROY**)

<hr>
 - <b>[INSTALA Y USA WIBREAK](https://github.com/Invertebr4do/WiBreak)</b>
<hr>
