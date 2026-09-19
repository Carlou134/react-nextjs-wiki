# Tipografía, Espaciado y Cajas

## La convención de nombres

Antes de ver las utilidades una por una, conviene entender la lógica que las une, porque una vez que la ves, dejás de necesitar memorizar la lista:

* El **prefijo** indica la propiedad CSS: `text-` (tamaño o color del texto), `font-` (peso o familia), `m-` (margin), `p-` (padding), `bg-` (fondo), `border-`, `rounded-`, `shadow-`.
* El **sufijo** indica el valor, casi siempre tomado de una **escala** predefinida: `sm`, `lg`, `2xl` para tamaños; `1`, `4`, `8` para espaciados; `blue-500` para colores.

Entonces `text-lg` es "tamaño de texto grande", `p-4` es "padding de la escala 4", y `border-red-500` es "borde de color rojo, intensidad 500".

-----

## Tipografía

### Tamaño del texto

```html
<p class="text-sm">Pequeño</p>
<p class="text-base">Normal</p>
<p class="text-lg">Grande</p>
<p class="text-2xl">Muy grande</p>
```

La escala sigue con `text-xl`, `text-3xl`, `text-4xl`, y así hasta tamaños de titular.

### Peso (negrita)

```html
<p class="font-thin">Muy fino</p>
<p class="font-normal">Normal</p>
<p class="font-bold">Negrita</p>
<p class="font-extrabold">Extra negrita</p>
```

### Estilo, alineación y espaciado de letras

```html
<p class="italic">Cursiva</p>
<p class="not-italic">Sin cursiva</p>

<p class="text-left">Izquierda</p>
<p class="text-center">Centrado</p>
<p class="text-right">Derecha</p>
<p class="text-justify">Justificado</p>

<p class="tracking-wide">Letras más separadas</p>
<p class="tracking-tight">Letras más juntas</p>
```

`tracking-` controla el espaciado entre letras (*letter-spacing*), y su hermana `leading-` controla la altura de línea (*line-height*), por ejemplo `leading-tight` o `leading-loose`.

### Todo junto

```html
<h1 class="text-4xl font-bold text-center text-blue-500">
  Hola mundo
</h1>
```

Se lee de corrido: texto muy grande, en negrita, centrado, de color azul.

-----

## Espaciado: margin y padding

El espaciado es probablemente lo más usado de Tailwind, y se apoya en la distinción básica del modelo de caja de CSS:

* **Margin (`m-`)**: el espacio **por fuera** del elemento, que lo separa de sus vecinos.
* **Padding (`p-`)**: el espacio **por dentro** del elemento, que separa su contenido de su propio borde.

```html
<div class="m-4 p-6 bg-blue-200">
  Margen afuera, espacio interno adentro
</div>
```

### La escala

Los números **no son píxeles directos**: son pasos de una escala en la que cada unidad equivale a `0.25rem`. Como el tamaño de fuente base de un navegador suele ser 16px, en la práctica:

| Clase | Valor |
| --- | --- |
| `m-1` / `p-1` | `0.25rem` (4px) |
| `m-4` / `p-4` | `1rem` (16px) |
| `m-6` / `p-6` | `1.5rem` (24px) |
| `m-8` / `p-8` | `2rem` (32px) |

Como es una escala de `rem`, se ajusta si el usuario cambia el tamaño de fuente de su navegador, algo que los píxeles fijos no hacen.

### Lados y ejes

Podés controlar un lado específico, o un eje completo:

| Clase | Aplica a |
| --- | --- |
| `mt-4` / `pt-4` | arriba (*top*) |
| `mb-4` / `pb-4` | abajo (*bottom*) |
| `ml-4` / `pl-4` | izquierda (*left*) |
| `mr-4` / `pr-4` | derecha (*right*) |
| `mx-4` / `px-4` | horizontal (izquierda y derecha) |
| `my-4` / `py-4` | vertical (arriba y abajo) |

-----

## Tamaños

El ancho y el alto se controlan con `w-` y `h-`, usando la misma escala de espaciado, además de valores especiales:

```html
<div class="w-32 h-32 bg-blue-400"></div>   <!-- 8rem x 8rem -->
<div class="w-full h-screen"></div>          <!-- todo el ancho, todo el alto de la ventana -->
```

Y cuando ningún valor de la escala te sirve, Tailwind permite un **valor arbitrario** entre corchetes:

```html
<div class="w-[137px] h-[3.5rem]"></div>
```

Es una válvula de escape útil para casos puntuales; si te encontrás usando el mismo valor arbitrario en muchos lugares, es señal de que ese valor debería ser parte de tu tema (lo vemos en la lección de personalización).

-----

## Bordes, redondeado y sombras

### Bordes

```html
<div class="border border-red-500">Borde de 1px rojo</div>
<div class="border-4 border-blue-500">Borde grueso azul</div>
```

`border` activa el borde (y define su grosor); `border-{color}` define su color.

### Esquinas redondeadas

```html
<div class="rounded-sm">Levemente redondeado</div>
<div class="rounded-xl">Muy redondeado</div>
<div class="rounded-full">Completamente redondo</div>
```

### Sombras

```html
<div class="shadow-md">Sombra media</div>
<div class="shadow-2xl">Sombra pronunciada</div>
```

Las sombras dan sensación de profundidad, y son lo que hace que un bloque se lea como una "tarjeta" que flota sobre la página.

> **Diferencias entre v3 y v4 en estas utilidades:** Tailwind v4 renombró las clases de la escala más chica de sombras, blur y esquinas, para que la escala sea más regular. Lo que en v3 era `shadow` pasó a ser `shadow-sm`, y el `shadow-sm` de v3 pasó a ser `shadow-xs`. Lo mismo ocurre con `rounded` (ahora `rounded-sm`) y `rounded-sm` (ahora `rounded-xs`). Además, el color de borde por defecto de `border` cambió de gris a `currentColor`: en v4 conviene indicar siempre un color explícito (`border border-gray-200`). Si copiás código de un tutorial de v3, estos son los lugares donde vas a notar diferencias.

-----

## Ejemplo completo: una tarjeta

```html
<div class="m-4 p-6 bg-blue-100 border border-blue-500 rounded-xl shadow-md">
  <h2 class="text-xl font-bold text-blue-900">Título</h2>
  <p class="mt-2 text-blue-800">Contenido de la tarjeta.</p>
</div>
```

Traducción de la caja exterior: `m-4` la separa de lo que la rodea, `p-6` da aire al contenido, `border border-blue-500` le pone un borde azul, `rounded-xl` suaviza las esquinas y `shadow-md` le da profundidad.

-----

## Resumen

* Las clases siguen una lógica de **prefijo (propiedad) + sufijo (valor de la escala)**.
* `text-` controla tamaño y color; `font-` el peso; `tracking-` y `leading-` el espaciado de letras y de líneas.
* **Margin** (`m-`) es el espacio de afuera, **padding** (`p-`) es el espacio de adentro; los números son pasos de `0.25rem`, no píxeles.
* Se puede apuntar a un lado (`mt-`, `pl-`) o a un eje (`mx-`, `py-`).
* `w-`/`h-` controlan el tamaño, y los valores arbitrarios (`w-[137px]`) resuelven casos puntuales.
* `border`, `rounded-` y `shadow-` completan el diseño de una caja; en v4 algunos nombres de la escala chica cambiaron.
