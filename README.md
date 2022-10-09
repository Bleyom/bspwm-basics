## Bspwm Basics
- Hecho por [dharmx](https://github.com/dharmx)
- [Post Original](https://dharmx.is-a.dev/bspwm-basics/)

![Imagen de Presentacion](https://dharmx.is-a.dev/bspwm-basics/images/featured-image.png)

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

### # Introduccion
#####  BSPWM es un gestor de ventanas con algunas caracteristicas interesantes. En este articulo analizare un poco su codigo fuente, los comandos IPC y algunas otras posibles extensiones. 

### # Antes de comenzar
##### Este es una articulo bastante intermedio que puede parecel facil de entender y/o seguir al principio, pero se volvera relativamente mas dificl de seguir a medida que el mismo avance. Necesitaras tener las siguientas cosas para progresar sin problemas

- Tal vez una pequeña cantidad de estructura de datos BST.
- Experienca con gestores de ventanas y X11/XORG
- Experiencia en programacion si es posible
##### Todo correcto! Comenzemos.

### # Acerca de bspwm
##### BSPWM es un gestor de ventanas de tipo tilling (mosaico), es decir que representa las ventanas como las hojas de un arbol binario **completo**
##### A diferencia de la mayoria de los otros administradores de ventas, no tiene un archivo de configuracion decica. Se usa el cliente IPC (bspc) para todo. Ya sea configurar los espacios entre ventanas, color de los borders, los diseños (layouts), monitores, etc. Para BSPWM, un archivo de configuracion el solo un shell script. El cual debe ser almacenado en `$HOME/.config/bspwm/bspwmrc`.
##### Es un shell script que debe tener comandos que inicien un demon (gestor) del teclado, un demonio (gestor) de notificaciones, configuracion de los montiores, boredes, espacios entre ventanas, y otras como llamadas a `bspc`.
##### Tambien puede tener comandos que iniciaran algunas aplicaciones como discord, barras de estado, etc.
##### Esto es preferido porque cuando BSPWM es iniciado por primera vez, no tiene ningun estado. Deberas ejecutar algunas llamadas a `bspc` los cuales estableceran un estado, el cual luego podra generar ventanas y otras cosas.
