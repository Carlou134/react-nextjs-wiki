# Cómo Tailwind Detecta las Clases

## Tailwind lee texto, no ejecuta código

Esta es una de las lecciones más importantes de todo el módulo, porque explica un error que rompe interfaces en producción y que en desarrollo puede pasar desapercibido.

Tailwind **no ejecuta tu código JavaScript**. Para saber qué clases generar, escanea tus archivos como **texto plano**, buscando cadenas que coincidan con el nombre de una clase que existe. Si encuentra `bg-blue-500` escrito tal cual en algún archivo, genera esa regla en el CSS final. Si no la encuentra escrita de forma completa, esa clase no existe para Tailwind.

La frase que lo resume: **si Tailwind no puede verlo como texto, no existe para él.**

-----

## El error clásico: construir clases dinámicamente

Mirá este componente de React:

```tsx
function Button({ color, children }) {
  return (
    <button className={`bg-${color}-600 hover:bg-${color}-500`}>
      {children}
    </button>
  );
}
```

Para vos, cuando `color` vale `"blue"`, el resultado es `bg-blue-600`. Pero Tailwind nunca ejecuta ese código: lo que ve al escanear el archivo es el texto `` `bg-${color}-600` ``, que no es una clase válida. Como consecuencia, **`bg-blue-600` no se genera en el CSS**, y el botón queda sin estilo. Lo peor es que, según cómo trabajes en desarrollo, a veces puede parecer que funciona (por ejemplo, si esa clase se usó literalmente en otro archivo) y romperse recién en producción.

Lo mismo ocurre con cualquier concatenación:

```js
"bg-" + color             // Tailwind ve: "bg-" + color
`text-${size}`            // Tailwind ve: text-${size}
```

-----

## La solución: clases completas escritas literalmente

La regla es que **cada clase que necesites tiene que aparecer completa, como texto, en algún archivo de tu proyecto.** La forma de lograrlo con valores dinámicos es un **mapa de clases**: un objeto donde cada opción está escrita completa.

```tsx
const colorVariants = {
  blue: "bg-blue-600 hover:bg-blue-500",
  red: "bg-red-600 hover:bg-red-500",
};

function Button({ color, children }) {
  return <button className={colorVariants[color]}>{children}</button>;
}
```

Ahora Tailwind ve, al escanear el archivo, los textos completos `"bg-blue-600 hover:bg-blue-500"` y `"bg-red-600 hover:bg-red-500"`, y genera todas esas clases. En tiempo de ejecución, `colorVariants[color]` simplemente elige cuál de las dos cadenas ya existentes usar.

Un condicional común también funciona por la misma razón:

```tsx
<div className={isActive ? "bg-blue-500" : "bg-gray-500"}>
```

Las dos clases, `"bg-blue-500"` y `"bg-gray-500"`, están escritas completas en el código, aunque cuál se aplica se decida en tiempo de ejecución.

> **En TypeScript:** el mapa de clases se combina muy bien con los tipos. Con `as const` y `keyof typeof`, las opciones válidas de la prop quedan derivadas del propio objeto, así que agregar un color nuevo es agregar una línea al mapa, y TypeScript te avisa si en algún lado pasás un color que no existe:
>
> ```tsx
> const colorVariants = {
>   blue: "bg-blue-600 hover:bg-blue-500",
>   red: "bg-red-600 hover:bg-red-500",
> } as const;
>
> type Color = keyof typeof colorVariants; // "blue" | "red"
>
> function Button({ color, children }: { color: Color; children: React.ReactNode }) {
>   return <button className={colorVariants[color]}>{children}</button>;
> }
> ```

-----

## Qué archivos escanea Tailwind (v4)

En Tailwind v4 la detección es **automática**: no hace falta indicarle dónde buscar. Escanea todos los archivos de tu proyecto, con estas excepciones:

* los archivos que estén en tu `.gitignore`,
* la carpeta `node_modules`,
* archivos binarios (imágenes, videos, etc.),
* archivos CSS y archivos de lock.

Esto tiene una consecuencia práctica que conviene tener presente: **si una clase vive únicamente en un archivo ignorado por git, no se detecta.** Y también: las clases que aparecen dentro de una librería de componentes instalada en `node_modules` (por ejemplo, una librería de UI que usa Tailwind) **no se detectan por defecto**.

-----

## Agregar fuentes de búsqueda: `@source`

Para esos casos, la directiva `@source` le indica a Tailwind dónde más mirar:

```css
@import "tailwindcss";
@source "../node_modules/@acmecorp/ui-lib";
```

También podés hacer lo contrario, excluir una ruta que no querés escanear:

```css
@source not "../src/components/legacy";
```

O tomar el control total, desactivando la detección automática y declarando vos las carpetas exactas:

```css
@import "tailwindcss" source(none);
@source "../admin";
@source "../shared";
```

-----

## Forzar una clase que no aparece en el código: safelist

A veces necesitás que exista una clase que **no está escrita en ningún archivo**: por ejemplo, si el nombre de la clase viene de una base de datos o de un CMS. Para eso, `@source inline()` genera esas clases directamente:

```css
@source inline("underline");
```

Acepta llaves para generar variantes y rangos sin escribir una por una:

```css
@source inline("{hover:,}bg-red-{50,{100..900..100},950}");
```

Esa línea genera `bg-red-50`, `bg-red-100`, `bg-red-200`... hasta `bg-red-950`, y también sus versiones con `hover:`. Es una herramienta de último recurso: si podés resolver el caso con un mapa de clases, es preferible, porque deja el código legible y buscable.

-----

## Resumen

* Tailwind escanea tus archivos como **texto plano**; no ejecuta JavaScript.
* Las clases construidas dinámicamente (`` `bg-${color}-600` ``) **no se generan**. Cada clase tiene que estar escrita completa en algún lugar.
* La solución estándar es un **mapa de clases**, con las opciones escritas completas; los condicionales con clases literales también funcionan.
* En v4 la detección es automática, pero ignora archivos de `.gitignore`, `node_modules`, binarios y CSS.
* `@source` agrega o excluye rutas de búsqueda, y `@source inline()` fuerza la generación de clases que no aparecen en ningún archivo.
