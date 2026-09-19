# Diseño Responsive

## El principio: mobile-first

En Tailwind, **responsive** significa cambiar estilos según el ancho de la pantalla, y se logra con **prefijos** delante de la clase. El punto que hay que entender bien, porque es la fuente de casi todos los errores, es que Tailwind es **mobile-first** (primero móvil):

* Una clase **sin prefijo** aplica en **todos** los tamaños de pantalla.
* Una clase **con prefijo** (`md:`) aplica **desde ese ancho en adelante**.

Es decir, escribís primero el diseño para móvil, sin prefijo, y después vas **sobrescribiendo** a medida que la pantalla crece.

-----

## Los breakpoints

Cada prefijo corresponde a un **ancho mínimo** (`min-width`):

| Prefijo | Ancho mínimo | En píxeles |
| --- | --- | --- |
| `sm:` | `40rem` | 640px |
| `md:` | `48rem` | 768px |
| `lg:` | `64rem` | 1024px |
| `xl:` | `80rem` | 1280px |
| `2xl:` | `96rem` | 1536px |

Fijate que las descripciones informales que circulan ("`sm` es celular grande, `md` es tablet, `lg` es laptop") son solo una aproximación orientativa: lo que define un breakpoint es el ancho mínimo en píxeles, no el tipo de dispositivo.

### El error más común: usar `sm:` para el móvil

```html
<!-- Incorrecto: esto SOLO centra el texto desde 640px en adelante -->
<div class="sm:text-center"></div>

<!-- Correcto: centrado en móvil, alineado a la izquierda desde 640px -->
<div class="text-center sm:text-left"></div>
```

`sm:` no significa "en pantallas chicas": significa "desde 640px hacia arriba". Para el móvil, simplemente no se usa prefijo.

-----

## Cómo se lee una cadena responsive

Como los estilos con prefijo se **acumulan** hacia arriba, una cadena se lee de izquierda a derecha como una secuencia de cambios:

```html
<div class="bg-blue-500 sm:bg-green-500 md:bg-yellow-500"></div>
```

* Por debajo de 640px: azul.
* Desde 640px: verde (sobrescribe el azul).
* Desde 768px: amarillo (sobrescribe el verde).

Y con texto:

```html
<p class="text-sm md:text-base lg:text-2xl">
```

Pequeño en móvil, tamaño normal desde 768px, y grande desde 1024px. La traducción mental es: "empiezo chico, y lo agrando a medida que hay lugar".

-----

## Mostrar y ocultar según el tamaño

Un patrón muy común es mostrar algo solo en cierto rango de pantallas:

```html
<!-- Visible en móvil, oculto desde 640px -->
<div class="block sm:hidden">Menú móvil</div>

<!-- Oculto en móvil, visible desde 768px -->
<div class="hidden md:block">Menú de escritorio</div>
```

Ambos patrones siguen la regla: sin prefijo es el estado en móvil, y el prefijo cambia el estado desde ese ancho.

-----

## Limitar a un rango con `max-*`

Como los prefijos normales son "desde este ancho hacia arriba", a veces necesitás lo opuesto: aplicar algo **solo por debajo de** un ancho, o solo **entre dos anchos**. Para eso existen las variantes `max-*`, que se pueden apilar con las normales:

```html
<!-- Solo entre 768px y 1280px -->
<div class="md:max-xl:flex"></div>

<!-- Solo por debajo de 768px -->
<div class="max-md:hidden"></div>
```

-----

## Contenedores: `container`, `mx-auto` y `max-w-*`

Un sitio que ocupa todo el ancho de una pantalla grande se ve mal: las líneas de texto se vuelven demasiado largas para leer cómodamente. Lo habitual es que el contenido esté **centrado y con un ancho controlado**. Hay tres piezas para eso:

* **`mx-auto`**: márgenes horizontales automáticos, que **centran** un bloque que tiene un ancho definido.
* **`max-w-*`**: define un **ancho máximo** (`max-w-md` es `28rem`, `max-w-3xl` es `48rem`, y así). El elemento crece hasta ese límite y deja de crecer.
* **`container`**: fija un ancho máximo **igual al breakpoint activo**, así que el contenedor se va ensanchando por escalones a medida que crece la pantalla.

Hay dos recetas habituales, y conviene **elegir una** en lugar de combinarlas, porque tanto `container` como `max-w-*` controlan el ancho máximo del mismo elemento:

```html
<!-- Receta 1: contenedor con escalones por breakpoint -->
<div class="container mx-auto px-4">...</div>

<!-- Receta 2: ancho máximo fijo, centrado -->
<div class="mx-auto max-w-3xl px-4">...</div>
```

`px-4` agrega el margen interno lateral, para que el contenido no quede pegado al borde en pantallas chicas.

`max-w-*` también se puede volver responsive: `max-w-sm md:max-w-lg` es un contenedor angosto en móvil que se ensancha desde 768px.

> **Diferencia entre v3 y v4:** en v3, la clase `container` se podía configurar para centrarse sola y tener padding por defecto desde `tailwind.config.js`. En v4 esas opciones no existen, y la personalización se hace con la directiva `@utility container { margin-inline: auto; padding-inline: 2rem; }` en el CSS. Por eso, en v4 siempre conviene escribir `mx-auto` explícitamente.

-----

## Proporciones: `aspect-ratio`

Otro problema clásico de responsive son los videos y las imágenes, que deben mantener su proporción al cambiar de tamaño. Un `<iframe>` de YouTube con `width="560" height="315"` fijos se desborda en un móvil; con Tailwind se resuelve con las utilidades de aspect-ratio:

```html
<iframe class="aspect-video w-full" src="..."></iframe>
```

* `aspect-video`: proporción 16:9.
* `aspect-square`: proporción 1:1.
* `aspect-auto`: la proporción natural del elemento.
* `aspect-[5/4]`: cualquier proporción con un valor arbitrario.

Estas utilidades son parte del **núcleo de Tailwind**: no hace falta instalar ningún plugin. (Vas a encontrar tutoriales viejos que instalan `@tailwindcss/aspect-ratio`, pero ese plugin es un resto de versiones anteriores del framework, cuando esa funcionalidad todavía no venía incluida.)

-----

## Breakpoints propios

Si los breakpoints por defecto no encajan con tu diseño, en v4 se personalizan desde el CSS, en el bloque `@theme`:

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

-----

## Resumen

* Tailwind es **mobile-first**: sin prefijo es el estilo base (aplica siempre), y con prefijo aplica **desde ese ancho en adelante**.
* Los breakpoints son anchos mínimos: `sm` 640px, `md` 768px, `lg` 1024px, `xl` 1280px, `2xl` 1536px.
* `sm:` **no** significa "móvil": para el móvil no se usa prefijo.
* `max-*` limita a un rango (`md:max-xl:flex`).
* Para centrar contenido con ancho controlado: `mx-auto` + `max-w-*`, o `container mx-auto`; elegí una receta, no las dos a la vez.
* `aspect-video` y compañía mantienen proporciones sin necesidad de plugins.
