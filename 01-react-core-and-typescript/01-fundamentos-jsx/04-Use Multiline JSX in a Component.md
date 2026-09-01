# Use Multiline JSX in a Component

En esta lección, aprenderás algunas formas comunes en que **JSX** y los **componentes de React** trabajan juntos. Te sentirás más cómodo tanto con JSX como con los componentes de React mientras aprendes algunos trucos nuevos.

Mira este HTML:

```html
<blockquote>
  <p>
    The world is full of objects, more or less interesting; I do not wish to add any more.
  </p>
  <cite>
    <a target="_blank"
      href="https://en.wikipedia.org/wiki/Douglas_Huebler">
      Douglas Huebler
    </a>
  </cite>
</blockquote>
```

¿Cómo podrías hacer que un componente de React devuelva este HTML?

Selecciona **QuoteMaker.js** para ver una manera de hacerlo.

Lo más importante a notar en **QuoteMaker** son los paréntesis en la sentencia `return`, en las líneas 4 y 16. Hasta ahora, las sentencias `return` de tus componentes funcionales se veían así, sin paréntesis:

```js
return <h1>Hello world</h1>;
```

Sin embargo, una expresión **JSX de varias líneas** siempre debe estar envuelta entre paréntesis. ¡Por eso la sentencia `return` de **QuoteMaker** tiene paréntesis alrededor!

----

## Use a Variable Attribute in a Component

Mira este objeto de JavaScript llamado **redPanda**:

```js
const redPanda = {
  src: 'https://upload.wikimedia.org/wikipedia/commons/b/b2/Endangered_Red_Panda.jpg',
  alt: 'Red Panda',
  width: '200px'
};
```

¿Cómo podrías renderizar un componente de React con una imagen de **redPanda** y sus propiedades?

Selecciona **RedPanda.js** para ver una manera de hacerlo.

Fíjate en todas las **inserciones de JavaScript entre llaves** dentro de la sentencia `return`. Puedes, y a menudo lo harás, **inyectar JavaScript dentro de JSX** dentro de la sentencia `return`.

------

## Putting Logic in a Function Component

Un **componente funcional** debe tener una sentencia **return**. Sin embargo, eso no es todo lo que puede contener.

También puedes poner **cálculos simples** que necesiten hacerse antes de devolver tu elemento **JSX** dentro del componente funcional.

Aquí tienes un ejemplo de algunos cálculos que se pueden realizar dentro de un componente funcional:

```js
function RandomNumber() {
  // Primero, algo de lógica que debe ocurrir antes del return
  const n = Math.floor(Math.random() * 10 + 1);
  // Después, una sentencia return usando esa lógica:
  return <h1>{n}</h1>
}
```

Ten cuidado con este error común:

```js
function RandomNumber() {
  return (
    const n = Math.floor(Math.random() * 10 + 1);
    <h1>{n}</h1>
  )
}
```

En el ejemplo anterior, la línea con la declaración `const n` causará un **error de sintaxis**, ya que debería ir **antes del return**.

----

## Use a Conditional in a Function Component

¿Cómo podrías usar una **sentencia condicional** dentro de un componente funcional?

Selecciona **TodaysPlan.js** para ver una manera de hacerlo.

Fíjate en que la sentencia **if** se encuentra dentro del componente funcional, pero **antes de la sentencia return**.

-------

## Event Listener and Event Handlers in a Component

Tus **componentes funcionales** pueden incluir **manejadores de eventos**. Con los manejadores de eventos, podemos ejecutar código en respuesta a interacciones con la interfaz, como hacer clic.

```js
function MyComponent(){
  function handleHover() {
    alert('Stop it. Stop hovering.');
  }
  return <div onHover={handleHover}></div>;
}
```

En el ejemplo anterior, el manejador de eventos es `handleHover()`. Se pasa como una **prop** al elemento JSX `<div>`. Hablaremos más sobre las props en una lección posterior, pero por ahora entiende que **las props son información que se pasa a una etiqueta JSX**.

La lógica de lo que debería ocurrir cuando se pasa el mouse sobre el `<div>` está dentro de la función `handleHover()`. Luego, la función se pasa al elemento `<div>`.

Las funciones manejadoras de eventos se definen dentro del componente funcional y, por convención, comienzan con la palabra **“handle”** seguida del tipo de evento que están manejando.

Hay un pequeño detalle al que debes prestar atención. Mira esta línea de nuevo:

```js
return <div onHover={handleHover}></div>
```

La función `handleHover()` se pasa **sin los paréntesis** que normalmente veríamos al llamar a una función. Esto se debe a que pasarla como `handleHover` indica que **solo debe ejecutarse cuando ocurra el evento**. Pasarla como `handleHover()` **ejecutaría la función inmediatamente**, ¡así que ten cuidado!

-----

## Review

¡Felicidades! Has terminado la lección sobre **componentes de React**.

Aquí tienes un breve resumen de los conceptos presentados en esta lección:

* Los **componentes funcionales** pueden devolver múltiples líneas de **JSX** anidando los elementos dentro de un elemento padre.
* Se pueden usar **atributos variables** dentro de un componente de React mediante inyecciones de JavaScript.
* Los componentes de React soportan **lógica** colocando las sentencias lógicas **antes** de las sentencias `return`.
* Los componentes pueden devolver elementos JSX de forma **condicional** usando sentencias condicionales dentro del componente.
* Los componentes pueden **responder a eventos** definiendo manejadores de eventos y pasándolos a los elementos JSX.

¡Excelente trabajo enfrentándote a estos temas complejos! Has dedicado mucho tiempo a estudiar los componentes de React de manera aislada. Ahora es momento de aprender cómo los componentes **encajan en el mundo que los rodea**.

-----

