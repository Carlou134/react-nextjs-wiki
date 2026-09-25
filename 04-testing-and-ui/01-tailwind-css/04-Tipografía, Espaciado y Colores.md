# Tipografía, espaciado y colores

## En una frase

Casi todo el estilo visual de una interfaz se resuelve con tres familias de utilidades de Tailwind: tipografía (`text-`, `font-`), espaciado y cajas (`m-`, `p-`, `w-`, `border`, `rounded-`, `shadow-`) y color (`bg-blue-500`, `text-slate-900`), todas con la misma lógica de **prefijo + valor de una escala**.

-----

## Antes de empezar

Conviene que ya conozcas:

* Qué es una clase utilitaria y cómo se combinan en el HTML: [Fundamentos de Tailwind](01-Fundamentos%20de%20Tailwind.md).
* Que Tailwind solo genera las clases que encuentra escritas completas en tu código: [Cómo Tailwind detecta las clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Escala:** conjunto ordenado de valores predefinidos (tamaños, espaciados, colores) entre los que eliges en lugar de inventar números.
* **`rem`:** unidad relativa al tamaño de fuente base del navegador (normalmente 16px). `1rem` equivale a 16px salvo que el usuario cambie esa base.
* **Modelo de caja:** en CSS, todo elemento es una caja formada por contenido, padding, borde y margin.
* **Opacidad:** cuánto deja ver lo que hay detrás de un color. `100%` es sólido; `0%`, invisible.
* **Valor arbitrario:** un valor puntual entre corchetes, como `w-[137px]`, que no pertenece a la escala.

-----

## El problema

En CSS tradicional, cada decisión visual pide inventar un valor: `padding: 13px`, `font-size: 17px`, `#3b82f6`, `#3a81f4`. Con el tiempo aparecen decenas de valores casi iguales, y la interfaz pierde consistencia.

Tailwind resuelve esto con **escalas cerradas**. No eliges "13px": eliges un paso (`p-3`, `p-4`). Los valores salen del tema, así que todo el proyecto comparte los mismos tamaños, espaciados y colores.

-----

## Cómo funciona

### La convención de nombres

Una vez que se entiende la lógica, no hace falta memorizar la lista de clases.

* El **prefijo** indica la propiedad CSS: `text-` (tamaño o color del texto), `font-` (peso), `m-` (margin), `p-` (padding), `bg-` (fondo), `border-`, `rounded-`, `shadow-`.
* El **sufijo** indica el valor, casi siempre tomado de una escala: `sm`, `lg`, `2xl` para tamaños; `1`, `4`, `8` para espaciados; `blue-500` para colores.

Por eso `text-lg` es "tamaño de texto grande", `p-4` es "padding del paso 4" y `border-red-500` es "borde rojo, intensidad 500".

> Preflight, el reinicio de estilos que Tailwind aplica por defecto, deja los títulos `h1` a `h6` sin tamaño ni peso propios (`font-size: inherit; font-weight: inherit`) y elimina los márgenes del navegador. Un `<h1>` sin clases se ve como texto normal: el tamaño lo decides tú.

### Tipografía

#### Tamaño del texto

Cada tamaño de la escala trae también una altura de línea por defecto.

| Clase | Tamaño |
| --- | --- |
| `text-xs` | `0.75rem` (12px) |
| `text-sm` | `0.875rem` (14px) |
| `text-base` | `1rem` (16px) |
| `text-lg` | `1.125rem` (18px) |
| `text-xl` | `1.25rem` (20px) |
| `text-2xl` | `1.5rem` (24px) |
| `text-3xl` | `1.875rem` (30px) |
| `text-4xl` | `2.25rem` (36px) |

La escala continúa hasta `text-9xl` (`8rem`). Para fijar tamaño y altura de línea juntos existe la sintaxis `text-{tamaño}/{altura}`:

```html
<p class="text-sm/6">14px de tamaño y altura de línea del paso 6 (1.5rem)</p>
```

#### Peso

| Clase | Peso |
| --- | --- |
| `font-thin` | 100 |
| `font-light` | 300 |
| `font-normal` | 400 |
| `font-medium` | 500 |
| `font-semibold` | 600 |
| `font-bold` | 700 |
| `font-extrabold` | 800 |
| `font-black` | 900 |

También existe `font-extralight` (200). Los pesos que se ven realmente dependen de que la fuente cargada los incluya.

#### Estilo, alineación y espaciado

```html
<p class="italic">Cursiva</p>
<p class="not-italic">Sin cursiva</p>

<p class="text-left">Izquierda</p>
<p class="text-center">Centrado</p>
<p class="text-right">Derecha</p>
<p class="text-justify">Justificado</p>

<p class="tracking-wide">Letras más separadas</p>
<p class="tracking-tight">Letras más juntas</p>
<p class="leading-tight">Líneas más juntas</p>
<p class="leading-relaxed">Líneas más separadas</p>
```

* `tracking-` controla el espaciado entre letras (*letter-spacing*): de `tracking-tighter` (`-0.05em`) a `tracking-widest` (`0.1em`).
* `leading-` controla la altura de línea (*line-height*). Los nombres (`leading-tight`, `leading-relaxed`) son relativos al tamaño de la fuente; `leading-6` es una altura fija tomada de la escala de espaciado.

El prefijo `text-` sirve para tres cosas distintas, y Tailwind distingue por el sufijo: `text-lg` (tamaño), `text-center` (alineación) y `text-blue-500` (color).

### Espaciado: margin y padding

Se apoya en el modelo de caja de CSS:

* **Margin (`m-`):** espacio **por fuera** del elemento. Lo separa de sus vecinos.
* **Padding (`p-`):** espacio **por dentro**. Separa el contenido del borde.

```
+----------------------------------------+
|  margin                                |
|  +----------------------------------+  |
|  |  border                          |  |
|  |  +----------------------------+  |  |
|  |  |  padding                   |  |  |
|  |  |  +----------------------+  |  |  |
|  |  |  |  contenido           |  |  |  |
|  |  |  +----------------------+  |  |  |
|  |  +----------------------------+  |  |
|  +----------------------------------+  |
+----------------------------------------+
   m-4    border    p-6      w- / h-
```

```html
<div class="m-4 p-6 bg-blue-200">
  Margen afuera, espacio interno adentro
</div>
```

#### La escala

En Tailwind v4 todo el espaciado sale de una única variable del tema, `--spacing`, que vale `0.25rem`. Una clase como `p-4` se calcula como `calc(var(--spacing) * 4)`. Los números **no son píxeles**: son múltiplos de esa unidad.

| Clase | Valor |
| --- | --- |
| `p-1` | `0.25rem` (4px) |
| `p-2` | `0.5rem` (8px) |
| `p-4` | `1rem` (16px) |
| `p-6` | `1.5rem` (24px) |
| `p-8` | `2rem` (32px) |

Como la escala está en `rem`, se ajusta si el usuario cambia el tamaño de fuente del navegador. Los píxeles fijos no lo hacen. Además existen `p-px` (`1px`) y, solo en margin, `m-auto`.

#### Lados y ejes

| Margin | Padding | Aplica a |
| --- | --- | --- |
| `mt-4` | `pt-4` | arriba (*top*) |
| `mb-4` | `pb-4` | abajo (*bottom*) |
| `ml-4` | `pl-4` | izquierda (*left*) |
| `mr-4` | `pr-4` | derecha (*right*) |
| `mx-4` | `px-4` | eje horizontal |
| `my-4` | `py-4` | eje vertical |
| `ms-4` | `ps-4` | inicio (según la dirección del texto) |
| `me-4` | `pe-4` | fin (según la dirección del texto) |

Detalles útiles:

* `mx-auto` centra horizontalmente un bloque con ancho definido.
* El margin acepta valores negativos con un guion delante: `-mt-4`. El padding no.
* `space-x-4` y `space-y-4` (en el contenedor) separan a los hijos entre sí sin tocar el margin de cada uno.

### Tamaños

El ancho y el alto se controlan con `w-` y `h-`, con la misma escala de espaciado más algunos valores especiales:

| Clase | CSS |
| --- | --- |
| `w-32` | `width: 8rem` (escala de espaciado) |
| `w-1/2` | `width: 50%` (fracciones) |
| `w-full` | `width: 100%` |
| `w-screen` | `width: 100vw` |
| `h-screen` | `height: 100vh` |
| `h-dvh` | `height: 100dvh` |
| `w-fit` | `width: fit-content` |
| `w-auto` | `width: auto` |
| `size-16` | ancho y alto a la vez |

También hay una escala de tamaños de contenedor (`w-xs` = `20rem`, `w-md` = `28rem`, `w-3xl` = `48rem`), que se usa mucho con `max-w-`.

`h-dvh` usa la altura **dinámica** del viewport, que se ajusta cuando la barra del navegador móvil aparece o desaparece. `h-screen` usa `100vh`, que en móviles puede quedar más alto que el área visible.

Cuando ningún valor de la escala sirve, se usa un **valor arbitrario** entre corchetes, o entre paréntesis si es una variable CSS:

```html
<div class="w-[137px] h-[3.5rem]"></div>
<div class="w-(--sidebar-width)"></div>
```

Es una válvula de escape para casos puntuales. Si repites el mismo valor arbitrario en muchos lugares, conviene moverlo al tema (se ve en [Tema, modo oscuro y plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md)).

### Bordes, redondeado y sombras

#### Bordes

```html
<div class="border border-red-500">Borde de 1px rojo</div>
<div class="border-4 border-blue-500">Borde grueso azul</div>
<div class="border-b-2 border-slate-300">Solo borde inferior</div>
```

`border` activa el borde con `1px`; `border-4` lo pone de `4px`; `border-t`, `border-b`, `border-x` y similares lo limitan a un lado o eje. `border-{color}` define el color.

Preflight deja todos los elementos con `border: 0 solid`, y por eso basta la clase `border` para que aparezca un borde sólido. En v4 su color por defecto es `currentColor`, es decir, **el color del texto**. Si quieres otro, indícalo.

#### Esquinas redondeadas

| Clase | Radio |
| --- | --- |
| `rounded-xs` | `0.125rem` (2px) |
| `rounded-sm` | `0.25rem` (4px) |
| `rounded-md` | `0.375rem` (6px) |
| `rounded-lg` | `0.5rem` (8px) |
| `rounded-xl` | `0.75rem` (12px) |
| `rounded-2xl` | `1rem` (16px) |
| `rounded-full` | totalmente redondo |
| `rounded-none` | sin redondeo |

Se puede limitar a un lado (`rounded-t-lg`) o a una esquina (`rounded-tl-lg`). `rounded-full` da un círculo en un elemento cuadrado y una forma de píldora en uno rectangular.

#### Sombras

| Clase | Efecto |
| --- | --- |
| `shadow-2xs` | sombra mínima (una línea fina) |
| `shadow-xs` | muy sutil |
| `shadow-sm` | pequeña |
| `shadow-md` | media |
| `shadow-lg` | grande |
| `shadow-xl` | más grande |
| `shadow-2xl` | pronunciada |
| `shadow-none` | sin sombra |

Las sombras dan sensación de profundidad: hacen que un bloque se lea como una tarjeta que flota sobre la página. El color de la sombra se cambia con `shadow-{color}` (por ejemplo, `shadow-blue-500`).

#### Nombres que cambiaron entre v3 y v4

Tailwind v4 desplazó un paso la escala más pequeña de sombras, desenfoques y esquinas para que fuera más regular.

| En v3 | En v4 |
| --- | --- |
| `shadow-sm` | `shadow-xs` |
| `shadow` | `shadow-sm` |
| `rounded-sm` | `rounded-xs` |
| `rounded` | `rounded-sm` |
| `blur-sm` | `blur-xs` |
| `blur` | `blur-sm` |
| `ring` (3px) | `ring-3` (`ring` ahora es 1px) |

Esto afecta el código copiado de tutoriales antiguos: `shadow-sm` sigue funcionando, pero es más pequeño que el de v3.

### Colores

#### La forma de un color

Los colores de la paleta por defecto siguen el patrón `{utilidad}-{color}-{intensidad}`.

```html
<p class="text-blue-500">Texto azul</p>
<div class="bg-green-200">Fondo verde claro</div>
<div class="border border-red-600">Borde rojo oscuro</div>
```

* La **utilidad** indica la propiedad: `text-` (texto), `bg-` (fondo), `border-` (borde), `shadow-`, `ring-`, entre otras.
* El **color** es el nombre de la familia: `blue`, `red`, `green`, `pink`, y los grises `slate`, `gray`, `zinc`, `neutral` y `stone`.
* La **intensidad** va de `50` (muy claro) a `950` (muy oscuro), con `500` como tono medio.

| Intensidad | Se ve como |
| --- | --- |
| `50`, `100` | casi blanco, tinte muy suave |
| `500` | tono medio |
| `900`, `950` | casi negro, tinte oscuro |

Los colores de la paleta están definidos en el espacio de color OKLCH, y cada uno está disponible como variable CSS: `var(--color-blue-500)`.

#### Transparencia: el modificador `/`

Se agrega una barra y el porcentaje de opacidad al color:

```html
<div class="bg-black/50">Fondo negro al 50% de opacidad</div>
<div class="bg-sky-500/75">Azul cielo al 75%</div>
<div class="bg-pink-500/[71.37%]">Valor arbitrario</div>
```

En v3 existían utilidades separadas (`bg-opacity-50`, `text-opacity-50`). En v4 se eliminaron: el modificador `/` es la única forma.

#### Contraste

Combinar cualquier color de texto con cualquier fondo es fácil, pero no siempre legible. `bg-green-200 text-white` es texto blanco sobre verde claro: casi no se lee.

Regla práctica:

* Fondos claros (`50` a `200`) con textos oscuros (`800` a `950`).
* Fondos oscuros (`700` a `950`) con textos claros (`50` a `200`).

Es un tema de accesibilidad, no solo de estética: existen ratios de contraste mínimos recomendados (WCAG) para que el texto sea legible.

#### Gradientes

Un degradado combina una **dirección** con hasta tres **paradas de color**:

```html
<div class="bg-linear-to-r from-purple-400 via-pink-500 to-red-500 p-4 text-white">
  Degradado
</div>
```

* `bg-linear-to-r`: dirección (de izquierda a derecha). Las otras son `-t`, `-tr`, `-b`, `-br`, `-bl`, `-l` y `-tl`.
* `from-`: color inicial. `via-`: color intermedio (opcional). `to-`: color final.
* `bg-linear-65` fija un ángulo exacto en grados.
* Cada parada acepta una posición en porcentaje:

```html
<div class="bg-linear-to-r from-indigo-500 from-10% via-sky-500 via-30% to-emerald-500 to-90%"></div>
```

Por defecto, los degradados interpolan los colores en el espacio `oklab`. Existen además `bg-radial` y `bg-conic` para degradados radiales y cónicos. Por eso v4 renombró `bg-gradient-to-r` a `bg-linear-to-r`: el nombre `linear` los distingue.

#### Colores propios (pasos básicos)

Cuando la paleta no alcanza (por ejemplo, para los colores de una marca), no escribas valores sueltos en cada clase. Agrégalos al **tema** con `@theme`:

```css
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.72 0.11 178);
}
```

Desde ese momento funcionan como cualquier otro color: `bg-brand`, `text-brand`, `border-brand`, y también `bg-brand/50`. La variable queda disponible como `var(--color-brand)`.

El tema completo (escalas, fuentes, sobrescribir la paleta) y el modo oscuro con `dark:` se explican en [Tema, modo oscuro y plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md).

-----

## Ejemplo completo

Una tarjeta de plan con encabezado en degradado:

```html
<article class="m-4 max-w-sm overflow-hidden rounded-xl border border-slate-200 bg-white shadow-md">
  <div class="bg-linear-to-r from-indigo-500 to-sky-500 p-4">
    <h2 class="text-xl font-bold text-white">Plan Pro</h2>
  </div>

  <div class="p-6">
    <p class="text-sm/6 text-slate-700">
      Proyectos ilimitados y soporte prioritario.
    </p>
    <p class="mt-4 text-3xl font-bold tracking-tight text-slate-900">$12</p>
    <button class="mt-6 w-full rounded-lg bg-indigo-600 px-4 py-2 font-semibold text-white">
      Elegir plan
    </button>
  </div>
</article>
```

Lectura de la caja exterior:

1. `m-4` la separa de lo que la rodea y `max-w-sm` limita su ancho a `24rem`.
2. `rounded-xl` redondea las esquinas. `overflow-hidden` recorta el encabezado para que respete ese redondeo.
3. `border border-slate-200` agrega un borde gris claro (con color explícito, porque en v4 el color por defecto es el del texto).
4. `shadow-md` le da profundidad.
5. Adentro, `p-4` y `p-6` dan aire al contenido, y `mt-4` y `mt-6` separan los bloques verticalmente.

-----

## En React

En React, las clases van en `className`:

```tsx
<button className="rounded-lg bg-indigo-600 px-4 py-2 font-semibold text-white">
  Guardar
</button>
```

Cuando el estilo depende de una prop, asigna **nombres de clase completos** en un objeto y elige uno. No los armes concatenando fragmentos:

```tsx
import type { ReactNode } from 'react';

const variants = {
  info: 'bg-blue-100 text-blue-900 border-blue-500',
  error: 'bg-red-100 text-red-900 border-red-500',
} as const;

type AlertProps = { kind: keyof typeof variants; children: ReactNode };

function Alert({ kind, children }: AlertProps) {
  return (
    <div className={`rounded-lg border p-4 ${variants[kind]}`}>{children}</div>
  );
}
```

Escribir `` `bg-${color}-100` `` no funciona: Tailwind lee tu código como texto y nunca ve la clase completa, por lo que no genera su CSS (ver [Cómo Tailwind detecta las clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md)). Si el valor es realmente dinámico (un ancho calculado en tiempo de ejecución), usa el atributo `style` o una variable CSS.

-----

## Errores comunes

### 1. Usar nombres de v3 en un proyecto v4

```html
<div class="bg-gradient-to-r from-red-500 to-yellow-400 bg-opacity-50"></div>
```

**Qué pasa:** el degradado no aparece y la opacidad no cambia.
**Por qué:** `bg-gradient-to-*` pasó a `bg-linear-to-*`, y las utilidades `*-opacity-*` fueron eliminadas.
**Arreglo:** `bg-linear-to-r from-red-500 to-yellow-400` y el modificador `/` en el color (`bg-black/50`).

### 2. Esperar que `shadow-sm` o `rounded-sm` se vean igual que en v3

**Qué pasa:** la sombra o el redondeo se ven más pequeños que en el tutorial.
**Por qué:** v4 desplazó la escala: el `shadow-sm` de v3 es ahora `shadow-xs`, y el `rounded-sm` de v3 es ahora `rounded-xs`.
**Arreglo:** usa el nombre que corresponde a la versión instalada (tabla de renombres arriba).

### 3. Bordes con el color del texto

```html
<div class="border p-4 text-slate-900">...</div>
```

**Qué pasa:** el borde sale oscuro, del mismo color que el texto, cuando esperabas un gris suave.
**Por qué:** en v4 el color de borde por defecto es `currentColor`.
**Arreglo:** indica el color siempre: `border border-slate-200`.

### 4. Armar clases con fragmentos dinámicos

```tsx
<div className={`bg-${color}-500`} />
```

**Qué pasa:** el elemento queda sin fondo.
**Por qué:** Tailwind solo genera CSS para las clases que aparecen completas en el código fuente.
**Arreglo:** un objeto que mapea cada valor posible a su clase completa (ejemplo de la sección "En React").

### 5. Confundir los números de la escala con píxeles

**Qué pasa:** `w-4` sale de `16px`, no de `4px`, y `p-13` no es `13px`.
**Por qué:** el número multiplica `--spacing` (`0.25rem`).
**Arreglo:** para píxeles exactos usa un valor arbitrario (`w-[4px]`); para el resto, piensa en múltiplos de `0.25rem`.

### 6. Texto ilegible por falta de contraste

**Qué pasa:** el texto casi no se distingue del fondo.
**Por qué:** se combinaron tonos de intensidad parecida (`bg-green-200 text-white`).
**Arreglo:** separa las intensidades (claro con oscuro) y revisa el contraste con una herramienta de accesibilidad.

### 7. Usar `h-screen` para una pantalla completa en móvil

**Qué pasa:** en algunos móviles el contenido queda cortado por debajo de la barra del navegador.
**Por qué:** `100vh` no descuenta la interfaz del navegador que aparece y desaparece.
**Arreglo:** usa `h-dvh` (altura dinámica del viewport).

-----

## Cuándo sí y cuándo no

**Usa las escalas de Tailwind** para casi todo: tamaños, espaciados, colores, radios y sombras. Es lo que mantiene coherente la interfaz.

**Usa valores arbitrarios (`w-[137px]`)** para casos puntuales que no se repiten, como una medida exacta de un diseño.

**Mueve el valor al tema (`@theme`)** cuando lo uses en varios lugares o represente una decisión de diseño, como el color de la marca o un ancho de sidebar.

**No abuses de los degradados ni de las sombras grandes.** Un par bien elegido se nota; muchos compiten entre sí. Prefiere sombras de la parte baja de la escala (`shadow-sm`, `shadow-md`) para elementos comunes.

**No confíes solo en el color** para comunicar estado (por ejemplo, error en rojo): suma un ícono o un texto para quien no distingue los colores.

-----

## Resumen en 5 líneas

1. Las clases siguen la lógica **prefijo (propiedad) + sufijo (valor de una escala)**: `text-lg`, `p-4`, `border-red-500`.
2. Margin (`m-`) es el espacio de afuera y padding (`p-`) el de adentro; los números multiplican `0.25rem`, no son píxeles.
3. Los colores usan `{utilidad}-{color}-{intensidad}` (50 a 950); la transparencia se escribe con `/` (`bg-black/50`).
4. Los degradados son `bg-linear-to-r` con `from-`, `via-` y `to-`; en v4 varias clases se renombraron (`shadow-xs`, `rounded-xs`, `bg-linear-*`).
5. Para casos fuera de la escala, valores arbitrarios; para colores propios y valores repetidos, `@theme`.

-----

## Para profundizar

<details>
<summary>Por qué v4 calcula el espaciado a partir de una sola variable</summary>

En v3 la escala de espaciado era una lista fija de pares nombre-valor en `tailwind.config.js`. En v4, `p-<número>`, `m-<número>`, `w-<número>` y similares se calculan como `calc(var(--spacing) * <número>)`. Consecuencias:

* Todo el sistema de espaciado se puede reescalar cambiando una sola variable en el tema (`@theme { --spacing: 0.3rem; }`).
* Las mismas reglas aplican a `margin`, `padding`, `gap`, `width`, `height`, `leading-<número>` y otras utilidades que usan la escala.
* Los valores derivados siguen la unidad que defina el tema: si `--spacing` fuera `1px`, `p-4` daría `4px`.

</details>

<details>
<summary>Tabla de renombres de v3 a v4 relacionados con esta lección</summary>

| En v3 | En v4 |
| --- | --- |
| `shadow-sm` | `shadow-xs` |
| `shadow` | `shadow-sm` |
| `rounded-sm` | `rounded-xs` |
| `rounded` | `rounded-sm` |
| `blur-sm` | `blur-xs` |
| `blur` | `blur-sm` |
| `ring` | `ring-3` |
| `bg-gradient-to-*` | `bg-linear-to-*` |
| `bg-opacity-*`, `text-opacity-*`, `border-opacity-*` | modificador `/` (`bg-black/50`) |
| `border` sin color (gris) | `border` sin color (`currentColor`) |

</details>

<details>
<summary>Los colores en OKLCH y la interpolación de gradientes</summary>

La paleta de v4 se define en OKLCH, un espacio de color basado en la percepción humana. Los degradados interpolan por defecto en `oklab`, otro espacio perceptual, en lugar de sRGB.

Se puede cambiar la interpolación con un modificador sobre la utilidad:

```html
<div class="bg-linear-to-r/srgb from-indigo-500 to-teal-400"></div>
<div class="bg-linear-to-r/oklch from-indigo-500 to-teal-400"></div>
```

Los ángulos con `bg-linear-<grados>` también interpolan en `oklab`. Para ángulos negativos se usa el prefijo `-bg-linear-`.

</details>

<details>
<summary>Sombras internas y anillos (ring)</summary>

Además de `shadow-*`, existen `inset-shadow-*` (sombra hacia adentro) y `ring-*` (un contorno hecho con `box-shadow`, sin afectar el layout). En v4, `ring` vale `1px`; para el grosor de v3 se usa `ring-3`. Un anillo se colorea con `ring-{color}`, por ejemplo `ring-blue-500`. Se usa mucho para indicar foco con `focus:ring-3`.

</details>

-----

## En entrevista

### Respuesta corta (junior)

En Tailwind cada clase controla una propiedad CSS y toma su valor de una escala: `text-lg` es tamaño de texto, `p-4` es padding y `bg-blue-500` es un color de fondo. El margin es el espacio de afuera y el padding el de adentro, y los números de la escala son múltiplos de `0.25rem`. Con ellas armas una tarjeta sin escribir CSS propio.

### Respuesta ampliada (semi-senior)

* **Sistema de escalas:** tamaños, espaciados, colores, radios y sombras salen del tema. Eso da consistencia y evita valores "casi iguales" repartidos por el proyecto.
* **Espaciado en v4:** las utilidades numéricas se calculan como `calc(var(--spacing) * n)`, con `--spacing: 0.25rem`. Se reescala todo cambiando esa variable.
* **Unidades relativas:** al usar `rem`, el diseño respeta el tamaño de fuente que configure el usuario. Los valores arbitrarios (`w-[137px]`) son válvula de escape; los valores repetidos van al tema.
* **Color:** paleta en OKLCH con intensidades 50 a 950; opacidad con el modificador `/`, que en v4 reemplazó a `bg-opacity-*`; colores de marca con `@theme` y variables `--color-*`.
* **Gradientes:** `bg-linear-to-*` más `from-`, `via-` y `to-`, con interpolación en `oklab` por defecto. Existen también `bg-radial` y `bg-conic`.
* **Cambios v3 a v4:** escala desplazada en sombras, blur y radios (`shadow-xs`, `rounded-xs`), borde por defecto en `currentColor`, `ring` de 1px y gradientes renombrados.
* **Clases dinámicas en React:** Tailwind lee el código como texto, así que las clases deben aparecer completas. Se mapean valores a clases en un objeto en lugar de interpolar fragmentos.
* **Accesibilidad:** el contraste entre texto y fondo debe cumplir los ratios WCAG; no se depende solo del color para comunicar estado.

### Preguntas frecuentes de seguimiento

**1. ¿`p-4` son 4 píxeles?**
No. Son 4 pasos de la escala, y cada paso vale `0.25rem`. `p-4` equivale a `1rem`, que son 16px con la fuente base habitual.

**2. ¿Cuál es la diferencia entre margin y padding?**
El padding está dentro del borde y separa el contenido del propio elemento; el margin está fuera y lo separa de sus vecinos. El fondo (`bg-`) cubre el padding pero no el margin.

**3. ¿Cómo aplicas transparencia a un color en v4?**
Con el modificador `/` sobre el color: `bg-black/50`. Las utilidades `bg-opacity-*` y similares de v3 fueron eliminadas.

**4. ¿Qué cambió en las sombras y esquinas de v3 a v4?**
Se desplazó la escala pequeña: `shadow-sm` pasó a `shadow-xs` y `shadow` a `shadow-sm`; con `rounded` ocurre lo mismo (`rounded-xs` y `rounded-sm`). También se agregó `shadow-2xs`.

**5. ¿Por qué `bg-${color}-500` no funciona?**
Porque Tailwind no ejecuta tu código: busca en los archivos nombres de clase completos como texto. Si la clase nunca aparece completa, su CSS no se genera. Se resuelve con un objeto que mapee cada valor a la clase completa.

**6. ¿Cuándo usas un valor arbitrario y cuándo el tema?**
Arbitrario para un valor puntual que no se repite. Si aparece en varios lugares o representa una decisión de diseño (color de marca), se define en `@theme` para que sea una utilidad más.

-----

## Siguiente lección

Ya sabes dar estilo a una caja y a su contenido. El siguiente paso es organizar varias cajas en la página y adaptarlas a distintos tamaños de pantalla: [Layout y responsive](05-Layout%20y%20Responsive.md).
