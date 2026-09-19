# Colores, Gradientes y Modo Oscuro

## La forma de un color en Tailwind

Todos los colores de la paleta por defecto siguen el mismo patrón: `{utilidad}-{color}-{intensidad}`.

```html
<p class="text-blue-500">Texto azul</p>
<div class="bg-green-200">Fondo verde claro</div>
<div class="border border-red-600">Borde rojo oscuro</div>
```

La **utilidad** dice a qué propiedad se aplica el color (`text-` para el texto, `bg-` para el fondo, `border-` para el borde, entre otras), el **color** es el nombre (`blue`, `red`, `green`, `pink`...) y la **intensidad** es un número que va de `50` (muy claro) a `950` (muy oscuro), con `500` como tono medio.

| Intensidad | Se ve como |
| --- | --- |
| `50`, `100` | casi blanco, tinte muy suave |
| `500` | el color "puro", tono medio |
| `900`, `950` | casi negro, tinte oscuro |

### Transparencia: el modificador `/`

Para aplicar un color con transparencia, se agrega una barra y el porcentaje de opacidad directamente al color:

```html
<div class="bg-black/50">Fondo negro al 50% de opacidad</div>
```

En Tailwind v3 existían utilidades separadas para esto (`bg-opacity-50`); en v4 esas fueron eliminadas y el modificador `/` es la única forma.

### Cuidado con el contraste

Combinar cualquier color de texto con cualquier color de fondo es fácil de hacer, pero no siempre legible. Por ejemplo, `bg-green-200 text-white` es texto blanco sobre un verde muy claro: casi no se lee. Una regla práctica: fondos claros (`50`-`200`) con textos oscuros (`800`-`950`), y fondos oscuros (`700`-`950`) con textos claros (`50`-`200`). Es un tema de accesibilidad, no solo de estética: hay ratios de contraste mínimos recomendados para que el texto sea legible para todas las personas.

-----

## Gradientes

Un degradado se arma combinando una **dirección** con hasta tres **paradas de color**:

```html
<div class="bg-linear-to-r from-purple-400 via-pink-500 to-red-500 text-white p-4">
  Degradado
</div>
```

* `bg-linear-to-r`: la dirección (de izquierda a derecha). Las otras direcciones siguen el mismo patrón: `-t` (arriba), `-b` (abajo), `-l` (izquierda), `-tr` (arriba a la derecha), y así.
* `from-`: el color inicial.
* `via-`: un color intermedio (opcional).
* `to-`: el color final.

También se puede indicar un ángulo exacto (`bg-linear-65`) y controlar en qué punto empieza cada color con porcentajes:

```html
<div class="bg-linear-to-r from-indigo-500 from-10% via-sky-500 via-30% to-emerald-500 to-90%"></div>
```

> **Diferencia entre v3 y v4:** en Tailwind v3 el degradado se escribía `bg-gradient-to-r`. En v4 el nombre pasó a ser `bg-linear-to-r` (porque ahora existen también degradados radiales y cónicos, y `linear` los distingue). La documentación de v4 ya no menciona el nombre anterior, así que en un proyecto v4 conviene usar siempre `bg-linear-*`.

-----

## Modo oscuro

Para aplicar un estilo solo en modo oscuro, se usa el prefijo `dark:`:

```html
<div class="bg-white dark:bg-gray-800">
  <h3 class="text-gray-900 dark:text-white">Título</h3>
  <p class="text-gray-500 dark:text-gray-400">Descripción</p>
</div>
```

Cada elemento define su versión clara (sin prefijo) y su versión oscura (con `dark:`). Por defecto, `dark:` se activa según la preferencia del **sistema operativo** del usuario, a través de la media query `prefers-color-scheme`: si el usuario tiene su sistema en modo oscuro, se aplican los estilos `dark:`, sin que tengas que hacer nada más.

### Cambio manual con un botón

Muchas aplicaciones quieren dejar que el usuario elija el tema independientemente del sistema. Para eso, se le indica a Tailwind que `dark:` se active según una **clase** en lugar de la preferencia del sistema, con la directiva `@custom-variant`:

```css
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));
```

Con esto, los estilos `dark:` se aplican cuando el elemento, o alguno de sus ancestros, tiene la clase `dark`. Lo habitual es poner o sacar esa clase en el elemento `<html>`:

```html
<html class="dark">
  <body>
    <div class="bg-white dark:bg-black">...</div>
  </body>
</html>
```

> **En Tailwind v3**, el equivalente se configuraba en `tailwind.config.js` con `darkMode: 'class'`. Si trabajás con un proyecto viejo, esa es la opción que vas a encontrar.

### Conexión con React

En React, el cambio manual de tema es un caso perfecto para combinar lo que ya practicaste: el hook `useLocalStorage` del ejercicio de custom hooks puede guardar la preferencia (`'light'` o `'dark'`) entre recargas, y un `useEffect` puede agregar o quitar la clase `dark` en `document.documentElement` cada vez que ese valor cambia:

```tsx
const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');

useEffect(() => {
  document.documentElement.classList.toggle('dark', theme === 'dark');
}, [theme]);
```

Y si necesitás que el tema esté disponible en toda la aplicación, ese estado es exactamente el caso de uso del Context que vimos en la lección de hooks.

-----

## Colores propios

Cuando la paleta por defecto no alcanza (por ejemplo, para los colores de una marca), no hace falta escribir colores sueltos en cada lugar: se agregan al **tema**, y pasan a funcionar como cualquier otro color de Tailwind (`bg-brand`, `text-brand-dark`). Lo vemos en la lección de personalización del tema.

-----

## Resumen

* Los colores siguen el patrón `{utilidad}-{color}-{intensidad}`, con intensidades de `50` a `950`.
* La opacidad se controla con el modificador `/` (`bg-black/50`); `bg-opacity-*` fue eliminado en v4.
* Combiná fondos claros con textos oscuros y viceversa: el contraste es un tema de accesibilidad.
* Los degradados usan `bg-linear-to-{dirección}` con `from-`, `via-` y `to-` (en v3 se llamaban `bg-gradient-to-*`).
* `dark:` sigue por defecto la preferencia del sistema; para un botón manual, se redefine con `@custom-variant dark`.
