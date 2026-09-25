# Fundamentos de Tailwind: construir estilos combinando utilidades

## En una frase

**Tailwind CSS** es un framework de CSS *utility-first*: en lugar de escribir reglas propias con nombres de clase inventados, se compone la interfaz aplicando en el marcado clases pequeñas ya definidas, y cada una de ellas hace una sola cosa.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué son un selector, una clase y una propiedad CSS.
* Los conceptos básicos de Flexbox y de padding y margin.
* Cómo se pasa `className` en JSX: [Estilos en React](../../01-react-core-and-typescript/05-patrones-estilos-lifecycle/02-React%20Styles.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Utility-first:** enfoque en el que la interfaz se construye con clases de propósito único (utilidades), sin escribir CSS propio para cada componente.
* **Utilidad:** clase que aplica una sola declaración CSS o un grupo muy pequeño de ellas, por ejemplo `text-center`.
* **Variante:** prefijo que hace condicional a una utilidad, por ejemplo `hover:` o `md:`.
* **Escala de diseño (theme):** conjunto de valores predefinidos (espaciados, colores, tamaños) de los que salen las utilidades.
* **Preflight:** hoja de estilos base que Tailwind incluye para normalizar el aspecto inicial de los elementos.

-----

## El problema

Con CSS tradicional, cada elemento pide tres pasos: inventar un nombre, definir la regla en un archivo CSS y aplicar la clase en el HTML.

```css
.btn {
  background: blue;
  padding: 10px;
}
```

```html
<button class="btn">Enviar</button>
```

Este flujo tiene costos que aparecen cuando el proyecto crece:

1. **Nombrar.** Hay que inventar y recordar nombres (`btn`, `btn-primary`, `btn-primary-large`).
2. **CSS que solo crece.** Cada variante (`btn-blue`, `btn-green`) añade reglas nuevas y casi nunca se borran.
3. **Cambios con efectos laterales.** Una clase compartida afecta a todos los elementos que la usan. Modificarla puede romper elementos en los que no pensabas.
4. **Estilo lejos del marcado.** Para entender un elemento hay que saltar entre el HTML y la hoja de estilos.

Tailwind evita estos costos moviendo el estilo al marcado, con clases que ya existen.

-----

## Cómo funciona

### Una utilidad, una declaración CSS

Cada clase de Tailwind corresponde a CSS concreto. El botón de antes queda así:

```html
<button class="bg-blue-500 px-5 py-2 rounded-sm text-white">Enviar</button>
```

| Clase | Qué hace en CSS |
| --- | --- |
| `bg-blue-500` | color de fondo tomado de la escala de azules del tema |
| `px-5` | padding horizontal (`padding-inline`) de 5 unidades de espaciado |
| `py-2` | padding vertical (`padding-block`) de 2 unidades de espaciado |
| `rounded-sm` | `border-radius` pequeño (`0.25rem` en v4) |
| `text-white` | color del texto blanco |

El botón surge de **componer** esas piezas, sin una sola línea de CSS propio. Otros ejemplos de correspondencia directa:

| Clase | Qué hace en CSS |
| --- | --- |
| `flex` | `display: flex` |
| `hidden` | `display: none` |
| `text-center` | `text-align: center` |
| `mx-auto` | márgenes horizontales `auto` (centra un bloque con ancho definido) |
| `px-4` | padding horizontal de `1rem` con el tema por defecto |

En v4 el espaciado usa una sola variable base, `--spacing`, cuyo valor por defecto es `0.25rem`. Una utilidad como `px-4` genera `padding-inline: calc(var(--spacing) * 4)`, es decir, `1rem`.

### Tailwind no reemplaza a CSS

Tailwind es una forma más rápida de escribir CSS, no un sustituto de saber CSS. Cada utilidad es una propiedad CSS con otro nombre. Si lees `flex items-center justify-center` y no sabes qué es Flexbox, no podrás entender qué hace ni corregirlo cuando algo falle. Si lo sabes, lo lees como "un contenedor flexible con sus hijos centrados en ambos ejes".

Regla práctica: primero se entiende CSS y después se usa Tailwind con soltura.

### Tailwind frente a CSS tradicional

Con Tailwind, el estilo se declara **en el mismo lugar donde se usa**:

```html
<button class="bg-blue-500 p-2">Enviar</button>
```

Cambiar el diseño de ese botón es cambiar una clase (`bg-blue-500` por `bg-pink-500`). Como el cambio está en el elemento y no en una regla compartida, ningún otro elemento se ve afectado. La documentación oficial resume así las ventajas de este enfoque:

* **No hay que nombrar clases.** La parte más tediosa de CSS desaparece.
* **Los cambios son locales y seguros.** Modificar una utilidad en un elemento solo afecta a ese elemento.
* **El CSS deja de crecer con cada pieza nueva.** Las utilidades se reutilizan, así que agregar una pantalla casi no agrega CSS.
* **El mantenimiento es más simple.** Para cambiar algo se busca el elemento y se editan sus clases, en lugar de recordar cómo funcionaba una regla escrita meses antes.

### Tailwind frente a estilos en línea

Escribir utilidades puede parecerse a usar `style="..."`, pero no es lo mismo. Las utilidades ofrecen tres cosas que el atributo `style` no da:

1. **Diseño con restricciones.** Los valores salen de una escala predefinida (el tema), no de números arbitrarios. Es difícil que dos pantallas usen `13px` y `14px` para lo mismo por accidente.
2. **Estados.** `hover:`, `focus:` y otros prefijos aplican estilos según el estado. `style` no admite pseudoclases.
3. **Media queries.** Los prefijos responsive (`md:`) aplican estilos según el ancho de pantalla. `style` tampoco admite media queries.

### Tailwind frente a Bootstrap

Ambos resuelven el problema de distinta forma.

* **Bootstrap** entrega **componentes ya armados** (botón, card, barra de navegación) con un aspecto definido, que se sobrescribe si se quiere algo distinto.
* **Tailwind** entrega **piezas pequeñas** y tú armas el diseño. Exige más trabajo inicial, pero no te ata a un aspecto prehecho.

La diferencia es de filosofía: Bootstrap parte de componentes con opinión visual, y Tailwind parte de propiedades sueltas sin ella.

### Variantes: estilos según el estado o la pantalla

Un prefijo delante de la clase la vuelve condicional:

```html
<button class="bg-blue-500 hover:bg-blue-600 md:text-lg dark:bg-blue-900">
  Enviar
</button>
```

* `hover:` aplica el estilo al pasar el mouse por encima. En v4 solo actúa en dispositivos cuyo método de entrada principal admite *hover* (`@media (hover: hover)`), para que no se "pegue" en pantallas táctiles.
* `md:` aplica el estilo desde cierto ancho hacia arriba. Es un `min-width`: en `md` equivale a `@media (width >= 48rem)`, es decir, `768px` con el tamaño de fuente por defecto.
* `dark:` aplica el estilo en modo oscuro. Por defecto se basa en `prefers-color-scheme: dark`, la preferencia del sistema operativo. Se puede cambiar para que dependa de una clase, algo que se ve en [Tema, Modo Oscuro y Plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md).

Las variantes se pueden apilar. En v4 se leen de izquierda a derecha, como una condición: `md:hover:bg-red-500` significa "desde pantallas medianas y al pasar el mouse, fondo rojo".

Una clase con variante **solo aporta el estilo condicional**. `hover:bg-blue-600` no pinta nada por sí sola: por eso el ejemplo conserva también `bg-blue-500` para el estado normal.

#### Mobile-first

Las utilidades sin prefijo aplican en todos los tamaños. Las que tienen prefijo aplican en ese breakpoint **y en los mayores**. Por eso, para estilar el móvil se usa la clase sin prefijo:

```html
<!-- Centrado en móvil; alineado a la izquierda desde 640px -->
<div class="text-center sm:text-left">...</div>
```

`sm:` no significa "en pantallas pequeñas", sino "desde el breakpoint `sm` (`40rem`) en adelante". El detalle completo está en [Layout y Responsive](05-Layout%20y%20Responsive.md).

### Preflight: el punto de partida de los estilos

Tailwind incluye **Preflight**, una hoja base construida sobre `modern-normalize`. Se inyecta al importar Tailwind y cambia el aspecto por defecto de varios elementos:

* Quita márgenes y padding de todos los elementos.
* Los encabezados `h1` a `h6` heredan tamaño y grosor de fuente: dejan de verse como títulos.
* Las listas (`ol`, `ul`) pierden viñetas y numeración.
* Imágenes, `svg`, `video` y similares pasan a `display: block`, y `img` y `video` tienen `max-width: 100%`.
* Los bordes parten de `border: 0 solid`, por lo que la clase `border` añade un borde sólido de `1px` en `currentColor`.

Es una decisión de diseño: todo se estila de forma explícita, sin heredar valores del navegador. Si un `<h1>` "no tiene estilo", no es un error: hay que darle `text-3xl font-bold`.

### Por qué el CSS final es pequeño

Tailwind lee tus archivos fuente **como texto plano**, sin interpretarlos como código. Busca fragmentos que podrían ser nombres de clase, intenta generar CSS para cada uno y descarta los que no corresponden a ninguna utilidad conocida. El CSS resultante contiene solo lo que usas.

```
Archivos fuente (HTML, JSX...)
        |
        v   lectura como texto plano
Fragmentos candidatos a clase
        |
        v   se descartan los que no son utilidades
CSS generado: solo las clases detectadas
```

Esa detección es la causa de un error común (clases armadas con interpolación). Se estudia a fondo en [Cómo Tailwind Detecta las Clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md).

-----

## Ejemplo completo

Una tarjeta con un botón, responsive, con estados y modo oscuro:

```tsx
type ProfileCardProps = {
  name: string;
  role: string;
};

export function ProfileCard({ name, role }: ProfileCardProps) {
  return (
    <article className="mx-auto flex max-w-sm flex-col gap-4 rounded-xl bg-white p-6 shadow-lg md:max-w-md md:flex-row md:items-center dark:bg-gray-800">
      <div className="flex-1">
        <h2 className="text-xl font-bold text-gray-900 dark:text-white">{name}</h2>
        <p className="text-gray-600 dark:text-gray-300">{role}</p>
      </div>
      <button className="rounded-sm bg-blue-500 px-5 py-2 text-white hover:bg-blue-600">
        Contactar
      </button>
    </article>
  );
}
```

Qué hace cada grupo de clases:

1. **Contenedor:** `flex flex-col gap-4` apila los hijos con separación. `mx-auto max-w-sm` lo centra con un ancho máximo.
2. **Apariencia:** `rounded-xl bg-white p-6 shadow-lg` da bordes redondeados, fondo, padding y sombra.
3. **Responsive:** `md:flex-row md:items-center md:max-w-md` cambia el layout a fila desde `768px`. Por debajo de ese ancho se aplican las clases sin prefijo.
4. **Modo oscuro:** `dark:bg-gray-800`, `dark:text-white` y `dark:text-gray-300` reemplazan colores cuando el sistema prefiere modo oscuro.
5. **Estado:** `hover:bg-blue-600` oscurece el botón al pasar el mouse.

Para escribir el equivalente en CSS tradicional harían falta nombres de clase, selectores, una media query y otra para el modo oscuro.

-----

## En React

Todo lo anterior vale igual en un proyecto con React, con un único cambio: `class` es una palabra reservada de JavaScript, así que en JSX el atributo se llama `className`.

```tsx
function SubmitButton() {
  return (
    <button className="rounded-sm bg-blue-500 px-5 py-2 text-white hover:bg-blue-600">
      Enviar
    </button>
  );
}
```

El costo de Tailwind es que las cadenas de clases se alargan. Si copias el mismo bloque de veinte clases en diez lugares, repites código, igual que antes pero con otra sintaxis. La solución no es volver a CSS: es **extraer un componente**. `SubmitButton` ya es la forma correcta de reutilizar ese conjunto de clases, porque las clases viven en un solo lugar y el resto de la app importa el componente.

La documentación de Tailwind propone tres estrategias contra la duplicación:

* **Bucles:** un elemento repetido se renderiza con `map`, así que su lista de clases se escribe una vez.
* **Componentes:** en React, el mecanismo natural (`<Button>`, `<Card>`).
* **CSS propio:** para patrones simples en un solo elemento, con `@layer components`.

Como dice la documentación, si puedes editar a la vez todas las copias de una lista de clases, no hace falta ninguna abstracción adicional. Las prácticas de reutilización se profundizan en [Buenas Prácticas y Flujo de Trabajo](07-Buenas%20Pr%C3%A1cticas%20y%20Flujo%20de%20Trabajo.md).

-----

## Errores comunes

### 1. Una clase mal escrita no da error

```html
<button class="bg-blu-500">Enviar</button>
```

**Qué pasa:** no aparece ningún error y el botón no cambia de color.
**Por qué:** `bg-blu-500` no corresponde a ninguna utilidad. Tailwind descarta ese fragmento al generar el CSS y, para el navegador, es una clase que no existe.
**Arreglo:** instala la extensión oficial **Tailwind CSS IntelliSense** (autocompleta, marca errores y muestra el CSS de cada clase al pasar el mouse) y consulta la documentación (`tailwindcss.com/docs`) en lugar de adivinar nombres.

### 2. Construir nombres de clase con interpolación

```jsx
<div className={`bg-${color}-500`} />   // no funciona de forma fiable
```

**Qué pasa:** el color no se aplica, o solo se aplica si esa clase completa aparece escrita en otro lugar del proyecto.
**Por qué:** Tailwind lee el código como texto y no entiende concatenación ni interpolación. Nunca ve el fragmento `bg-red-500` completo.
**Arreglo:** escribe las clases completas y elígelas con un objeto de correspondencias.

```jsx
const colors = {
  blue: 'bg-blue-500 hover:bg-blue-600',
  red: 'bg-red-500 hover:bg-red-600',
};

<div className={colors[color]} />
```

### 3. Esperar que los elementos conserven el estilo del navegador

**Qué pasa:** un `h1` se ve como texto normal, una lista sin viñetas y una imagen se comporta como bloque.
**Por qué:** Preflight normaliza estos elementos a propósito.
**Arreglo:** aplica las utilidades que necesites (`text-3xl font-bold`, `list-disc`, etc.).

### 4. Usar `sm:` para el móvil

```html
<div class="sm:text-center">...</div>
```

**Qué pasa:** el texto no se centra en pantallas pequeñas.
**Por qué:** el enfoque es mobile-first. `sm:` aplica desde `40rem` hacia arriba, no por debajo.
**Arreglo:** escribe la clase base sin prefijo para el móvil y usa prefijos para pantallas mayores (`text-center md:text-left`).

### 5. Copiar la misma cadena de clases en muchos lugares

**Qué pasa:** un cambio de diseño obliga a editar decenas de archivos.
**Por qué:** se duplicó el estilo en vez de compartirlo.
**Arreglo:** extrae un componente (`<Button>`) o, si se repite dentro de una lista, renderízala con `map`.

### 6. Usar nombres de utilidades de v3 en un proyecto v4

```html
<div class="shadow-sm rounded-sm">...</div>
```

**Qué pasa:** el resultado visual difiere de lo que muestra un tutorial escrito para v3.
**Por qué:** en v4 se renombraron varias utilidades de la escala. `shadow-sm` de v3 es `shadow-xs` en v4, y `shadow` pasó a `shadow-sm`. Lo mismo con `rounded-sm` (v3) que ahora es `rounded-xs`, y `rounded` que pasó a `rounded-sm`.
**Arreglo:** verifica en la documentación de v4 y ten presente la versión de cada tutorial. La tabla completa de renombres está en la guía de actualización oficial.

-----

## Cuándo sí y cuándo no

**Tailwind es una buena opción cuando:**

* El proyecto usa componentes (React, Vue, plantillas con partials), porque el componente resuelve la reutilización.
* El equipo quiere consistencia visual apoyada en una escala de valores compartida.
* Se prioriza construir rápido sin mantener hojas de estilo por componente.

**Conviene otra herramienta, o combinarla con CSS propio, cuando:**

* Hay que estilar HTML que no controlas (contenido Markdown, un CMS) y no puedes añadir clases a los elementos.
* Los estilos dependen de valores calculados en tiempo de ejecución: para eso sirve `style` o una variable CSS.
* Debes soportar navegadores anteriores a Safari 16.4, Chrome 111 o Firefox 128: v4 no está pensado para ellos, y en ese caso corresponde quedarse en v3.4.
* Un patrón complejo se repite en un solo elemento sin componente: un `@layer components` puede ser más limpio.

Tailwind y CSS propio se combinan. Las clases definidas en la capa `components` se pueden sobrescribir con utilidades.

-----

## Resumen en 5 líneas

1. Tailwind es **utility-first**: se compone la interfaz con clases pequeñas que aplican una sola declaración CSS cada una.
2. El estilo va donde se usa, sin nombres inventados: los cambios son locales y el CSS deja de crecer con cada pieza.
3. Difiere de `style` porque sus valores salen de una escala y soporta estados (`hover:`) y media queries (`md:`).
4. **No reemplaza a CSS:** cada utilidad es CSS, y saber CSS es lo que permite usarla bien. Una clase mal escrita no da error.
5. La contracara son las cadenas largas de clases: se resuelven con **componentes** y no con copiar y pegar.

-----

## Para profundizar

<details>
<summary>El CSS que genera una variante</summary>

Una utilidad con variante se traduce a CSS condicional. Según la documentación de v4, un breakpoint (`sm:` en el ejemplo oficial) y `dark:` generan, en esencia:

```css
.sm\:grid-cols-3 {
  @media (width >= 40rem) {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}

.dark\:bg-gray-800 {
  @media (prefers-color-scheme: dark) {
    background-color: var(--color-gray-800);
  }
}
```

Y `hover:` genera el estado `:hover` dentro de `@media (hover: hover)`. Los nombres con `\:` son el escape de CSS para poder usar `:` dentro de un nombre de clase. Los valores provienen de variables del tema (`--color-gray-800`, `--text-lg`), que Tailwind emite como variables CSS en `:root` solo si se usan.

</details>

<details>
<summary>Qué cambió de v3 a v4 en lo básico</summary>

* **Importación:** en v4 se usa un `@import "tailwindcss";` normal. En v3 eran las directivas `@tailwind base; @tailwind components; @tailwind utilities;`.
* **Utilidades renombradas:** `shadow-sm` pasó a `shadow-xs` y `shadow` a `shadow-sm`. Lo mismo ocurre con `blur`, `drop-shadow` y `rounded`. `outline-none` pasó a `outline-hidden` y `ring` a `ring-3`.
* **Orden de las variantes apiladas:** en v3 se aplicaban de derecha a izquierda. En v4 se aplican de izquierda a derecha.
* **`hover:`:** en v4 solo se aplica cuando el dispositivo principal admite hover.
* **Color de borde por defecto:** en v3 `border` usaba `gray-200`. En v4 usa `currentColor`.
* **Navegadores:** v4 está diseñado para Safari 16.4+, Chrome 111+ y Firefox 128+.

</details>

<details>
<summary>Tailwind en el contexto de otros enfoques de estilos</summary>

En React hay varias formas de estilar: `style`, CSS global, CSS Modules, Sass, CSS-in-JS y Tailwind. Tailwind genera CSS estático durante el build y no tiene costo de generación en el navegador. Por eso no depende del contexto de React en el navegador, a diferencia del CSS-in-JS en tiempo de ejecución. La comparación completa está en [Estilos en React](../../01-react-core-and-typescript/05-patrones-estilos-lifecycle/02-React%20Styles.md).

</details>

<details>
<summary>Herramientas que ayudan a escribir clases</summary>

* **Tailwind CSS IntelliSense:** extensión oficial para VS Code (también funciona en Cursor; Zed tiene soporte integrado). Ofrece autocompletado, detección de errores y vista previa del CSS de cada clase.
* **Plugin oficial de Prettier:** ordena las clases de cada elemento con un orden recomendado, lo que hace más legibles las cadenas largas.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Tailwind es un framework de CSS *utility-first*. En lugar de escribir CSS propio con nombres de clase, se construye la interfaz combinando clases pequeñas que aplican una sola propiedad cada una, por ejemplo `px-4` o `text-center`. El estilo queda en el mismo lugar donde se usa, y el CSS final incluye solo las clases que el código utiliza. No reemplaza a CSS: hay que conocer CSS para usarlo bien.

### Respuesta ampliada (semi-senior)

* **Modelo:** cada utilidad mapea a una declaración CSS con valores tomados del tema (`--spacing`, `--color-*`). El tema es la fuente de consistencia: el diseño se restringe a una escala en lugar de valores arbitrarios.
* **Generación:** Tailwind escanea los archivos fuente como texto plano, sin parsearlos como código, y genera CSS solo para los fragmentos que coinciden con una utilidad. Por eso el CSS es pequeño y por eso no funcionan los nombres armados con interpolación.
* **Frente a `style`:** las utilidades admiten estados, media queries y pseudoelementos mediante variantes, y no admiten valores fuera de la escala salvo que se usen explícitamente valores arbitrarios (`w-[137px]`).
* **Frente a Bootstrap:** Bootstrap entrega componentes con opinión visual que hay que sobrescribir. Tailwind entrega propiedades y deja el diseño libre, a cambio de más trabajo inicial y de marcado con más clases.
* **Trade-offs:** ventajas en velocidad, cambios locales y consistencia. Costos en la legibilidad del marcado, en la curva de aprendizaje del vocabulario y en la disciplina necesaria para reutilizar con componentes en vez de copiar y pegar.
* **Variantes:** son prefijos apilables que se leen de izquierda a derecha en v4. Son mobile-first: sin prefijo aplica siempre, con prefijo aplica desde ese breakpoint.
* **Versión:** v4 se configura desde CSS con `@import "tailwindcss"` y cambió nombres de utilidades y el soporte de navegadores respecto a v3. Es importante distinguirlas al leer material antiguo.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué no usar estilos en línea, si es parecido?**
Porque `style` no admite pseudoclases ni media queries, y sus valores son libres. Las utilidades permiten `hover:` y `md:`, y sus valores salen de una escala compartida.

**2. ¿Tailwind no repite código en cada elemento?**
Sí, repite clases en el marcado, pero no repite CSS: cada utilidad se define una vez. La repetición de bloques largos se resuelve con componentes en React, con bucles o, en casos simples, con `@layer components`.

**3. ¿Cómo evita generar un CSS enorme?**
Escanea los archivos fuente como texto plano y genera solo las utilidades cuyo nombre encuentra. Las variables del tema también se emiten únicamente si se usan.

**4. ¿Qué significa que sea mobile-first?**
Que una utilidad sin prefijo aplica en todos los tamaños, y una con prefijo (`md:`) aplica desde ese ancho mínimo hacia arriba. Para estilar el móvil se escribe la clase sin prefijo.

**5. ¿Por qué una clase mal escrita no falla?**
Porque Tailwind descarta los fragmentos que no coinciden con ninguna utilidad, y el navegador ignora las clases que no existen en el CSS. Se detecta con IntelliSense o revisando el resultado.

**6. ¿Qué cambió en v4 que afecte a los fundamentos?**
Se importa con `@import "tailwindcss"` en lugar de las directivas `@tailwind`, se renombraron utilidades (`shadow-sm`, `rounded-sm`), las variantes apiladas se aplican de izquierda a derecha y `hover:` solo actúa en dispositivos con hover. Además, requiere navegadores modernos (Safari 16.4+, Chrome 111+, Firefox 128+).

-----

## Siguiente lección

Ahora que sabes cómo se componen las utilidades, el paso que sigue es instalar Tailwind en un proyecto y conectarlo con el bundler: [Instalación e Integración](02-Instalaci%C3%B3n%20e%20Integraci%C3%B3n.md). Más adelante, en el módulo siguiente, verás componentes construidos sobre Tailwind: [Qué es shadcn/ui](../02-shadcn-ui/01-Qu%C3%A9%20es%20shadcn-ui.md).
