# Advanced JSX

## class vs className

Esta lección cubrirá **JSX más avanzado**. Aprenderás algunos trucos poderosos y algunos errores comunes que debes evitar.

La **gramática en JSX** es mayormente igual que en HTML, pero hay **diferencias sutiles** a las que hay que prestar atención. La más frecuente tiene que ver con la palabra **class**.

En HTML, es común usar **class** como nombre de atributo:

```html
<h1 class="big">Título</h1>
```

En JSX, **no puedes usar la palabra class**. ¡Debes usar **className** en su lugar!

```jsx
<h1 className="big">Título</h1>
```

Esto se debe a que JSX se traduce a JavaScript, y **class** es una palabra reservada en JavaScript.

Cuando JSX se renderiza, los atributos **className** de JSX se renderizan automáticamente como **class** en el HTML final.

---

## Self-Closing Tags

Otro error común en JSX tiene que ver con las **etiquetas autocerradas**.

**¿Qué es una etiqueta autocerrada?**

La mayoría de los elementos HTML usan **dos etiquetas**: una etiqueta de apertura (`<div>`) y una etiqueta de cierre (`</div>`). Sin embargo, algunos elementos HTML, como `<img>` y `<input>`, usan **solo una etiqueta**. La etiqueta que pertenece a un elemento de una sola etiqueta **no es de apertura ni de cierre**, sino una **etiqueta autocerrada**.

Cuando escribes una etiqueta autocerrada en HTML, es **opcional** incluir una barra diagonal `/` justo antes del ángulo final:

```html
<!-- Correcto en HTML con barra: -->
<br />

<!-- También correcto, sin la barra: -->
<br>
```

Pero, en **JSX**, **tienes que incluir la barra**. Si escribes una etiqueta autocerrada en JSX y olvidas la barra, **se generará un error**:

```jsx
// Correcto en JSX:
<br />

// TOTALMENTE INCORRECTO en JSX:
<br>
```

---

## JavaScript In Your JSX In Your JavaScript

Hasta ahora, nos hemos enfocado en escribir **expresiones JSX**. Es similar a escribir fragmentos de HTML, pero **dentro de un archivo JavaScript**.

En esta lección, vamos a agregar algo nuevo: **JavaScript regular**, escrito **dentro de una expresión JSX**, dentro de un **archivo JavaScript**.

---

## Curly Braces in JSX

El código del último ejercicio **no se comportó como uno podría esperar**. En lugar de sumar 2 y 3, **se imprimió “2 + 3” como un texto**. ¿Por qué?

Esto sucedió porque **2 + 3 estaba ubicado entre las etiquetas `<h1>` y `</h1>`**.

Cualquier código que se encuentre **entre las etiquetas de un elemento JSX** será leído como JSX, ¡no como JavaScript normal! JSX **no suma números**, sino que los interpreta como texto, igual que HTML.

Necesitas una forma de escribir código que diga:
*"Aunque estoy ubicado entre etiquetas JSX, trátame como JavaScript ordinario y no como JSX."*

Puedes hacer esto **envolviendo tu código entre llaves `{}`**.

---

## 20 Digits of Pi in JSX


**¡Ahora puedes inyectar JavaScript normal dentro de expresiones JSX!** Esto será extremadamente útil.

Acá tenés una expresión JSX que muestra los primeros veinte dígitos de pi:

```jsx
const pi = (
  <div>
    <h1>PI, a Special Number</h1>
    <p>The number pi is an important number. It is approximately {Math.PI.toFixed(20)}</p>
  </div>
);
```

Estudia la expresión y observa lo siguiente:
* El código está escrito en un archivo JavaScript. Por defecto, todo se tratará como JavaScript normal.
* Busca `<div>` en la primera línea del `return`. Desde ahí y hasta `</div>`, el código se tratará como JSX.
* Busca `Math`. Desde ahí y hasta `(20)`, el código volverá a tratarse como JavaScript normal.
* Las llaves `{}` en sí mismas no se tratarán ni como JSX ni como JavaScript. Son marcadores que señalan el inicio y el final de una inyección de JavaScript dentro de JSX, de forma similar a como las comillas señalan los límites de una cadena de texto.

----

## Variables in JSX

Cuando inyectas JavaScript dentro de **JSX**, ese JavaScript forma parte del mismo entorno que el resto del JavaScript en tu archivo.

Eso significa que puedes acceder a variables mientras estás dentro de una expresión JSX, incluso si esas variables fueron declaradas fuera del bloque de código JSX.

> ```js
> // Declara una variable:
> const name = 'Gerdo';
>
> // Accede a tu variable dentro de una expresión JSX:
> const greeting = <p>Hola, {name}!</p>;
> ```

----

## Variable Attributes in JSX

Al escribir **JSX**, es común usar variables para establecer atributos.

Aquí tienes un ejemplo de cómo podría funcionar esto:

> ```js
> // Usa una variable para establecer los atributos `height` y `width`:
>
> const sideLength = "200px";
>
> const panda = (
>   <img 
>     src="images/panda.jpg" 
>     alt="panda" 
>     height={sideLength} 
>     width={sideLength} />
> );
> ```

Observa cómo en este ejemplo cada atributo del `<img />` está en su propia línea. Esto puede hacer que tu código sea más legible si tienes muchos atributos para un solo elemento.

Las propiedades de objetos también se usan a menudo para establecer atributos:

> ```js
> const pics = {
>   panda: "http://bit.ly/1Tqltv5",
>   owl: "http://bit.ly/1XGtkM3",
>   owlCat: "http://bit.ly/1Upbczi"
> }; 
>
> const panda = (
>   <img 
>     src={pics.panda} 
>     alt="Lazy Panda" />
> );
>
> const owl = (
>   <img 
>     src={pics.owl} 
>     alt="Unimpressed Owl" />
> );
>
> const owlCat = (
>   <img 
>     src={pics.owlCat} 
>     alt="Ghastly Abomination" />
> ); 
> ```

----

## Event Listeners in JSX

Los elementos **JSX** pueden tener escuchadores de eventos, igual que los elementos HTML. Programar en React implica trabajar constantemente con escuchadores de eventos.

Creas un escuchador de eventos dándole a un elemento JSX un atributo especial. Aquí tienes un ejemplo:

```jsx
<img onClick={clickAlert} />
```

El nombre del atributo del escuchador de eventos debe ser algo como `onClick` u `onMouseOver`: la palabra **on** más el tipo de evento que estás escuchando. Puedes revisar la [lista de componentes comunes en la documentación de React](https://react.dev/reference/react-dom/components/common#) para ver los nombres de eventos compatibles.

El valor del atributo del escuchador de eventos debe ser una función. El ejemplo anterior solo funcionaría si `clickAlert` fuera una función válida definida en otro lugar:

```js
function clickAlert() {
  alert('¡Hiciste clic en esta imagen!');
}

<img onClick={clickAlert} />
```

Ten en cuenta que en HTML los nombres de los escuchadores de eventos se escriben completamente en minúsculas, como `onclick` u `onmouseover`. En JSX, los nombres de los escuchadores de eventos se escriben en **camelCase**, como `onClick` u `onMouseOver`.

---

## JSX Conditionals: If Statements That Don't Work

¡Excelente trabajo! Has aprendido cómo usar llaves `{}` para inyectar JavaScript dentro de una expresión **JSX**.

Aquí hay una regla que necesitas conocer: **no puedes inyectar una sentencia `if` dentro de una expresión JSX**.

Este código se romperá:

```jsx
(
  <h1>
    {
      if (purchase.complete) {
        '¡Gracias por realizar tu pedido!'
      }
    }
  </h1>
)
```

¿Qué pasa si quieres que una expresión JSX se renderice solo bajo ciertas circunstancias? No puedes inyectar una sentencia `if`. ¿Qué puedes hacer entonces?

Tienes muchas opciones. En las próximas lecciones exploraremos algunas formas sencillas de escribir condicionales (expresiones que solo se ejecutan bajo ciertas condiciones) en JSX.

---

## JSX Conditionals: If Statements That Do Work

¿Cómo puedes escribir un condicional si no puedes inyectar una sentencia `if` dentro de JSX?

Una opción es escribir una sentencia `if` y **no** inyectarla dentro de **JSX**.

```jsx
function ConcertInfo({ price }) {
  let ticketInfo;

  if (price === 0) {
    ticketInfo = <h2>Entrada gratuita</h2>;
  } else {
    ticketInfo = <h2>Entrada: ${price}</h2>;
  }

  return (
    <div>
      <h1>Próximo show</h1>
      {ticketInfo}
    </div>
  );
}
```

Este componente funciona porque las palabras `if` y `else` no están inyectadas entre etiquetas JSX: la sentencia `if` está por fuera, en su propia declaración de variable, y solo la variable resultante (`ticketInfo`) se inyecta con llaves dentro del JSX. No es necesaria ninguna inyección de JavaScript de la sentencia `if` en sí.

Esta es una forma común de expresar condicionales en JSX.

---

## JSX Conditionals: The Ternary Operator

Hay una forma más compacta de escribir condicionales en JSX: **el operador ternario**.

El operador ternario funciona de la misma manera en React que en JavaScript normal. Sin embargo, aparece sorprendentemente a menudo en React.

Recuerda cómo funciona: se escribe `x ? y : z`, donde `x`, `y` y `z` son expresiones de JavaScript. Cuando se ejecuta el código, `x` se evalúa como “verdadero” (*truthy*) o “falso” (*falsy*). Si `x` es verdadero, entonces todo el operador ternario devuelve `y`. Si `x` es falso, entonces todo el operador ternario devuelve `z`.

Así es como podrías usar el operador ternario dentro de una expresión **JSX**:

```jsx
const headline = (
  <h1>
    { age >= drinkingAge ? 'Buy Drink' : 'Do Teen Stuff' }
  </h1>
);
```

En el ejemplo anterior, si `age` es mayor o igual que `drinkingAge`, entonces `headline` será igual a `<h1>Buy Drink</h1>`. De lo contrario, `headline` será igual a `<h1>Do Teen Stuff</h1>`.

---

## JSX Conditionals: &&

Vamos a cubrir una última forma de escribir condicionales en React: **el operador `&&`**.

Al igual que el operador ternario, `&&` no es específico de React, pero aparece muy a menudo en React.

En los dos últimos ejercicios, escribiste sentencias que a veces renderizaban un gatito y otras veces un perrito. `&&` no habría sido la mejor opción para ese código.

`&&` funciona mejor para condicionales que a veces realizan una acción y otras veces no hacen nada en absoluto.

Aquí tienes un ejemplo:

```jsx
const tasty = (
  <ul>
    <li>Applesauce</li>
    { !baby && <li>Pizza</li> }
    { age > 15 && <li>Brussels Sprouts</li> }
    { age > 20 && <li>Oysters</li> }
    { age > 25 && <li>Grappa</li> }
  </ul>
);
```

Si la expresión a la izquierda de `&&` se evalúa como verdadera, entonces el **JSX** a la derecha de `&&` se renderizará. Sin embargo, si la primera expresión es falsa, el JSX a la derecha de `&&` se ignorará y no se renderizará.

---

## .map in JSX

El método de arreglos **`.map()`** aparece con frecuencia en React. Es bueno acostumbrarse a usarlo junto con JSX.

Si quieres crear una lista de elementos JSX, usar **`.map()`** suele ser la forma más eficiente. Al principio puede verse un poco extraño:

```js
const strings = ['Home', 'Shop', 'About Me'];

const listItems = strings.map(string => <li>{string}</li>);

<ul>{listItems}</ul>
```

En el ejemplo anterior, comenzamos con un arreglo de cadenas de texto. Llamamos a **`.map()`** sobre este arreglo, y la llamada a `.map()` devuelve un nuevo arreglo de elementos `<li>`.

En la última línea del ejemplo, observa que `{listItems}` se evaluará como un arreglo, ¡porque es el valor devuelto por `.map()`! Los `<li>` en JSX no tienen que estar en un arreglo como este, pero pueden estarlo.

```jsx
// Esto es válido en JSX, no en un arreglo explícito:
<ul>
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
</ul>

// ¡Esto también es válido!
const liArray = [
  <li>item 1</li>, 
  <li>item 2</li>, 
  <li>item 3</li>
];

<ul>{liArray}</ul>
```

---

## Keys

Cuando creas una lista en **JSX**, a veces tu lista necesitará incluir algo llamado **keys** (claves):

```jsx
<ul>
  <li key="li-01">Example1</li>
  <li key="li-02">Example2</li>
  <li key="li-03">Example3</li>
</ul>
```

Una **key** es un atributo de JSX. El nombre del atributo es `key`. El valor del atributo debe ser algo único, similar a un atributo `id`.

Las **keys** no hacen nada visible. React las usa internamente para llevar el control de las listas. Si no usas keys cuando deberías, React podría mezclar accidentalmente los elementos de la lista en un orden incorrecto.

No todas las listas necesitan keys. Una lista necesita keys si se cumple alguna de las siguientes condiciones:

* Los elementos de la lista tienen memoria de un renderizado al siguiente. Por ejemplo, cuando se renderiza una lista de tareas, cada elemento debe “recordar” si fue marcado como completado. Los elementos no deberían perder esa información al renderizarse.
* El orden de la lista puede cambiar. Por ejemplo, una lista de resultados de búsqueda podría reorganizarse de un renderizado a otro.

Si ninguna de estas condiciones se cumple, entonces no tienes que preocuparte por las keys. ¡Y si no estás seguro, nunca está de más usarlas! 😄

----

## React.createElement

¡Puedes escribir código React sin usar **JSX** en absoluto!

La mayoría de los programadores de React sí usan JSX, pero debes entender que **es posible escribir código React sin él**.

La siguiente expresión JSX:

```js
const h1 = <h1>Hello world</h1>;
```

puede reescribirse sin JSX, así:

```js
const h1 = React.createElement(
  "h1",
  null,
  "Hello world"
);
```

Cuando un elemento JSX se compila, el compilador transforma el elemento JSX en el método que ves arriba: **React.createElement()**. Cada elemento JSX es, en secreto, una llamada a `React.createElement()`.

No entraremos en detalle sobre cómo funciona `React.createElement()`, pero puedes consultar [la documentación de React sobre `createElement()`](https://react.dev/reference/react/createElement) para aprender más.

----

## Review

¡Felicidades! Has aprendido una gran variedad de conceptos de JSX. Si sientes que aún no los dominas todos, ¡no te preocupes! Estos conceptos aparecerán una y otra vez a lo largo de tu aprendizaje de React.

¡Ahora estás listo para poner en práctica tus conocimientos de JSX!

-----