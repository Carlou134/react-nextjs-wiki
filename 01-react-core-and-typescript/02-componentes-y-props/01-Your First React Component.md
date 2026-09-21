# Tu primer componente de React

## En una frase

Un componente de React es una función de JavaScript cuyo nombre empieza con mayúscula y que devuelve JSX; se exporta desde su archivo y se monta en la página con `createRoot(...).render(<Componente />)`.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es JSX y cómo se escribe: [fundamentos de JSX](../01-fundamentos-jsx/README.md).
* Cómo funcionan `import` y `export` en módulos de JavaScript.

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Componente:** pieza reutilizable de interfaz. En React moderno se define como una función.
* **Raíz (root):** punto de la página donde React toma el control de la interfaz.
* **Renderizar:** producir la interfaz que describe un componente y mostrarla en pantalla.
* **PascalCase:** convención donde cada palabra del nombre empieza con mayúscula (`MyComponent`).

-----

## El problema

Una interfaz real se compone de muchas partes: barra de navegación, buscador, lista, formulario. Si todo vive en un solo bloque de HTML, es difícil reutilizarlo y mantenerlo. Los componentes permiten dividir la interfaz en piezas independientes, cada una con una responsabilidad clara, y componerlas entre sí.

-----

## Cómo funciona

### 1. Un componente es una función que devuelve JSX

```jsx
function MyComponent() {
  return <h1>Hola mundo</h1>;
}
```

La función no muestra nada por sí sola. Solo **describe** qué interfaz corresponde. React la ejecuta cuando el componente se usa y convierte el resultado en elementos del DOM.

Antes de los Hooks (React 16.8) también existían componentes de clase. Hoy los componentes de función son la forma estándar de escribir componentes nuevos.

### 2. El nombre empieza con mayúscula

Los nombres de componentes van en PascalCase: `MyComponent`, `NavBar`, `UserCard`. No es solo estilo. React usa la primera letra para distinguir:

* `<section />` (minúscula): una etiqueta HTML.
* `<Profile />` (mayúscula): un componente.

Si nombras un componente en minúscula, React lo trata como una etiqueta HTML y no lo encuentra como componente.

### 3. El `return` es obligatorio

El cuerpo de la función puede tener cualquier código de JavaScript, pero debe terminar devolviendo lo que se va a mostrar (normalmente JSX). Si el JSX ocupa varias líneas, se envuelve en paréntesis abiertos en la misma línea del `return`:

```jsx
function BackButton() {
  return (
    <button>
      Volver al inicio
    </button>
  );
}
```

Sin esos paréntesis, JavaScript inserta un punto y coma tras `return` y el JSX de la línea siguiente nunca se ejecuta (ver "Errores comunes").

### 4. Exportar e importar

Cada componente suele vivir en su propio archivo. Para usarlo desde otro, se exporta y se importa:

```jsx
// App.jsx
export default function App() {
  return <h1>Hola mundo</h1>;
}
```

```jsx
// main.jsx
import App from './App';
```

Con `export default`, el archivo expone un valor principal y quien importa elige el nombre. Por convención, se importa con el mismo nombre del componente. También existe la exportación con nombre (`export function App() {}` e `import { App } from './App'`), donde el nombre importado debe coincidir.

### 5. Usarlo como una etiqueta

Un componente se usa escribiendo su nombre como etiqueta JSX. Sin contenido dentro, se escribe autocerrado:

```jsx
<MyComponent />
```

### 6. Montarlo en la página con `createRoot`

`createRoot` viene del paquete `react-dom/client`, porque conecta React con el DOM del navegador. Los paquetes se reparten así:

* `react`: lo propio de React (hooks, JSX, definición de componentes). No toca el DOM.
* `react-dom`: la conexión con el DOM del navegador. El DOM existe sin React; por eso esta parte va aparte.

```jsx
import { createRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');
const root = createRoot(container);
root.render(<App />);
```

Paso a paso:

1. `document.getElementById('root')` obtiene un elemento que ya existe en el HTML.
2. `createRoot(container)` crea una raíz de React sobre ese elemento.
3. `root.render(<App />)` le indica a React qué mostrar dentro de la raíz.

A partir de ahí, React administra todo lo que hay dentro de ese elemento. Una aplicación normal crea la raíz una sola vez y el resto de componentes se agregan desde `App`.

Sobre importar `React`: con el JSX transform actual (React 17 en adelante) no hace falta `import React from 'react'` para escribir JSX. Solo se importa desde `react` lo que realmente se usa, por ejemplo `useState`.

-----

## Ejemplo completo

`index.html` (fragmento):

```html
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
```

`src/Greeting.jsx`:

```jsx
export default function Greeting() {
  return (
    <section>
      <h1>Hola, soy un componente de función</h1>
      <p>Me definí una vez y puedo usarme cuantas veces quieras.</p>
    </section>
  );
}
```

`src/main.jsx`:

```jsx
import { createRoot } from 'react-dom/client';
import Greeting from './Greeting';

createRoot(document.getElementById('root')).render(<Greeting />);
```

Flujo: el navegador carga el HTML, ejecuta `main.jsx`, se crea la raíz sobre `#root` y React inserta el resultado de `Greeting` dentro de ese elemento.

-----

## Errores comunes

### 1. Nombre en minúscula

```jsx
function greeting() {
  return <h1>Hola</h1>;
}

<greeting />
```

**Qué pasa:** React busca una etiqueta HTML llamada `greeting`; el componente no se renderiza como tal.
**Por qué:** la primera letra decide si la etiqueta es HTML o componente.
**Solución:** renombra a `Greeting`.

### 2. `return` sin paréntesis con JSX en otra línea

```jsx
function Bad() {
  return
    <h1>Hola</h1>;
}
```

**Qué pasa:** el componente devuelve `undefined` y no se muestra nada.
**Por qué:** la inserción automática de punto y coma de JavaScript convierte `return` en `return;`.
**Solución:** abre el paréntesis en la misma línea: `return (`.

### 3. Olvidar el `return`

```jsx
function Bad() {
  <h1>Hola</h1>;
}
```

**Qué pasa:** el componente no muestra nada. La función devuelve `undefined`, que no es una interfaz válida para renderizar.
**Por qué:** una función sin `return` devuelve `undefined`.
**Solución:** agrega `return`.

### 4. Olvidar exportar o importar

**Qué pasa:** error del tipo `X is not defined` o el import falla.
**Por qué:** los módulos son privados por defecto. Un componente no exportado no existe fuera de su archivo.
**Solución:** agrega `export` en el archivo del componente e impórtalo donde se use. Verifica que el tipo de exportación (por defecto o con nombre) coincida con el tipo de import.

### 5. Usar `ReactDOM.render`

```jsx
ReactDOM.render(<App />, document.getElementById('root'));
```

**Qué pasa:** en React 19 no existe.
**Por qué:** se eliminó en favor de `createRoot`, introducido en React 18.
**Solución:** usa `createRoot` desde `react-dom/client`, como en el ejemplo.

### 6. Definir un componente dentro de otro

**Qué pasa:** errores de estado y rendimiento.
**Por qué:** cada render del padre crea una función nueva y React la trata como un componente distinto.
**Solución:** define cada componente en el nivel superior del archivo, y pasa datos con props (ver [Props](03-Props.md)).

-----

## En TypeScript

Un componente sin props no necesita anotaciones. El tipo de retorno se infiere:

```tsx
export default function Greeting() {
  return <h1>Hola</h1>;
}
```

En proyectos y tutoriales antiguos verás `React.FC`. Hoy se recomienda evitarlo: la función simple es más clara y permite tipar `children` solo en los componentes que lo reciben. Las props se tipan en [Props](03-Props.md).

Al montar la raíz, `document.getElementById()` devuelve `HTMLElement | null`, mientras que `createRoot` espera un `HTMLElement`. Hay dos formas de resolverlo:

```tsx
// 1. Aserción de no-nulo: le dices al compilador que el elemento existe.
createRoot(document.getElementById('root')!).render(<App />);

// 2. Verificación explícita: falla con un mensaje claro si no existe.
const container = document.getElementById('root');
if (!container) throw new Error('No se encontró el elemento #root');
createRoot(container).render(<App />);
```

El `!` es aceptable aquí porque el elemento está en tu propio `index.html`, pero solo es una promesa al compilador: si el elemento no existe, fallará en ejecución. La segunda opción da un error más legible.

-----

## Cuándo sí y cuándo no

* **Un componente por responsabilidad:** extrae un componente cuando una pieza de interfaz tiene identidad propia o se repite.
* **Un archivo por componente:** es la convención más común; facilita encontrar y reutilizar el código.
* **`export default` o exportación con nombre:** ambas son válidas. La exportación con nombre obliga a usar el mismo nombre al importar; la de por defecto permite elegirlo. Sigue la convención de tu equipo.
* **No crees más de una raíz sin motivo:** una aplicación normal tiene una sola. Varias raíces se usan al integrar React en una página que no es de React.

-----

## Resumen en 5 líneas

1. Un componente es una función que devuelve JSX y describe una parte de la interfaz.
2. Su nombre va en PascalCase; la mayúscula lo distingue de una etiqueta HTML.
3. El `return` es obligatorio; si el JSX ocupa varias líneas, va entre paréntesis.
4. Se exporta desde su archivo, se importa donde se necesite y se usa como `<Componente />`.
5. Se muestra en la página con `createRoot(elemento).render(<App />)`, de `react-dom/client`.

-----

## Para profundizar

<details>
<summary>Por qué la mayúscula cambia el significado</summary>

El JSX se transforma en llamadas a funciones que reciben el "tipo" del elemento. Si la etiqueta empieza con minúscula, el tipo es una cadena (`'section'`) y React crea un elemento del DOM. Si empieza con mayúscula, el tipo es la referencia a una variable (`Profile`) y React llama a esa función. Por eso el nombre debe estar en el alcance y escrito con mayúscula inicial.

</details>

<details>
<summary>Volver a llamar a render sobre la misma raíz</summary>

La documentación de React indica que llamar a `render` de nuevo sobre la misma raíz es válido: React actualiza el DOM para reflejar el nuevo JSX. Es poco común, porque normalmente los componentes cambian su interfaz mediante estado y no volviendo a llamar a `render`.

</details>

<details>
<summary>Componentes de clase</summary>

Antes de React 16.8 el estado y el ciclo de vida solo estaban disponibles en componentes de clase. Los Hooks permitieron hacerlo en funciones y hoy los componentes de función son la opción estándar. Los de clase siguen soportados, pero no se recomiendan para código nuevo.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Un componente de React es una función de JavaScript que devuelve JSX y describe una parte de la interfaz. Su nombre empieza con mayúscula para que React lo distinga de una etiqueta HTML. Se exporta desde su archivo y se monta en la página con `createRoot` y `render`.

### Respuesta ampliada (semi-senior)

* **Definición:** una función que recibe props y devuelve lo que React debe renderizar. Debe ser pura respecto a sus entradas: mismas props, mismo resultado.
* **Nombre con mayúscula:** JSX distingue tipo cadena (etiqueta del DOM) y tipo referencia (componente) según la primera letra.
* **Punto de entrada:** `createRoot` de `react-dom/client` crea la raíz sobre un nodo del DOM; `render` indica el árbol a mostrar. Sustituye a `ReactDOM.render`, eliminado en React 19.
* **Separación de paquetes:** `react` define componentes y hooks sin depender del DOM; `react-dom` los conecta con el navegador. Esa separación permite otros renderizadores (por ejemplo, React Native).
* **Módulos:** cada componente se exporta e importa como cualquier módulo ES.
* **Definición en el nivel superior:** declarar un componente dentro de otro lo recrea en cada render y hace que React lo trate como un tipo distinto.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué el nombre de un componente debe empezar con mayúscula?**
Porque JSX usa esa letra para decidir si es una etiqueta HTML (minúscula) o un componente (mayúscula).

**2. ¿Qué devuelve un componente?**
Normalmente JSX, pero también puede devolver `null` para no mostrar nada. Si la función no tiene `return`, devuelve `undefined`, que suele indicar un olvido.

**3. ¿Hay que importar React para escribir JSX?**
No con el JSX transform actual (React 17 en adelante). Solo se importa desde `react` lo que se use, como hooks.

**4. ¿Qué diferencia hay entre `react` y `react-dom`?**
`react` define componentes y hooks sin conocer el DOM. `react-dom` conecta React con el DOM del navegador.

**5. ¿Qué reemplaza `createRoot` y desde cuándo?**
Reemplaza a `ReactDOM.render`. Se introdujo en React 18 y `render` se eliminó en React 19.

**6. ¿`export default` o exportación con nombre?**
Ambas funcionan. La de por defecto deja elegir el nombre al importar; la de nombre obliga a usar el mismo. Depende de la convención del proyecto.

-----

## Siguiente lección

Ya sabes definir, exportar y montar un componente. Ahora se usa un componente dentro de otro: [Los componentes renderizan otros componentes](02-Components%20Render%20Other%20Components.md). Después seguirá [Props](03-Props.md), y más adelante [The State Hook](../04-hooks-y-context/01-The%20State%20Hook.md).
