---
layout: single
title: Audacity - Esteganografía
excerpt: "Veamos cómo podemos visualizar el mensaje oculto en un audio (como el que creamos con Coagula), además de cómo podemos unirlo con otro archivo de audio, como lo podría ser una canción."
date: 2022-06-27
classes: wide
header:
  teaser: /assets/images/software-audacity/Audacity.png
  teaser_home_page: true
  icon: /assets/images/steganography.png
categories:
  - esteganografía
  - software
  - tutorial
tags:
  - linux
  - windows
  - audacity
---

<p align="center">
<img src="/assets/images/software-audacity/img_header.jpg">
</p>

## VER EL ESPECTROGRAMA
Para poder ver el mensaje oculto en el archivo .wav que creamos en el post de [Coagula](Coagula/What%20is%20Coagula?), podemos usar diferentes herramientas como **Audacity**, de la siguiente manera.

> Si no sabes como crear el *.wav* con la imagen oculta, puedes ver [este post](https://invertebr4do.github.io/software-coagula) donde lo explico paso a paso.

- Abrimos **Audacity**, clickeamos en **File** ~» **Import** ~» **Audio**.

<p align="center">
<img src="/assets/images/software-audacity/img1.png">
</p>

- **Seleccionamos** y **abrimos** el archivo **.wav**.

<p align="center">
<img src="/assets/images/software-audacity/img2.png">
</p>

- Ahora tendríamos que tener algo así:

<p align="center">
<img src="/assets/images/software-audacity/img3.png">
</p>

- Lo único que nos queda por hacer es: dirigirnos al lado izquierdo de la pista, seleccionar el nombre de nuestro archivo (*en mi caso s3cret*) y marcar la opción **Spectrogram**.

<p align="center">
<img src="/assets/images/software-audacity/img4.png">
</p>

- Y si ajustamos el zoom con *CTRL* + *SCROLL*, podremos ver la imagen oculta.

<p align="center">
<img src="/assets/images/software-audacity/img5.png">
</p>

## AÑADIR LA IMAGEN OCULTA A UN ARCHIVO DE MÚSICA
Para hacer esto, solo tenemos que repetir los primeros pasos para importar un archivo y añadir algun detalle siguiendo estos pasos.

- Seleccionamos la canción que queremos usar, para ocultar la imagen.

<p align="center">
<img src="/assets/images/software-audacity/img6.png">
</p>

<p align="center">
<img src="/assets/images/software-audacity/img7.png">
</p>

- Deberíamos nuevamente poder ver las ondas de audio de nuestra canción.

<p align="center">
<img src="/assets/images/software-audacity/img8.png">
</p>

> Obviamente el sonido de la imagen oculta va a sonar cuando reproduzcamos la canción, por lo tanto vamos a quitar la imagen del inicio de la canción, para que sea menos sospechoso.

- Para finalizar usamos la herramienta **Time Shift Tool**, para mover la imagen con el mouse, a donde nos interese.

<p align="center">
<img src="/assets/images/software-audacity/img9.png">
</p>

<p align="center">
<img src="/assets/images/software-audacity/img10.png">
</p>

- Ahora solo quedaría exportarlo en **File** ~» **Export** y elegir el tipo de audio que deseemos (*en mi caso mp3*).

<p align="center">
<img src="/assets/images/software-audacity/img11.png">
</p>

## OTRA FORMA DE VER EL ESPECTROGRAMA
Si por algún motivo, no nos funcionó el anterior método, o no queremos/podemos usar audacity, hay algunas herramientas online que nos pueden servir para esto. Una por ejemplo es [academo](https://academo.org/demos/spectrum-analyzer).

<p align="center">
<img src="/assets/images/software-audacity/img12.png">
</p>

Cada uno que elija la opción que le convenga.
