# Buenas Prácticas y Flujo de Trabajo

## Pensar en componentes, no en pantallas

Cuando el diseño se arma con clases utilitarias, la tentación es escribir cada pantalla completa con sus cadenas de clases, copiando y pegando partes de una a otra. Esa es exactamente la repetición que el CSS tradicional intentaba evitar, y Tailwind no la elimina por sí mismo.

La forma de mantener el código ordenado es pensar en **componentes reutilizables** (una barra de navegación, una tarjeta, un botón), y hacer que **cada componente sea el único lugar donde viven sus clases**:

```tsx
type ButtonProps = {
  children: React.ReactNode;
  onClick?: () => void;
};

function Button({ children, onClick }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      className="rounded-md bg-blue-600 px-4 py-2 font-semibold text-white hover:bg-blue-700"
    >
      {children}
    </button>
  );
}
```

Si mañana el azul de los botones cambia, se modifica en un único lugar. Esto es lo mismo que vimos en los patrones de composición: el mecanismo de reutilización no es una clase CSS, es un **componente**.

-----

## `@apply`: con moderación

Tailwind permite agrupar utilidades en una clase propia con la directiva `@apply`:

```css
.btn {
  @apply rounded-md bg-blue-600 px-4 py-2 font-semibold text-white;
}
```

Funciona, pero conviene tener cuidado: reintroduce nombres de clases inventados (`.btn`) y archivos CSS que mantener, es decir, parte de los problemas que se querían evitar. En un proyecto con componentes, casi siempre es preferible extraer un **componente** en lugar de una clase con `@apply`. Es más útil en casos donde no tenés componentes: HTML plano, o estilos que necesitás aplicar a contenido que no controlás.

-----

## Combinar clases condicionales

Cuando un componente tiene variantes (un botón primario y uno secundario, un botón deshabilitado), aparece la necesidad de combinar clases según condiciones. La forma más simple, respetando lo que vimos sobre detección de clases, es un mapa de clases completas:

```tsx
const variants = {
  primary: "bg-blue-600 text-white hover:bg-blue-700",
  secondary: "bg-gray-200 text-gray-900 hover:bg-gray-300",
} as const;
```

Cuando la lógica se complica, dos utilidades muy usadas en el ecosistema ayudan. **`clsx`** arma una cadena de clases a partir de condiciones, y **`tailwind-merge`** resuelve conflictos entre clases: como en CSS la clase que gana no depende del orden en que la escribís en el `className` sino del orden en que Tailwind las genera, escribir `p-2 p-4` no garantiza que gane `p-4`. `tailwind-merge` se ocupa de que la última prevalezca. Es común combinarlas en una función auxiliar (habitualmente llamada `cn`) que se usa en todo el proyecto.

-----

## Legibilidad de las cadenas largas

Las cadenas de clases largas son el costo principal de Tailwind, y hay tres hábitos que ayudan a que sigan siendo legibles:

* **Un orden consistente.** El plugin oficial de Prettier para Tailwind (**`prettier-plugin-tailwindcss`**) ordena las clases automáticamente al guardar, siguiendo un orden estándar, así que todo el equipo las lee igual.
* **Una clase por línea** cuando la cadena es muy larga, con saltos de línea dentro del `className`.
* **Extraer el componente** cuando una misma cadena se repite: si copiaste las mismas veinte clases dos veces, ya es hora de un componente.

-----

## Accesibilidad

Tailwind facilita la accesibilidad, pero no la garantiza: es una decisión tuya. Tres hábitos básicos:

* **Contraste.** Elegí combinaciones de color legibles (fondo claro con texto oscuro, y viceversa); el ejemplo de `bg-green-200 text-white` de la lección de colores es lo que hay que evitar.
* **Estados de foco visibles.** Un usuario que navega con el teclado necesita ver qué elemento está activo. Se controla con variantes como `focus-visible:outline-2` o `focus:ring-2`. Quitar el contorno sin ofrecer un reemplazo deja la interfaz inusable con teclado.
* **Texto solo para lectores de pantalla.** La clase `sr-only` oculta un elemento visualmente pero lo deja disponible para tecnologías de asistencia (por ejemplo, para describir un botón que solo tiene un ícono).

-----

## Reutilizar diseños ya hechos

Un atajo legítimo, y muy usado, es partir de una **colección de componentes ya armados con Tailwind**, como HyperUI (`hyperui.dev`): copiás un componente (un encabezado, un formulario, una tarjeta), lo pegás en tu proyecto, y ya tenés un punto de partida que se ve bien.

Lo importante es **qué hacés después**: no lo dejes tal cual. Adaptá los colores y espaciados a los de tu tema (para que use `bg-brand` en vez de un azul cualquiera), extraelo a un componente si lo vas a repetir, y revisá que use la sintaxis de la versión de Tailwind que tenés. Un componente copiado sin adaptar es el origen más común de una interfaz que "se siente" inconsistente.

-----

## Trabajar con asistentes de IA

Los asistentes de IA son buenos generando código de Tailwind rápidamente: un login, una barra de navegación, tres variantes de un encabezado. Pero el código generado es un **borrador**, no un producto terminado, y hay cosas concretas para revisar, varias de las cuales vimos en este módulo:

* **La versión de Tailwind.** Es habitual que el código generado use sintaxis de v3 (`@tailwind base;`, `bg-gradient-to-r`, un `tailwind.config.js`, `shadow` a secas) aunque tu proyecto esté en v4. Si algo no funciona, es lo primero a sospechar.
* **Clases dinámicas.** Si el código arma clases con plantillas de texto (`` `bg-${color}-500` ``), no va a funcionar en el build.
* **Consistencia con tu proyecto.** Ajustar los valores a tu tema y a tus componentes existentes, en lugar de sumar estilos sueltos.
* **Accesibilidad.** Que el resultado tenga contraste suficiente y estados de foco.

Y una regla que vale para cualquier herramienta: si no entendés qué hace una clase del código que te devolvió, **buscala en la documentación** antes de aceptarla. Usar Tailwind bien sigue dependiendo de entender CSS, como vimos en el primer capítulo.

-----

## Qué revisar cuando "no se ven los estilos"

Un procedimiento en orden, de lo más común a lo menos común:

1. **¿El CSS se está generando y cargando?** En un proyecto con CLI, `--watch` tiene que estar corriendo; en un framework, el servidor de desarrollo. Revisá que el `<link>` (o el import del CSS) apunte al archivo correcto.
2. **¿La clase está bien escrita?** Un error de tipeo no da error, simplemente no se aplica.
3. **¿La clase se construye dinámicamente?** Tiene que estar escrita completa en el código.
4. **¿Estás mezclando versiones?** Una clase o directiva de v3 en un proyecto v4 (o al revés).
5. **¿El archivo con las clases está fuera de la detección?** Por ejemplo, ignorado por `.gitignore`, o dentro de `node_modules`.
6. **¿Reiniciaste el servidor?** Especialmente después de cambiar plugins o la configuración.

-----

## Resumen

* Cada componente es el **único lugar donde viven sus clases**: reutilizás importando un componente, no copiando cadenas.
* `@apply` se usa con moderación; en un proyecto con componentes, casi siempre es mejor extraer un componente.
* Para variantes, un **mapa de clases completas**; `clsx` y `tailwind-merge` ayudan cuando la lógica crece.
* El **plugin de Prettier** mantiene un orden consistente en las clases.
* Accesibilidad: contraste, estados de foco visibles y `sr-only`.
* Las colecciones de componentes (HyperUI) y la IA son buenos puntos de partida, pero **siempre hay que adaptar y revisar**: versión de Tailwind, clases dinámicas y consistencia con tu tema.
