# Instalación e integración de Tailwind

## En una frase

Tailwind es una herramienta de build que lee tus archivos, detecta las clases que usaste y genera un CSS con solo esas reglas; se puede ejecutar de tres formas: en el navegador (Play CDN, solo para probar), con su CLI, o como plugin del build de tu framework (Vite, Next.js, Astro).

-----

## Antes de empezar

Necesitas:

* **Node.js** y **npm** instalados (`node -v` para comprobarlo). La documentación indica Node.js 20 o superior para la herramienta de migración de v4; Vite, Next.js y Astro tienen sus propios requisitos de versión.
* Una terminal y un editor de código.
* Saber qué es Tailwind y qué es una clase utilitaria: [Fundamentos de Tailwind](01-Fundamentos%20de%20Tailwind.md).

Palabras nuevas (más términos en el [Glosario](Glosario.md)):

* **Herramienta de build:** programa que transforma tus archivos fuente en archivos listos para el navegador, antes de que la app se ejecute.
* **CSS de entrada (input):** el archivo CSS que escribes tú. Contiene `@import "tailwindcss"`.
* **CSS de salida (output):** el archivo CSS que genera Tailwind. Es el que carga el navegador.
* **Watch mode (modo watch):** modo en que un programa queda ejecutándose y repite su trabajo cada vez que guardas un archivo.
* **Plugin:** pieza que se conecta al sistema de build de otra herramienta para agregarle una capacidad.
* **PostCSS:** herramienta que procesa CSS mediante plugins. Next.js la usa para transformar el CSS.
* **Servidor de desarrollo (dev server):** servidor local que sirve la app mientras la programas y recarga el navegador cuando guardas.

-----

## El problema

El navegador solo entiende CSS. Una clase como `bg-pink-500` no significa nada para él hasta que existe una regla CSS con ese nombre.

Hay dos formas ingenuas de resolverlo, y ambas fallan:

1. **Enviar un CSS con todas las clases posibles.** Tailwind ofrece miles de combinaciones de utilidades y valores; el archivo sería enorme.
2. **Escribir cada regla a mano.** Es justo el trabajo que Tailwind quiere quitarte.

La solución es un **paso de compilación**: una herramienta lee tu código, encuentra qué clases usaste y genera un CSS que contiene solo esas. Toda la instalación de Tailwind consiste en decidir **quién ejecuta ese paso**: el navegador, la CLI o tu framework.

-----

## Cómo funciona

### 1. Qué hace Tailwind por dentro

**Tailwind es una herramienta de build.** No se ejecuta en el navegador del usuario final. Lee tus archivos como texto, busca cadenas que coincidan con utilidades conocidas y genera un archivo CSS normal.

```
tus archivos (HTML, JSX...)  ->  Tailwind (build)  ->  archivo CSS final
      clases escritas              analiza el texto      solo lo que usaste
```

Consecuencias:

* **En producción solo viaja el CSS resultante**, nunca Tailwind. No hay costo de ejecución en el navegador.
* **El tamaño del CSS final depende de cuántas clases distintas usaste**, no de cuántas ofrece Tailwind.
* **Si Tailwind no compila, no hay estilos.** Casi todos los problemas de instalación se reducen a esto.

Cómo decide Tailwind qué archivos leer se explica en [Cómo Tailwind detecta las clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).

Las tres formas de ejecutar el build, de la más rápida a la más real:

```
Play CDN     compila en el navegador          solo para probar
CLI          compila desde la terminal        proyectos HTML simples
Plugin       compila dentro del build         proyectos reales (Vite, Next.js, Astro)
```

### 2. Play CDN: probar sin instalar nada

Basta una etiqueta `<script>` en el HTML:

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

Con eso ya puedes usar clases de Tailwind. Para personalizar el tema, se agrega un bloque `<style type="text/tailwindcss">`:

```html
<style type="text/tailwindcss">
  @theme {
    --color-clifford: #da373d;
  }
</style>

<h1 class="text-3xl font-bold underline text-clifford">Hola mundo</h1>
```

Sirve para experimentar, hacer una demo o reproducir un problema en un playground. La documentación oficial es explícita: **el Play CDN está pensado solo para desarrollo, no para producción**. Compila las clases en el navegador, en tiempo de ejecución, en lugar de entregar un CSS ya generado.

### 3. La CLI de Tailwind (v4)

Para un proyecto HTML sin framework, la CLI es el camino directo. Necesita cuatro piezas.

**Instalar Tailwind y su CLI** (en v4 la CLI es un paquete aparte):

```bash
npm install tailwindcss @tailwindcss/cli
```

**Un CSS de entrada** (por ejemplo, `src/input.css`) con una sola línea:

```css
@import "tailwindcss";
```

**El comando de compilación**, que lee el CSS de entrada, escanea tus archivos y escribe el CSS de salida. Con `--watch` queda escuchando cambios:

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./src/output.css --watch
```

**Un enlace al CSS generado** en el HTML, y las clases en los elementos:

```html
<link href="./output.css" rel="stylesheet">

<h1 class="text-3xl font-bold underline">Hola mundo</h1>
```

En v4 **no hace falta un archivo de configuración ni indicar en qué archivos buscar clases**: Tailwind los detecta automáticamente. La salida es un CSS listo para usar, que puedes minificar con las herramientas habituales.

### 4. Watch mode: por qué "no veo los estilos"

El problema más común al empezar: agregas una clase nueva (por ejemplo, `bg-pink-500`), guardas y **no ves ningún cambio**.

Causa: esa clase todavía no existe en `output.css`, porque Tailwind no volvió a generarlo después de tu cambio. Sin la regla en el CSS, el navegador no tiene qué aplicar.

`--watch` lo resuelve: Tailwind queda **escuchando** los archivos y, al guardar, detecta las clases nuevas y actualiza `output.css`.

Hay un detalle que suele confundir: **son dos procesos distintos que hacen dos trabajos distintos.**

| Herramienta | Qué hace |
| --- | --- |
| Tailwind (`--watch`) | Genera el CSS cuando cambian tus clases |
| Live Server (o el dev server de tu framework) | Recarga la página en el navegador cuando cambia un archivo |

En un proyecto HTML simple necesitas ambos a la vez (dos terminales, o el Live Server del editor más la terminal con Tailwind). Si falta uno, o el CSS no se actualiza, o el navegador no se refresca.

### 5. Integración con un framework

En un proyecto real ya existe un sistema de build que compila tu código y recarga el navegador al guardar. Ejecutar la CLI a mano, en paralelo, duplicaría ese trabajo.

Por eso Tailwind se conecta como **plugin del build**: compila y recarga dentro del mismo proceso de `npm run dev`. Los tres casos comparten la misma idea: **una línea en el CSS (`@import "tailwindcss"`) y un plugin en la configuración del build.**

```
npm run dev
    |
    v
[ build del framework ]  --plugin de Tailwind-->  CSS generado  -->  navegador
  (Vite / PostCSS)                                                    (recarga)
```

#### Vite (React, Vue, Svelte, etc.)

Vite tiene un plugin oficial de Tailwind, la forma recomendada de integrarlo:

```bash
npm install tailwindcss @tailwindcss/vite
```

Se agrega en `vite.config.ts`:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

Y en el CSS principal (por ejemplo, `src/index.css`):

```css
@import "tailwindcss";
```

La guía oficial de Vite también sirve como base para otros frameworks que usan Vite: por ejemplo Laravel, SvelteKit, React Router, Nuxt y SolidJS.

#### Astro

Astro usa Vite por debajo, así que la integración es la misma, dentro de la clave `vite` de la configuración de Astro:

```bash
npm install tailwindcss @tailwindcss/vite
```

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
});
```

```css
/* src/styles/global.css */
@import "tailwindcss";
```

Ese CSS se importa en las páginas o layouts que lo necesiten, dentro del bloque de código de un archivo `.astro`:

```astro
---
import "../styles/global.css";
---

<h1 class="text-3xl font-bold underline">Hola mundo</h1>
```

Un archivo `.astro` tiene dos partes separadas por `---`: arriba, un bloque opcional de JavaScript (imports, datos); debajo, el HTML de la página.

> **Sobre `npx astro add tailwind`:** aparece en muchos tutoriales y corresponde a la integración anterior (`@astrojs/tailwind`). La guía actual de Tailwind para Astro indica usar el plugin `@tailwindcss/vite`, no esa integración. Ante la duda, sigue la documentación oficial de Tailwind.

#### Next.js

Next.js no usa Vite: procesa el CSS con **PostCSS**. Por eso Tailwind se integra con otro paquete:

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

Se crea `postcss.config.mjs` en la raíz del proyecto:

```js
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
```

Y en el CSS global (`app/globals.css`):

```css
@import "tailwindcss";
```

Con eso puedes usar clases en cualquier componente:

```tsx
export default function Home() {
  return (
    <h1 className="text-3xl font-bold underline">
      Hola mundo
    </h1>
  );
}
```

#### Qué paquete va con cada herramienta

| Herramienta | Paquetes | Dónde se configura |
| --- | --- | --- |
| HTML plano (CLI) | `tailwindcss`, `@tailwindcss/cli` | Se ejecuta por línea de comandos |
| Vite (React, Vue...) | `tailwindcss`, `@tailwindcss/vite` | `vite.config.ts` |
| Astro | `tailwindcss`, `@tailwindcss/vite` | `astro.config.mjs` |
| Next.js | `tailwindcss`, `@tailwindcss/postcss`, `postcss` | `postcss.config.mjs` |

La regla para elegir: **si la herramienta se apoya en Vite, plugin de Vite; si procesa CSS con PostCSS, plugin de PostCSS.** Cuando puedes elegir, Tailwind recomienda el plugin de Vite, que además rinde mejor que el de PostCSS.

En todos los casos el CSS lleva `@import "tailwindcss"`, y los colores, fuentes y demás tokens se definen en el propio CSS con `@theme` (se ve en [Tema, modo oscuro y plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md)).

### 6. Cómo saber si un proyecto es v3 o v4

Tailwind v4 se publicó el 22 de enero de 2025 y cambió la instalación y la configuración. Es probable que encuentres proyectos y tutoriales de v3, así que conviene reconocer cuál tienes delante.

Así se instalaba y ejecutaba en v3:

```bash
npm install -D tailwindcss
npx tailwindcss init
```

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```js
// tailwind.config.js
module.exports = {
  content: ["./index.html"],
  // ...
};
```

```bash
npx tailwindcss -i ./input.css -o ./styles.css --watch
```

Diferencias principales:

| | Tailwind v3 | Tailwind v4 |
| --- | --- | --- |
| CLI | `npx tailwindcss` | `npx @tailwindcss/cli` (paquete aparte) |
| Plugin de PostCSS | el propio paquete `tailwindcss` | `@tailwindcss/postcss` (paquete aparte) |
| Plugin de Vite | no aplica (se usaba PostCSS) | `@tailwindcss/vite` (plugin oficial) |
| Activar en el CSS | tres directivas `@tailwind base/components/utilities` | una línea: `@import "tailwindcss"` |
| Archivo de configuración | `tailwind.config.js` | opcional; la configuración va en el CSS con `@theme` |
| Dónde buscar clases | opción `content` | detección automática |

Regla rápida para identificar la versión:

* El CSS tiene `@tailwind base;` -> **v3**.
* El CSS tiene `@import "tailwindcss";` -> **v4**.
* Confirmación: la versión de `tailwindcss` en `package.json`.

#### Si ves un `tailwind.config.js` o `tailwind.config.ts`

Es la forma de configurar v3. **En v4 ese archivo ya no se detecta automáticamente.** Si un proyecto v4 todavía lo necesita, se carga de forma explícita desde el CSS con la directiva `@config`:

```css
@config "../../tailwind.config.js";
```

La documentación describe `@config` como una vía para archivos de configuración heredados en JavaScript y su ejemplo usa `.js`; no documenta el uso de un archivo `.ts`. Además, las opciones `corePlugins`, `safelist` y `separator` de la configuración JavaScript no son compatibles en v4. Para proyectos nuevos, configura todo en el CSS.

> **Compatibilidad:** Tailwind v4 usa características modernas de CSS y requiere Safari 16.4 o superior, Chrome 111 o superior y Firefox 128 o superior. Si necesitas navegadores más antiguos, la documentación oficial indica quedarse en v3.4.

-----

## Ejemplo completo

Proyecto nuevo con Vite, React y TypeScript, con Tailwind v4:

```bash
npm create vite@latest mi-app -- --template react-ts
cd mi-app
npm install
npm install tailwindcss @tailwindcss/vite
```

`vite.config.ts`:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

`src/index.css` (reemplaza su contenido; la plantilla lo importa desde `main.tsx`):

```css
@import "tailwindcss";
```

`src/App.tsx`:

```tsx
function App() {
  return (
    <h1 className="text-3xl font-bold underline text-pink-500">
      Hola, Tailwind
    </h1>
  );
}

export default App;
```

```bash
npm run dev
```

Resultado: un encabezado grande, en negrita, subrayado y de color rosa. Si cambias `text-pink-500` por otra clase y guardas, el navegador se actualiza solo, porque Vite compila con el plugin y recarga en el mismo proceso.

Comprueba la versión con estas dos señales: `src/index.css` tiene `@import "tailwindcss"` (v4) y no existe ningún `tailwind.config`.

-----

## En React

* En JSX las clases van en **`className`**, no en `class`. Es una regla de JSX, no de Tailwind.
* El CSS con `@import "tailwindcss"` debe **importarse una vez desde el punto de entrada**: en Vite, normalmente `main.tsx` (`import './index.css'`); en Next.js con App Router, en la plantilla por defecto `app/layout.tsx` importa `globals.css`.
* Si el CSS no se importa desde ningún lado, el plugin no tiene nada que procesar y no habrá estilos.

-----

## Errores comunes

### 1. Cambio una clase y no veo el cambio

**Qué pasa:** guardas el archivo y la página no cambia.
**Por qué:** con la CLI, `output.css` no se regeneró (falta `--watch` o el proceso se detuvo), o el HTML no enlaza el CSS generado. También puede ser que el navegador no se recargó.
**Arreglo:** ejecuta la CLI con `--watch`, confirma que el HTML enlaza `output.css` y recuerda que Tailwind (genera CSS) y el servidor de recarga (actualiza el navegador) son procesos distintos.

### 2. Usar el Play CDN en producción

**Qué pasa:** el sitio funciona, pero compila el CSS en el navegador de cada visitante.
**Por qué:** el Play CDN existe para prototipar; la documentación indica que no es para producción.
**Arreglo:** migra a la CLI, al plugin de Vite o al de PostCSS según tu proyecto.

### 3. Mezclar comandos y paquetes de v3 y v4

**Qué pasa:** `npx tailwindcss init` no funciona, o Tailwind muestra un error al usar `tailwindcss` como plugin de PostCSS.
**Por qué:** en v4 la CLI y el plugin de PostCSS pasaron a paquetes aparte (`@tailwindcss/cli`, `@tailwindcss/postcss`), y el flujo de `init` y `tailwind.config.js` es de v3.
**Arreglo:** identifica la versión (`@tailwind base;` o `@import "tailwindcss";`) y sigue la guía oficial de esa versión.

### 4. Usar el paquete equivocado para el framework

**Qué pasa:** en Next.js, instalar `@tailwindcss/vite` no produce estilos.
**Por qué:** cada plugin se engancha a un sistema de build distinto, y Next.js no usa Vite.
**Arreglo:** Vite y Astro usan `@tailwindcss/vite`; Next.js usa `@tailwindcss/postcss` con `postcss.config.mjs`.

### 5. Olvidar importar el CSS

**Qué pasa:** el plugin está bien configurado, pero no hay estilos.
**Por qué:** el archivo con `@import "tailwindcss"` no se importa desde el punto de entrada.
**Arreglo:** importa ese CSS en `main.tsx` (Vite), `app/layout.tsx` (Next.js) o en la página o layout `.astro` (Astro).

### 6. Mi `tailwind.config` no hace nada

**Qué pasa:** cambias colores o rutas en `tailwind.config.js` o `.ts` y no se aplican.
**Por qué:** en v4 ese archivo no se detecta automáticamente.
**Arreglo:** mueve la configuración al CSS con `@theme`, o carga el archivo de forma explícita con `@config`.

-----

## Cuándo sí y cuándo no

| Situación | Opción |
| --- | --- |
| Probar una idea, una demo, reproducir un problema | **Play CDN** |
| Página HTML simple sin framework | **CLI** con `--watch` |
| React, Vue, Svelte u otro framework sobre Vite | **`@tailwindcss/vite`** |
| Astro | **`@tailwindcss/vite`** en `astro.config.mjs` |
| Next.js | **`@tailwindcss/postcss`** con `postcss.config.mjs` |
| Debes soportar navegadores anteriores a Safari 16.4, Chrome 111 o Firefox 128 | **Tailwind v3.4** (según la documentación oficial) |

-----

## Resumen en 5 líneas

1. Tailwind es una herramienta de build: lee tus clases y genera un CSS con solo lo que usaste; en producción viaja únicamente ese CSS.
2. El Play CDN sirve para probar; nunca para producción.
3. Con la CLI de v4: `npm install tailwindcss @tailwindcss/cli`, un CSS con `@import "tailwindcss"` y `npx @tailwindcss/cli -i ... -o ... --watch`.
4. En un framework, Tailwind es un plugin del build: `@tailwindcss/vite` para Vite y Astro, `@tailwindcss/postcss` para Next.js, y siempre `@import "tailwindcss"` en el CSS.
5. `@tailwind base;` indica v3 y `@import "tailwindcss";` indica v4; en v4 un `tailwind.config` solo se usa si lo cargas con `@config`.

-----

## Para profundizar

<details>
<summary>Ejecutable independiente de la CLI</summary>

La documentación de la CLI menciona que existe un **ejecutable independiente** (standalone), publicado en las releases de Tailwind en GitHub, como alternativa cuando no quieres depender de Node.js para compilar el CSS. La instalación con npm sigue siendo la vía principal.

</details>

<details>
<summary>Por qué el plugin de Vite y no PostCSS</summary>

El anuncio de Tailwind v4 presenta el plugin de Vite como una integración de primera parte con mejor rendimiento que la del plugin de PostCSS. Por eso, cuando el proyecto usa Vite, se recomienda `@tailwindcss/vite`. PostCSS sigue siendo la vía natural en herramientas que ya procesan el CSS con PostCSS, como Next.js. En v4, con el plugin de PostCSS ya no hace falta instalar `postcss-import` ni `autoprefixer`.

</details>

<details>
<summary>Migrar un proyecto de v3 a v4</summary>

La guía de actualización oficial ofrece una herramienta automática que actualiza dependencias, configuración y plantillas:

```bash
npx @tailwindcss/upgrade
```

Requiere Node.js 20 o superior. La documentación recomienda ejecutarla en una rama nueva y revisar los cambios antes de integrarlos: v4 renombró o eliminó algunas utilidades y cambió algunos valores por defecto.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Tailwind es una herramienta de build: lee mis archivos, detecta las clases que usé y genera un CSS con solo esas reglas. Para proyectos reales lo instalo como plugin del sistema de build: `@tailwindcss/vite` con Vite o Astro, y `@tailwindcss/postcss` con Next.js. En el CSS agrego `@import "tailwindcss"`. El Play CDN solo lo uso para probar, no para producción.

### Respuesta ampliada (semi-senior)

* **Modelo:** Tailwind compila en tiempo de build y no tiene costo de ejecución en el navegador. El CSS final contiene solo las utilidades detectadas, así que su tamaño depende del uso.
* **Formas de ejecutarlo:** Play CDN (compila en el navegador, solo desarrollo), CLI (`@tailwindcss/cli`, con `--watch`) y plugins de build (`@tailwindcss/vite`, `@tailwindcss/postcss`).
* **Elección del plugin:** depende de cómo procesa CSS la herramienta. Vite y Astro (que usa Vite) usan el plugin de Vite; Next.js usa PostCSS. El plugin de Vite es la opción recomendada y de mejor rendimiento cuando está disponible.
* **Dos procesos:** Tailwind genera CSS; el dev server o Live Server recarga el navegador. Con la CLI hacen falta ambos.
* **v3 vs v4:** v4 (enero de 2025) reemplaza las tres directivas `@tailwind` por `@import "tailwindcss"`, separa la CLI y el plugin de PostCSS en paquetes propios, detecta el contenido automáticamente y mueve la configuración al CSS con `@theme`.
* **Compatibilidad de configuración:** en v4 el `tailwind.config` no se detecta solo; se carga con `@config`, y `corePlugins`, `safelist` y `separator` no son compatibles.
* **Navegadores:** v4 exige Safari 16.4+, Chrome 111+ y Firefox 128+; para navegadores más antiguos, la documentación indica v3.4.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué Tailwind no se envía al navegador en producción?**
Porque es un paso de build: genera un CSS estático con las clases usadas. El navegador solo recibe ese CSS. El Play CDN es la excepción y por eso no se recomienda en producción.

**2. ¿Cómo sé si un proyecto usa v3 o v4?**
Miro el CSS: `@tailwind base;` es v3 y `@import "tailwindcss";` es v4. También reviso la versión de `tailwindcss` en `package.json`.

**3. ¿Qué diferencia hay entre `@tailwindcss/vite` y `@tailwindcss/postcss`?**
Ambos hacen lo mismo, pero se enganchan a sistemas de build distintos. El de Vite se usa con Vite y Astro; el de PostCSS con herramientas que procesan CSS con PostCSS, como Next.js. Cuando hay Vite, se recomienda el de Vite por rendimiento.

**4. ¿Por qué no veo cambios al agregar una clase con la CLI?**
Porque `output.css` no se regeneró. Hay que ejecutar la CLI con `--watch` y, además, tener algo que recargue el navegador. Son dos procesos distintos.

**5. ¿Qué pasa con un `tailwind.config.js` en un proyecto v4?**
No se detecta automáticamente. Se puede cargar de forma explícita con `@config` en el CSS, pero para proyectos nuevos se recomienda configurar con `@theme`.

**6. ¿Cuándo me quedaría en Tailwind v3?**
Cuando debo soportar navegadores anteriores a Safari 16.4, Chrome 111 o Firefox 128, ya que v4 usa características modernas de CSS. La documentación oficial recomienda v3.4 en ese caso.

-----

## Siguiente lección

Ya sabes instalar Tailwind y ejecutarlo. Ahora toca entender cómo decide qué clases incluir en el CSS y por qué algunas no aparecen: [Cómo Tailwind detecta las clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).
