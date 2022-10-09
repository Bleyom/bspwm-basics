## Bspwm Basics
- Hecho por [dharmx](https://github.com/dharmx)
- [Post Original](https://dharmx.is-a.dev/bspwm-basics/)

![Imagen de Presentacion](https://dharmx.is-a.dev/bspwm-basics/images/featured-image.png)

<h1>
  <a href="#--------">
    <img alt="" align="left" src="https://img.shields.io/github/stars/Bleyom/bspwm-basics?color=0b0d10&label=Stars%20%E2%AD%90&style=for-the-badge"/>
  </a>
  <a href="#--------">
    <img alt="" align="right" src="https://img.shields.io/github/forks/Bleyom/bspwm-basics?color=0b0d10&label=Forks%20%F0%9F%94%B1&style=for-the-badge"/>
  </a>
</h1>

#

### Contenido 📖

- [Antes de empezar]()
- [Acerca de bspwm]()
- [Monitores]()
- [Espacios de Trabajo]()
- [La estructura de datos BST]()
- [Modo Automatico]()
- [Modo Manual]()
- [Estados, Banderas y diseño]()
- [BSPC]()
- [Referencias]()
- [Herramientas]()
- [Nota Final]()

### ![#c5f015](https://via.placeholder.com/15/c5f015/c5f015.png) Introduccion
#####  BSPWM es un gestor de ventanas con algunas caracteristicas interesantes. En este articulo analizare un poco su codigo fuente, los comandos IPC y algunas otras posibles extensiones. 

### ![#1589F0](https://via.placeholder.com/15/1589F0/1589F0.png) Antes de comenzar
##### Este es una articulo bastante intermedio que puede parecel facil de entender y/o seguir al principio, pero se volvera relativamente mas dificl de seguir a medida que el mismo avance. Necesitaras tener las siguientas cosas para progresar sin problemas

- Tal vez una pequeña cantidad de estructura de datos BST.
- Experienca con gestores de ventanas y X11/XORG
- Experiencia en programacion si es posible
##### Todo correcto! Comenzemos.

### ![#1589F0](https://via.placeholder.com/15/1589F0/1589F0.png) Acerca de bspwm
##### BSPWM es un gestor de ventanas de tipo tilling (mosaico), es decir que representa las ventanas como las hojas de un arbol binario **completo**
##### A diferencia de la mayoria de los otros administradores de ventas, no tiene un archivo de configuracion decica. Se usa el cliente IPC (bspc) para todo. Ya sea configurar los espacios entre ventanas, color de los borders, los diseños (layouts), monitores, etc. Para BSPWM, un archivo de configuracion el solo un shell script. El cual debe ser almacenado en `$HOME/.config/bspwm/bspwmrc`.
##### Es un shell script que debe tener comandos que inicien un demon (gestor) del teclado, un demonio (gestor) de notificaciones, configuracion de los montiores, boredes, espacios entre ventanas, y otras como llamadas a `bspc`.
##### Tambien puede tener comandos que iniciaran algunas aplicaciones como discord, barras de estado, etc.
##### Esto es preferido porque cuando BSPWM es iniciado por primera vez, no tiene ningun estado. Deberas ejecutar algunas llamadas a `bspc` los cuales estableceran un estado, el cual luego podra generar ventanas y otras cosas.

### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Estructura de visualizacion

##### Cabe aclara que BSPWM es un administrador de ventanas, lo cual significa que administra ventanas ~~(yeah)~~. Y lo hace representando el arbol de ventanas como hojas de un BST completo.
##### Para que las aplicaciones puedan engendrarse, BSPWM primero necesita definir una ventana raiz, que en la [jerga](https://es.wikipedia.org/wiki/Jerga) de los WM se denomina espacios de trabjo. Y, despues de definir estos espacios de trabajo, ya podra abrir una ventana, la cual sera administrada por BSPWM.

##### Ademas de esta informacion, debe tenerse en cuenta que BSPWM se diesño originalmente para computadoras con un solo monitor.

##### Si aun estas confundido, la siguiente imagen lo ayudara a entenderlo mejor

![Monitores](https://dharmx.is-a.dev/bspwm-basics/svgs/bspwm-mon-ws.svg)

