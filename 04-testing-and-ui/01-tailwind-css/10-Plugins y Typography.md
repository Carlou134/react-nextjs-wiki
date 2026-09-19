# Plugins y Typography

## Qué es un plugin de Tailwind

Un **plugin** agrega funcionalidad a Tailwind que no viene en el núcleo: nuevas utilidades, nuevos componentes, o estilos base. Los dos plugins oficiales más usados son **`@tailwindcss/typography`** (para dar estilo a texto largo) y **`@tailwindcss/forms`** (para normalizar los controles de formulario).

El flujo es el mismo con cualquier plugin: instalarlo, activarlo en la configuración, y usar sus clases.

-----

## Cómo se activa un plugin: v4 y v3

Se instala como dependencia de desarrollo:

```bash
npm install -D @tailwindcss/typography
```

Y la activación cambia entre versiones.

**En Tailwind v4**, se activa desde el propio CSS con la directiva `@plugin`:

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

**En Tailwind v3**, se agregaba al arreglo `plugins` del archivo de configuración:

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@tailwindcss/typography')],
};
```

> **Una confusión frecuente:** existe la idea de que "en v4 ya no hacen falta los plugins porque vienen integrados". No es correcto. Lo que cambió en v4 es **cómo se activan** (con `@plugin` en el CSS en lugar de un arreglo en el config), no si hacen falta. `@tailwindcss/typography` sigue siendo un paquete aparte que hay que instalar y activar explícitamente. Lo que sí quedó obsoleto es el plugin `@tailwindcss/aspect-ratio`, pero por otro motivo: sus utilidades (`aspect-video`, `aspect-square`) pasaron al núcleo del framework, como vimos en la lección de diseño responsive.

-----

## Typography: la clase `prose`

### El problema

Cuando el contenido viene de un lugar que no controlás —un artículo escrito en **Markdown**, el resultado de un CMS, un blog—, el HTML que recibís tiene etiquetas sin clases (`<h1>`, `<p>`, `<ul>`, `<table>`, `<code>`). Tailwind, por diseño, **resetea** los estilos por defecto del navegador, así que esas etiquetas se ven sin ningún formato: los títulos no se distinguen del texto, las listas pierden sus viñetas. Y no podés agregarles clases una por una, porque no escribiste ese HTML.

### La solución

El plugin Typography agrega la clase **`prose`**. Se aplica **una sola vez, en el contenedor padre**, y da estilo automáticamente a todo lo que esté adentro:

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

Con eso, los títulos tienen jerarquía, los párrafos tienen espaciado de lectura cómodo, las listas recuperan sus viñetas, y el código y las tablas tienen formato. Si no funciona, lo primero a revisar es que `prose` esté en el **contenedor** del contenido y no en un elemento suelto.

### Tamaños

`prose` tiene modificadores de tamaño que escalan toda la tipografía junta:

| Clase | Tamaño |
| --- | --- |
| `prose-sm` | chico |
| `prose-base` | el valor por defecto de `prose` |
| `prose-lg` | grande |
| `prose-xl` | extra grande |
| `prose-2xl` | el más grande |

Se combinan con los breakpoints igual que cualquier otra clase:

```html
<article class="prose md:prose-lg lg:prose-xl">
```

### Colores, modo oscuro y excepciones

* **Escala de grises:** `prose-slate`, `prose-zinc`, `prose-neutral` o `prose-stone` cambian el tono del texto. Siempre se usan junto con `prose`.
* **Modo oscuro:** `prose-invert` invierte los colores para fondos oscuros. Se combina con `dark:`: `class="prose dark:prose-invert"`.
* **Excluir un bloque:** `not-prose` saca a un elemento (y a sus hijos) del estilo de Typography, útil para insertar un componente propio dentro de un artículo sin que herede el formato de texto.

Además, existen **modificadores de elemento** para ajustar una etiqueta específica dentro de `prose` sin tocar el HTML, por ejemplo `prose-a:text-blue-600` para cambiar el color de los enlaces del contenido.

-----

## Forms: controles de formulario

Los controles de formulario (`<input>`, `<select>`, `<textarea>`, checkboxes) tienen estilos por defecto del navegador que son difíciles de sobrescribir de forma consistente entre navegadores. El plugin **`@tailwindcss/forms`** los normaliza con una base uniforme y sencilla, de modo que después puedas darles el aspecto que quieras usando utilidades comunes (`rounded-md`, `border-gray-300`, `focus:ring-2`...).

```css
@import "tailwindcss";
@plugin "@tailwindcss/forms";
```

Es importante entender qué **no** hace: no convierte los formularios en algo "bonito" automáticamente. Lo que hace es dejarlos en un punto de partida neutro y predecible. El diseño lo seguís armando vos con utilidades; muchas veces, copiando y adaptando un formulario ya hecho de alguna colección de componentes, que es un uso habitual (lo vemos en la próxima lección).

Una forma de comprobar que el plugin está activo: sacarlo y ver que los controles vuelven al aspecto del navegador, o dejarlo y ver que se ven todos iguales entre sí.

-----

## Si un plugin no funciona: lista de revisión

1. ¿Está **instalado** (`npm install -D ...`)? Aparece en el `package.json`.
2. ¿Está **activado**? En v4, la línea `@plugin "..."` en el CSS; en v3, dentro de `plugins: [...]` en `tailwind.config.js`.
3. ¿Reiniciaste el servidor de desarrollo? Cambios en la configuración de plugins suelen requerirlo.
4. ¿Estás en la versión correcta? Un tutorial de v3 (con `require(...)`) no funciona tal cual en un proyecto v4.
5. Para Typography: ¿`prose` está en el **contenedor padre** del contenido?

-----

## Resumen

* Un **plugin** agrega funcionalidad al núcleo: `@tailwindcss/typography` y `@tailwindcss/forms` son los oficiales más usados.
* En **v4** se activan con `@plugin "..."` en el CSS; en **v3**, con `plugins: [require(...)]` en el config. No es que "ya no hagan falta" en v4: cambia cómo se activan.
* `prose` (Typography) da estilo a HTML que no controlás, y se aplica **en el contenedor padre**; se ajusta con `prose-lg`, `prose-invert`, `not-prose` y modificadores de elemento.
* `@tailwindcss/forms` da una **base neutra** a los controles de formulario, no un diseño terminado.
* `@tailwindcss/aspect-ratio` quedó obsoleto porque `aspect-video` y afines son parte del núcleo.
