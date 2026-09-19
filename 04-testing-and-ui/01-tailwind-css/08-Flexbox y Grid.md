# Flexbox y Grid

## Dos herramientas para dos problemas

Tailwind expone los dos sistemas de layout modernos de CSS, Flexbox y Grid, mediante utilidades. Elegir entre ellos no es una cuestión de gusto: cada uno resuelve un problema distinto.

* **Flexbox** organiza elementos en **una dimensión**: una fila o una columna.
* **Grid** organiza elementos en **dos dimensiones** a la vez: filas y columnas.

Una regla práctica: **Flexbox para alinear cosas, Grid para estructurar layouts.**

-----

## Flexbox

### Activarlo

```html
<div class="flex">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

Con solo `flex`, los hijos directos pasan a organizarse en una fila. A partir de ahí se controla su distribución con utilidades sobre el contenedor.

### Los dos ejes

Para entender `justify-` e `items-`, hay que pensar en los dos ejes de Flexbox:

* El **eje principal** es la dirección en la que se colocan los elementos: horizontal si la dirección es fila (`flex-row`, el valor por defecto), vertical si es columna (`flex-col`).
* El **eje transversal** es el perpendicular al principal.

Entonces:

* `justify-*` alinea sobre el **eje principal**.
* `items-*` alinea sobre el **eje transversal**.

En una fila (el caso por defecto), `justify-` es horizontal e `items-` es vertical, y por eso se suele memorizar así. Pero **si cambiás a `flex-col`, los ejes se invierten**: `justify-` pasa a ser vertical e `items-` horizontal. La regla "justify es horizontal" es una simplificación que solo vale para filas.

### Los patrones más comunes

**Centrar algo en toda la pantalla:**

```html
<div class="flex justify-center items-center h-screen">
  <div class="w-32 h-32 bg-blue-400"></div>
</div>
```

`flex` activa Flexbox, `justify-center` centra sobre el eje principal, `items-center` centra sobre el transversal, y `h-screen` hace que el contenedor ocupe todo el alto de la ventana (sin una altura, no habría espacio vertical en el que centrar).

**Una barra de navegación (logo a la izquierda, menú a la derecha):**

```html
<nav class="flex justify-between items-center">
  <div>Logo</div>
  <ul class="flex gap-4">
    <li>Inicio</li>
    <li>Contacto</li>
  </ul>
</nav>
```

`justify-between` empuja el primer hijo al inicio y el último al final, repartiendo el espacio sobrante en el medio. `gap-4` agrega separación **entre** los elementos, sin necesidad de márgenes individuales.

**Una columna centrada con separación:**

```html
<div class="flex flex-col items-center gap-4">
  <p>Uno</p>
  <p>Dos</p>
</div>
```

### Valores de `justify-*` e `items-*`

| Clase | Efecto |
| --- | --- |
| `justify-start` / `items-start` | al inicio del eje |
| `justify-center` / `items-center` | centrado |
| `justify-end` / `items-end` | al final del eje |
| `justify-between` | primero al inicio, último al final, espacio en el medio |
| `justify-around` / `justify-evenly` | espacio repartido alrededor de los elementos |

### Cómo se reparte el espacio entre hijos

Los **hijos** también tienen utilidades que controlan cuánto espacio ocupan:

```html
<div class="flex">
  <div class="flex-1">Ocupa el espacio disponible</div>
  <div class="flex-1">Ocupa el mismo espacio</div>
  <div class="flex-none">Tamaño fijo, no crece</div>
</div>
```

* `flex-1`: el elemento **crece** para ocupar el espacio libre (si hay varios, se lo reparten en partes iguales).
* `flex-none`: el elemento **mantiene su tamaño**, no crece ni se achica.

Para que los elementos pasen a la línea siguiente cuando no caben, se agrega `flex-wrap` al contenedor.

-----

## Grid

### Activarlo y definir columnas

```html
<div class="grid grid-cols-3 gap-4">
  <div>1</div>
  <div>2</div>
  <div>3</div>
  <div>4</div>
  <div>5</div>
  <div>6</div>
</div>
```

* `grid` activa el sistema de cuadrícula.
* `grid-cols-3` define **tres columnas de igual ancho**.
* `gap-4` agrega separación entre las celdas.

Los hijos se acomodan solos, de izquierda a derecha y de arriba a abajo: seis elementos en tres columnas forman dos filas, sin que tengas que indicar la posición de ninguno.

### Un caso real: una galería

```html
<div class="grid grid-cols-4 gap-4">
  <img src="1.jpg" alt="" />
  <img src="2.jpg" alt="" />
  <img src="3.jpg" alt="" />
  <img src="4.jpg" alt="" />
</div>
```

### Que un elemento ocupe más celdas

```html
<div class="grid grid-cols-4 gap-4">
  <div class="col-span-2">Ocupa dos columnas</div>
  <div>Normal</div>
  <div>Normal</div>
</div>
```

`col-span-2` hace que el elemento se extienda sobre dos columnas. Existen equivalentes para las filas (`grid-rows-*` y `row-span-*`).

### Grid y responsive

Grid se combina naturalmente con los breakpoints, y este es probablemente su uso más frecuente:

```html
<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
  ...
</div>
```

Una columna en móvil, dos desde 768px, tres desde 1024px: la misma grilla, sin tocar el HTML de los hijos.

-----

## Cuál usar y cuándo

| Situación | Herramienta |
| --- | --- |
| Alinear el contenido de una barra de navegación | Flexbox |
| Centrar un elemento | Flexbox |
| Un botón con ícono y texto alineados | Flexbox |
| Galería de tarjetas o imágenes | Grid |
| El layout de una página completa (encabezado, lateral, contenido) | Grid |
| Contenido que fluye en una sola dirección y se ajusta | Flexbox |

No son excluyentes: es muy común usar **Grid para la estructura general** y **Flexbox dentro de cada componente** para alinear su contenido interno.

-----

## Resumen

* **Flexbox** es de una dimensión (fila o columna); **Grid** es de dos (filas y columnas).
* `flex` activa Flexbox; `justify-*` alinea sobre el eje principal e `items-*` sobre el transversal. Con `flex-col` los ejes se invierten.
* `gap-*` separa elementos sin necesidad de márgenes; `flex-1` y `flex-none` controlan cuánto ocupa cada hijo.
* `grid` activa Grid; `grid-cols-N` define las columnas y `col-span-N` hace que un elemento ocupe varias.
* Combinar Grid con breakpoints (`grid-cols-1 md:grid-cols-3`) es el patrón responsive más usado.
