# Instalación y Compilación

## Cómo funciona Tailwind por dentro

Antes de instalar nada, conviene entender el modelo, porque explica casi todos los problemas que aparecen después: **Tailwind es una herramienta de build.** No se ejecuta en el navegador del usuario: lee tus archivos, detecta qué clases usaste, y genera un archivo CSS normal que contiene únicamente esas reglas.

```
tus archivos (HTML, JSX...)  →  Tailwind (build)  →  archivo CSS final
      clases escritas              analiza el texto      solo lo que usaste
```

Tailwind es, entonces, una **herramienta de desarrollo**: lo necesitás para *generar* el CSS, pero en producción el navegador solo recibe el CSS resultante, nunca Tailwind en sí. El tamaño de ese archivo final depende únicamente de cuántas clases distintas usaste.

Hay tres formas de usarlo, de la más rápida a la más completa.

-----

## Opción 1: Play CDN (solo para probar)

Para probar Tailwind sin instalar nada, alcanza con una etiqueta `<script>` en tu HTML:

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

Con eso ya podés usar clases de Tailwind en la página, y podés personalizar el tema con un bloque `<style type="text/tailwindcss">`:

```html
<style type="text/tailwindcss">
  @theme {
    --color-clifford: #da373d;
  }
</style>
```

Es ideal para experimentar, hacer una demo, o reproducir un problema en un playground. Pero la propia documentación de Tailwind es explícita: **el Play CDN está pensado solo para desarrollo, no para producción**, porque compila las clases en el navegador del usuario en cada carga.

-----

## Opción 2: la CLI de Tailwind (v4)

Para un proyecto HTML simple, sin framework, la CLI es el camino directo. El flujo tiene seis pasos:

**1. Crear el proyecto:**

```bash
npm init -y
```

**2. Instalar Tailwind y su CLI:**

```bash
npm install tailwindcss @tailwindcss/cli
```

**3. Crear el archivo CSS de entrada** (`src/input.css`) con una única línea:

```css
@import "tailwindcss";
```

**4. Compilar en modo watch:**

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./src/output.css --watch
```

**5. Enlazar el CSS generado desde el HTML:**

```html
<link href="./output.css" rel="stylesheet">
```

**6. Usar las clases:**

```html
<h1 class="text-3xl font-bold underline">Hola mundo</h1>
```

Fijate que en la versión 4 **no hace falta un archivo de configuración** ni indicarle a Tailwind en qué archivos buscar clases: lo detecta solo (lo vemos en la lección sobre cómo Tailwind detecta las clases).

-----

## Watch mode: por qué "no veo los estilos"

El problema más común al empezar es este: cambiás el HTML, agregás una clase nueva (por ejemplo `bg-pink-500`), guardás, y **no ves ningún cambio**. La causa es siempre la misma: esa clase todavía no existe en `output.css`, porque Tailwind todavía no volvió a generar el archivo después de tu cambio. Sin ninguna clase en el CSS generado, el navegador no tiene qué aplicar.

Por eso se usa la bandera `--watch`: Tailwind queda **escuchando** los cambios en tus archivos, y cada vez que guardás detecta las clases nuevas y actualiza `output.css` automáticamente.

Hay un detalle que suele confundir: **son dos procesos distintos que hacen dos trabajos distintos.**

| Herramienta | Qué hace |
| --- | --- |
| Tailwind (`--watch`) | Genera el CSS cuando cambian tus clases |
| Live Server (o el dev server de tu framework) | Recarga la página en el navegador cuando cambia un archivo |

Para trabajar con un proyecto HTML simple necesitás los dos corriendo a la vez (en dos terminales, o con el Live Server de tu editor más la terminal con Tailwind). Si falta uno, o el navegador no se refresca, o el CSS no se actualiza. La frase que resume todo: **si Tailwind no compila, no genera estilos.**

-----

## Opción 3: integrado con un framework

En un proyecto real con Vite, Next.js o Astro, no corrés la CLI a mano: Tailwind se integra con el sistema de build del framework, que ya compila y recarga por vos. Cada uno lo hace de una forma distinta, que vemos en la próxima lección.

-----

## Tailwind v3 y v4: cómo reconocer con cuál estás trabajando

Tailwind v4 se publicó en enero de 2025, y cambió bastante la instalación y la configuración. Es muy probable que te cruces con proyectos y tutoriales de la versión 3, así que conviene reconocer la diferencia. En v3, el flujo era:

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

Los cambios principales de v3 a v4, en una tabla:

| | Tailwind v3 | Tailwind v4 |
| --- | --- | --- |
| CLI | `npx tailwindcss` | `npx @tailwindcss/cli` (paquete aparte) |
| Plugin de PostCSS | el propio paquete `tailwindcss` | `@tailwindcss/postcss` (paquete aparte) |
| Activar en el CSS | tres directivas `@tailwind base/components/utilities` | una línea: `@import "tailwindcss"` |
| Archivo de configuración | `tailwind.config.js` obligatorio en la práctica | opcional; la configuración va en el CSS con `@theme` |
| Dónde buscar clases | opción `content` | detección automática |

Una regla rápida para saber qué versión tiene un proyecto: si el CSS tiene `@tailwind base;`, es v3; si tiene `@import "tailwindcss";`, es v4. También podés mirar la versión de `tailwindcss` en el `package.json`.

> **Compatibilidad:** Tailwind v4 usa características modernas de CSS y requiere navegadores relativamente recientes (Safari 16.4 o superior, Chrome 111 o superior, Firefox 128 o superior). Si necesitás soportar navegadores más viejos, la documentación oficial indica quedarse en v3.4.

-----

## Resumen

* Tailwind es una **herramienta de build**: lee tus clases y genera un CSS final con solo lo que usaste. En producción, solo viaja ese CSS.
* El **Play CDN** sirve para probar y prototipar, nunca para producción.
* Con la **CLI de v4**: `npm install tailwindcss @tailwindcss/cli`, un CSS con `@import "tailwindcss"`, y `npx @tailwindcss/cli -i ... -o ... --watch`.
* Si cambiás una clase y no ves el cambio, es porque el CSS no se regeneró: usá `--watch`. Tailwind (genera CSS) y Live Server (recarga la página) son dos procesos distintos.
* v3 y v4 tienen instalaciones distintas: `@tailwind base;` indica v3, `@import "tailwindcss";` indica v4.
