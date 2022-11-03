---
layout: single
title: EASY-PWM - Tool
excerpt: "En esta ocasión traigo mi primera herramienta propia, hecha totalmente en bash, creada para la automatización de mi configuración de bspwm, tomando como referencia el entorno de s4vitar, gracias a EASY-PWM podrás renovar tu entorno, mientras te haces algo de comer, duermes, sales, etc"
date: 2021-09-20
classes: wide
header:
  teaser: /assets/images/github-tool-easy-pwm/WorkEnvironment.png
  teaser_home_page: true
  icon: /assets/images/github.png
categories:
  - github
  - tool
  - bash
tags:
  - bspwm
  - sxhkd
  - bash
  - scripting
  - linux
  - configuration
---

<p align="center">
<img src="/assets/images/github-tool-easy-pwm/img_header.png">
</p>

## En este post presento la primera de mis herramientas | EASY-PWM

Este script hecho en bash está creado para la automatización de mi configuración de bspwm, es casi la misma de *s4vitar*, pero con algunos cambios.
Esta utilidad está pensada para que sea completamente automática, esto te permitirá salir a tomarte un café mientras se configura por completo tu entorno bspwm.

## Testeado en los siguientes OS:
- Kali linux 2022.3 Changelog.
- Parrot security 5.1 Electro Ara.
- Ubuntu 22.04.1 LTS.

## Despues de un rato de haber ejecutado el script, debería quedar así tu entorno:

![](https://github.com/Invertebr4do/EASY-PWM/blob/main/img/WorkEnvironment.png?raw=true)

## Implementaciones
- **kitty** *~»* [kovid goyal](https://sw.kovidgoyal.net/kitty/)
- **neovim con nvchad** *~»* [nvchad](https://nvchad.com/)
- **bspwm** *~»* [baskerville](https://github.com/baskerville)
- **sxhkd** *~»* [baskerville](https://github.com/baskerville/sxhkd.git)
- **blue-sky** *~»* [VaughnValle](https://github.com/VaughnValle/blue-sky.git)
- **polybar** *~»* [polybar](https://github.com/polybar/polybar)
- **rofi** *~»* [adi1090x](https://github.com/adi1090x/rofi)
- **picom** *~»* [ibhagwan](https://github.com/ibhagwan/picom.git)
- **powerlevel10k** *~»* [romkatv](https://github.com/romkatv/powerlevel10k)
- **feh** *~»* [derf](https://github.com/derf/feh)
- **fzf** *~»* [junegunn](https://github.com/junegunn/fzf)
- **sudo plugin** *~»* [ohmyzsh](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/sudo/sudo.plugin.zsh)
- **hack nerd font** *~»* [Nerd Fonts](https://www.nerdfonts.com/font-downloads)
- **i3lock / i3lock-color** *~»* [Raymo111](https://github.com/Raymo111/i3lock-color)
- **extractPorts** *~»* [s4vitar](https://www.youtube.com/s4vitar)
- **mkt** *~»* [s4vitar](https://www.youtube.com/s4vitar)
- **rmk** *~»* [s4vitar](https://www.youtube.com/s4vitar)
- **settarget / cleartarget** *~»* [s4vitar](https://www.youtube.com/s4vitar)
- **firejail** *~»* [netblue30](https://github.com/netblue30/firejail)
- **flameshot** *~»* [flameshot](https://github.com/flameshot-org/flameshot)
- **zsh**

## Shortcuts

| Acción | Shortcut |
| -------- | -------- |
| Cambiar de workspace específico | <kbd>Windows</kbd> + <kbd>[1\|2\|3\|4\|5\|6\|7\|8\|9\|0]</kbd> |
| Cambiar de workspace a el siguiente o el anterior | <kbd>Windows</kbd> + <kbd>]\|[</kbd> |
| Alternar entre los dos últimos workspaces | <kbd>Windows</kbd> + <kbd>Tab</kbd> |
| Seleccionar ventana abierta en el workspace actual | <kbd>Windows</kbd> + <kbd>⇦⇧⇩⇨</kbd> | 
| Mover la ventana seleccionada a otro workspace | <kbd>Windows</kbd> + <kbd>Shift</kbd> + <kbd>[1\|2\|3\|4\|5\|6\|7\|8\|9\|0]</kbd> |
| Abrir terminal (*Kitty*) | <kbd>Windows</kbd> + <kbd>Enter</kbd> |
| Abrir navegador (*Firefox*) | <kbd>Windows</kbd> + <kbd>Shift</kbd> + <kbd>F</kbd> |
| Abrir navegador (*Chromium*) | <kbd>Windows</kbd> + <kbd>Shift</kbd> + <kbd>G</kbd> |
| Abrir *BurpSuite* | <kbd>Windows</kbd> + <kbd>Shift</kbd> + <kbd>B</kbd> |
| Abrir menú de aplicaciones (*Rofi*) | <kbd>Windows</kbd> + <kbd>D</kbd> |
| Tomar screenshot (*Flameshot*) | <kbd>Windows</kbd> + <kbd>Shift</kbd> + <kbd>S</kbd> |
| Bloquear pantalla (*i3lock-color*) | <kbd>Windows</kbd> + <kbd>Shift</kbd> + <kbd>X</kbd> |
| Abrir menú para apagar, suspender, etc. El sistema (*Rofi*) | <kbd>Windows</kbd> + <kbd>P</kbd>|
| Cerrar ventana seleccionada | <kbd>Windows</kbd> + <kbd>W</kbd> |
| Abrir preselección | <kbd>Windows</kbd> + <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>Space</kbd> + <kbd>⇦⇧⇩⇨</kbd> |
| Resize de preselección | <kbd>Windows</kbd> + <kbd>Ctrl</kbd> + <kbd>[1\|2\|3\|4\|5\|6\|7\|8\|9\|0]</kbd> |
| Cerrar preselección | <kbd>Windows</kbd> + <kbd>Ctrl</kbd> + <kbd>Space</kbd> |
| Cambiar la ventana seleccionada a modo *Flotante* | <kbd>Windows</kbd> + <kbd>S</kbd> |
| Cambiar la ventana seleccionada a modo *Terminal | <kbd>Windows</kbd> + <kbd>T</kbd> |
| Cambiar layout de la ventana seleccionada entre *Tiled* y *Monocle* | <kbd>Windows</kbd> + <kbd>M</kbd> |
| Resize de ventanas flotantes | <kbd>Windows</kbd> + <kbd>Alt</kbd> + <kbd>⇦⇧⇩⇨</kbd> |
| Mover ventanas flotantes | <kbd>Windows</kbd> + <kbd>Ctrl</kbd> + <kbd>⇦⇧⇩⇨</kbd> |
| Recargar configuración (*bspwm*) | <kbd>Windows</kbd> + <kbd>Alt</kbd> + <kbd>R</kbd> |
| Cerrar sesión | <kbd>Windows</kbd> + <kbd>Alt</kbd> + <kbd>Q</kbd> |
| Recargar configuración (*sxhkd*) | <kbd>Windows</kbd> + <kbd>Esc</kbd> |
| Subir volumen del sistema | <kbd>Fn</kbd> + <kbd>F3</kbd> |
| Bajar volumen del sistema | <kbd>Fn</kbd> + <kbd>F2</kbd> |
| Subir brillo del monitor | <kbd>Fn</kbd> + <kbd>F6</kbd> |
| Bajar brillo del monitor | <kbd>Fn</kbd> + <kbd>F5</kbd> |


<hr>
<b>[CLONA Y USA EASY-PWM](https://github.com/Invertebr4do/EASY-PWM)</b>
<hr>

## Tutorial para la configuración manual:
- **s4vitar:** [https://www.youtube.com/watch?v=mHLwfI1nHHY&t=3013s](https://www.youtube.com/watch?v=mHLwfI1nHHY&t=3013s)

**Disfruta :)**
