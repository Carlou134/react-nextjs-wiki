# Integración con Vite, Astro y Next.js

## Por qué se integra en lugar de usar la CLI

En un proyecto real, ya existe un sistema de build que compila tu código y recarga el navegador cuando guardás (Vite, Next.js, Astro). Correr la CLI de Tailwind a mano, en paralelo, sería duplicar ese trabajo. En cambio, Tailwind se conecta como un **plugin** de ese mismo sistema de build, de modo que compila el CSS, y lo recarga, dentro del mismo proceso que ya tenías corriendo con `npm run dev`.

Cada herramienta lo hace de forma distinta, y los tres casos comparten la misma idea: una línea en el CSS (`@import "tailwindcss"`) y un plugin en la configuración del build.

-----

## Vite (React, Vue, etc.)

Vite tiene un plugin oficial de Tailwind, que es la forma recomendada de integrarlo:

```bash
npm install tailwindcss @tailwindcss/vite
```

Se agrega el plugin en `vite.config.ts`:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

Y en el archivo CSS principal (por ejemplo `src/index.css`), la línea de importación:

```css
@import "tailwindcss";
```

Con eso, cualquier componente de React puede usar clases de Tailwind en su `className`. Es la configuración que corresponde a proyectos creados con Vite, como el de los ejercicios de práctica.

-----

## Astro

Astro también usa Vite por debajo, así que la integración es la misma que la anterior, dentro de la configuración de Astro. Los pasos de la guía oficial actual son:

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

Y se importa ese CSS en las páginas o layouts donde se necesite, dentro del bloque de código de un archivo `.astro`:

```astro
---
import "../styles/global.css";
---

<h1 class="text-3xl font-bold underline">Hola mundo</h1>
```

Un archivo `.astro` tiene dos partes separadas por `---`: arriba, un bloque de JavaScript opcional (imports, datos), y debajo, el HTML de la página. Para el estilo, alcanza con escribir clases de Tailwind en ese HTML.

> **Nota sobre tutoriales anteriores:** en muchos tutoriales vas a ver el comando `npx astro add tailwind`. Es una integración anterior, ligada a Tailwind v3. La guía actual de Tailwind para Astro ya no la menciona y usa el plugin de Vite que se muestra arriba, así que ante la duda seguí la documentación oficial de Tailwind.

-----

## Next.js

Next.js no usa Vite, sino **PostCSS** para procesar el CSS, y por eso Tailwind se integra con un paquete distinto:

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

Se crea un archivo `postcss.config.mjs` en la raíz del proyecto:

```js
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
```

Y en el CSS global (`app/globals.css`), la importación:

```css
@import "tailwindcss";
```

Con eso ya se pueden usar las clases en cualquier componente:

```tsx
export default function Home() {
  return (
    <h1 className="text-3xl font-bold underline">
      Hola mundo
    </h1>
  );
}
```

Igual que en Vite y Astro, no hace falta un archivo `tailwind.config` para este setup básico.

-----

## Qué paquete va con cada herramienta

| Herramienta | Paquetes | Dónde se configura |
| --- | --- | --- |
| HTML plano (CLI) | `tailwindcss`, `@tailwindcss/cli` | se corre por línea de comandos |
| Vite (React, Vue...) | `tailwindcss`, `@tailwindcss/vite` | `vite.config.ts` |
| Astro | `tailwindcss`, `@tailwindcss/vite` | `astro.config.mjs` |
| Next.js | `tailwindcss`, `@tailwindcss/postcss`, `postcss` | `postcss.config.mjs` |

En todos los casos, el CSS lleva `@import "tailwindcss"`, y la configuración de estilo (colores, fuentes) se hace en el propio CSS, como vemos en la lección de personalización del tema.

-----

## Si ves un `tailwind.config.ts` en un proyecto

En proyectos creados hace tiempo (o con tutoriales de v3) es normal encontrar un archivo `tailwind.config.js` o `tailwind.config.ts`. Es la forma de configurar Tailwind v3. Si el proyecto está en v4 y todavía necesitás ese archivo, se puede cargar explícitamente desde el CSS con la directiva `@config`, pero para proyectos nuevos la recomendación es configurar todo directamente en el CSS.

-----

## Resumen

* En un framework, Tailwind se integra como **plugin del build**: no corrés la CLI aparte.
* **Vite y Astro** usan `@tailwindcss/vite`; **Next.js** usa `@tailwindcss/postcss`.
* En los tres casos el CSS necesita una sola línea, `@import "tailwindcss"`, y no hace falta un `tailwind.config` para empezar.
* `npx astro add tailwind` es un camino de tutoriales anteriores; la guía actual usa el plugin de Vite.
* En JSX, las clases van en `className`, no en `class`.
