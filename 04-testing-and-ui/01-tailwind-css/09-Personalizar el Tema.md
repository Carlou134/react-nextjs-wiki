# Personalizar el Tema

## Qué es el tema

Cuando usás `bg-blue-500`, `p-4`, `text-lg` o `md:`, todos esos valores salen de un mismo lugar: el **tema** de Tailwind, un conjunto de valores predefinidos (los colores, la escala de espaciado, los tamaños de texto, las fuentes, los breakpoints) al que también se lo llama **design tokens**.

Personalizar el tema significa agregar o modificar esos valores para que reflejen el diseño de **tu** proyecto, de manera que las clases de Tailwind hablen el idioma de tu marca: `bg-brand`, `font-heading`, `text-xxs`. Esto es lo que da consistencia: si los colores de tu marca viven en el tema, no aparecen `#3AB0FF` escritos a mano en cuarenta lugares.

La forma de hacerlo cambió con Tailwind v4, así que vemos primero la versión actual y después la anterior, que vas a encontrar en proyectos existentes.

-----

## Tailwind v4: el bloque `@theme` en el CSS

En v4 el tema se define directamente en el CSS, con la directiva `@theme`, usando **variables CSS con nombres especiales**. El prefijo de cada variable determina qué tipo de utilidad genera:

```css
@import "tailwindcss";

@theme {
  --color-brand: #007bff;
  --font-heading: "Ubuntu", sans-serif;
  --text-xxs: 0.65rem;
}
```

Con esas tres líneas, ya existen estas clases: `bg-brand`, `text-brand`, `border-brand` (por el prefijo `--color-`), `font-heading` (por `--font-`) y `text-xxs` (por `--text-`).

Los prefijos, o **namespaces**, más usados:

| Prefijo de la variable | Utilidades que genera |
| --- | --- |
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*`... |
| `--font-*` | `font-sans`, `font-heading`... (familias de fuente) |
| `--text-*` | `text-xs`, `text-xxs`... (tamaños de texto) |
| `--font-weight-*` | `font-bold`... |
| `--tracking-*` | `tracking-wide`... |
| `--leading-*` | `leading-tight`... |
| `--spacing-*` | `p-4`, `mt-8`, `max-h-16`... |
| `--breakpoint-*` | los prefijos responsive (`sm:`, `md:`...) |
| `--radius-*` | `rounded-sm`, `rounded-lg`... |
| `--shadow-*` | `shadow-md`, `shadow-lg`... |
| `--animate-*` | `animate-spin`... |

### Una paleta con variantes de un mismo color

```css
@theme {
  --color-brand: #007bff;
  --color-brand-light: #3ab0ff;
  --color-brand-dark: #0056b3;
}
```

```html
<button class="bg-brand hover:bg-brand-dark text-white">Enviar</button>
<div class="bg-brand-light"></div>
```

### Un espaciado propio

```css
@theme {
  --spacing-gutter: 1.5rem;
}
```

```html
<div class="p-gutter"></div>
```

### `@theme` no es lo mismo que `:root`

Una variable dentro de `:root` es una variable CSS común: existe, pero no genera ninguna clase de Tailwind. Una variable dentro de `@theme` hace las dos cosas: crea la variable CSS **y** las utilidades. Por eso, los valores que querés poder usar como clases (`bg-brand`) van en `@theme`, y los valores que solo necesitás como variable suelta van en `:root`.

-----

## Extender, sobrescribir o reemplazar el tema

Agregar una variable en `@theme` **suma** un valor a los que ya existen, sin quitar nada: `--font-script` agrega `font-script` y los `font-sans`, `font-serif` por defecto siguen funcionando.

Si en cambio querés **redefinir** un valor existente, se le da un nuevo valor con el mismo nombre:

```css
@theme {
  --breakpoint-sm: 30rem; /* ahora sm: se activa desde 30rem en vez de 40rem */
}
```

Y para **eliminar** todos los valores por defecto de una categoría y quedarte solo con los tuyos, se inicializa con `initial`:

```css
@theme {
  --color-*: initial;
  --color-white: #fff;
  --color-midnight: #121063;
}
```

Con esto, `bg-red-500` y el resto de la paleta por defecto dejan de existir: solo funcionan `white` y `midnight`. Y para partir completamente de cero, `--*: initial;` elimina todo el tema por defecto. Es una decisión fuerte, que tiene sentido cuando un sistema de diseño reemplaza por completo al de Tailwind.

-----

## Fuentes: una aclaración importante

Definir `--font-heading: "Ubuntu", sans-serif;` le dice a Tailwind **qué nombre usar**, pero no descarga la fuente. Para que el navegador pueda mostrar "Ubuntu", tenés que cargarla por separado (con un `<link>` a Google Fonts, un `@font-face`, o `next/font` en Next.js). Si la fuente no está cargada, el navegador usa la de respaldo (`sans-serif`) sin avisar.

### Combinar con `next/font`

Cuando la fuente se carga con `next/font` en Next.js, se expone como una variable CSS, y el tema tiene que **referenciar** esa variable. Para ese caso, `@theme inline` evita que la referencia se resuelva en el lugar equivocado:

```css
@theme inline {
  --font-sans: var(--font-inter);
}
```

Sin `inline`, la variable se resolvería donde está *definida* en lugar de donde se *usa*, y el valor de respaldo terminaría pisando a la fuente real. Es un detalle de las variables CSS, y por eso la documentación lo indica específicamente para las variables que apuntan a otras variables.

-----

## Tailwind v3: `tailwind.config.js`

En v3, la personalización vivía en un archivo de JavaScript, dentro de `theme.extend` (el `extend` es lo que **suma** valores; si escribías los valores directo en `theme`, reemplazabas la categoría entera):

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{html,js,jsx,tsx,astro}"],
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: "#007bff",
          light: "#3ab0ff",
          dark: "#0056b3",
        },
      },
      fontFamily: {
        heading: ["Ubuntu", "sans-serif"],
      },
      fontSize: {
        xxs: "0.65rem",
      },
    },
  },
};
```

Una sutileza de v3 que confunde: la clave especial **`DEFAULT`** (en mayúsculas) es la que genera la clase sin sufijo, `bg-brand`; así lo indica la documentación de v3. Esa clave es un nombre reservado con tratamiento especial, así que una clave escrita en minúsculas (`default`) no lo recibe: se comporta como cualquier otro nombre y generaría `bg-brand-default`. La equivalencia entre las dos versiones:

| Objetivo | Tailwind v3 (`tailwind.config.js`) | Tailwind v4 (CSS) |
| --- | --- | --- |
| Color de marca | `colors: { brand: { DEFAULT: "#007bff" } }` | `--color-brand: #007bff;` |
| Variante clara | `colors: { brand: { light: "..." } }` | `--color-brand-light: ...;` |
| Fuente propia | `fontFamily: { heading: [...] }` | `--font-heading: ...;` |
| Tamaño de texto | `fontSize: { xxs: "0.65rem" }` | `--text-xxs: 0.65rem;` |
| Dónde buscar clases | `content: [...]` | detección automática |

### ¿Puedo usar un config de v3 en un proyecto v4?

Sí, pero de forma explícita: v4 ya no lo detecta solo, hay que cargarlo desde el CSS con la directiva `@config`:

```css
@config "../../tailwind.config.js";
@import "tailwindcss";
```

Algunas opciones antiguas (`corePlugins`, `safelist` y `separator`) no son compatibles con v4. Para el safelist, el reemplazo es `@source inline()` (lo vemos en la lección sobre detección de clases).

-----

## Qué versión conviene usar

Una nota de tutoriales anteriores decía "no te cambies aún a v4, v3 es el estándar". Esa recomendación tenía sentido cuando v4 recién salía, pero **ya no aplica**: v4 es la versión actual, es la que muestra la documentación oficial y la que usan los proyectos nuevos. La única razón legítima para quedarte en v3 es un proyecto existente que ya la usa, o la necesidad de dar soporte a navegadores más viejos que los que v4 soporta.

Para aprender, conviene hacerlo con v4, sabiendo **leer** v3, porque vas a encontrar las dos en el mundo real.

-----

## Resumen

* El **tema** es el conjunto de valores (colores, fuentes, espaciados, breakpoints) de los que salen las clases de Tailwind; personalizarlo hace que las clases reflejen tu diseño.
* En **v4** se define en el CSS con `@theme`, con variables cuyo **prefijo** (`--color-*`, `--font-*`, `--text-*`...) determina qué clases se generan.
* Agregar variables **suma**; redefinir una variable la **sobrescribe**; `--color-*: initial` **elimina** los valores por defecto de esa categoría.
* Definir una fuente en el tema no la descarga: hay que cargarla aparte. `@theme inline` sirve cuando la variable apunta a otra (como con `next/font`).
* En **v3** todo esto vivía en `tailwind.config.js`, dentro de `theme.extend`; la clave `DEFAULT` genera la clase sin sufijo.
* Un config de v3 se puede cargar en v4 con `@config`, pero para proyectos nuevos se recomienda configurar en CSS.
