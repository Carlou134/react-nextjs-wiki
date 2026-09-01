# Intro to JSX

## Why React?

React.js es una biblioteca de JavaScript desarrollada por ingenieros de Facebook. Estas son solo algunas de las razones por las que las personas eligen programar con React:

* **React es rápido.** Las aplicaciones hechas con React pueden manejar actualizaciones complejas y aun así sentirse ágiles y responsivas.
* **React es modular.** En lugar de escribir archivos de código grandes y densos, puedes crear muchos archivos más pequeños y reutilizables. La modularidad de React puede ser una solución elegante a los problemas de mantenibilidad de JavaScript.
* **React es escalable.** Los programas grandes que muestran muchos datos cambiantes son donde React se desempeña mejor.
* **React es flexible.** Puedes usar React para proyectos interesantes que no tienen nada que ver con crear una aplicación web. La gente todavía está descubriendo el potencial de React. Hay mucho espacio para explorar.
* **React es popular.** Aunque esta razón tiene poco que ver con la calidad de React, la verdad es que entender React te hará más empleable.

Si eres nuevo en React, entonces este curso es para ti: no se espera ningún conocimiento previo de React. Comenzaremos desde lo más básico y avanzaremos lentamente. Al final, estarás listo para programar en React con una comprensión real de lo que estás haciendo.

-----

## Hello World

Observa la siguiente línea de código:

```js
const h1 = <h1>Hello world</h1>;
```

¿Qué tipo de código híbrido extraño es ese? ¿Es JavaScript? ¿HTML? ¿O algo diferente?

Parece que debe ser JavaScript, ya que comienza con `const` y termina con `;`. Si intentaras ejecutarlo en un archivo HTML, no funcionaría.

Sin embargo, el código también contiene `<h1>Hello world</h1>`, que se ve exactamente como HTML. Esa parte no funcionaría si intentaras ejecutarla en un archivo JavaScript.

¿Qué está pasando?

-----

## The Mystery, Revealed

Vuelve a observar la línea de código que escribiste. ¿Este código pertenece a un archivo JavaScript, a un archivo HTML o a algún otro lugar?

La respuesta es… ¡un archivo JavaScript! A pesar de lo que parece, tu código en realidad no contiene nada de HTML.

La parte que parece HTML, `<h1>Hello world</h1>`, es algo llamado **JSX**.

Haz clic en **Next** para aprender sobre JSX.

-----

## What is JSX?

**JSX** es una extensión de sintaxis para JavaScript. Fue creada para usarse con React. El código JSX se parece mucho al HTML.

¿Qué significa “extensión de sintaxis”?

En este caso, significa que JSX no es JavaScript válido. ¡Los navegadores web no pueden leerlo!

Si un archivo JavaScript contiene código JSX, entonces ese archivo tendrá que ser compilado. Esto significa que, antes de que el archivo llegue a un navegador web, un compilador de JSX traducirá cualquier JSX a JavaScript normal.

Los servidores de Codecademy ya tienen un compilador de JSX instalado, así que por ahora no tienes que preocuparte por eso. Más adelante veremos cómo configurar un compilador de JSX en tu computadora personal.

----

## JSX Elements

Una unidad básica de **JSX** se llama **elemento JSX**.

Aquí tienes un ejemplo de un elemento JSX:

```jsx
<h1>Hello world</h1>
```

¡Este elemento JSX se ve exactamente igual que HTML! La única diferencia notable es que lo encontrarías en un archivo JavaScript, en lugar de en un archivo HTML.

-----

## JSX Elements And Their Surroundings

Los **elementos JSX** se tratan como **expresiones de JavaScript**. Pueden ir en cualquier lugar donde puedan ir las expresiones de JavaScript. Esto significa que un elemento JSX puede guardarse en una variable, pasarse a una función, almacenarse en un objeto o en un arreglo… lo que se te ocurra.

Aquí tienes un ejemplo de un elemento JSX guardado en una variable:

```js
const navBar = <nav>I am a nav bar</nav>;
```

Copiar al portapapeles

Aquí tienes un ejemplo de varios elementos JSX almacenados en un objeto:

```js
const myTeam = {
  center: <li>Benzo Walli</li>,
  powerForward: <li>Rasha Loa</li>,
  smallForward: <li>Tayshaun Dasmoto</li>,
  shootingGuard: <li>Colmar Cumberbatch</li>,
  pointGuard: <li>Femi Billon</li>
};
```

-----

## Attributes In JSX

Los **elementos JSX** pueden tener **atributos**, al igual que los elementos HTML.

Un atributo JSX se escribe usando una sintaxis similar a HTML: un nombre, seguido de un signo igual, seguido de un valor. El valor debe ir entre comillas, así:

```txt
my-attribute-name="my-attribute-value"
```

Aquí tienes algunos elementos JSX con atributos:

```jsx
<a href='http://www.example.com'>Welcome to the Web</a>;

const title = <h1 id='title'>Introduction to React.js: Part I</h1>;
```

Un solo elemento JSX puede tener muchos atributos, igual que en HTML:

```jsx
const panda = <img src='images/panda.jpg' alt='panda' width='500px' height='500px'>;
```

----

## Nested JSX

Puedes **anidar elementos JSX dentro de otros elementos JSX**, igual que en HTML.

Aquí tienes un ejemplo de un elemento JSX `<h1>` anidado dentro de un elemento JSX `<a>`:

```jsx
<a href="https://www.example.com"><h1>Click me!</h1></a>
```

Para que sea más legible, puedes usar **saltos de línea e indentación al estilo HTML**:

```jsx
<a href="https://www.example.com">
  <h1>
    Click me!
  </h1>
</a>
```

Si una expresión JSX ocupa más de una línea, entonces debes **envolver la expresión JSX de varias líneas entre paréntesis**. Al principio puede parecer extraño, pero te acostumbras:

```jsx
(
  <a href="https://www.example.com">
    <h1>
      Click me!
    </h1>
  </a>
)
```

Las expresiones JSX anidadas pueden guardarse en variables, pasarse a funciones, etc., ¡igual que las expresiones JSX no anidadas! Aquí tienes un ejemplo de una expresión JSX anidada guardada en una variable:

```jsx
const theExample = (
  <a href="https://www.example.com">
    <h1>
      Click me!
    </h1>
  </a>
);
```

----

## JSX Outer Elements

Hay una regla que aún no hemos mencionado: **una expresión JSX debe tener exactamente un solo elemento externo**.

En otras palabras, este código **sí funcionará**:

```jsx
const paragraphs = (
  <div id="i-am-the-outermost-element">
    <p>I am a paragraph.</p>
    <p>I, too, am a paragraph.</p>
  </div>
);
```

Pero este código **no funcionará**:

```jsx
const paragraphs = (
  <p>I am a paragraph.</p> 
  <p>I, too, am a paragraph.</p>
);
```

La primera etiqueta de apertura y la última etiqueta de cierre de una expresión JSX **deben pertenecer al mismo elemento JSX**.

Es fácil olvidar esta regla y terminar con errores difíciles de diagnosticar.

Si notas que una expresión JSX tiene varios elementos externos, la solución suele ser simple: **envuelve la expresión JSX dentro de un elemento `<div>`**.

----

## Rendering JSX

¡Ya aprendiste cómo escribir **elementos JSX**! Ahora es momento de aprender cómo **renderizarlos**.

Renderizar una expresión JSX significa **hacer que aparezca en la pantalla**.

----

## Rendering JSX Explained

Vamos a examinar el código que acabas de escribir en el último ejercicio:

```js
const container = document.getElementById('app');
const root = createRoot(container);
root.render(<h1>Hello world</h1>);
```

Antes de comenzar, es esencial entender que **React se basa en dos cosas para renderizar**: **qué contenido renderizar** y **dónde colocar el contenido**.

Con eso en mente, veamos la primera línea:

```js
const container = document.getElementById('app')
```

Esta línea:

* Usa el objeto `document`, que representa nuestra página web.
* Usa el método `getElementById()` de `document` para obtener el objeto `Element` que representa el elemento HTML con el id proporcionado (`app`).
* Almacena el elemento en la variable `container`.

En la siguiente línea:

```js
const root = createRoot(container)
```

Usamos `createRoot()` de la biblioteca `react-dom/client`, que **crea una raíz de React a partir de `container`** y la almacena en `root`.
`root` puede usarse para **renderizar una expresión JSX**. Esta es la parte de React que responde a **“dónde colocar el contenido”**.

Finalmente, la última línea:

```js
root.render(<h1>Hello world</h1>)
```

Usa el método `render()` de `root` para **renderizar el contenido pasado como argumento**.
Aquí pasamos un elemento `<h1>`, que muestra *Hello world*.
Esta es la parte de React que responde a **“qué contenido renderizar”**.

-----

## Passing a Variable to render()

En el ejercicio anterior, vimos cómo podemos crear una raíz de React usando `createRoot()` y usar su método `render()` para renderizar JSX.

El argumento del método `render()` no necesita ser JSX directamente, pero sí debe evaluarse como una expresión JSX. El argumento también podría ser una variable, siempre que esa variable se evalúe como una expresión JSX.

En este ejemplo, guardamos una expresión JSX en una variable llamada `toDoList`. Luego, pasamos `toDoList` como argumento de `render()`:

```javascript
const toDoList = (
  <ol>
    <li>Aprender React</li>
    <li>Convertirse en Desarrollador</li>
  </ol>
);

const container = document.getElementById('app');
const root = createRoot(container);
root.render(toDoList);
```

---

## The Virtual DOM

Una cosa especial del método `render()` de una raíz de React es que **solo actualiza los elementos del DOM que han cambiado**.

Eso significa que si renderizas exactamente lo mismo dos veces seguidas, la segunda renderización **no hará nada**:

```javascript
const hello = <h1>Hola mundo</h1>;

// Esto añadirá "Hola mundo" a la pantalla:
root.render(hello, document.getElementById('app'));

// Esto no hará absolutamente nada:
root.render(hello, document.getElementById('app'));
```

¡Esto es importante! Solo actualizar los elementos necesarios del DOM es una gran parte de lo que hace que React sea tan eficiente. Esto se logra usando el **DOM virtual de React**.

---

## Review

¡Felicidades! ¡Has aprendido a crear y renderizar elementos **JSX**! Este es el primer paso para volverte fluido en React.

En esta lección, aprendimos que:

* React es un framework **modular, escalable, flexible y popular** para el front-end.
* JSX es una **extensión de sintaxis de JavaScript** que nos permite tratar el HTML como expresiones.
* ¡Los elementos JSX se pueden almacenar en variables, objetos, arreglos y más!
* Los elementos JSX pueden tener **atributos** y **anidarse** entre sí, igual que en HTML.
* JSX debe tener **exactamente un elemento exterior**, y otros elementos pueden estar anidados dentro de él.
* `createRoot()` de `react-dom/client` se puede usar para **crear una raíz de React** en un elemento específico del DOM.
* El método `render()` de una raíz de React se puede usar para **renderizar JSX en la pantalla**.
* El método `render()` de una raíz de React **solo actualiza los elementos del DOM que han cambiado**, usando el **DOM virtual**.

A medida que continúes aprendiendo más sobre React, descubrirás cosas poderosas que puedes hacer con JSX, algunos problemas comunes de JSX y cómo **evitarlos**.

---

