# Props

## Props

Cuando pensamos en el contexto de una aplicación React, los componentes son pequeñas partes de un todo. Juntos, conforman la interfaz que los usuarios verán.

Con cada componente desempeñando un papel en la interfaz, hay momentos en los que los componentes deben poder comunicarse con otros componentes.

En esta lección, aprenderás otra forma en que los componentes pueden interactuar: un componente pasando información a otro componente.

La información que se pasa de un componente a otro se conoce como **props**.

Las **props** se pueden usar para personalizar la salida de cada componente, dependiendo de la información que se pase.

Al permitir que los componentes se comuniquen entre sí, podemos agregar un nivel de flexibilidad que antes no era posible.

Al final de esta lección, deberías ser capaz de:

* Pasar, acceder y mostrar **props**.
* Usar **props** para crear sentencias condicionales.
* Definir manejadores de eventos en un componente y pasarlos a otros componentes.
* Trabajar con los **children** de un componente.
* Asignar valores predeterminados a las **props**.

¡Comencemos!

------

## Access a Component's props

Cada componente tiene algo llamado **props**.

Las **props** de un componente son un objeto. Contienen información sobre ese componente.

¡Probablemente ya lo has visto antes, pero tal vez no te habías dado cuenta! Echemos un vistazo al tag HTML de un botón. Hay varias piezas de información que podemos pasar al tag de botón, como el tipo del botón.

```html
<button type="submit" value="Submit"> Submit </button>
```

En este ejemplo, hemos pasado dos piezas de información al tag del botón: un tipo y un valor. Dependiendo del atributo **type** que le demos al elemento `<button>`, este tratará el formulario de manera diferente. De la misma manera, podemos pasar información a nuestros propios componentes para especificar cómo se comportan.

Las **props** sirven para el mismo propósito en los componentes que los argumentos para las funciones.

Para acceder al objeto **props** de un componente, puedes hacer referencia al objeto **props** y usar la notación de punto para sus propiedades. Aquí tienes un ejemplo:

```jsx
props.name
```

Esto recuperaría la propiedad **name** del objeto **props**.

-----

## Pass `props` to a Component

Para aprovechar las **props**, necesitamos pasar información a un componente de React. En el ejercicio anterior, renderizamos un objeto **props** vacío porque no pasamos ninguna **prop** a nuestro componente **PropsDisplayer**.

¿Cómo pasamos las **props**? Dándole un atributo al componente:

```jsx
<Greeting name="Jamel" />
```

Supongamos que quieres pasar un mensaje al componente, como "¡Somos geniales!". Así es como lo harías:

```jsx
<SloganDisplay message="¡Somos geniales!" />
```

Como puedes ver, para pasar información a un componente, necesitas un nombre para la información que deseas pasar.

En el ejemplo anterior, usamos el nombre **message**. Puedes usar cualquier nombre que desees.

Si quieres pasar información que no sea una cadena de texto, entonces envuelve esa información en llaves. Así es como pasarías un arreglo:

```jsx
<Greeting myInfo={["Astronauta", "Narek", "43"]} />
```

En este siguiente ejemplo, pasamos varias piezas de información al componente `<Greeting />`. Los valores que no son cadenas de texto están envueltos en llaves:

```jsx
<Greeting name="The Queen Mary" city="Long Beach, California" age={56} haunted={true} />
```

-----

## Render a Component's props

Las **props** nos permiten personalizar el componente al pasarle información.

Hemos aprendido cómo pasar información al objeto **props** de un componente. A menudo querrás que un componente muestre la información que le pasas.

Para asegurarte de que un componente de función pueda usar el objeto **props**, define tu componente de función con **props** como parámetro:

```jsx
function Button(props) {
  return <button>{props.displayText}</button>;
}
```

En este ejemplo, **props** se acepta como parámetro y los valores del objeto se acceden con el patrón de notación de punto (objeto.nombreDePropiedad).

Alternativamente, dado que **props** es un objeto, también puedes usar la sintaxis de desestructuración de esta manera:

```jsx
function Button({displayText}) {
  return <button>{displayText}</button>;
}
```

-----

## Pass props From Component To Component

Has aprendido cómo pasar una **prop** a un componente:

```jsx
<Greeting firstName="Esmerelda" />
```

También has aprendido cómo acceder y mostrar una **prop** pasada:

```jsx
return <h1>{props.firstName}</h1>;
```

El uso más común de las **props** es pasar información a un componente desde otro componente.

Las **props** en React viajan en una sola dirección, de arriba a abajo, de padre a hijo.

Vamos a explorar un poco más la relación padre-hijo al pasar **props**.

```jsx
function App() {
    return <Product name="Apple Watch" price={399} rating="4.5/5.0" />;
}
```

En este ejemplo, **App** es el componente padre y **Product** es el componente hijo. **App** pasa tres **props** a **Product** (name, price y rating), que luego pueden ser leídas dentro del componente hijo.

Las **props** pasadas son inmutables, lo que significa que no se pueden cambiar. Si un componente quiere nuevos valores para sus **props**, debe depender del componente padre para que le pase los nuevos valores.

¡Vamos a practicar esto!

-----

## Render Different UI Based on props

Puedes hacer más con las **props** que solo mostrarlas. También puedes usar las **props** para tomar decisiones.

```jsx
function LoginMsg(props) {
  if (props.password === 'a-tough-password') {
    return <h2>Inicio de sesión exitoso.</h2>;
  } else {
    return <h2>Falló el inicio de sesión.</h2>;
  }
}
```

En este ejemplo, usamos las **props** pasadas para tomar una decisión, en lugar de renderizar el valor en la pantalla.

Si la contraseña recibida es igual a `'a-tough-password'`, el mensaje resultante en `<h2></h2>` será diferente.

¡La contraseña pasada no se muestra en ninguno de los casos! La **prop** se usa para decidir qué se va a mostrar. ¡Esta es una técnica común!

-----

## Put an Event Handler in a Function Component

Puedes, y a menudo lo harás, pasar funciones como **props**. Es especialmente común pasar funciones de manejo de eventos.

En el siguiente ejercicio, pasaremos una función de manejo de eventos como una **prop**. Sin embargo, primero debemos definir un manejador de eventos antes de poder pasarlo a cualquier lugar. En este ejercicio, vamos a definir una función de manejo de eventos.

¿Cómo definimos un manejador de eventos en React?

Definimos un manejador de eventos como un método en el componente de función.

Echa un vistazo al archivo **Example.js** en el editor de código. En las líneas 4 a 8, se define un método de manejo de eventos. En la línea 10, ese método de manejo de eventos se adjunta a un evento (en este caso, un evento de clic).

-----

## Pass an Event Handler as a prop

¡Bien! Has definido un nuevo método dentro del componente **Talker**. Ahora estás listo para pasar esa función a otro componente.

Puedes pasar un método de la misma manera exacta que pasas cualquier otro dato: usando llaves `{}`.

### 🧠 Qué está pasando (visión general)

Este componente:

* **define una función**
* **se la pasa a otro componente**
* **ese otro componente la ejecuta**

Nada más.

#### 🔍 Paso a paso (línea por línea)

```js
import Button from './Button';
```

👉 Importa un **componente React** llamado `Button`.

```js
function Talker() {
```

👉 Define un **function component**.

```js
function talk() {
```

👉 Define una **función normal de JavaScript** dentro del componente.

```js
let speech = '';
for (let i = 0; i < 10000; i++) {
  speech += 'blah ';
}
```

👉 Construye un string largo:
`"blah blah blah ..."` (10,000 veces)

```js
alert(speech);
```

👉 Muestra ese texto en una alerta.

```jsx
return <Button talk={talk} />;
```

👉 Aquí está **lo importante**:

* `talk={talk}`
* **NO se ejecuta la función**
* Se pasa la **referencia** de la función al componente `Button`

```js
export default Talker;
```

👉 Exporta el componente.

### 🧠 Qué pasa en tiempo de ejecución (mental model)

1️⃣ React renderiza `Talker`
2️⃣ `Talker` crea la función `talk`
3️⃣ `Talker` pasa `talk` a `Button` como prop
4️⃣ `Button` decide **cuándo llamar `talk()`**
(normalmente en un `onClick`)

Ejemplo típico dentro de `Button`:

```jsx
function Button({ talk }) {
  return <button onClick={talk}>Talk</button>;
}
```

## 🎯 Regla clave (memorízala)

👉 **Las funciones se pasan sin paréntesis**
👉 **Se ejecutan donde corresponde (eventos)**

### Resumen en una línea

> `Talker` define una función y se la entrega a `Button` para que `Button` la ejecute cuando el usuario haga algo.

-----

## Receive an Event Handler as a prop

¡Genial! Acabas de pasar una función de `<Talker />` a `<Button />`.

Mira el archivo **Button.js** en el editor de código. Observa que **Button** devuelve un elemento `<button>`.

Si un usuario hace clic en este elemento `<button>`, quieres que se llame a la función **talk()** que pasaste. Esto significa que necesitas adjuntar **talk()** al elemento `<button>` como un manejador de eventos.

¿Cómo se hace eso? De la misma manera que adjuntas cualquier manejador de eventos a un elemento JSX: le das a ese elemento JSX un atributo especial. El nombre del atributo debe ser un nombre de evento, como **onClick** o **onHover**. El valor del atributo debe ser el manejador de eventos que quieres adjuntar.

### 🧠 Qué está pasando exactamente

Tienes esto (implícito):

```jsx
<Button talk={talk} />
```

Y en `Button` algo como:

```jsx
function Button(props) {
  return <button onClick={props.talk}>Talk</button>;
}
```

### 🔍 ¿Por qué `props` es un objeto?

Porque **React agrupa TODAS las props en un solo objeto**.

Piensa así 👇

Cuando haces:

```jsx
<Button talk={talk} color="red" />
```

React lo convierte internamente en algo como:

```js
props = {
  talk: talk,
  color: "red"
};
```

👉 **Siempre es un objeto**

👉 Cada atributo del componente = una propiedad del objeto

### 🔑 Entonces, ¿por qué `props.talk`?

Porque:

* `talk` fue pasado como prop
* React lo guardó dentro del objeto `props`
* Accedes a esa función como cualquier propiedad JS

```js
props.talk   // es la función talk
```

Y cuando el usuario hace click:

```js
props.talk() // se ejecuta la función
```

### 🧠 Analogía rápida (para fijarlo)

Imagina que llamas a una función así:

```js
Button({
  talk: talk
});
```

Dentro de la función:

```js
props.talk();
```

👉 Es **JavaScript puro**, no magia de React.

### ✨ Forma moderna (más común)

En vez de:

```js
function Button(props) {
  return <button onClick={props.talk}>Talk</button>;
}
```

Usamos **destructuring**:

```js
function Button({ talk }) {
  return <button onClick={talk}>Talk</button>;
}
```

👉 Es exactamente lo mismo
👉 Más limpio y legible

---

## handleEvent, onEvent, and props.onEvent

Hablemos sobre cómo nombrar cosas.

Cuando pasas un manejador de eventos como **prop**, como acabas de hacer, hay dos nombres que debes elegir. Ambas elecciones de nombres ocurren en el componente padre, es decir, el componente que define el manejador de eventos y lo pasa.

El primer nombre que debes elegir es el nombre del propio manejador de eventos.

Mira **Talker.js**, líneas 5 a 11. Este es nuestro manejador de eventos. Elegimos nombrarlo **talk**.

El segundo nombre que debes elegir es el nombre de la **prop** que usarás para pasar el manejador de eventos. Esto es lo mismo que el nombre del atributo.

Para el nombre de nuestra **prop**, también elegimos **talk**, como se muestra en la línea 12:

```jsx
return <Button talk={talk} />;
```

Estos dos nombres pueden ser lo que queramos. Sin embargo, hay una convención de nombres que se usa comúnmente.

Así funciona la convención de nombres: primero, piensa en el tipo de evento al que estás escuchando. En nuestro ejemplo, el tipo de evento era `"click"`. Si estás escuchando un evento `"click"`, entonces nombras tu manejador de eventos **handleClick**. Si estás escuchando un evento `"hover"`, entonces nombras tu manejador de eventos **handleHover**:

```jsx
function myClass() {
  function handleHover() {
    alert('Soy un manejador de eventos.');
    alert('Se llamará en respuesta a eventos "hover".');
  }
}
```

El nombre de tu **prop** debe ser la palabra **on**, más el tipo de evento. Si estás escuchando un evento `"click"`, entonces nombras tu **prop** **onClick**. Si estás escuchando un evento `"hover"`, entonces nombras tu **prop** **onHover**:

```jsx
function myClass(){
  function handleHover() {
    alert('Soy un manejador de eventos.');
    alert('Escucharé un evento "hover".');
  }
  return <Child onHover={handleHover} />;
}
```

Perfecto bro 👌 esto es **MUY importante** y confunde a muchos.
Te lo explico **ultra simple y directo**.

### 🧠 Idea clave (una sola frase)

👉 **`onClick` solo es “especial” cuando se usa en elementos HTML (`<button>`, `<div>`, etc.)**

👉 **En componentes propios (`<Button />`) es solo un nombre de prop**

#### 🔍 Caso 1: HTML real (SÍ es especial)

```jsx
<button onClick={props.onClick}>
  Click me!
</button>
```

Aquí:

* `<button>` 👉 es HTML real
* `onClick` 👉 **React sabe que es un evento**
* React crea un **event listener**
* Cuando haces click 👉 se ejecuta la función

✅ Aquí `onClick` **TIENE significado especial**

### 🔍 Caso 2: Componente propio (NO es especial)

```jsx
<Button onClick={handleClick} />
```

Aquí:

* `<Button />` 👉 **NO es HTML**
* Es solo un **componente React**
* `onClick` 👉 **NO crea ningún evento**
* Es solo una **prop con ese nombre**

👉 React lo trata como:

```js
props = {
  onClick: handleClick
}
```

❌ No hay listener todavía

### 🔁 Entonces… ¿cuándo funciona el click?

Funciona cuando **tú conectas las cosas** 👇

#### Button.js

```jsx
function Button(props) {
  return (
    <button onClick={props.onClick}>
      Click me!
    </button>
  );
}
```

Aquí pasa la magia:

* El `onClick` del **HTML button**
* recibe la función que vino como prop
* ahora SÍ hay evento

### 🧠 Modelo mental definitivo (memorízalo)

```text
onClick en HTML  → evento real
onClick en componente → solo una prop
```

O más claro aún:

> Los eventos solo existen en elementos HTML, no en componentes React.

### 🎯 Por qué esto es así

Porque React:

* No sabe qué eventos quieres en tus componentes
* Solo entiende eventos cuando ve **HTML real**
* Los componentes solo **reciben datos y funciones**

### ✅ Resumen en 3 bullets

* `<button onClick={...}>` → evento
* `<Button onClick={...} />` → prop normal
* El evento nace SOLO cuando llega a HTML

----

## props.children

Cada objeto **props** de un componente tiene una propiedad llamada **children**.

`props.children` devolverá todo lo que esté entre las etiquetas JSX de apertura y cierre de un componente.

Hasta ahora, todos los componentes que has visto han sido etiquetas autocerradas, como `<MyFunctionComponent />`. ¡No tienen que serlo! Podrías escribir `<MyFunctionComponent></MyFunctionComponent>` y aún así funcionaría.

`props.children` devolvería todo lo que esté entre `<MyFunctionComponent>` y `</MyFunctionComponent>`.

Al usar **props.children**, podemos separar el componente exterior, en este caso **MyFunctionComponent**, del contenido, lo que lo hace flexible y reutilizable.

Mira **BigButton.js**:

* En el Ejemplo 1, `props.children` de `<BigButton>` sería igual al texto: “I am a child of BigButton.”
* En el Ejemplo 2, `props.children` de `<BigButton>` sería igual a un componente `<LilButton />`.
* En el Ejemplo 3, `props.children` de `<BigButton>` sería **undefined**.

Si un componente tiene más de un hijo entre sus etiquetas JSX, entonces `props.children` devolverá esos hijos en un arreglo. Sin embargo, si un componente tiene solo un hijo, entonces `props.children` devolverá ese único hijo, **sin envolverlo en un arreglo**.

----

## Giving Default Values to props

Mira el componente **Button**. Fíjate que en la línea 6, **Button** espera recibir una **prop** llamada **text**. El **text** recibido se mostrará dentro de un elemento `<button>`.

¿Qué pasa si nadie le pasa texto a **Button**?

Si nadie le pasa texto a **Button**, entonces lo que se mostrará será un botón vacío. Sería mejor si **Button** pudiera mostrar un mensaje predeterminado en su lugar.

Puedes hacer que esto suceda especificando un valor predeterminado para la **prop**. ¡Hay dos formas de hacerlo!

El primer método es especificar el valor predeterminado directamente en la definición de la función:

```jsx
function Example({text='This is default text'}) {
   return <h1>{text}</h1>;
}
```

También puedes establecer el valor predeterminado dentro del cuerpo de la función:

```jsx
function Example(props) {
  const {text = 'This is default text'} = props;
  return <h1>{text}</h1>;
}
```

Si a `<Example />` no se le pasa ningún texto, entonces mostrará “This is default text”.

Si a `<Example />` se le pasa algún texto, entonces mostrará ese texto pasado como **prop**.

-----

## Review

¡Eso completa nuestra lección sobre **props**! Aquí están algunas de las habilidades que has aprendido:

* Pasar una **prop** al darle un atributo a una instancia de componente
* Acceder a una **prop** pasada a través de `props.propName`
* Mostrar una **prop**
* Usar una **prop** para tomar decisiones sobre qué mostrar
* Definir un manejador de eventos en un componente de función
* Pasar un manejador de eventos como una **prop**
* Recibir un manejador de eventos como **prop** y adjuntarlo a un escuchador de eventos
* Seguir las convenciones de nombres para los manejadores de eventos y sus atributos
* Acceder a **props.children**
* Asignar valores predeterminados a las **props**

¡Eso es mucho! No te preocupes si todo esto parece un poco confuso. ¡Pronto tendrás mucha práctica!

-----