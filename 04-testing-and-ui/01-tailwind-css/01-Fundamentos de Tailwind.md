# Fundamentos de Tailwind

## Qué es Tailwind CSS

**Tailwind CSS** es un framework de CSS del tipo **utility-first** (las utilidades primero): en lugar de escribir reglas CSS propias con selectores y nombres de clase inventados, construís la interfaz combinando clases pequeñas que ya vienen definidas, cada una de las cuales hace **una sola cosa**.

```html
<button class="bg-blue-500 px-5 py-2 rounded-sm text-white">Enviar</button>
```

Cada clase aplica una única propiedad CSS: `bg-blue-500` el color de fondo, `px-5` el padding horizontal, `rounded-sm` el borde redondeado, `text-white` el color del texto. El botón final surge de **componer** esas piezas, sin escribir una sola línea de CSS propio.

-----

## El enfoque tradicional frente al utility-first

Con CSS tradicional, el flujo es: inventar un nombre para el elemento, definir esa clase en un archivo CSS, y después aplicarla en el HTML.

```css
.btn {
  background: blue;
  padding: 10px;
}
```

```html
<button class="btn">Enviar</button>
```

Este enfoque tiene costos que aparecen a medida que el proyecto crece: hay que inventar y recordar nombres (`btn`, `btn-primary`, `btn-primary-large`...), el CSS crece con cada variante (`btn-blue`, `btn-green`), y modificar una clase compartida puede romper elementos que no tenías en mente al tocarla.

Con Tailwind, el estilo se declara **en el mismo lugar donde se usa**:

```html
<button class="bg-blue-500 p-2">Enviar</button>
```

Y cambiar el diseño de ese único botón es tan simple como cambiar una clase (`bg-blue-500` por `bg-pink-500`), con la garantía de que ningún otro elemento se ve afectado.

### Tailwind frente a Bootstrap

Es habitual comparar ambos, pero resuelven el problema de forma distinta. **Bootstrap** te da **componentes ya armados** (un botón, una card, una barra de navegación) con un aspecto definido, que después tenés que sobrescribir si querés algo distinto. **Tailwind** te da **piezas pequeñas** y vos armás el diseño: más trabajo inicial, pero sin depender de un aspecto prehecho que te limite.

-----

## Qué ganás (y qué cuesta)

Las ventajas concretas son cuatro:

* **No inventás nombres.** La parte más tediosa de CSS, nombrar cosas, desaparece.
* **El CSS final es chico.** Tailwind genera únicamente las clases que efectivamente usás en tu código, no un archivo gigante con todo lo posible.
* **Consistencia.** Los valores (espaciados, tamaños de texto, colores) salen de una escala predefinida, así que es difícil que dos pantallas usen `13px` y `14px` para lo mismo por accidente.
* **Cambios locales y seguros.** Editar las clases de un elemento no afecta a ningún otro.

El costo también es real y conviene conocerlo desde el principio: las cadenas de clases se vuelven **largas**, y si copiás y pegás el mismo bloque de veinte clases en diez lugares, terminaste repitiendo código, igual que antes pero con otra sintaxis. La solución no es volver a CSS: en un proyecto con React, la forma correcta de reutilizar es **extraer un componente** (`<Button>`), de modo que las clases vivan en un solo lugar. Lo vemos en la lección de buenas prácticas.

-----

## Tailwind no reemplaza a CSS

Este es el punto más importante de toda la lección: **Tailwind es una forma más rápida de escribir CSS, no un sustituto de saber CSS.** Cada utilidad es una propiedad CSS con otro nombre:

| Clase de Tailwind | Qué hace en CSS |
| --- | --- |
| `flex` | `display: flex` |
| `mx-auto` | margen horizontal automático (centra el bloque) |
| `px-4` | padding horizontal de `1rem` |
| `text-center` | `text-align: center` |
| `hidden` | `display: none` |

Por eso, si mirás `flex items-center justify-center` y no sabés qué es Flexbox, no vas a entender qué hace ni cómo corregirlo cuando algo se rompe. Si lo sabés, leés esa línea como "un contenedor flexible con sus hijos centrados en ambos ejes". La regla práctica: **primero entendé CSS, después usá Tailwind mejor.**

-----

## Variantes: estilos según el estado o la pantalla

Tailwind agrega comportamiento condicional con **prefijos** delante de la clase:

```html
<button class="bg-blue-500 hover:bg-blue-600 md:text-lg dark:bg-blue-900">
  Enviar
</button>
```

* `hover:` aplica el estilo cuando pasás el mouse por encima.
* `md:` lo aplica solo desde cierto ancho de pantalla en adelante (lo vemos en la lección de diseño responsive).
* `dark:` lo aplica en modo oscuro (lo vemos en la lección de colores).

Los prefijos se pueden combinar y se leen de izquierda a derecha, como una condición: `md:hover:bg-red-500` significa "en pantallas medianas o más grandes, y al pasar el mouse, fondo rojo".

-----

## Un detalle que confunde al empezar

Si escribís mal el nombre de una clase (`bg-blu-500` en lugar de `bg-blue-500`), **no aparece ningún error**: el navegador simplemente no aplica nada, porque para él es una clase que no existe. Dos herramientas ayudan a evitarlo: la extensión oficial **Tailwind CSS IntelliSense** para tu editor (autocompleta clases y marca las inválidas), y la **documentación oficial** (`tailwindcss.com/docs`), que tiene buscador y muestra, para cada propiedad CSS, qué clase le corresponde.

La regla de aprendizaje es esa: **no adivines clases, buscalas en la documentación.** Con la práctica, las que más usás las vas a memorizar sin esfuerzo.

-----

## En React

Todo lo que vimos vale igual en un proyecto con React, con un único cambio: como `class` es una palabra reservada de JavaScript, en JSX el atributo se llama `className`:

```tsx
function SubmitButton() {
  return (
    <button className="bg-blue-500 px-5 py-2 rounded-sm text-white hover:bg-blue-600">
      Enviar
    </button>
  );
}
```

Este `SubmitButton` ya es, en sí mismo, la forma correcta de reutilizar ese conjunto de clases: en lugar de repetir la cadena de clases en cada lugar donde necesitás un botón, importás el componente.

-----

## Resumen

* Tailwind es **utility-first**: se construye combinando clases pequeñas que hacen una sola cosa.
* El estilo se declara donde se usa, sin inventar nombres ni mantener archivos CSS por componente.
* El CSS final incluye solo las clases que usás, así que pesa poco.
* La contracara es que las cadenas de clases son largas: se resuelve extrayendo **componentes**, no copiando y pegando.
* **No reemplaza a CSS:** cada utilidad es una propiedad CSS. Saber CSS es lo que te permite usar Tailwind bien.
* Una clase mal escrita no da error, simplemente no se aplica: usá IntelliSense y la documentación oficial.
