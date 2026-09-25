# Estilos en React: cómo aplicar CSS a los componentes

## En una frase

React no impone una forma de dar estilos: un componente puede recibirlos como objeto (`style`), como clases CSS (`className`), o mediante herramientas que generan esas clases (CSS Modules, Sass, CSS-in-JS, Tailwind). Cada enfoque resuelve de forma distinta el mismo problema: que los estilos de un componente no choquen con los de otro.

-----

## Antes de empezar

Conviene que ya sepas cómo escribir JSX y pasar props a un componente. Términos usados en esta nota (también están en el [Glosario](Glosario.md)):

* **Estilo en línea (inline style):** estilo que se aplica con el atributo `style` de un elemento.
* **Selector de clase:** regla CSS que se aplica a los elementos que tienen cierta clase, por ejemplo `.card`.
* **Ámbito (scope):** zona donde un nombre de clase es válido. En CSS normal, el ámbito es toda la página.
* **CSS-in-JS:** enfoque en el que el CSS se escribe dentro de archivos JavaScript.
* **Utility-first:** enfoque en el que se compone la interfaz con clases pequeñas que aplican una sola propiedad cada una.
* **Bundler:** herramienta (Vite, Next.js, etc.) que procesa y empaqueta el código y los estilos antes de servirlos al navegador.

-----

## El problema

El CSS del navegador es **global**: cualquier regla `.title` afecta a todos los elementos con esa clase, sin importar en qué componente estén.

```css
/* Header.css */
.title { color: red; }

/* Card.css */
.title { color: blue; }
```

Si ambos archivos se cargan, el resultado depende del orden en que se cargan, y un componente puede alterar a otro sin que lo notes. A medida que la app crece, hace falta un enfoque que:

1. Evite choques de nombres.
2. Mantenga el estilo cerca del componente que lo usa.
3. Permita variar el estilo según props o estado.

-----

## Cómo funciona

### Estilos en línea con un objeto

El atributo `style` recibe un **objeto**, no un string. Por eso se ven dobles llaves: las externas abren una expresión JavaScript y las internas son el objeto literal.

```jsx
<h1 style={{ color: 'red', marginTop: 20 }}>Hola</h1>
```

Reglas:

* Los nombres de propiedades van en **camelCase**: `backgroundColor`, no `background-color`. Sigue la convención de la propiedad `style` del DOM.
* Los valores son strings o números. Un número recibe `px` automáticamente, salvo en propiedades sin unidad (como `opacity` o `zIndex`). Para otra unidad, usa un string: `{ fontSize: '2em' }`.
* El objeto puede guardarse en una variable para reutilizarlo:

```jsx
const darkMode = { color: 'white', backgroundColor: 'black' };

<h1 style={darkMode}>Hola</h1>
```

Limitaciones: `style` no admite pseudoclases (`:hover`), pseudoelementos ni media queries, y los estilos no se pueden sobrescribir fácilmente desde una hoja externa. La documentación de React recomienda usarlo solo para valores dinámicos que no se conocen de antemano, y usar clases (`className`) en los demás casos.

```jsx
// Caso adecuado: el valor viene de un dato
<div style={{ width: `${progress}%` }} />
```

### Hojas de estilo importadas y `className`

Se escribe un archivo `.css` y se importa desde el componente. El bundler lo incluye en la página. Los elementos usan el atributo `className`:

```jsx
import './Card.css';

function Card() {
  return <div className="card">Contenido</div>;
}
```

`className` reemplaza a `class` porque `class` es una palabra reservada de JavaScript. Para combinar clases de forma condicional, arma el string con una plantilla o con una utilidad como `clsx`:

```jsx
<button className={`btn ${isActive ? 'btn-active' : ''}`}>Enviar</button>
```

### Varias hojas de estilo

Una hoja por componente mantiene el CSS organizado, pero **importar un archivo `.css` no lo limita a ese componente**: sus reglas siguen siendo globales una vez cargadas. Dos archivos con `.title` chocan igual. Las salidas son:

* Prefijar los nombres a mano (`.card__title`, convención BEM). Funciona, pero depende de la disciplina del equipo.
* Usar CSS Modules, que automatiza el aislamiento.

### CSS Modules

Un archivo con extensión `.module.css` se trata como módulo: el bundler reescribe cada nombre de clase para hacerlo único y devuelve un objeto que asocia el nombre original con el generado.

```css
/* Card.module.css */
.card { border-radius: 8px; }
.title { color: #2563eb; }
```

```jsx
import styles from './Card.module.css';

function Card() {
  return (
    <div className={styles.card}>
      <h2 className={styles.title}>Título</h2>
    </div>
  );
}
```

El `.title` de este archivo no choca con el de otro módulo, porque cada uno recibe un nombre generado distinto. Vite y Next.js lo soportan sin configuración extra. Si el nombre de la clase tiene guiones (`.card-title`), se accede con corchetes (`styles['card-title']`) o se escribe en camelCase (`.cardTitle`).

### Sass

**Sass** es un preprocesador: se escribe en una sintaxis que extiende CSS y se **compila a CSS plano** antes de llegar al navegador. Añade variables, anidamiento, mixins y módulos (`@use`).

```scss
// Card.module.scss
$primary: #2563eb;

.card {
  border-radius: 8px;

  .title {
    color: $primary;
  }
}
```

Para usarlo, instala el paquete `sass` (`npm install -D sass`). Vite y Next.js lo compilan automáticamente y no requieren un plugin adicional. Un archivo `.module.scss` combina Sass con CSS Modules, y se importa igual que un `.module.css`.

En Sass, `@use` es el reemplazo del antiguo `@import`; para código nuevo se usa `@use`.

Hoy CSS nativo ya ofrece variables (`--color`) y anidamiento, así que Sass es menos imprescindible que antes; sigue siendo útil por sus mixins, funciones y módulos.

### Styled Components y CSS-in-JS

**styled-components** define componentes que llevan su CSS incorporado, escrito en un template literal. Genera un nombre de clase único por componente.

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background: ${(props) => (props.$primary ? '#2563eb' : '#6b7280')};
  color: white;
  padding: 8px 16px;
`;

<Button $primary>Confirmar</Button>
<Button>Cancelar</Button>
```

Puntos a tener en cuenta:

* `Button` es un componente de React que renderiza un `<button>`.
* Las reglas pueden depender de props. El prefijo `$` (transient props) evita que esa prop llegue al elemento del DOM.
* Genera el CSS **en tiempo de ejecución** en el navegador (y en el servidor si hay SSR, con configuración adicional).
* Estado actual: los mantenedores anunciaron en marzo de 2025 que la librería entra en **modo de mantenimiento** (solo correcciones críticas y de seguridad) y no recomiendan adoptarla en proyectos nuevos. Además, no funciona en React Server Components sin marcar el archivo con `'use client'`. Verifica el estado vigente en el repositorio antes de decidir.

### Tailwind CSS

**Tailwind CSS** es un framework utility-first: la interfaz se construye combinando clases predefinidas directamente en el JSX.

```jsx
<h1 className="text-3xl font-bold underline">Hola</h1>
```

Cada clase aplica una propiedad CSS. Tailwind analiza tu código durante el build y genera solo el CSS de las clases que encuentra, por lo que no hay costo de generación en el navegador.

Con Vite, la instalación en Tailwind v4 es:

```bash
npm install tailwindcss @tailwindcss/vite
```

```js
// vite.config.js
import tailwindcss from '@tailwindcss/vite';

export default { plugins: [tailwindcss()] };
```

```css
/* CSS principal */
@import "tailwindcss";
```

En Next.js se usa el plugin de PostCSS (`@tailwindcss/postcss`) en lugar del de Vite. En Tailwind v3 el proceso era distinto (archivo `tailwind.config.js` y directivas `@tailwind base/components/utilities`).

Más detalle en el módulo de Tailwind del wiki: [Fundamentos de Tailwind](../../04-testing-and-ui/01-tailwind-css/01-Fundamentos%20de%20Tailwind.md) e [Instalación e integración](../../04-testing-and-ui/01-tailwind-css/02-Instalaci%C3%B3n%20e%20Integraci%C3%B3n.md). Para componentes construidos sobre Tailwind, ver [Qué es shadcn/ui](../../04-testing-and-ui/02-shadcn-ui/01-Qu%C3%A9%20es%20shadcn-ui.md).

-----

## Ejemplo completo

Un botón con variantes, usando CSS Modules y una prop tipada:

```css
/* Button.module.css */
.button { padding: 8px 16px; border: 0; border-radius: 4px; color: white; }
.primary { background: #2563eb; }
.secondary { background: #6b7280; }
```

```tsx
// Button.tsx
import styles from './Button.module.css';

type ButtonProps = {
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
};

export function Button({ variant = 'primary', children }: ButtonProps) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

```tsx
<Button variant="primary">Confirmar</Button>
<Button variant="secondary">Cancelar</Button>
```

La variante se elige con una prop, y las clases quedan aisladas al componente.

-----

## Errores comunes

### 1. Usar `class` en lugar de `className`

**Qué pasa:** en JSX, `class` no aplica la clase y React muestra una advertencia en consola.
**Por qué:** JSX se compila a JavaScript y la propiedad del DOM se llama `className`.
**Solución:** escribe `className`.

### 2. Estilos en línea con unidades incorrectas

```jsx
<div style={{ fontSize: 20 }} />       // 20px
<div style={{ width: '50' }} />        // inválido: un string sin unidad se ignora
<div style={{ width: '50%' }} />       // correcto
```

**Por qué:** un número recibe `px`, pero un string se pasa tal cual al navegador, que lo descarta si no es un valor CSS válido. Usa números para píxeles y strings con unidad para el resto.

### 3. Usar guiones o strings en `style`

```jsx
<div style={{ 'background-color': 'red' }} />   // no funciona
<div style="color: red" />                      // error: style espera un objeto
```

**Solución:** objeto con propiedades en camelCase.

### 4. Estilos globales que se pisan

**Qué pasa:** dos componentes definen `.title` y uno cambia el aspecto del otro.
**Por qué:** un `.css` importado es global; importarlo desde un componente no lo aísla.
**Solución:** CSS Modules, un prefijo consistente (BEM) o clases utilitarias.

### 5. Depender del orden de los imports

**Qué pasa:** con reglas de igual especificidad, gana la que se carga después, y ese orden depende del orden de los imports (y de cómo el bundler agrupa los archivos).
**Por qué:** la cascada de CSS resuelve empates por orden de aparición.
**Solución:** evita depender de ese orden: usa clases aisladas, importa los estilos globales una sola vez en el punto de entrada y verifica el resultado con el build de producción, no solo en desarrollo.

-----

## En TypeScript

`React.CSSProperties` tipa un objeto de estilos y valida nombres y valores:

```tsx
import type { CSSProperties } from 'react';

const box: CSSProperties = { marginTop: 20, backgroundColor: 'green' };

<div style={box} />
```

Las variables CSS personalizadas no forman parte de `CSSProperties`; se pueden pasar con un cast:

```tsx
<div style={{ '--accent': color } as CSSProperties} />
```

Los módulos CSS se importan como un objeto de strings. Los tipos de `vite/client` (o los que provee Next.js) declaran `*.module.css`, así que `styles.card` es un `string` sin errores. Si quieres que el compilador valide los nombres de clase, hay herramientas que generan tipos por archivo, aunque no son obligatorias.

Las variantes se tipan como una unión de literales en las props, como en el ejemplo anterior (`'primary' | 'secondary'`). Con styled-components, las props se declaran con un genérico: `styled.button<{ $primary?: boolean }>`.

-----

## Cuándo sí y cuándo no

| Enfoque | Ventajas | Desventajas | Cuándo elegirlo |
| ------- | -------- | ----------- | --------------- |
| Estilo en línea | Simple; ideal para valores dinámicos | Sin pseudoclases ni media queries; difícil de reutilizar | Valores calculados en runtime (ancho, posición) |
| CSS global (`.css`) | Estándar, sin herramientas extra | Nombres globales que pueden chocar | Reset, tipografía base, variables globales |
| CSS Modules | Aislamiento automático, CSS estándar, sin costo en runtime | Menos cómodo para variantes muy dinámicas | Estilos propios de componentes; buena opción por defecto |
| Sass | Variables, mixins, módulos | Paso de compilación extra; CSS nativo cubre parte de lo que ofrece | Proyectos con mucho CSS compartido |
| CSS-in-JS en runtime (styled-components) | Estilos ligados a props, todo en un archivo | Costo en runtime; poco compatible con Server Components; styled-components está en mantenimiento | Proyectos existentes que ya lo usan |
| Tailwind CSS | Sin nombres que inventar, CSS generado en build, sistema de diseño consistente | JSX con muchas clases; requiere aprender el vocabulario | Proyectos nuevos y equipos que priorizan velocidad y consistencia |

La documentación de Next.js recomienda Tailwind para la mayor parte del estilado y CSS Modules cuando las utilidades no alcanzan.

-----

## Resumen en 5 líneas

1. El CSS es global por defecto: el reto en React es evitar choques de nombres entre componentes.
2. `style` recibe un objeto en camelCase; úsalo solo para valores dinámicos.
3. Las hojas `.css` importadas siguen siendo globales; usa `className`, no `class`.
4. CSS Modules aísla los nombres de clase por archivo sin costo en runtime; Sass se compila a CSS y se combina con ellos.
5. Tailwind genera el CSS en build; styled-components lo genera en runtime y está en modo de mantenimiento.

-----

## Para profundizar

<details>
<summary>Cómo funciona el aislamiento de CSS Modules</summary>

Durante el build, el bundler reemplaza cada nombre de clase por uno único (por ejemplo `Card_title__x7Yk2`) en el CSS y en el objeto que exporta el módulo. La app no cambia el nombre en tu código fuente: solo el resultado final. Las clases se aíslan, pero los selectores globales (`body`, `h1`) no; para eso existe `:global(...)`.

</details>

<details>
<summary>Por qué el CSS-in-JS en runtime es un problema con Server Components</summary>

Los Server Components se ejecutan en el servidor y no tienen estado de cliente ni contexto de React en el navegador. Las librerías de CSS-in-JS en runtime suelen depender de contexto y de inyectar estilos mientras se renderiza, por lo que necesitan componentes de cliente (`'use client'`) y configuración adicional para SSR. Los enfoques que generan CSS en build (CSS Modules, Tailwind) no tienen esa dependencia.

</details>

<details>
<summary>Combinar clases de Tailwind sin conflictos</summary>

Al construir componentes con variantes, las clases pueden contradecirse (`p-2` y `p-4`). Una práctica frecuente es usar `clsx` para armar clases condicionales y `tailwind-merge` para resolver conflictos. shadcn/ui lo formaliza en una utilidad llamada `cn`.

</details>

-----

## En entrevista

### Respuesta corta (junior)

En React se puede dar estilo con `style` (un objeto en camelCase), con hojas CSS y `className`, con CSS Modules, con Sass, con librerías CSS-in-JS como styled-components o con Tailwind. Los CSS Modules generan nombres de clase únicos para que los estilos de un componente no afecten a otros. Los estilos en línea sirven para valores dinámicos.

### Respuesta ampliada (semi-senior)

* **Aislamiento:** CSS normal es global. CSS Modules lo resuelve en build, CSS-in-JS genera nombres únicos en runtime y Tailwind evita nombrar clases.
* **Rendimiento:** CSS Modules y Tailwind producen CSS estático, cacheable y sin trabajo extra en el navegador. El CSS-in-JS en runtime serializa y inyecta estilos al renderizar, lo que añade costo, sobre todo con muchos componentes o actualizaciones frecuentes.
* **SSR y Server Components:** los enfoques estáticos funcionan igual en servidor y cliente. styled-components necesita `'use client'` y soporte de SSR, y su estado de mantenimiento lo hace poco recomendable para proyectos nuevos.
* **Mantenibilidad:** CSS Modules mantiene CSS estándar y separado del marcado. Tailwind reduce nombres y CSS muerto, pero requiere disciplina para no acumular cadenas de clases largas (se resuelve extrayendo componentes). El CSS-in-JS junta lógica y estilo, pero ata el proyecto a una librería.
* **Decisión típica:** Tailwind o CSS Modules por defecto, estilo en línea para valores dinámicos y variables CSS para temas. Sass solo si el proyecto ya lo usa o necesita sus funciones.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué `className` y no `class`?**
`class` es palabra reservada en JavaScript, y JSX se compila a JavaScript. La propiedad del DOM que representa el atributo se llama `className`.

**2. ¿Un `.css` importado desde un componente queda aislado a ese componente?**
No. El bundler lo incluye en la página y sus reglas son globales. El aislamiento requiere CSS Modules u otra técnica.

**3. ¿Cuándo conviene `style` frente a una clase?**
Cuando el valor es dinámico y no se conoce de antemano, por ejemplo un ancho calculado. Para estilos fijos, una clase es más eficiente y permite pseudoclases y media queries.

**4. ¿Qué diferencia hay entre Sass y CSS Modules?**
Resuelven problemas distintos: Sass es un lenguaje que se compila a CSS (variables, mixins), y CSS Modules aísla nombres de clase. Se combinan con `.module.scss`.

**5. ¿Por qué Tailwind no tiene costo en runtime?**
Porque analiza el código durante el build y genera un CSS estático con solo las clases usadas. El navegador recibe una hoja de estilos normal.

**6. ¿Se puede seguir usando styled-components?**
Sí, en proyectos existentes: sigue funcionando y recibe correcciones críticas. Sus mantenedores no recomiendan adoptarlo en proyectos nuevos, y con Server Components requiere `'use client'`.

-----

## Siguiente lección

Después de decidir cómo se ve un componente, el paso que sigue es entender **cuándo** se crea, se actualiza y se destruye: [Component Lifecycle Methods](03-Component%20Lifecycle%20Methods.md).
