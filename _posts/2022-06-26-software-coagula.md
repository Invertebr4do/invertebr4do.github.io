---
layout: single
title: Coagula - Esteganografía
excerpt: "En este post veremos cómo usando Coagula (un sintetizador de imágenes), para generar sonido a partir de una imagen, logrando ocultar mensajes en archivos de audio."
date: 2022-06-26
classes: wide
header:
  teaser: /assets/images/software-coagula/Coagula.png
  teaser_home_page: true
  icon: /assets/images/steganography.png
categories:
  - esteganografía
  - software
  - tutorial
tags:
  - windows
  - coagula
---

<p align="center">
<img src="/assets/images/software-coagula/img_header.png">
</p>

Coagula es un **software** creado para **windows**, el cual nos sirve tanto como **editor** de imagenes, como un **generador** de **sonido** apartir de una **imagen**.
A pesar de hacerle falta várias herramientas para la edición de imágenes que generalmente estarian incluidas en un editor de imágenes, tiene un **montón** de **herramientas específicas** que nos pueden ser de utilidad.

> Puedes descargar e installar Coagula [aquí](https://ccm.net/downloads/sound/6141-coagula/).

Ahora empecemos.

## IMPORTAR IMAGEN

> Web que usé para generar la imagen con texto: [imageonline](https://text.imageonline.co/es/).

1. Damos click en **File** y **Open image**.

<p align="center">
<img src="/assets/images/software-coagula/img1.png">
</p>

2. Aquí seleccionamos la imagen que queramos convertir a audio (*no tiene que ser un texto, pero en mi opinión, es lo más sencillo*).

<p align="center">
<img src="/assets/images/software-coagula/img2.png">
</p>

## RENDERIZACIÓN DE LA IMAGEN
1. Ya que tenemos la imagen abierta, vamos a **Tools** y **Render Options**.

<p align="center">
<img src="/assets/images/software-coagula/img3.png">
</p>

2. Ahora damos en **Render** y deberíamos escuchar un sonido un poco molesto.

<p align="center">
<img src="/assets/images/software-coagula/img4.png">
</p>

3. Luego de esto, vamos a **Sound** y **Render Without Blue** (ahora deberíamos escuchar otro sonido similar al anterior, pero un poco más agudo).

<p align="center">
<img src="/assets/images/software-coagula/img5.png">
</p>

## EXPORTAR ARCHIVO WAV
Ahora solo nos falta exportar el audio que generamos en el paso anterior, a un archivo wav.

1. Clickeamos en **File** y **Save Sound As**.

<p align="center">
<img src="/assets/images/software-coagula/img6.png">
</p>

2. Ahora solo elegimos la ruta donde lo queremos guardar y el nombre con el que queremos exportar el archivo.

<p align="center">
<img src="/assets/images/software-coagula/img7.png">
</p>

## VISUALIZAR LA IMAGEN EN EL ESPECTOGRAMA
Si te interesa ver algunas formas de poder ver la imagen que anteriormente pasamos a audio, además de saber de que manera podemos insertarla en otro archivo de audio, como una canción, puedes leer [este otro post](https://invertebr4do.github.io/software-audacity).
