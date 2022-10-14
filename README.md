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

- [Introduccion](#-introduccion)
- [Antes de comenzar](#-antes-de-comenzar)
- [Acerca de bspwm](#-acerca-de-bspwm)
- [Estructura de visualizacion](#--estructura-de-visualizacion)
- [Monitores](#-monitores)
- [Espacios de Trabajo](#--espacios-de-trabajo)
- [Relacion monitor-area de trabajo](#--relacion-monitor-area-de-trabajo)
- [La estructura de datos BST](#-estructura-de-datos-bst)
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

##### Un nodo BST tendria el siguiente aspecto en el lenguaje C.
```c
struct node {
    int some_data;
    node_t *left_child;
    node_t *right-child;
};
```

##### Este nodo contiene un puntero al hijo izquiero y otro al hijo derech. Los hijos izquierdo y derecho son hermanos, como se ilustra en el diagram. Y `some_data` cuales son los datos que transportara el nodo. Ahora, considere el siguiente codigo para obtener una vista mas clara de como se incializa el nodo y como funcionan algunos componentes basicos.

<details>
  <summary>Implementacion minimalista de BST en el lenguaje C</summary>
  
  ```c
  typedef struct node_t node_t;
struct node_t {
    int data;
    node_t* left;
    node_t* right;
};
  ```
</details>
  </summary>
</details>

###### Nota ✏️
> No intente entender la funcion de cada variable. Simplemente tome nota de las similitudes entre las dos definiciones de nodo.

```c
typedef struct node_t node_t;
struct node_t {
    uint32_t id;
    split_type_t split_type;
    double split_ratio;
    presel_t *presel;
    xcb_rectangle_t rectangle;
    constraints_t constraints;
    bool vacant;
    bool hidden;
    bool sticky;
    bool private;
    bool locked;
    bool marked;
    node_t *first_child;
    node_t *second_child;
    node_t *parent;
    client_t *client;
};
```

##### Esta solo es una suposicion, pero la razon por la que el autor llama `left_child` y `righ_child`, `first_child` y `second_child`, es por la forma como se generan las ventanas (nodos). Es decir se genera la **primera** ventana que es por nombre, el primogenito del padre actual y el **segundo** hijo es el segundo hijo del padre. Se hace para guiar la intuicion al leer el codigo.


### ![#1589F0](https://via.placeholder.com/15/1589F0/1589F0.png) Modo automatico

##### Esto es un poco dificil de explicar. Por lo tanto, debemos andar cuidadosamente para evitar las confusiones.

##### Anteriormente vimos que BSPWM representa monitores y escritorios como una lista doblemente enlazada. Las ventanas se representan como hojas de un arbol binario completo. Abrir una nueva ventana en BSPWM se denomina "insercion", donde la ventana es un nodo que se inserta en un punto del arbol binario como hijo de un padre.

##### Ahora, la insercion tiene dos modos en BSPWM, el automatico y el manual. El modo automatico consta de varios tipos o, mas bien, las formas, los patrones de como se debe "enmarcar" un nodo y/o ventana.

- Esquema alternativo
- Esquema del lado mas largo
- Esquema de tipo espiral

##### Puede cambiar el esquema automatico emitiendo una llama a `bspc` tal como la que se muestra a continuacion

```bash
$ bspc config automatic_scheme # impreme el valor actual
> alternate
$ bspc config automatic_scheme longest_side
$ bspc config automatic_scheme alternate
$ bspc config automatic_scheme spiral
```

##### Ahora, en mi opinion comprender el esquema alternativo (predeterminado) seria la forma mas rapida para que un principiante confundido comprenda el funcionamente general de BSPWM. 
##### Entonces, intentare explicar eso primero.

### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Esquema alternativo

##### Explicare esto con un ejemplo en el que se abriran nuevas ventanas paso a paso y en esos pasos se explicara como BSPWM organiza esas ventanas, Estare asignando nuevos alias a estos pasos por cero, inicial, primero, segundo, ...  y ultimo.

### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Estado nulo

![null-state](https://dharmx.is-a.dev/bspwm-basics/images/alternate-0.png)

> El estado nulo del esquema alternativo.

##### Bueno, en realidad ese solo es mi fondo de pantalla 😢, De todos modos, este es un estado en el que las ventanas aun no se han generado


### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Estado Inicial

![initial-state](https://dharmx.is-a.dev/bspwm-basics/svgs/alternate-1.svg)

> El estado inicial del esquema alternativo (diagrama).

![initial](https://dharmx.is-a.dev/bspwm-basics/images/alternate-1.png)

> El estado de escritorio inicial del esquema alternativo.

##### Aqui, acabamos de generar una ventana de terminal. Ahora, el escritorio esta en un estado inicial. Donde una nueva ventana/nodo ha sido insertado.

<details>
  <summary>📖 ❝❞ Citando el README de BSPWM</summary>
  Por defecto el punto de inserción es la ventana enfocada y su modo de inserción es automático.
</details>

### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Segundo estado

![segundo-estado](https://dharmx.is-a.dev/bspwm-basics/svgs/alternate-2.svg)

> El segundo diagrama de estado del esquema alternativo (diagrama).

![2-state](https://dharmx.is-a.dev/bspwm-basics/images/alternate-2.png)

> El segundo estado de escritorio del esquema alternativo

```c
# assert new_node != first_child
if (is_first_child(new_node))
  node->second_child = new_node;
else
  node->first_child = new_node;
```

##### El fragmento anterior comprueba si `new_node` que se va a insertar es el primer hijo del nodo padre o no, y si lo es, el atributo del segundo nodo del nodo padre apuntará a `new_node`.

##### Ahora, `first_child` tiene un hermano, es decir `second_child`. Ellos son hermanos. Al usar `bspc` podras verificar si realmente son hermanos o no. Intente ejecutar el siguiente comando. Puede enfocarse en la ventana izquiera o derecha.

##### Para verificarlo primero necesita obtener los IDs de esas dos ventanas. Digamos que la ventana derecha es `0x3000006` y la ventana izquierda `0x2600006`, en mi caso. Puedes ver la imagen de arriba con dos ventanas abiertas como referencia.

##### De todas formas, me centrare en la ventana con ID `0x3000006` (derecha) y ejecutare el siguiente comando. Ahora comprobaremos si al escribir eso se obtiene el ID de la ventana opuesta (izquierda).

```bash
$ bspc query --nodes --node @brother
> 0x3000006
```

##### Como puedes ver, muestra el ID de la ventana de la izquierda. Ademas, cierra la ventana de la izquierda. Ademas cierre la ventana de la izquierda y escriba el mismo comando en la terminal. Vera que `STDERR` se esta dando como respuesta en su lugar, ya que ahora `first_child` se le ha asignado el puntero `NULL`. Ejecute el siguiente comando para verificar.

```bash 
$ bspc query --nodes --node @brother && echo YES || echo NO
> NO
```

### ![ac45cc](https://via.placeholder.com/15/ac45cc/ac45cc.png) | Tercer Estado
![tercer-estado](https://dharmx.is-a.dev/bspwm-basics/svgs/alternate-3.svg)
 
>  El tercer estado de escritorio del esquema alternativo

![](https://dharmx.is-a.dev/bspwm-basics/images/alternate-3.png)

##### Este es probablemente un buen momento para mencionar lo que significan los nodos `A` y `B` Actualmente, el nodo `A` es el padre del nodo `B` y `1` y el nodo `B` es el padre del nodo `2` y `3`.

##### Se llaman nodos padre o, como lo define BSPWM, nodos internos, solo aparecerán cuando a `second_child` esté a punto de nacer, así que, por ejemplo, volviendo al segundo estado , debemos entender que el padre Ano aparecerá hasta que esté tiempo para 2 que nazca un segundo hijo . Entonces, en este caso, antes de que nazca el segundo hijo, el primer hijo se desplaza a la izquierda de un nuevo nodo y el segundo hijo se adjunta a la derecha de ese nuevo nodo. El nodo ahora se convierte en el padre de esos dos nodos. Además, tenga en cuenta que los nodos internos no son visibles. Cuando un nodo se divide en dos, se crea un nodo interno y permanece debajo de `first_child` y `second_child`.
 
