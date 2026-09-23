# Introducción a JSX

## En una frase

JSX es una sintaxis que te deja escribir estructura de interfaz con forma de HTML dentro de JavaScript; una herramienta de compilación la transforma en llamadas a funciones que producen los **elementos de React**.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es una expresión en JavaScript (algo que produce un valor) y cómo se usan variables, objetos y arreglos.
* Qué es el DOM: la representación del documento HTML que el navegador expone a JavaScript.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **JSX:** extensión de sintaxis de JavaScript que permite escribir etiquetas con forma de HTML.
* **Elemento de React:** objeto ligero que describe qué debe mostrarse en pantalla. Un elemento JSX produce uno.
* **Compilar (transformar):** convertir JSX en JavaScript estándar antes de que llegue al navegador.
* **Raíz (root):** el punto del DOM donde React toma el control y dibuja la interfaz.
* **Renderizar:** hacer que React calcule y muestre en pantalla lo que describe un elemento.

-----

## El problema

Una interfaz mezcla estructura (qué se muestra) con lógica (cuándo y con qué datos). Separarlas en archivos distintos, HTML por un lado y JavaScript por otro, obliga a mantenerlas sincronizadas a mano. React propone agrupar por **componente** en lugar de por tecnología, y JSX es la sintaxis que permite escribir la estructura junto a la lógica que la controla.

Esto genera una duda legítima con esta línea:

```jsx
const heading = <h1>Hola mundo</h1>;
```

Empieza con `const` y termina con `;`, así que parece JavaScript. Pero contiene `<h1>`, que un intérprete de JavaScript no entiende. Tampoco es HTML: no funcionaría dentro de un archivo `.html`. La respuesta es que es **JSX**, y vive en un archivo JavaScript.

-----

## Cómo funciona

### JSX no es HTML ni JavaScript válido

JSX es una extensión de sintaxis. Un navegador no puede ejecutarla directamente; un compilador (Babel, SWC, esbuild o el propio TypeScript) la convierte antes en JavaScript estándar. Los proyectos creados con herramientas como Vite o Next.js ya traen esa configuración.

Con la transformación actual, esto:

```jsx
const heading = <h1 className="title">Hola mundo</h1>;
```

se convierte, de forma aproximada, en:

```js
import { jsx as _jsx } from 'react/jsx-runtime';

const heading = _jsx('h1', { className: 'title', children: 'Hola mundo' });
```

El resultado de esa llamada es un elemento de React, es decir, un objeto que describe `<h1>`. No es un nodo del DOM. Cómo React usa esos objetos para actualizar la pantalla se explica en [El DOM virtual](02-The%20Virtual%20Dom.md).

### Un elemento JSX es una expresión

Como cada elemento se transforma en una llamada a función, se comporta como cualquier expresión: puedes guardarlo en una variable, pasarlo a una función, meterlo en un objeto o en un arreglo.

```jsx
const navBar = <nav>Menú principal</nav>;

const team = {
  center: <li>Ana</li>,
  guard: <li>Luis</li>,
};

const items = [<li key="a">Uno</li>, <li key="b">Dos</li>];
```

(Los elementos dentro de un arreglo necesitan la prop `key`; se verá al trabajar con listas.)

### Atributos

Los elementos aceptan atributos con una sintaxis parecida a la de HTML. El valor puede ser un texto entre comillas o, entre llaves, cualquier expresión de JavaScript:

```jsx
const link = <a href="https://example.com">Ir al sitio</a>;
const photo = <img src="images/panda.jpg" alt="Un panda" width={500} height={500} />;
```

Las diferencias con HTML son deliberadas, porque JSX se traduce a propiedades de JavaScript:

| HTML | JSX |
| ---- | --- |
| `class` | `className` |
| `for` | `htmlFor` |
| `onclick` | `onClick` |
| `stroke-width` | `strokeWidth` |

Los atributos `aria-*` y `data-*` conservan el guion. Las llaves y sus reglas se estudian en [JSX avanzado](03-Advanced%20JSX.md).

### Etiquetas siempre cerradas

En JSX toda etiqueta debe cerrarse. Los elementos sin contenido usan autocierre:

```jsx
<img src="a.png" alt="" />
<input type="text" />
<br />
```

### Elementos anidados y varias líneas

Puedes anidar elementos igual que en HTML. Cuando la expresión ocupa varias líneas, es habitual envolverla entre paréntesis:

```jsx
const card = (
  <a href="https://example.com">
    <h1>Haz clic</h1>
  </a>
);
```

Los paréntesis no son sintaxis de JSX: son paréntesis de agrupación de JavaScript. Evitan que la inserción automática de punto y coma cause problemas, sobre todo con `return`. Se profundiza en [Varias líneas de JSX en un componente](04-Use%20Multiline%20JSX%20in%20a%20Component.md).

### Un solo elemento raíz

Una expresión JSX debe tener **un único elemento exterior**. Esto funciona:

```jsx
const paragraphs = (
  <div>
    <p>Soy un párrafo.</p>
    <p>Yo también.</p>
  </div>
);
```

Esto no:

```jsx
const paragraphs = (
  <p>Soy un párrafo.</p>
  <p>Yo también.</p>
);
```

La razón es que cada expresión JSX se convierte en una sola llamada que devuelve un solo objeto, y una función no puede devolver dos valores a la vez.

Si necesitas agrupar elementos pero no quieres añadir un `<div>` extra al DOM, usa un **fragmento**. Es un componente especial de React (`React.Fragment`) cuyo único trabajo es agrupar hijos sin crear ningún nodo en el HTML final. Tiene una forma corta, `<>...</>`, que es la que se usa casi siempre:

```jsx
const paragraphs = (
  <>
    <p>Soy un párrafo.</p>
    <p>Yo también.</p>
  </>
);
```

En el HTML resultante no aparece nada que represente ese fragmento: quedan los dos `<p>` directamente, sin ningún contenedor alrededor. Esto importa cuando la estructura le importa al CSS del padre (por ejemplo, un `display: grid` o `display: flex` que espera que sus hijos directos sean los elementos reales, no un `div` de más metido en el medio).

La forma corta `<>` no admite atributos. Si necesitas pasarle una `key` (por ejemplo, al generar una lista de fragmentos con `.map`), tienes que usar la forma larga:

```jsx
<React.Fragment key={item.id}>
  <dt>{item.term}</dt>
  <dd>{item.description}</dd>
</React.Fragment>
```

### Renderizar: mostrar JSX en pantalla

Escribir un elemento no lo muestra. Para eso hay que decirle a React dos cosas: **dónde** dibujar y **qué** dibujar.

```jsx
import { createRoot } from 'react-dom/client';

const container = document.getElementById('app');
const root = createRoot(container);
root.render(<h1>Hola mundo</h1>);
```

* `document.getElementById('app')` obtiene el nodo del DOM que servirá de contenedor.
* `createRoot(container)` crea una raíz de React sobre ese nodo. Responde a "dónde".
* `root.render(...)` recibe lo que se quiere mostrar. Responde a "qué".

El argumento de `render` puede ser un elemento directo o una variable que lo contenga:

```jsx
const toDoList = (
  <ol>
    <li>Aprender React</li>
    <li>Conseguir un empleo</li>
  </ol>
);

root.render(toDoList);
```

`createRoot` es la API vigente desde React 18. La antigua `ReactDOM.render` fue eliminada en React 19.

Si llamas a `root.render` varias veces, React conserva lo que no cambió y solo actualiza las diferencias en el DOM. Ese mecanismo, el DOM virtual, se explica en la [lección siguiente](02-The%20Virtual%20Dom.md).

-----

## Ejemplo completo

```jsx
import { createRoot } from 'react-dom/client';

const user = { name: 'Ana', avatar: 'images/ana.png' };

const profile = (
  <section className="profile">
    <img src={user.avatar} alt={`Foto de ${user.name}`} width={80} height={80} />
    <h2>{user.name}</h2>
    <p>Estudiando React.</p>
  </section>
);

const container = document.getElementById('app');
const root = createRoot(container);
root.render(profile);
```

Cada elemento es una expresión, hay un solo elemento raíz (`section`), las etiquetas están cerradas, se usa `className` y los valores dinámicos van entre llaves.

-----

## Errores comunes

**1. Devolver dos elementos sin envolver.**
Qué pasa: error de compilación (por ejemplo, "Adjacent JSX elements must be wrapped in an enclosing tag").
Por qué: una expresión JSX debe producir un solo elemento.
Arreglo: envuelve en un elemento contenedor o en un fragmento `<>...</>`.

**2. Usar `class` en lugar de `className`.**
Qué pasa: React muestra una advertencia y, según la versión, la clase puede no aplicarse como esperas.
Por qué: JSX usa los nombres de las propiedades del DOM en camelCase.
Arreglo: `className`.

**3. Olvidar cerrar una etiqueta.**
Qué pasa: error de compilación.
Por qué: JSX es más estricto que HTML y no admite etiquetas sin cerrar.
Arreglo: `<img />`, `<br />`, `<input />`.

**4. `createRoot(null)`.**
Qué pasa: error "Target container is not a DOM element".
Por qué: `getElementById` no encontró el elemento (id mal escrito o script ejecutado antes de que exista el nodo).
Arreglo: revisa el id y el orden de carga del script.

**5. Pasar el componente en lugar del nodo.**
Qué pasa: el mismo error anterior.
Por qué: `createRoot` recibe un nodo del DOM, no JSX.
Arreglo: `createRoot(domNode)` y luego `root.render(<App />)`.

-----

## En TypeScript

Un archivo que usa JSX con TypeScript lleva la extensión `.tsx`, no `.ts`. Además, `tsconfig.json` necesita la opción `"jsx"` (por ejemplo `"react-jsx"`). Se explica en [Archivo tsconfig](../00-typescript-fundamentals/02-Archivo%20tsconfig.md).

Tipos relevantes en esta lección:

* Una expresión JSX guardada en una variable tiene tipo `React.JSX.Element`. No hace falta anotarlo: TypeScript lo infiere. En React 19 el espacio de nombres `JSX` global fue retirado de los tipos; si necesitas escribirlo, usa `React.JSX.Element`.
* Para tipar "cualquier cosa que React pueda renderizar" (elementos, texto, números, `null`, arreglos), el tipo es `React.ReactNode`. Es más amplio que `JSX.Element`.
* `document.getElementById()` devuelve `HTMLElement | null`, mientras que `createRoot` exige un nodo no nulo. Dos formas habituales de resolverlo:

```tsx
// Aserción de no nulo: asumes que el elemento existe
const root = createRoot(document.getElementById('app')!);

// Más seguro: comprobar y fallar con un mensaje claro
const container = document.getElementById('app');
if (!container) throw new Error('No se encontró el elemento #app');
createRoot(container).render(<h1>Hola mundo</h1>);
```

El compilador también rechaza etiquetas sin cerrar o varios elementos raíz, y el editor lo marca antes de ejecutar. Se retoma en [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).

-----

## Cuándo sí y cuándo no

**Úsalo cuando:**

* Describes qué se muestra en función de datos: JSX mantiene estructura y lógica juntas.
* Quieres que el compilador detecte errores de estructura antes de ejecutar.

**Ten en cuenta:**

* JSX es opcional. React puede usarse con `createElement` o `jsx()` directamente, aunque casi nadie lo hace por legibilidad.
* JSX no es una plantilla de texto: no puedes escribir cualquier cosa de HTML y esperar que funcione igual (atributos, cierre de etiquetas, `class`).

-----

## Resumen en 5 líneas

1. JSX es una sintaxis que se compila a llamadas `jsx()`; no es HTML y el navegador no la entiende sin compilar.
2. Cada elemento JSX es una expresión y produce un elemento de React (un objeto descriptivo).
3. Atributos en camelCase (`className`, `onClick`), etiquetas siempre cerradas.
4. Una expresión JSX tiene un único elemento raíz; usa un fragmento `<>...</>` para no añadir nodos.
5. `createRoot(container).render(jsx)` indica dónde y qué mostrar; en TypeScript se usa `.tsx`.

-----

## Para profundizar

<details>
<summary>Qué hace realmente el compilador</summary>

Desde React 17 existe la "nueva transformación de JSX", que inserta automáticamente las importaciones desde `react/jsx-runtime`. Por eso ya no es necesario escribir `import React from 'react'` en cada archivo solo para usar JSX. El resultado de `jsx()` es un objeto simple con propiedades como `type` y `props`, que React lee para decidir qué mostrar.

</details>

<details>
<summary>Por qué "un solo elemento raíz"</summary>

No es una regla arbitraria: viene de JavaScript. Una función devuelve un solo valor, y cada expresión JSX se convierte en una sola llamada. Los fragmentos existen para cumplir esa regla sin añadir un nodo extra al DOM.

</details>

-----

## En entrevista

### Respuesta corta (junior)

JSX es una extensión de sintaxis de JavaScript que permite escribir estructura parecida a HTML dentro del código. No es HTML: un compilador la transforma en llamadas a funciones que crean elementos de React. Cada elemento JSX es una expresión, y debe tener un único elemento raíz.

### Respuesta ampliada (semi-senior)

JSX es azúcar sintáctico: `<h1 className="t">Hola</h1>` se compila a `jsx('h1', { className: 't', children: 'Hola' })`, que devuelve un objeto plano (un elemento de React) que describe la UI. Ese objeto no es un nodo del DOM; React lo compara con el anterior para decidir qué actualizar. Como es una expresión, se puede almacenar, pasar y devolver como cualquier valor, y por eso un componente puede devolver JSX. Las reglas (raíz única, etiquetas cerradas, atributos en camelCase) se derivan de que se traduce a JavaScript. Para montar la app se usa `createRoot(container).render(...)`, que sustituye a `ReactDOM.render`, eliminada en React 19. En TypeScript se usa `.tsx` y la opción `jsx` de `tsconfig`.

### Preguntas frecuentes de seguimiento

**1. ¿JSX es obligatorio para usar React?**
No. Se puede usar `createElement` o `jsx()` directamente, pero JSX es mucho más legible y es el estándar.

**2. ¿Por qué `className` y no `class`?**
JSX se traduce a propiedades de JavaScript y `class` es una palabra reservada; además, React usa los nombres de propiedades del DOM en camelCase.

**3. ¿Para qué sirven los fragmentos?**
Para agrupar varios elementos bajo un único elemento raíz sin añadir un nodo extra al DOM.

**4. ¿Un elemento JSX es un nodo del DOM?**
No. Es un objeto que describe lo que debería mostrarse. React lo usa para crear o actualizar el DOM real.

**5. ¿Qué diferencia hay entre `JSX.Element` y `ReactNode`?**

| | `JSX.Element` | `ReactNode` |
| --- | --- | --- |
| Qué es | El tipo de una expresión JSX evaluada | Todo lo que React puede renderizar como hijo |
| Qué acepta | Solo `<Algo />` | JSX, string, number, boolean, `null`, `undefined`, arreglos de todo eso |
| Relación | Es un subconjunto de `ReactNode` | Incluye a `JSX.Element` y mucho más |

Todo `JSX.Element` es `ReactNode`, pero no al revés: un `string` es un `ReactNode` válido y no es un `JSX.Element`. Por eso `children` casi siempre se tipa como `ReactNode` (acepta texto, `null` de un `&&` condicional, arreglos), y `JSX.Element` casi no se usa salvo que necesites forzar que algo sea exactamente un elemento JSX.

**6. ¿Por qué ya no hace falta importar React para usar JSX?**
Porque la nueva transformación (React 17 en adelante) importa automáticamente `jsx` desde `react/jsx-runtime`.

-----

## Siguiente lección

[El DOM virtual](02-The%20Virtual%20Dom.md)
