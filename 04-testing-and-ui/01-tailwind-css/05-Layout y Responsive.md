# Layout y Responsive: Flexbox, Grid y breakpoints

## En una frase

El layout en Tailwind se construye con **utilidades de Flexbox (una dimensión) y Grid (dos dimensiones)**, y se adapta al tamaño de pantalla con **prefijos mobile-first** (`md:`, `lg:`) que aplican desde un ancho mínimo en adelante.

-----

## Antes de empezar

Conviene que ya sepas:

* Que Tailwind se compone con clases de utilidad sobre el elemento: [Fundamentos de Tailwind](01-Fundamentos%20de%20Tailwind.md).
* Cómo usar la escala de espaciado (`p-4`, `gap-4`, `mx-auto`): [Tipografía, Espaciado y Colores](04-Tipograf%C3%ADa%2C%20Espaciado%20y%20Colores.md).
* Por qué las clases deben escribirse completas en el código para que Tailwind las genere: [Cómo Tailwind Detecta las Clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).

Palabras nuevas (también en el [Glosario](Glosario.md)):

* **Breakpoint:** ancho de pantalla a partir del cual un prefijo empieza a aplicar (`md:` = desde 48rem).
* **Mobile-first:** enfoque en el que el estilo base es el del móvil y los prefijos lo sobrescriben en pantallas más anchas.
* **Eje principal / eje transversal:** en Flexbox, la dirección en que se colocan los elementos y su perpendicular.
* **Contenedor flex / grid:** el elemento con `flex` o `grid`. Sus hijos directos son los **ítems**.
* **Viewport:** el área visible de la ventana del navegador.

-----

## El problema

Un mismo diseño debe verse bien en un móvil de 360px y en un monitor de 1920px. Con CSS clásico eso implica escribir hojas de estilo con `@media` y decidir, para cada componente, cómo se reparte el espacio.

Tailwind resuelve dos cosas por separado:

1. **Cómo se acomodan los elementos entre sí:** Flexbox y Grid.
2. **Cómo cambia esa disposición según el ancho:** prefijos responsive.

Ambas se expresan con clases en el propio elemento, sin salir del HTML.

-----

## Cómo funciona

### Mobile-first: qué significa un prefijo

En Tailwind, **responsive** significa cambiar estilos según el ancho de pantalla, usando **prefijos** delante de la clase.

* Una clase **sin prefijo** aplica en **todos** los tamaños.
* Una clase **con prefijo** (`md:`) aplica **desde ese ancho en adelante**.

Se escribe primero el diseño del móvil, sin prefijo, y se **sobrescribe** a medida que la pantalla crece.

### Los breakpoints por defecto

Cada prefijo genera una media query de ancho mínimo (`width >= ...`):

| Prefijo | Ancho mínimo | Píxeles |
| --- | --- | --- |
| `sm:` | `40rem` | 640px |
| `md:` | `48rem` | 768px |
| `lg:` | `64rem` | 1024px |
| `xl:` | `80rem` | 1280px |
| `2xl:` | `96rem` | 1536px |

Lo que define un breakpoint es su ancho mínimo, no el tipo de dispositivo. Frases como "`md` es tablet" son solo una aproximación.

### El error de `sm:` para el móvil

```html
<!-- Incorrecto: centra el texto SOLO desde 640px -->
<div class="sm:text-center"></div>

<!-- Correcto: centrado en móvil, a la izquierda desde 640px -->
<div class="text-center sm:text-left"></div>
```

`sm:` no significa "pantallas chicas". Significa "desde el breakpoint `sm`". Para el móvil no se usa prefijo.

### Cómo se lee una cadena responsive

Los estilos con prefijo se **acumulan** hacia arriba. Se lee de izquierda a derecha como una secuencia de cambios:

```html
<div class="bg-blue-500 sm:bg-green-500 md:bg-yellow-500"></div>
```

* Por debajo de 640px: azul.
* Desde 640px: verde (sobrescribe el azul).
* Desde 768px: amarillo (sobrescribe el verde).

```html
<p class="text-sm md:text-base lg:text-2xl">...</p>
```

Pequeño en móvil, tamaño normal desde 768px y grande desde 1024px.

### Mostrar y ocultar según el tamaño

```html
<!-- Visible en móvil, oculto desde 768px -->
<div class="md:hidden">Menú móvil</div>

<!-- Oculto en móvil, visible desde 768px -->
<div class="hidden md:block">Menú de escritorio</div>
```

Sin prefijo es el estado del móvil; el prefijo cambia el estado desde ese ancho. `hidden` es `display: none`, y `md:block` lo revierte a `display: block`.

### Limitar a un rango con `max-*`

Los prefijos normales significan "desde este ancho hacia arriba". Las variantes `max-*` significan lo contrario: **por debajo de** ese ancho (`width < ...`). Se pueden apilar para definir un rango:

```html
<!-- Solo por debajo de 768px -->
<div class="max-md:hidden"></div>

<!-- Solo entre 768px (incluido) y 1280px (excluido) -->
<div class="md:max-xl:flex"></div>
```

Para un valor puntual que no está en la escala, existen las variantes arbitrarias: `min-[320px]:text-center` y `max-[600px]:bg-sky-300`.

-----

### Flexbox: una dimensión

`flex` convierte al elemento en un **contenedor flex**. Sus hijos directos se organizan en una fila (o en una columna con `flex-col`).

```html
<div class="flex">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

#### Los dos ejes

* El **eje principal** es la dirección en la que se colocan los ítems: horizontal con `flex-row` (el valor por defecto), vertical con `flex-col`.
* El **eje transversal** es el perpendicular.

Sobre ellos:

* `justify-*` alinea sobre el **eje principal**.
* `items-*` alinea sobre el **eje transversal**.

```
flex-row (por defecto)                 flex-col

  eje transversal (items-*)              eje principal (justify-*)
         ^                                      ^
         |  +---+ +---+ +---+                    |  +-----------+
         |  | 1 | | 2 | | 3 |                    |  |     1     |
         |  +---+ +---+ +---+                    |  +-----------+
         +---------------------> eje              |  +-----------+
           eje principal (justify-*)              |  |     2     |
                                                  |  +-----------+
                                                  +-------------------> eje
                                                    eje transversal (items-*)
```

En una fila, `justify-` es horizontal e `items-` es vertical, y por eso suele memorizarse así. Pero con `flex-col` **los ejes se intercambian**: `justify-` pasa a ser vertical e `items-` horizontal. "justify es horizontal" es una simplificación válida solo para filas.

#### Valores de `justify-*` e `items-*`

| Clase | Efecto |
| --- | --- |
| `justify-start` / `items-start` | al inicio del eje |
| `justify-center` / `items-center` | centrado |
| `justify-end` / `items-end` | al final del eje |
| `justify-between` | primero al inicio, último al final, espacio repartido entre ítems |
| `justify-around` | espacio a ambos lados de cada ítem (los extremos reciben la mitad) |
| `justify-evenly` | espacio idéntico entre ítems y en los extremos |
| `items-stretch` | los ítems se estiran para ocupar el eje transversal (valor por defecto) |

`items-*` también admite `items-baseline`, que alinea por la línea base del texto. `justify-*` tiene además variantes `-safe` (por ejemplo `justify-center-safe`), que alinean al inicio si el contenido no cabe en lugar de desbordarse por ambos lados.

#### Patrones frecuentes

**Centrar un elemento en toda la pantalla:**

```html
<div class="flex h-screen items-center justify-center">
  <div class="h-32 w-32 bg-blue-400"></div>
</div>
```

`justify-center` centra sobre el eje principal, `items-center` sobre el transversal. `h-screen` da al contenedor el alto de la ventana (`100vh`); sin una altura, no habría espacio vertical en el que centrar.

**Barra de navegación (logo a la izquierda, menú a la derecha):**

```html
<nav class="flex items-center justify-between">
  <div>Logo</div>
  <ul class="flex gap-4">
    <li>Inicio</li>
    <li>Contacto</li>
  </ul>
</nav>
```

`gap-4` (`1rem`) separa los ítems **entre sí**, sin márgenes individuales. Existen `gap-x-*` y `gap-y-*` para controlar cada eje por separado.

**Columna centrada con separación:**

```html
<div class="flex flex-col items-center gap-4">
  <p>Uno</p>
  <p>Dos</p>
</div>
```

#### Cómo se reparte el espacio entre ítems

Los **ítems** tienen utilidades que controlan cuánto espacio ocupan:

| Clase | CSS | Comportamiento |
| --- | --- | --- |
| `flex-1` | `flex: 1` | Crece y se encoge; **ignora el tamaño inicial** del ítem. |
| `flex-auto` | `flex: auto` | Crece y se encoge, **partiendo de su tamaño inicial**. |
| `flex-initial` | `flex: 0 auto` | Puede encogerse, pero no crece. |
| `flex-none` | `flex: none` | No crece ni se encoge. |

```html
<div class="flex">
  <div class="flex-1">Espacio libre, parte igual</div>
  <div class="flex-1">Espacio libre, parte igual</div>
  <div class="flex-none">Tamaño fijo</div>
</div>
```

Con `flex-1` en varios ítems, se reparten el espacio libre en partes iguales (porque todos parten de cero). Con `flex-auto`, cada uno parte de su contenido y el reparto queda desigual.

Por defecto los ítems **no** pasan a otra línea: se encogen. Para que salten a la línea siguiente cuando no caben, se agrega `flex-wrap` al contenedor.

-----

### Grid: dos dimensiones

`grid` convierte al elemento en un **contenedor grid**. `grid-cols-N` define `N` columnas de igual ancho:

```html
<div class="grid grid-cols-3 gap-4">
  <div>1</div> <div>2</div> <div>3</div>
  <div>4</div> <div>5</div> <div>6</div>
</div>
```

```
grid-cols-3  +  gap-4

+-----+   +-----+   +-----+
|  1  |   |  2  |   |  3  |
+-----+   +-----+   +-----+
                              <- gap (separación entre filas)
+-----+   +-----+   +-----+
|  4  |   |  5  |   |  6  |
+-----+   +-----+   +-----+
   ^          ^          ^
   columnas de igual ancho, separadas por gap
```

Los ítems se colocan solos, de izquierda a derecha y de arriba a abajo (*auto-placement*). Seis ítems en tres columnas forman dos filas sin indicar posición.

En v4, `grid-cols-3` equivale a `grid-template-columns: repeat(3, minmax(0, 1fr))`. El `minmax(0, 1fr)` evita que un contenido ancho estire una columna más allá de su parte.

#### Galería

```html
<div class="grid grid-cols-4 gap-4">
  <img src="1.jpg" alt="" />
  <img src="2.jpg" alt="" />
  <img src="3.jpg" alt="" />
  <img src="4.jpg" alt="" />
</div>
```

#### Que un ítem ocupe más celdas

```html
<div class="grid grid-cols-4 gap-4">
  <div class="col-span-2">Ocupa dos columnas</div>
  <div>Normal</div>
  <div>Normal</div>
</div>
```

`col-span-2` extiende el ítem sobre dos columnas. Para filas existen `grid-rows-*` y `row-span-*`.

#### Cuándo Flexbox y cuándo Grid

| Situación | Herramienta |
| --- | --- |
| Alinear el contenido de una barra de navegación | Flexbox |
| Centrar un elemento | Flexbox |
| Botón con ícono y texto alineados | Flexbox |
| Contenido que fluye en una dirección y se ajusta | Flexbox |
| Galería de tarjetas o imágenes | Grid |
| Layout de página (encabezado, lateral, contenido) | Grid |

Regla práctica: **Flexbox para alinear, Grid para estructurar.** No son excluyentes: es común usar Grid para la estructura general y Flexbox dentro de cada componente.

-----

### Combinar layout con responsive

Los prefijos se aplican a cualquier utilidad de layout:

```html
<div class="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">...</div>
```

Una columna en móvil, dos desde 768px y tres desde 1024px. El HTML de los hijos no cambia.

Con Flexbox, un patrón equivalente es cambiar la dirección:

```html
<div class="flex flex-col gap-4 md:flex-row">
  <aside class="md:w-64 md:flex-none">Lateral</aside>
  <main class="flex-1">Contenido</main>
</div>
```

Apilado en móvil; lado a lado desde 768px, con el lateral a ancho fijo y el contenido ocupando el resto.

-----

### Contenedores: `container`, `mx-auto` y `max-w-*`

En una pantalla grande, el contenido a todo el ancho produce líneas de texto demasiado largas. Se centra con un ancho controlado, con tres piezas:

* **`mx-auto`:** márgenes horizontales automáticos. Centran un bloque cuyo ancho es menor que el de su padre.
* **`max-w-*`:** ancho máximo. `max-w-md` es `28rem` y `max-w-3xl` es `48rem`. El elemento crece hasta ese límite y ahí se detiene.
* **`container`:** fija el `max-width` al breakpoint activo (`40rem`, `48rem`, `64rem`, `80rem`, `96rem`), así que se ensancha por escalones. Además aplica `width: 100%`.

Dos recetas habituales. Conviene elegir **una**, porque `container` y `max-w-*` controlan el ancho máximo del mismo elemento:

```html
<!-- Receta 1: escalones por breakpoint -->
<div class="container mx-auto px-4">...</div>

<!-- Receta 2: ancho máximo fijo, centrado -->
<div class="mx-auto max-w-3xl px-4">...</div>
```

`px-4` añade relleno lateral para que el contenido no quede pegado al borde en pantallas chicas. `max-w-*` también puede ser responsive: `max-w-sm md:max-w-lg`.

> **v3 vs v4:** en v3, `container` podía centrarse solo y tener padding por defecto desde `tailwind.config.js` (`center`, `padding`). En v4 esas opciones no existen: `container` no se centra ni añade padding. Se escribe `mx-auto` y `px-*` explícitamente, o se personaliza con `@utility container { ... }` en el CSS.

-----

### Proporciones: `aspect-ratio`

Un `<iframe>` de YouTube con `width` y `height` fijos se desborda en un móvil. Las utilidades de aspect-ratio mantienen la proporción al cambiar el ancho:

```html
<iframe class="aspect-video w-full" src="..."></iframe>
```

| Clase | CSS |
| --- | --- |
| `aspect-video` | `16 / 9` |
| `aspect-square` | `1 / 1` |
| `aspect-auto` | `auto` (proporción natural) |
| `aspect-3/2` | `3 / 2` (fracción con la sintaxis `aspect-<ratio>`) |
| `aspect-[4/3]` | cualquier valor arbitrario |

Son parte del núcleo de Tailwind: no hace falta el plugin `@tailwindcss/aspect-ratio`, que corresponde a versiones antiguas.

-----

### Breakpoints propios

En v4 se personalizan en el CSS, dentro de `@theme`, con variables `--breakpoint-*`:

```css
@import "tailwindcss";

@theme {
  --breakpoint-xs: 30rem;
  --breakpoint-3xl: 120rem;
}
```

```html
<div class="grid xs:grid-cols-2 3xl:grid-cols-6"></div>
```

Usa la misma unidad (por defecto `rem`) en todos los breakpoints, para evitar un orden de override inesperado. Para quitar uno: `--breakpoint-2xl: initial;`. Para reemplazar todos: `--breakpoint-*: initial;` y luego defines los tuyos.

-----

## Ejemplo completo

Una sección con cabecera, tarjetas en cuadrícula responsive y un video con proporción fija:

```html
<div class="container mx-auto px-4">
  <nav class="flex items-center justify-between py-4">
    <a href="/" class="font-bold">Logo</a>

    <ul class="hidden gap-4 md:flex">
      <li>Inicio</li>
      <li>Contacto</li>
    </ul>

    <button class="md:hidden">Menú</button>
  </nav>

  <section class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
    <article class="rounded-lg border p-4">Tarjeta 1</article>
    <article class="rounded-lg border p-4">Tarjeta 2</article>
    <article class="rounded-lg border p-4 sm:col-span-2 lg:col-span-1">Tarjeta 3</article>
  </section>

  <iframe class="mt-8 aspect-video w-full" src="..."></iframe>
</div>
```

Puntos clave:

1. `container mx-auto px-4` centra y limita el ancho.
2. La `nav` usa **Flexbox** (una dimensión): logo a la izquierda, menú a la derecha.
3. El menú de escritorio es `hidden md:flex` y el botón `md:hidden`: cada uno aparece en su rango.
4. Las tarjetas usan **Grid** (dos dimensiones): 1 columna en móvil, 2 desde 640px, 3 desde 1024px.
5. La tercera tarjeta ocupa toda la fila en el rango de 2 columnas y vuelve a una celda con 3 (`lg:col-span-1`).
6. El video mantiene 16:9 en cualquier ancho.

-----

## En React

En JSX el atributo es `className`, y las clases funcionan igual:

```jsx
function CardGrid({ items }) {
  return (
    <section className="grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3">
      {items.map((item) => (
        <article key={item.id} className="rounded-lg border p-4">
          {item.title}
        </article>
      ))}
    </section>
  );
}
```

Las clases responsive viven en el contenedor; los hijos no necesitan saber cuántas columnas hay. Si el número de columnas depende de una prop, no construyas la clase con una plantilla:

```jsx
// Mal: Tailwind no ve "grid-cols-3" escrito completo y no genera el CSS
<div className={`grid grid-cols-${cols}`} />

// Bien: mapa con nombres de clase completos
const colsClass = { 1: 'grid-cols-1', 2: 'grid-cols-2', 3: 'grid-cols-3' };
<div className={`grid ${colsClass[cols]}`} />
```

El motivo está en [Cómo Tailwind Detecta las Clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).

-----

## Errores comunes

### 1. Usar `sm:` para el estilo móvil

**Qué pasa:** `sm:text-center` no centra nada en un móvil.
**Por qué:** los prefijos son de ancho mínimo (`width >= 40rem`); sin prefijo es lo que aplica en pantallas chicas.
**Arreglo:** estilo base sin prefijo y sobrescribe hacia arriba: `text-center sm:text-left`.

### 2. `justify-*` o `items-*` que no hacen nada

**Qué pasa:** la alineación no cambia.
**Por qué:** esas utilidades solo actúan si el padre es un contenedor `flex` o `grid`.
**Arreglo:** añade `flex` (o `grid`) al elemento padre.

### 3. Confundir los ejes con `flex-col`

**Qué pasa:** `justify-center` deja de centrar horizontalmente al cambiar a columna.
**Por qué:** `justify-*` sigue al eje principal, que en columna es vertical.
**Arreglo:** en `flex-col`, usa `items-center` para el horizontal y `justify-center` para el vertical.

### 4. Centrar verticalmente sin altura

**Qué pasa:** `items-center` no centra en pantalla completa.
**Por qué:** el contenedor mide lo que mide su contenido; no hay espacio sobrante.
**Arreglo:** dale altura: `h-screen` o `min-h-screen`.

### 5. Combinar `container` con `max-w-*`

**Qué pasa:** el ancho máximo no es el que esperabas.
**Por qué:** ambas clases definen `max-width` sobre el mismo elemento y una pisa a la otra.
**Arreglo:** elige una receta. Para anidar, usa un elemento distinto por cada una.

### 6. Esperar que `container` centre y dé padding (herencia de v3)

**Qué pasa:** el contenido queda pegado a la izquierda y a los bordes.
**Por qué:** en v4, `container` no centra ni añade padding.
**Arreglo:** `container mx-auto px-4`.

### 7. Repartir mal el espacio con `flex-1`

**Qué pasa:** con `flex-1` los ítems miden lo mismo aunque su contenido difiera.
**Por qué:** `flex-1` ignora el tamaño inicial.
**Arreglo:** si el contenido debe influir, usa `flex-auto`.

-----

## Cuándo sí y cuándo no

**Usa Flexbox** para alinear en una dirección: barras de navegación, filas de botones, centrado, ítems con ícono y texto.

**Usa Grid** para estructuras de filas y columnas: galerías, tarjetas, layouts de página.

**Usa `container mx-auto`** cuando quieras que el ancho siga los breakpoints. **Usa `mx-auto max-w-*`** cuando quieras un ancho máximo fijo, como una columna de lectura.

**No uses breakpoints por costumbre.** Si el layout ya se adapta solo (`flex-wrap`, columnas fluidas), añadir prefijos suma clases sin aportar. Pon los prefijos donde el diseño realmente cambia.

**No basta con mirar un solo tamaño.** Verifica al menos el ancho móvil, uno intermedio y uno de escritorio.

-----

## Resumen en 5 líneas

1. Tailwind es **mobile-first**: sin prefijo aplica siempre; con prefijo, **desde ese ancho en adelante** (`sm` 640, `md` 768, `lg` 1024, `xl` 1280, `2xl` 1536).
2. `sm:` no es "móvil": para el móvil no se usa prefijo. `max-*` limita hacia abajo y se apila para rangos (`md:max-xl:flex`).
3. **Flexbox** (`flex`) es de una dimensión: `justify-*` sigue el eje principal e `items-*` el transversal; con `flex-col` se intercambian.
4. **Grid** (`grid`, `grid-cols-N`, `col-span-N`) es de dos dimensiones; `grid-cols-1 md:grid-cols-3` es el patrón responsive más común.
5. Para centrar y limitar el ancho: `container mx-auto px-4` o `mx-auto max-w-3xl px-4` (una, no ambas); `aspect-video` mantiene proporciones sin plugin.

-----

## Para profundizar

<details>
<summary>Container queries: responsive según el contenedor, no la ventana</summary>

En v4, las container queries vienen en el núcleo (en v3 requerían un plugin). Se marca el padre con `@container` y los hijos usan variantes con `@`:

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row">...</div>
</div>
```

`@md` aplica cuando el **contenedor** mide al menos `28rem` (448px), sin importar el viewport. También existen `@max-md:`, rangos (`@sm:@max-md:`) y contenedores con nombre (`@container/main` y `@sm/main:`). Son útiles para componentes reutilizables que viven en sitios de distinto ancho (una barra lateral y una columna principal, por ejemplo).

Las escalas `@sm`, `@md`, etc. **no coinciden** con `sm`, `md` de los breakpoints de viewport: `@sm` es `24rem`, mientras que `sm:` es `40rem`.

</details>

<details>
<summary>Grid con columnas automáticas</summary>

Para que el número de columnas se ajuste solo al ancho disponible, sin prefijos, se usa un valor arbitrario con `repeat` y `auto-fit`:

```html
<div class="grid grid-cols-[repeat(auto-fit,minmax(16rem,1fr))] gap-4">...</div>
```

Cada columna mide al menos `16rem` y se reparte el sobrante. Es una alternativa a escribir `md:grid-cols-2 lg:grid-cols-3` cuando lo único que importa es un ancho mínimo por tarjeta.

</details>

<details>
<summary>Cómo funciona por dentro el orden de los prefijos</summary>

Las variantes de breakpoint se emiten en el CSS de menor a mayor. Con `md:` y `lg:` sobre la misma propiedad, gana `lg:` cuando ambas coinciden: aparece después en la hoja. Las clases tienen la misma especificidad, así que decide el orden de aparición.

La documentación recomienda usar una sola unidad (por defecto `rem`) en todos los breakpoints personalizados para evitar un comportamiento de override inesperado.

</details>

<details>
<summary>Valores de `max-w-*` útiles</summary>

`max-w-*` usa la escala de contenedores: `max-w-xs` (20rem), `max-w-sm` (24rem), `max-w-md` (28rem), `max-w-lg` (32rem), `max-w-xl` (36rem), `max-w-2xl` (42rem), `max-w-3xl` (48rem), `max-w-4xl` (56rem), `max-w-5xl` (64rem), `max-w-6xl` (72rem) y `max-w-7xl` (80rem). También hay `max-w-full`, `max-w-none`, `max-w-fit`, `max-w-min`, `max-w-max`, valores numéricos (`max-w-96`) y arbitrarios (`max-w-[220px]`).

</details>

-----

## En entrevista

### Respuesta corta (junior)

Tailwind es mobile-first: las clases sin prefijo aplican a todas las pantallas y las que llevan prefijo (`md:`) aplican desde ese ancho mínimo en adelante. Flexbox (`flex`) organiza elementos en una dimensión y Grid (`grid`) en dos. Se combinan, por ejemplo con `grid-cols-1 md:grid-cols-3`, para adaptar el layout al tamaño de pantalla.

### Respuesta ampliada (semi-senior)

* **Mobile-first:** los prefijos generan media queries de ancho mínimo (`@media (width >= 48rem)`). El estilo base es el móvil y se sobrescribe hacia arriba. `sm:` significa "desde 640px", no "en móviles".
* **Breakpoints por defecto:** `sm` 40rem, `md` 48rem, `lg` 64rem, `xl` 80rem, `2xl` 96rem. En v4 se personalizan en CSS con `--breakpoint-*` dentro de `@theme`, no en `tailwind.config.js`.
* **Rangos:** `max-*` genera `width < ...`. Se apilan: `md:max-xl:flex` aplica entre 48rem y 80rem. También hay variantes arbitrarias `min-[...]` y `max-[...]`.
* **Flexbox:** `justify-*` alinea sobre el eje principal e `items-*` sobre el transversal; en `flex-col` se intercambian. Los ítems se controlan con `flex-1` (`flex: 1`, ignora tamaño inicial), `flex-auto`, `flex-initial` y `flex-none`.
* **Grid:** `grid-cols-N` es `repeat(N, minmax(0, 1fr))`; `col-span-N` y `row-span-N` extienden ítems. Con Grid se define la estructura y con Flexbox se alinea el interior de cada componente.
* **Contenedores:** en v4, `container` fija `max-width` por breakpoint pero no se centra ni añade padding (cambio respecto a v3); se usa `container mx-auto px-4` o `mx-auto max-w-*`.
* **Container queries:** en v4 son nativas (`@container`, `@md:`) y dependen del tamaño del padre, no del viewport.
* **Trade-offs:** el responsive con prefijos repite clases en el marcado. Se compensa extrayendo componentes (en React) en lugar de duplicar el HTML.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué `sm:text-center` no centra el texto en un móvil?**
Porque `sm:` aplica desde 640px. Lo que aplica en pantallas menores es la clase sin prefijo, así que el patrón correcto es `text-center sm:text-left`.

**2. ¿Cuál es la diferencia entre `justify-*` e `items-*`?**
`justify-*` alinea sobre el eje principal e `items-*` sobre el transversal. En `flex-row` son horizontal y vertical respectivamente; en `flex-col`, al revés.

**3. ¿Cuándo usas Flexbox y cuándo Grid?**
Flexbox cuando el contenido fluye en una dirección y se trata de alinear o repartir (navbars, botones, centrado). Grid cuando necesitas controlar filas y columnas a la vez (galerías, layouts de página). Es común anidarlos.

**4. ¿Cómo aplicas un estilo solo entre dos anchos?**
Apilando una variante normal con una `max-*`: `md:max-xl:flex` aplica desde `48rem` hasta justo antes de `80rem`.

**5. ¿Qué cambió con `container` entre v3 y v4?**
En v3 se podía configurar `center` y `padding` en `tailwind.config.js`. En v4 no existen: `container` solo fija el `max-width` por breakpoint, así que se añaden `mx-auto` y `px-*` a mano, o se redefine con `@utility container`.

**6. ¿Cómo agregas un breakpoint propio en v4?**
En el CSS, dentro de `@theme`: `--breakpoint-3xl: 120rem;`. Después se usa `3xl:` como cualquier prefijo. Con `--breakpoint-*: initial;` se descartan los predeterminados.

-----

## Siguiente lección

Ya sabes ordenar y adaptar el layout. El paso siguiente es personalizar el sistema de diseño (colores, fuentes, breakpoints propios), el modo oscuro y los plugins: [Tema, Modo Oscuro y Plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md).
