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

> Cómo BSPWM define la relación entre monitores y espacios de trabajo.

### ![#1589F0](https://via.placeholder.com/15/1589F0/1589F0.png) Monitores
##### Si analizamos el codigo, veremos que los monitores no se representan como un arbol de multiples ramas, sino como una sola rama, es decir, una lista enlazada, donde el monitor actual tiene enlaces al monitor anterior y al siguiente.

> Parte del codigo fuente

 ```diff
288    typedef struct monitor_t monitor_t;
289    struct monitor_t {
290    char name[SMALEN];
291    uint32_t id;
292    xcb_randr_output_t randr_id;
293    xcb_window_t root;
294    bool wired;
295    padding_t padding;
296    unsigned int sticky_count;
297    int window_gap;
298    unsigned int border_width;
299    xcb_rectangle_t rectangle;
300    desktop_t *desk;
301    desktop_t *desk_head;
302    desktop_t *desk_tail;
! 303  monitor_t *prev;
! 304  monitor_t *next; 
305    };
```
```c
// code representation
const int MON_LEN = 2;
monitor_t monitors[MON_LEN];
```

##### ¿Ves esas variable `prev` y `next`? Intuitivamente, la variable `prev` apunt a la estructura previa de `monitor_t` y `next` siguiente. Y, si cualquiera de esas variables (punteros) tienen `NULL` como valor, se debera asumir que el monitor esta enfocado esta al final o, al principio de la lista de monitores

### ![#1589F0](https://via.placeholder.com/15/1589F0/1589F0.png)  Espacios de trabajo

##### Para las computadoras, la estructura es mas o menos la misma que la de los monitores

> Los espacios de trabajo se denominan escritorios en BSPWM. Se usan indistintamente. Vera que ambos aparecen en este articulo.

##### Por lo tanto, consule el siguiente fragmento de codigo.

```diff
273    typedef struct desktop_t desktop_t;
274    struct desktop_t {
275    char name[SMALEN];
276    uint32_t id;
277    layout_t layout;
278    layout_t user_layout;
279    node_t *root;
280    node_t *focus;
! 281    desktop_t *prev;
! 282    desktop_t *next;
283    padding_t padding;
284    int window_gap;
285    unsigned int border_width;
286    };

```

### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Relacion monitor-area de trabajo

##### La siguiente imagen intenta definir brevemente la relacion monitor-area de trabajo en BSPWM que tiene en cuenta lo anterios de `monitor_t` y `desktop_t`

<img src="https://dharmx.is-a.dev/bspwm-basics/svgs/linked-list-bspwm.svg">

> Representacion del espacio de trabajo del monitor en forma de listas enlazadas.

##### El primer monitor (marcado de color verde) tiene tres escritorios donde la etiqueta del primer escritorio se identifica por `(+)`, segundo por `(-)` y por ultimo por `(=)`.

##### Vale la pena señalar que es posible tener etiquetas duplicadas porque el administrador de ventanas no identifica los espacios de trabajo a partir de las etiquetas, sino que usan el ID de ese escritorio/area de trabajo. Puede verificaro cuales son los ID actuales con una simple consula a traves del client IPC `bspc`

```bash
$ bspc query --desktops # IDs
0x00400004
0x00400005
0x00400006

$ bspc query --names --desktops # Etiquetas
(+)
(-)
(=)

# Cambiar el numero de monitores en el monitor primario
$ bspc monitor primary --reset-desktops 1 2 3
```

### ![#1589F0](https://via.placeholder.com/15/1589F0/1589F0.png) Estructura de datos BST

##### La estructura de datos Binary Tree es una forma de representar datos en forma de arbol. UN BST es un caso especial de arbol que solo tiene dos hijos, las hojas se llaman nodos, que es el termino que usaremos de ahora en adelante

![Arbol binario ilustracion](https://dharmx.is-a.dev/bspwm-basics/svgs/bst.svg)

> Breve ilustracion descriptiva de un arbol binario

##### Tambien puedes notar que los nodos tienen un color direferente. Los colores significan niveles. Por ejemplo, el nivel raiz del nodo esta coloreado con verde ![verde](https://via.placeholder.com/15/79dcaa/79dcaa.png). El siguiente nivel esta coloreado con azul ![azul](https://via.placeholder.com/15/7ab0df/7ab0df.png) y por ultimo el 3er nodo el cual esta colerado con rojo ![rojo](https://via.placeholder.com/15/f87070/f87070.png)
