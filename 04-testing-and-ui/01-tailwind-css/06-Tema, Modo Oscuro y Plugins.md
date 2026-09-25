# Tema, modo oscuro y plugins

## En una frase

El **tema** es el conjunto de valores (colores, fuentes, espaciados, breakpoints) de los que salen las clases de Tailwind; en v4 se define en el CSS con `@theme`, el **modo oscuro** se controla con la variante `dark:` y los **plugins** agregan funcionalidad que el núcleo no trae, como los estilos de `prose` para texto largo.

-----

## Antes de empezar

Conviene que ya conozcas:

* Cómo se detectan y generan las clases: [Cómo Tailwind detecta las clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).
* Los colores y la escala tipográfica por defecto: [Tipografía, espaciado y colores](04-Tipograf%C3%ADa%2C%20Espaciado%20y%20Colores.md).
* Los prefijos responsive (`sm:`, `md:`), que también salen del tema: [Layout y responsive](05-Layout%20y%20Responsive.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Tema (theme):** el conjunto de valores predefinidos de los que Tailwind genera sus utilidades. También se llaman **design tokens**.
* **Variable de tema (theme variable):** una variable CSS declarada dentro de `@theme`. Su prefijo (`--color-`, `--font-`...) indica qué utilidades genera.
* **Namespace (espacio de nombres):** el prefijo de una variable de tema, como `--color-*` o `--breakpoint-*`.
* **Variante:** un prefijo que condiciona una utilidad (`hover:`, `md:`, `dark:`).
* **Plugin:** un paquete que agrega utilidades, componentes o estilos base a Tailwind.
* **Preflight:** la capa de estilos base de Tailwind. Resetea los estilos por defecto del navegador (títulos, listas, márgenes).
* **FOUC (flash of unstyled content):** el parpadeo que ocurre cuando la página se pinta con un estilo y después cambia a otro.

-----

## El problema

Las clases de Tailwind (`bg-blue-500`, `p-4`, `text-lg`, `md:`) salen de un tema por defecto. Ese tema no conoce tu marca. Si el azul de tu marca es `#007bff`, tienes dos caminos:

* Escribirlo a mano en cada lugar (`bg-[#007bff]`). Un cambio de marca obliga a buscar y reemplazar en todo el proyecto.
* Registrarlo una vez en el tema y usar `bg-brand`. Un cambio de marca toca una sola línea.

Aparecen dos necesidades más:

* **Modo oscuro:** cada elemento debe tener dos versiones de sus colores, y hay que decidir qué activa la versión oscura (la preferencia del sistema o un botón).
* **Contenido que no controlas:** el HTML que viene de Markdown o de un CMS no trae clases. Preflight lo deja sin formato visible: los títulos se ven como texto normal y las listas pierden sus viñetas. No puedes agregarle clases, porque no escribiste ese HTML.

Las tres necesidades se resuelven con el tema, la variante `dark:` y un plugin (Typography).

-----

## Cómo funciona

### El tema con `@theme` (Tailwind v4)

En v4 el tema se define en el propio CSS con la directiva `@theme`, usando variables CSS con nombres especiales. El prefijo de cada variable determina qué utilidades se generan:

```css
@import "tailwindcss";

@theme {
  --color-brand: #007bff;
  --font-heading: "Ubuntu", sans-serif;
  --text-xxs: 0.65rem;
}
```

Con esas tres líneas existen `bg-brand`, `text-brand` y `border-brand` (prefijo `--color-`), `font-heading` (prefijo `--font-`) y `text-xxs` (prefijo `--text-`).

Los namespaces más usados:

| Prefijo de la variable | Utilidades que genera |
| --- | --- |
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*`... |
| `--font-*` | `font-sans`, `font-heading`... (familias de fuente) |
| `--text-*` | `text-xs`, `text-xxs`... (tamaños de texto) |
| `--font-weight-*` | `font-bold`... |
| `--tracking-*` | `tracking-wide`... |
| `--leading-*` | `leading-tight`... |
| `--breakpoint-*` | las variantes responsive (`sm:`, `md:`...) |
| `--container-*` | variantes de container queries (`@sm:`) y tamaños como `max-w-md` |
| `--spacing-*` | utilidades de espaciado y tamaño (`px-4`, `max-h-16`...) |
| `--radius-*` | `rounded-sm`, `rounded-lg`... |
| `--shadow-*` | `shadow-md`, `shadow-lg`... |
| `--animate-*` | `animate-spin`... |

Sobre el espaciado: `--spacing` (sin sufijo) es un único valor base que Tailwind multiplica para generar la escala (`p-4` es cuatro veces ese valor). Además puedes registrar valores con nombre propio con `--spacing-*`.

#### Una paleta con variantes de un mismo color

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

#### Un espaciado con nombre

```css
@theme {
  --spacing-gutter: 1.5rem;
}
```

```html
<div class="p-gutter"></div>
```

#### `@theme` no es lo mismo que `:root`

Una variable dentro de `:root` es una variable CSS común: existe, pero no genera ninguna clase de Tailwind. Una variable dentro de `@theme` hace las dos cosas: crea la variable CSS y las utilidades. Los valores que quieres usar como clases (`bg-brand`) van en `@theme`. Los valores que solo necesitas como variable suelta van en `:root`.

Todas las variables de tema quedan disponibles como variables CSS normales, así que puedes usarlas en tu propio CSS:

```css
.card {
  color: var(--color-gray-700);
  border-radius: var(--radius-xl);
}
```

Por defecto, Tailwind incluye en el CSS final solo las variables que se usan. Si necesitas que salgan todas (por ejemplo, para leerlas desde JavaScript), se declara con `@theme static { ... }`.

### Extender, sobrescribir o reemplazar el tema

Hay tres operaciones distintas:

* **Extender.** Agregar una variable con un nombre nuevo suma un valor sin quitar nada. `--font-script: "Great Vibes", cursive;` agrega `font-script`, y `font-sans` y `font-serif` siguen funcionando.
* **Sobrescribir.** Redefinir una variable existente cambia su valor:

```css
@theme {
  --breakpoint-sm: 30rem; /* sm: se activa desde 30rem en vez de 40rem */
}
```

* **Reemplazar.** Inicializar un namespace con `initial` elimina todos sus valores por defecto:

```css
@theme {
  --color-*: initial;
  --color-white: #fff;
  --color-midnight: #121063;
}
```

Con esto, `bg-red-500` y el resto de la paleta por defecto dejan de existir: solo funcionan `white` y `midnight`. Para partir completamente de cero, `--*: initial;` elimina todo el tema por defecto. Es una decisión fuerte, que tiene sentido cuando un sistema de diseño reemplaza por completo al de Tailwind.

Un tema puede vivir en su propio archivo CSS y compartirse entre proyectos, porque es CSS común:

```css
/* app.css */
@import "tailwindcss";
@import "../brand/theme.css";
```

### Fuentes y `next/font`

Definir `--font-heading: "Ubuntu", sans-serif;` le dice a Tailwind qué nombre usar, pero no descarga la fuente. Para que el navegador muestre "Ubuntu" debes cargarla por separado: con un `<link>` a Google Fonts, un `@font-face` o `next/font` en Next.js. Si la fuente no está cargada, el navegador usa la de respaldo (`sans-serif`) sin avisar.

Cuando la fuente se carga con `next/font`, se expone como una variable CSS (por ejemplo `--font-inter`) y el tema debe referenciarla. Para ese caso se usa `@theme inline`:

```css
@theme inline {
  --font-sans: var(--font-inter);
}
```

Con `inline`, la utilidad generada usa el **valor** de la variable en lugar de referenciarla:

```css
.font-sans {
  font-family: var(--font-inter);
}
```

Sin `inline`, `var(--font-sans)` se resuelve en el elemento donde `--font-sans` está *definida*. Si `--font-inter` solo existe más abajo en el árbol, ahí no tiene valor, y el navegador termina usando el valor de respaldo en lugar de la fuente real. La documentación indica `@theme inline` para las variables de tema que apuntan a otras variables.

### Modo oscuro

#### La variante `dark:`

Para aplicar un estilo solo en modo oscuro se usa el prefijo `dark:`:

```html
<div class="bg-white dark:bg-gray-800">
  <h3 class="text-gray-900 dark:text-white">Título</h3>
  <p class="text-gray-500 dark:text-gray-400">Descripción</p>
</div>
```

Cada elemento define su versión clara (sin prefijo) y su versión oscura (con `dark:`). Por defecto, `dark:` se activa según la preferencia del sistema operativo, mediante la media query `prefers-color-scheme`. No necesitas hacer nada más.

#### Cambio manual con un botón

Si el usuario debe elegir el tema con independencia del sistema, hay que indicar que `dark:` se active por una clase en lugar de por la media query. Se hace redefiniendo la variante con `@custom-variant`:

```css
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));
```

Con esto, los estilos `dark:` se aplican cuando el elemento, o alguno de sus ancestros, tiene la clase `dark`. Lo habitual es poner o quitar esa clase en `<html>`:

```html
<html class="dark">
  <body>
    <div class="bg-white dark:bg-black">...</div>
  </body>
</html>
```

También puedes usar un atributo de datos en lugar de una clase:

```css
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

```html
<html data-theme="dark">
```

La documentación oficial propone un esquema de tres opciones (claro, oscuro y "seguir el sistema") con un script que corre al cargar la página. Debe ir en línea dentro del `<head>` para evitar el FOUC:

```js
document.documentElement.classList.toggle(
  "dark",
  localStorage.theme === "dark" ||
    (!("theme" in localStorage) && window.matchMedia("(prefers-color-scheme: dark)").matches),
);

// Cuando el usuario elige claro:
localStorage.theme = "light";
// Cuando elige oscuro:
localStorage.theme = "dark";
// Cuando elige seguir al sistema:
localStorage.removeItem("theme");
```

En Tailwind v3 el equivalente se configuraba en `tailwind.config.js` con `darkMode: 'class'`.

#### Conexión con React

En React, el tema es un estado. Ese estado tiene que hacer tres cosas: recordarse entre recargas, reflejarse como clase en `<html>` y estar disponible en toda la app. Cada una tiene su herramienta:

* `useState` con inicialización perezosa lee la preferencia guardada en `localStorage`.
* `useEffect` sincroniza la clase `dark` de `<html>` y guarda el valor cada vez que cambia.
* Context comparte el estado sin pasar props. Es exactamente el caso de la lección [React Context](../../01-react-core-and-typescript/04-hooks-y-context/04-React%20Context.md).

Tailwind no participa en nada de esto: solo reacciona a que `<html>` tenga o no la clase `dark`.

### Plugins

Un **plugin** agrega funcionalidad que no viene en el núcleo. Los dos oficiales más usados son `@tailwindcss/typography` (estilos para texto largo) y `@tailwindcss/forms` (base uniforme para controles de formulario). El flujo es siempre el mismo: instalar, activar y usar sus clases.

```bash
npm install -D @tailwindcss/typography
```

La activación cambia entre versiones.

**En Tailwind v4**, desde el CSS, con la directiva `@plugin` (acepta un nombre de paquete o una ruta local):

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

**En Tailwind v3**, en el arreglo `plugins` del archivo de configuración:

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@tailwindcss/typography')],
};
```

> **Una confusión frecuente:** "en v4 ya no hacen falta los plugins". No es correcto. Lo que cambió es cómo se activan (`@plugin` en el CSS en lugar de un arreglo en el config), no si hacen falta. `@tailwindcss/typography` sigue siendo un paquete aparte que hay que instalar y activar. El plugin `@tailwindcss/aspect-ratio` sí quedó obsoleto, pero desde Tailwind v3.0 (no v4): sus utilidades (`aspect-video`, `aspect-square`) pasaron al núcleo.

#### Typography: la clase `prose`

El plugin Typography agrega la clase `prose`. Se aplica **una sola vez, en el contenedor padre**, y da estilo a todo el HTML sin clases que tenga adentro:

```html
<article class="prose">
  <h1>Título del artículo</h1>
  <p>Un párrafo de texto...</p>
  <ul>
    <li>Un elemento de lista</li>
  </ul>
  <pre><code>const x = 1;</code></pre>
</article>
```

Los títulos recuperan jerarquía, los párrafos un espaciado de lectura cómodo, las listas sus viñetas, y el código y las tablas su formato.

**Tamaños.** Los modificadores de tamaño escalan toda la tipografía junta y siempre se usan junto con `prose`:

| Clase | Tamaño base del cuerpo |
| --- | --- |
| `prose-sm` | 14px |
| `prose-base` | 16px (valor por defecto de `prose`) |
| `prose-lg` | 18px |
| `prose-xl` | 20px |
| `prose-2xl` | 24px |

Se combinan con breakpoints como cualquier otra clase: `class="prose md:prose-lg lg:prose-xl"`.

**Colores.** `prose-gray` es el valor por defecto. `prose-slate`, `prose-zinc`, `prose-neutral` y `prose-stone` cambian la escala de grises, y también se usan junto con `prose`.

**Modo oscuro.** `prose-invert` invierte los colores para fondos oscuros y se combina con la variante: `class="prose dark:prose-invert"`.

**Excepciones.** `not-prose` saca un bloque (y sus hijos) del estilo de Typography. Sirve para insertar un componente propio dentro de un artículo sin que herede el formato de texto.

**Modificadores de elemento.** Ajustan una etiqueta concreta dentro de `prose` sin tocar el HTML: `prose-a:text-blue-600` cambia el color de los enlaces, y `prose-headings:` o `prose-img:` siguen el mismo patrón.

**Ancho.** `prose` incluye un ancho máximo para que las líneas sean legibles. Si el contenedor debe ocupar todo el ancho disponible, agrega `max-w-none`.

#### Forms: controles de formulario

Los controles de formulario (`<input>`, `<select>`, `<textarea>`, checkboxes) traen estilos del navegador difíciles de sobrescribir de forma consistente. `@tailwindcss/forms` los normaliza con una base uniforme, para que después puedas darles el aspecto que quieras con utilidades comunes (`rounded-md`, `border-gray-300`, `focus:ring-2`...).

```css
@import "tailwindcss";
@plugin "@tailwindcss/forms";
```

No convierte los formularios en algo "bonito": los deja en un punto de partida neutro y predecible. El diseño lo sigues armando con utilidades.

Por defecto usa la estrategia `base`, que aplica estilos globales a los elementos. La estrategia `class` genera solo clases (`form-input`, `form-select`, `form-checkbox`...) para aplicarlas donde las necesites. En v4 se elige dentro de la declaración del plugin:

```css
@plugin "@tailwindcss/forms" {
  strategy: "class";
}
```

### El tema en Tailwind v3 y cómo elegir versión

En v3 la personalización vivía en `tailwind.config.js`, dentro de `theme.extend`. El `extend` suma valores; si escribías los valores directo en `theme`, reemplazabas la categoría entera:

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

La clave **`DEFAULT`** (en mayúsculas) es la que genera la clase sin sufijo (`bg-brand`). Es un nombre reservado: escrita en minúsculas (`default`) se comporta como cualquier otra clave y generaría `bg-brand-default`.

Equivalencias entre versiones:

| Objetivo | Tailwind v3 (`tailwind.config.js`) | Tailwind v4 (CSS) |
| --- | --- | --- |
| Color de marca | `colors: { brand: { DEFAULT: "#007bff" } }` | `--color-brand: #007bff;` |
| Variante clara | `colors: { brand: { light: "..." } }` | `--color-brand-light: ...;` |
| Fuente propia | `fontFamily: { heading: [...] }` | `--font-heading: ...;` |
| Tamaño de texto | `fontSize: { xxs: "0.65rem" }` | `--text-xxs: 0.65rem;` |
| Modo oscuro por clase | `darkMode: 'class'` | `@custom-variant dark (...)` |
| Activar un plugin | `plugins: [require(...)]` | `@plugin "..."` |
| Dónde buscar clases | `content: [...]` | detección automática |

**Usar un config de v3 en un proyecto v4.** Es posible, pero de forma explícita: v4 ya no detecta el archivo solo. Hay que cargarlo desde el CSS con `@config`:

```css
@config "../../tailwind.config.js";
@import "tailwindcss";
```

Las opciones `corePlugins`, `safelist` y `separator` no son compatibles con v4. El reemplazo del safelist es `@source inline()`, que se vio en la lección de [detección de clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).

**Qué versión usar.** v4 es la versión actual, la que muestra la documentación oficial y la que corresponde a proyectos nuevos. Hay dos razones legítimas para quedarse en v3: un proyecto existente que ya la usa, o la necesidad de dar soporte a navegadores más viejos. v4 está diseñado para Safari 16.4+, Chrome 111+ y Firefox 128+; la documentación de migración indica quedarse en v3.4 si necesitas navegadores anteriores. Para aprender, conviene usar v4 y saber **leer** v3, porque ambas aparecen en proyectos reales.

-----

## Ejemplo completo

Una tarjeta de artículo con color de marca, tema claro y oscuro con clase en `<html>` y contenido Markdown estilizado con `prose`.

```css
/* app.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";

/* dark: se activa con la clase "dark" en <html> */
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-brand: #007bff;
  --color-brand-light: #3ab0ff;
  --color-brand-dark: #0056b3;
  --font-heading: "Ubuntu", sans-serif;
}
```

```html
<!-- index.html: aplica el tema antes de pintar para evitar el FOUC -->
<head>
  <script>
    const saved = localStorage.getItem('theme');
    const dark = saved ? saved === 'dark' : matchMedia('(prefers-color-scheme: dark)').matches;
    document.documentElement.classList.toggle('dark', dark);
  </script>
</head>
```

```tsx
// ThemeContext.tsx
import { createContext, useContext, useEffect, useMemo, useState } from 'react';

type Theme = 'light' | 'dark';

type ThemeContextType = {
  theme: Theme;
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

function getInitialTheme(): Theme {
  const saved = localStorage.getItem('theme');
  if (saved === 'light' || saved === 'dark') return saved;
  return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>(getInitialTheme);

  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark');
    localStorage.setItem('theme', theme);
  }, [theme]);

  const value = useMemo(
    () => ({
      theme,
      toggleTheme: () => setTheme((t) => (t === 'light' ? 'dark' : 'light')),
    }),
    [theme],
  );

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme debe usarse dentro de un ThemeProvider');
  }
  return context;
}
```

```tsx
// Article.tsx
import { useTheme } from './ThemeContext';

function ThemeButton() {
  const { theme, toggleTheme } = useTheme();
  return (
    <button
      onClick={toggleTheme}
      className="rounded-md bg-brand px-4 py-2 text-white hover:bg-brand-dark"
    >
      Tema: {theme}
    </button>
  );
}

export function Article({ html }: { html: string }) {
  return (
    <section className="bg-white p-6 dark:bg-gray-900">
      <h1 className="font-heading text-3xl text-gray-900 dark:text-white">Mi artículo</h1>
      <ThemeButton />
      {/* html viene de Markdown o de un CMS, sin clases */}
      <div className="prose dark:prose-invert" dangerouslySetInnerHTML={{ __html: html }} />
    </section>
  );
}
```

Qué ocurre:

1. `@theme` crea `bg-brand`, `hover:bg-brand-dark` y `font-heading`.
2. `@custom-variant` hace que `dark:` dependa de la clase `dark` en `<html>`.
3. El script del `<head>` fija esa clase antes del primer pintado. El `ThemeProvider` toma el relevo: lee el mismo valor y mantiene la clase y `localStorage` sincronizados.
4. `prose` da formato al HTML sin clases y `dark:prose-invert` invierte sus colores en modo oscuro.
5. `dangerouslySetInnerHTML` solo es seguro con HTML de una fuente de confianza o sanitizado. Si el contenido viene de usuarios, sanitízalo antes.

-----

## En React

* **Las clases de tema son estáticas.** `bg-brand` y `dark:bg-gray-900` se detectan como texto en el código, como cualquier clase. Lo único dinámico es la clase `dark` de `<html>`. No construyas nombres de clase concatenando (`` `bg-${color}` ``): Tailwind no los detecta.
* **El estado del tema es un caso típico de Context.** Lo leen muchos componentes (el botón, los gráficos, los iconos) y cambia con poca frecuencia. Ver [React Context](../../01-react-core-and-typescript/04-hooks-y-context/04-React%20Context.md).
* **Persistencia y SSR.** `localStorage` no existe en el servidor. En Next.js, leerlo dentro del inicializador de `useState` produce diferencias entre el HTML del servidor y el del cliente (error de hidratación). Lo habitual es aplicar la clase con un script en línea en el `<head>`, como el del ejemplo, y añadir `suppressHydrationWarning` a `<html>`, porque ese script modifica su clase antes de que React hidrate. Una librería como `next-themes` resuelve este patrón.
* **Fuentes en Next.js.** Se cargan con `next/font` y se conectan al tema con `@theme inline`, como se vio arriba.
* **Componentes con estilos de `prose`.** `prose` se aplica en el contenedor del contenido, no en cada elemento de React que lo compone.

-----

## Errores comunes

### 1. La clase del tema no existe (`bg-brand` no hace nada)

```css
:root {
  --color-brand: #007bff;   /* no genera bg-brand */
}
```

**Qué pasa:** la clase no se genera y el elemento queda sin color.
**Por qué:** una variable en `:root` es CSS común. Solo las variables dentro de `@theme` y con un prefijo de namespace (`--color-`, `--font-`...) generan utilidades. Un nombre sin prefijo válido tampoco genera nada.
**Arreglo:** declara la variable dentro de `@theme` y con el prefijo correcto (`--color-brand`).

### 2. Definir la fuente en el tema pero no cargarla

**Qué pasa:** `font-heading` se aplica, pero el texto se ve con la fuente de respaldo.
**Por qué:** `--font-heading` solo asigna un nombre. No descarga la fuente.
**Arreglo:** cárgala con `<link>`, `@font-face` o `next/font`. Si usas `next/font`, conecta la variable con `@theme inline`.

### 3. Perder toda la paleta sin querer

```css
@theme {
  --color-*: initial;
  --color-brand: #007bff;
}
```

**Qué pasa:** `bg-red-500`, `text-white` y demás colores dejan de funcionar.
**Por qué:** `--color-*: initial` elimina todos los colores por defecto.
**Arreglo:** para agregar un color, declara solo la variable nueva. Usa `initial` solo cuando quieras reemplazar la paleta completa, y en ese caso vuelve a declarar `--color-white`, `--color-black` y los que uses.

### 4. `dark:` no responde al botón

**Qué pasa:** el botón agrega o quita la clase `dark`, pero los estilos no cambian. O cambian solo cuando cambia el sistema operativo.
**Por qué:** sin `@custom-variant`, `dark:` sigue la media query `prefers-color-scheme` y ignora la clase. También falla si la clase se pone en un elemento que no es ancestro del contenido.
**Arreglo:** agrega `@custom-variant dark (&:where(.dark, .dark *));` y aplica la clase en `<html>`. En v3, `darkMode: 'class'`.

### 5. Parpadeo del tema al cargar la página

**Qué pasa:** la página se ve clara un instante y luego pasa a oscura.
**Por qué:** la clase `dark` se aplica desde un `useEffect`, que corre después del primer pintado.
**Arreglo:** fija la clase con un script en línea en el `<head>`, antes de que el navegador pinte.

### 6. Usar la configuración de v3 en un proyecto v4

**Qué pasa:** el `tailwind.config.js` no tiene efecto: los colores propios no existen.
**Por qué:** v4 ya no detecta ese archivo automáticamente.
**Arreglo:** migra el tema a `@theme` o carga el archivo con `@config`. Recuerda que `corePlugins`, `safelist` y `separator` no son compatibles.

### 7. `default` en minúsculas en un color de v3

**Qué pasa:** `bg-brand` no existe y aparece `bg-brand-default`.
**Por qué:** la clave reservada es `DEFAULT` en mayúsculas.
**Arreglo:** escribe `DEFAULT`.

### 8. Un plugin no funciona

Revisa en este orden:

1. ¿Está **instalado** (`npm install -D ...`)? Debe aparecer en el `package.json`.
2. ¿Está **activado**? En v4, la línea `@plugin "..."` en el CSS; en v3, dentro de `plugins: [...]` en `tailwind.config.js`.
3. ¿Reiniciaste el servidor de desarrollo? Los cambios en la configuración de plugins suelen requerirlo.
4. ¿Estás en la versión correcta? Un tutorial de v3 (con `require(...)`) no funciona tal cual en un proyecto v4.
5. Para Typography: ¿`prose` está en el **contenedor padre** del contenido y no en un elemento suelto?
6. Para Typography en modo oscuro: ¿agregaste `dark:prose-invert`?

### 9. El contenido de `prose` queda angosto

**Qué pasa:** el texto no ocupa todo el ancho de su contenedor.
**Por qué:** `prose` incluye un ancho máximo pensado para lectura.
**Arreglo:** agrega `max-w-none` al contenedor.

-----

## Cuándo sí y cuándo no

**Personaliza el tema cuando** un valor se repite o representa una decisión de diseño: colores de marca, fuentes, espaciados propios, breakpoints. Así un cambio de diseño toca un solo lugar.

**No lo personalices para:**

* Un valor que aparece una sola vez. Un valor arbitrario (`bg-[#007bff]`) es suficiente.
* Reemplazar todo el tema (`--*: initial`) sin tener un sistema de diseño propio que lo respalde.

**Usa `dark:` con la media query por defecto** cuando basta con respetar el sistema. **Usa `@custom-variant`** cuando el usuario debe poder elegir. Si necesitas ambas opciones, usa el esquema de tres opciones (claro, oscuro y sistema).

**Usa un plugin cuando** el problema es recurrente y lo resuelve un paquete oficial: `prose` para HTML que no controlas, `forms` para partir de una base uniforme. **No uses `prose`** para interfaces que armas tú con componentes y clases: ahí las utilidades directas son más claras.

-----

## Resumen en 5 líneas

1. En v4 el tema se define en el CSS con `@theme`; el prefijo de cada variable (`--color-*`, `--font-*`, `--text-*`...) determina qué clases se generan, y `:root` no genera ninguna.
2. Agregar una variable extiende, redefinirla sobrescribe y `--color-*: initial` reemplaza el namespace; una fuente definida en el tema hay que cargarla aparte (`@theme inline` con `next/font`).
3. `dark:` sigue por defecto la preferencia del sistema; para un botón manual se redefine con `@custom-variant dark` y se alterna la clase `dark` en `<html>`.
4. En React, el tema es un estado (`useState` + `useEffect` + Context) que solo alterna esa clase; un script en el `<head>` evita el parpadeo.
5. Los plugins se instalan y se activan con `@plugin` (v4) o `plugins: [...]` (v3); `prose` va en el contenedor padre y `forms` da una base neutra.

-----

## Para profundizar

<details>
<summary>Variantes personalizadas con `@custom-variant`</summary>

`@custom-variant` no sirve solo para el modo oscuro. Permite crear cualquier variante:

```css
@custom-variant theme-midnight (&:where([data-theme="midnight"] *));
```

Con eso puedes escribir `theme-midnight:bg-black` y `theme-midnight:text-white`. Es la base para temas múltiples (claro, oscuro, sepia) controlados por un atributo `data-theme` en `<html>`.

</details>

<details>
<summary>Animaciones propias en `@theme`</summary>

Una variable `--animate-*` puede acompañarse de su `@keyframes` dentro del mismo bloque:

```css
@theme {
  --animate-fade-in-scale: fade-in-scale 0.3s ease-out;

  @keyframes fade-in-scale {
    0% { opacity: 0; transform: scale(0.95); }
    100% { opacity: 1; transform: scale(1); }
  }
}
```

Esto genera `animate-fade-in-scale`. Si quieres que el `@keyframes` se incluya siempre, incluso sin la variable `--animate-*`, defínelo fuera de `@theme`.

</details>

<details>
<summary>Personalizar Typography en v4</summary>

Los colores de `prose` se controlan con variables CSS (`--tw-prose-body` y similares). En v4, un tema propio se define con `@utility`:

```css
@utility prose-pink {
  --tw-prose-body: var(--color-pink-800);
  --tw-prose-headings: var(--color-pink-900);
}
```

Se usa como `class="prose prose-pink"`. Consulta el README del plugin para la lista completa de variables.

</details>

<details>
<summary>Referencia rápida: qué cambió entre v3 y v4 en este tema</summary>

* Config: `tailwind.config.js` con `theme.extend` pasó a `@theme` en el CSS.
* Detección de contenido: `content: [...]` pasó a detección automática, con `@source` para casos especiales.
* Modo oscuro: `darkMode: 'class'` pasó a `@custom-variant dark`.
* Plugins: `plugins: [require(...)]` pasó a `@plugin "..."`.
* Config antiguo: se carga con `@config`, pero `corePlugins`, `safelist` y `separator` no son compatibles.
* Navegadores: v4 apunta a Safari 16.4+, Chrome 111+ y Firefox 128+.

</details>

-----

## En entrevista

### Respuesta corta (junior)

En Tailwind v4 el tema se define en el CSS con `@theme`: cada variable (`--color-brand`) genera sus utilidades (`bg-brand`). El modo oscuro usa la variante `dark:`, que por defecto sigue la preferencia del sistema; para un botón manual se redefine con `@custom-variant` y se alterna una clase `dark` en `<html>`. Los plugins como Typography (`prose`) agregan estilos para contenido sin clases y se activan con `@plugin`.

### Respuesta ampliada (semi-senior)

* **Tema:** las variables de `@theme` son variables CSS que además le indican a Tailwind qué utilidades generar, según su namespace. Las de `:root` no generan clases. Se extiende con nombres nuevos, se sobrescribe repitiendo el nombre y se reemplaza con `--color-*: initial` o `--*: initial`.
* **Referencias entre variables:** `@theme inline` hace que la utilidad use el valor de la variable y no una referencia. Evita que un `var()` se resuelva en el ámbito equivocado, típico con `next/font`.
* **Modo oscuro:** por defecto usa `prefers-color-scheme`. Con `@custom-variant dark (&:where(.dark, .dark *))` pasa a depender de una clase. En React se maneja con estado, Context y un efecto que alterna la clase en `<html>`, más un script en línea para evitar el FOUC y los problemas de hidratación en SSR.
* **Plugins:** siguen siendo paquetes aparte. En v4 se activan con `@plugin` en el CSS; en v3, con `plugins` en el config. Typography (`prose`) estiliza HTML que no controlas; Forms deja una base neutra en los controles.
* **Migración:** v4 no detecta `tailwind.config.js`; se carga con `@config`, salvo `corePlugins`, `safelist` y `separator`. v4 apunta a navegadores modernos, y v3.4 sigue siendo la opción si hay que soportar navegadores antiguos.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre `@theme` y `:root`?**
Ambos crean variables CSS, pero solo las de `@theme` con un prefijo válido generan utilidades. `:root` es para variables que no necesitan clase.

**2. ¿Cómo agrego un color de marca en v4?**
Declaro `--color-brand: #007bff;` dentro de `@theme`. Eso genera `bg-brand`, `text-brand`, `border-brand` y el resto de utilidades de color.

**3. ¿Cómo implementas un botón de modo oscuro?**
Redefino la variante con `@custom-variant dark (&:where(.dark, .dark *))` y alterno la clase `dark` en `<html>` desde un estado de React. Guardo la preferencia en `localStorage` y aplico la clase con un script en el `<head>` para evitar el parpadeo.

**4. ¿Para qué sirve `@theme inline`?**
Para variables de tema que apuntan a otras variables, como `--font-sans: var(--font-inter)`. Hace que la utilidad use directamente `var(--font-inter)` y no dependa del ámbito donde se resuelve `--font-sans`.

**5. ¿Los plugins siguen haciendo falta en v4?**
Sí. Lo que cambió es cómo se activan: `@plugin "@tailwindcss/typography"` en el CSS en lugar de `require(...)` en el config.

**6. ¿Cómo estilizas HTML que viene de Markdown o de un CMS?**
Con el plugin Typography: aplico `prose` al contenedor, ajusto tamaños con `prose-lg`, modo oscuro con `dark:prose-invert` y excluyo bloques con `not-prose`.

-----

## Siguiente lección

Con el tema, el modo oscuro y los plugins resueltos, queda ordenar el trabajo: cuándo extraer componentes, cómo evitar clases repetidas y cómo organizar el flujo diario: [Buenas prácticas y flujo de trabajo](07-Buenas%20Pr%C3%A1cticas%20y%20Flujo%20de%20Trabajo.md).
