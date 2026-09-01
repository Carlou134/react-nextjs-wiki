# Your First React Component

## React Components

Las aplicaciones de React están hechas de **componentes**.

Un componente es una pequeña pieza de código reutilizable que se encarga de una sola tarea. Esa tarea suele ser renderizar algo de HTML y volver a renderizarlo cuando cambian algunos datos.

Mira el código de abajo. Este código creará y renderizará un nuevo componente de React:

```js
import { createRoot } from 'react-dom/client';

function MyComponent() {
  return <h1>Hello world</h1>;
}

createRoot(document.getElementById('app')).render(<MyComponent />);
```

Muchas de estas cosas pueden parecer desconocidas, pero no te preocupes. Vamos a descomponer ese código, una pequeña parte a la vez. ¡Al final de esta lección, entenderás cómo construir un componente en React!

-----

## Import React and createRoot

Podemos importar la librería **React** siempre que nuestro código necesite algo del paquete `react`. Por ejemplo:

```js
import React from 'react';
```

Esto nos da acceso al objeto **React** y a las utilidades que proporciona. En React moderno, no es necesario importar React solo para escribir **JSX**, pero aun así podemos importarlo siempre que necesitemos alguna característica o función del paquete React.

Además de escribir componentes, también necesitamos una forma de conectar nuestra aplicación con la página. Para hacerlo, usamos la función **createRoot** del paquete `react-dom/client`. Así es como la importamos y la usamos:

```js
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('app')).render(<App />);
```


Usando **createRoot**, conectamos nuestro componente principal (normalmente `App`) a un elemento del archivo HTML para que nuestra aplicación de React pueda aparecer en la página.

-----

## Import ReactDOM

Otra importación que necesitamos, además de React, es **ReactDOM**:

```js
import ReactDOM from 'react-dom/client';
```

Los métodos importados desde `react-dom` interactúan con el **DOM**.

Los métodos importados desde `react` no trabajan con el DOM en absoluto. No interactúan directamente con nada que no sea parte de React.

Para aclarar: el DOM se usa en aplicaciones de React, pero **no es parte de React**. Después de todo, el DOM también se utiliza en muchísimas aplicaciones que no usan React. Los métodos importados desde `react` son solo para propósitos propios de React, como crear componentes o escribir elementos **JSX**.

-----

## Create a Function Component

Has aprendido que un componente de React es una pequeña pieza de código reutilizable que se encarga de una sola tarea, la cual a menudo implica renderizar HTML y volver a renderizarlo cada vez que cambian algunos datos.

Es útil pensar en los componentes como partes más pequeñas de nuestra interfaz. En conjunto, son los bloques de construcción que forman una aplicación de React. En un sitio web, podemos crear un componente para la barra de búsqueda, otro componente para la barra de navegación y otro componente para el contenido del panel principal.

Aquí hay otro dato sobre los componentes: podemos usar funciones de JavaScript para definir un nuevo componente de React. A esto se le llama **componente funcional**.

En el pasado, los componentes de React se definían usando clases de JavaScript. Pero desde la introducción de los **Hooks** (algo de lo que hablaremos más adelante), los componentes funcionales se han convertido en el estándar en las aplicaciones modernas de React.

Después de definir nuestro componente funcional, podemos usarlo para crear tantas instancias de ese componente como queramos.

Veamos el ejemplo del primer ejercicio:

```js
function MyComponent() {
  return <h1>Hello, I'm a functional React Component!</h1>;
}

export default MyComponent;
```

En la tercera línea, se define una función con el nombre **MyComponent**. Dentro de ella, la función devuelve un elemento de React en sintaxis **JSX**:

```js
return <h1>Hello, I'm a functional React Component!</h1>;
```

En conjunto, esto forma un componente funcional básico de React.

En la última línea del bloque de código anterior, **MyComponent** se exporta para que pueda usarse más adelante.

Mucho de esto todavía puede parecer desconocido, ¡pero ya entiendes más de lo que entendías antes! ¡Sigamos adelante! 🚀

-----

## Name a Functional Component

¡Bien! Crear una función de JavaScript es la forma de declarar un nuevo componente funcional.

Cuando declaras un nuevo componente funcional, necesitas darle un nombre a ese componente. En nuestro componente terminado, el nombre era **MyComponent**:

```js
function MyComponent() {
  return <h1>Hello world</h1>;
}
```

Los nombres de los componentes funcionales deben comenzar con mayúscula y, por convención, se crean usando **PascalCase**. Debido a la forma en que se compilan las etiquetas **JSX**, el uso de mayúsculas indica que se trata de un componente de React y no de una etiqueta HTML.

¡Este es un detalle específico de React! Si estás creando un componente, asegúrate de que su nombre comience con una letra mayúscula para que React lo interprete como un componente. Si comienza con una letra minúscula, React intentará buscar un componente integrado como `div` o `input` y fallará.

-----

## Function Component Instructions

¡Repasemos lo que hemos aprendido hasta ahora! Mira dentro de **App.js** y **index.js** y encuentra cada uno de estos puntos:

* En **App.js**, podemos importar React siempre que nuestro componente necesite algo del paquete `react`. En React moderno, no es obligatorio importar React solo para escribir **JSX**, pero aun así podemos importarlo cuando necesitemos alguna funcionalidad de la librería React.
* En **index.js**, importamos la función **createRoot** desde `'react-dom/client'`. Esta función permite que React conecte nuestra aplicación con el DOM del navegador.
* En **App.js**, definimos un componente funcional escribiendo una función normal de JavaScript. Un componente funcional es como una receta: no muestra nada por sí solo hasta que lo renderizamos en el DOM.
* Cada vez que creamos un componente funcional, necesitamos darle un nombre escrito en **PascalCase** (UpperCamelCase), como `MyComponent`.
* Algo que todavía no hemos comentado es el cuerpo de tu componente funcional: las llaves que van después de la declaración de la función y todo el código que hay dentro de ellas.

Como cualquier función de JavaScript, un componente necesita un cuerpo. Este cuerpo contiene las instrucciones que le dicen a React qué debe mostrar el componente.

Aquí está el cuerpo de la función de **App.js**:

```js
return <h1>Hello, this is a function component body.</h1>;
```

Puede parecer una simple línea de JSX, pero esta línea es la instrucción que le dice a React exactamente qué debe renderizar el componente.

-----

## The Return Keyword in Functional Components

Cuando definimos un componente funcional, básicamente estamos definiendo una **fábrica** que puede construir la combinación adecuada de elementos cada vez que hacemos referencia a su nombre. Lo hace consultando un conjunto de instrucciones que tú debes proporcionar.

Si estás pensando: *“Eso suena exactamente a para qué sirve una función normal de JavaScript”*, entonces tienes razón. Los componentes funcionales pueden entenderse de forma muy similar a las funciones normales de JavaScript, excepto que su trabajo es ensamblar una parte de la interfaz basándose en las instrucciones dadas.

Hablemos un poco más sobre estas instrucciones.

Para empezar, estas instrucciones deben tomar la forma del cuerpo de una función. Eso significa que estarán delimitadas por llaves, como en este ejemplo:

```js
function Button() {
  // Las instrucciones van aquí, entre las llaves.
}
```

Nuestras instrucciones pueden incluir una combinación de marcado, CSS y JavaScript para producir el resultado deseado. La única cosa que siempre debemos incluir es una sentencia **return**.

Se espera que la función produzca código **JSX** que pueda usarse para renderizar algo en la pantalla del navegador. Por lo tanto, cuando definimos componentes funcionales, debemos devolver un elemento JSX.

```js
function BackButton() {
  return <button>Back To Home</button>;
}
```

Por supuesto, esto todavía no hace que `<button>Back To Home</button>` se muestre en la pantalla del navegador. Solo hemos definido nuestro componente.

¡Sigamos adelante para ver cómo renderizarlo y por qué la sentencia `return` era necesaria! 🚀

------

## Importing and Exporting React Components

Hay un poco más de trabajo que debemos hacer antes de poder usar nuestro componente definido y lograr que se renderice en el DOM.

Mencionamos anteriormente que una aplicación de React normalmente tiene dos archivos principales: **App.js** e **index.js**. El archivo **App.js** es el nivel superior de tu aplicación, y **index.js** es el punto de entrada.

Hasta ahora, hemos definido el componente dentro de **App.js**, pero como **index.js** es el punto de entrada, tenemos que exportarlo a **index.js** para poder renderizarlo.

Los componentes en React son geniales porque son reutilizables. Podemos mantener las piezas de nuestros componentes separadas, organizadas y reutilizables colocándolas en archivos separados y exportándolas donde las necesitemos.

Para exportarlos, podemos anteponer la declaración **export** y especificar si se trata de una exportación por defecto o con nombre. En este caso, usaremos la exportación por defecto. Si necesitas repasar cómo funcionan las exportaciones, puedes consultar la documentación web de MDN.

Después de la definición del componente funcional, en **App.js**, podemos exportar nuestro componente por defecto de esta manera:

```js
export default MyComponent;
```

Luego, podemos ir a nuestro archivo **index.js** para importar el componente desde `'./App'`:

```js
import MyComponent from './App';
```

Esto nos permitirá usar **MyComponent** en **index.js**.

----

## Using and Rendering a Component

Ahora que ya tenemos un componente funcional definido, podemos empezar a usarlo.

Podemos usarlo con una sintaxis similar a HTML que se parece a una etiqueta autocerrada:

```js
<MyComponent />
```

Si necesitas anidar otros componentes dentro, también puedes usar una etiqueta de apertura y cierre:

```js
<MyComponent>
  <OtherComponent />
</MyComponent>
```

Sin embargo, para mostrar nuestro componente en el navegador, debemos usar los métodos **createRoot()** y **render()** de la librería `react-dom/client`. Esto se hace en nuestro archivo de entrada, normalmente **index.js**.

### Renderizado en React 19+

Primero, llamamos a **createRoot()** para crear una raíz de React. Una aplicación de React generalmente tiene un único elemento raíz en el DOM, y React gestiona todo lo que hay dentro de él.

```js
import { createRoot } from "react-dom/client";
```

Luego, pasamos un elemento del DOM desde **index.html** a **createRoot()**:

```js
const root = createRoot(document.getElementById("app"));
```

Veámoslo paso a paso:

* `document.getElementById("app")` selecciona un elemento del DOM desde **index.html**.
* `createRoot()` recibe ese elemento y lo convierte en una raíz de React.
* `createRoot()` devuelve un objeto con un método `.render()` que podemos usar para renderizar nuestro componente.

Finalmente, renderizamos nuestro componente:

```js
createRoot(document.getElementById("app")).render(<MyComponent />);
```

A partir de este punto, React toma el control de la interfaz de usuario dentro de la raíz. En una aplicación típica de React, solo se configura la raíz una vez, y todos los componentes adicionales se agregan a través de tu componente principal **App.js**.

-----

## Review

En esta lección, has aprendido un concepto fundamental de React: **los componentes**.

Antes de terminar, aquí tienes un resumen:

* Las aplicaciones de React se construyen a partir de componentes.
* Los componentes son responsables de renderizar partes de la interfaz de usuario.
* Para crear componentes, importamos desde `react` solo cuando es necesario (por ejemplo, hooks).
* Para renderizar componentes en el navegador, importamos **createRoot** desde `react-dom/client`.
* Los componentes de React pueden definirse usando funciones normales de JavaScript, conocidas como **componentes funcionales**.
* Los nombres de los componentes funcionales deben comenzar con una letra mayúscula, y **PascalCase** es la convención estándar de nombres.
* Los componentes funcionales deben devolver elementos de React escritos en **JSX**.
* Los componentes pueden exportarse e importarse entre archivos para mantener el código organizado y reutilizable.
* Un componente de React puede usarse como una etiqueta similar a HTML, a menudo como un elemento autocerrado.
* Para renderizar un componente de React, es necesario llamar a **createRoot()** para especificar un nodo raíz del DOM y luego llamar a **.render()** sobre la raíz devuelta.

¡Uf! Fue bastante información, pero los componentes están en el corazón de React y son una gran parte de lo que hace que React sea una herramienta tan poderosa 🚀

-----