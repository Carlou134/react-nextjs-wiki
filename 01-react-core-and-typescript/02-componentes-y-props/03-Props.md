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

## ¿Cuándo usar cada concepto de esta lección?

- **Acceder a props** (`props.name`) — siempre que un componente necesite mostrar o usar algo que le llega desde afuera.
- **Pasar props** — cuando el padre necesita personalizar o configurar a un hijo (`<Product name="..." price={...} />`).
- **Renderizado condicional según props** — cuando es el **hijo** el que tiene que decidir qué mostrar según lo que recibió, sin que el padre le diga literalmente "mostrate de tal forma".
- **Evento como prop** (el patrón `Talker`/`Button`) — cuando el padre necesita reaccionar a algo que pasa **dentro** del hijo, como un click. El padre define la lógica (`talk`), el hijo decide el momento en que se dispara (`onClick={talk}`).
- **Convención `handleX` / `onX`** — siempre que definís un manejador de eventos y lo pasás como prop, para que cualquiera que lea el código entienda de qué se trata sin tener que rastrear la implementación.
- **`children`** — cuando el contenido interno de un componente varía completamente según quien lo use (layouts, wrappers, cards), al punto de que enumerar ese contenido como props nombradas no tendría sentido.
- **Valores por defecto** — cuando una prop es opcional y existe un valor razonable para cuando nadie la pasa explícitamente.

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

> **En TypeScript:** acá es donde vas a sentir el mayor cambio respecto de JSX puro. Cada componente necesita declarar la forma de sus props con un `type` (o `interface`), y usarlo para anotar el parámetro:
>
> ```tsx
> type ButtonProps = {
>   displayText: string;
> };
>
> function Button({ displayText }: ButtonProps) {
>   return <button>{displayText}</button>;
> }
> ```
>
> A cambio de escribir ese tipo, ganás que TypeScript te avise en el momento —no cuando la app ya está corriendo— si te olvidaste de pasar una prop obligatoria, si le pasaste un tipo de dato equivocado (un número donde se esperaba un string), o si escribiste mal el nombre de una prop al usar el componente. Vemos el tipado de props en detalle, con más ejemplos, en [11-typescript-y-react/01-Tipado de Props y Funciones.md](../11-typescript-y-react/01-Tipado%20de%20Props%20y%20Funciones.md).

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

Puedes, y a menudo lo harás, pasar funciones como **props**. Es especialmente común pasar funciones de manejo de eventos: un componente padre define **qué** debe pasar cuando ocurre un evento, y se lo entrega a un componente hijo para que decida **cuándo** dispararlo.

Antes de poder pasar un manejador de eventos a algún lado, primero hay que definirlo. Se define exactamente igual que cualquier otra función dentro de un componente de función:

```jsx
function Talker() {
  function talk() {
    let speech = '';
    for (let i = 0; i < 10000; i++) {
      speech += 'blah ';
    }
    alert(speech);
  }

  return <Button talk={talk} />;
}

export default Talker;
```

En este componente **Talker**, `talk` es una función común de JavaScript, definida dentro del cuerpo del componente: cuando se la invoque, va a construir un texto largo y mostrarlo en una alerta. Hasta acá no hay nada nuevo — `talk` es solo una función. Lo interesante empieza en la línea `return`, donde esa función se pasa como **prop** al componente `Button`.

-----

## Pass an Event Handler as a prop

Fíjate en la línea `return` de **Talker**:

```jsx
return <Button talk={talk} />;
```

Acá está el punto clave de esta lección: `talk={talk}` pasa la **función en sí** — no el resultado de ejecutarla. Nota que no hay paréntesis después de `talk`. Si hubiéramos escrito `talk={talk()}`, la función se habría ejecutado inmediatamente durante el renderizado de `Talker` (mostrando la alerta enseguida), y lo que se pasaría como prop sería el valor que `talk()` devuelve — en este caso `undefined` — no la función. Al pasarla sin paréntesis, le entregamos a `Button` la **referencia** a la función, para que sea `Button` quien decida cuándo (y si) llamarla.

Esto conecta con algo que ya sabés: las funciones en JavaScript son valores como cualquier otro, así que se pueden pasar como props de la misma manera que pasarías un string o un número.

En tiempo de ejecución, la secuencia es la siguiente: React renderiza `Talker`, `Talker` crea la función `talk`, se la pasa a `Button` como prop, y `Button` decide en qué momento invocarla — normalmente dentro de un manejador de eventos como `onClick`.

-----

## Receive an Event Handler as a prop

Del lado de `Button`, la función `talk` llega como una propiedad más dentro del objeto `props`. Para dispararla cuando el usuario haga clic, hay que adjuntarla al elemento `<button>` como manejador del evento `onClick`:

```jsx
function Button(props) {
  return <button onClick={props.talk}>Talk</button>;
}
```

Vale la pena entender por qué `props` es un objeto. Cuando `Talker` escribe `<Button talk={talk} color="red" />`, React agrupa **todos** los atributos que le pasaste al componente dentro de un único objeto `props`, algo equivalente a:

```js
props = {
  talk: talk,
  color: "red"
};
```

Por eso accedés a la función con `props.talk`: es simplemente la propiedad `talk` de ese objeto. Y, al igual que en `Talker`, se la pasa a `onClick` sin paréntesis (`props.talk`, no `props.talk()`) — es React quien la va a invocar automáticamente cuando detecte el clic.

Dado que `props` es un objeto, también podés desestructurarlo directamente en los parámetros de la función, lo cual suele ser más legible cuando el componente usa pocas props:

```jsx
function Button({ talk }) {
  return <button onClick={talk}>Talk</button>;
}
```

Ambas versiones hacen exactamente lo mismo; la segunda es simplemente la forma más común de escribirlo en código moderno de React.

> **En TypeScript:** una prop que es una función también se tipa con una firma de función, no solo con un nombre genérico. Si `talk` no recibe argumentos y no devuelve nada útil, se tipa como `() => void`:
>
> ```tsx
> type ButtonProps = {
>   talk: () => void;
> };
>
> function Button({ talk }: ButtonProps) {
>   return <button onClick={talk}>Talk</button>;
> }
> ```
>
> Esto es lo que hace que el error de "pasar `talk()` en lugar de `talk`" sea mucho menos probable en la práctica: si en algún punto `talk` esperara un argumento y te olvidás de pasárselo al invocarla, TypeScript te lo va a marcar.

----

## handleEvent, onEvent, and props.onEvent

Cuando pasás un manejador de eventos como prop, hay dos nombres que tenés que elegir, y ambos se deciden en el componente padre (el que define el manejador y lo pasa hacia abajo):

1. El nombre del **manejador de eventos** en sí.
2. El nombre de la **prop** que usás para pasarlo.

En el ejemplo de `Talker`, elegimos llamar `talk` tanto a la función como a la prop:

```jsx
function talk() { /* ... */ }

return <Button talk={talk} />;
```

Estos dos nombres pueden ser cualquier cosa que quieras, pero existe una convención muy extendida en la comunidad de React que conviene seguir. Para el nombre del manejador, se usa la palabra **handle** seguida del tipo de evento: si escuchás un `"click"`, el manejador se llama **handleClick**; si escuchás un `"hover"`, se llama **handleHover**:

```jsx
function MyComponent() {
  function handleHover() {
    alert('Soy un manejador de eventos.');
    alert('Se llamará en respuesta a eventos "hover".');
  }

  return <Child onHover={handleHover} />;
}
```

Para el nombre de la prop, se usa la palabra **on** seguida del tipo de evento: **onClick**, **onHover**, y así sucesivamente — como se ve en el ejemplo anterior, donde la prop se llama `onHover`.

Ahora bien, hay un punto que suele generar confusión: `onClick` (o cualquier otro `on...`) **no es una palabra mágica que React reconozca en todos lados**. Su significado depende de dónde se use.

Cuando `onClick` se coloca sobre un **elemento HTML nativo** — como `<button>`, `<div>` o `<img>` — React sí le da un tratamiento especial: registra un verdadero **escuchador de eventos** del DOM, y ejecuta la función que le pasaste cuando ocurre el clic real.

```jsx
// <button> es un elemento HTML nativo: onClick SÍ es un evento real
<button onClick={props.onClick}>
  Click me!
</button>
```

Pero cuando `onClick` se coloca sobre un **componente propio** — como `<Button />` — React no le atribuye ningún significado especial. `<Button />` no es HTML, es una función de JavaScript que vos escribiste; React simplemente empaqueta `onClick` dentro del objeto `props` de ese componente, igual que haría con cualquier otro nombre de atributo (`talk`, `color`, `size`, lo que sea):

```jsx
// Button no es HTML: acá onClick es solo el nombre de una prop, sin comportamiento propio
<Button onClick={handleClick} />

// React internamente arma:
// props = { onClick: handleClick }
```

En este segundo caso, todavía **no existe ningún escuchador de eventos**. Solo se genera un evento real en el momento en que, dentro de la definición de `Button`, esa prop termina adjuntada a un elemento HTML nativo:

```jsx
function Button(props) {
  return (
    <button onClick={props.onClick}>
      Click me!
    </button>
  );
}
```

Ahí es donde ocurre la conexión: `props.onClick` (el nombre que elegiste para tu prop) se asigna al `onClick` del `<button>` (el evento real del DOM), y recién en ese punto el clic queda conectado a la función.

La regla general para tener en mente es esta: **los eventos del navegador solo existen sobre elementos HTML**. Sobre un componente propio, cualquier nombre que empiece con `on` —lo elijas vos— es apenas una convención de nomenclatura, no un mecanismo del lenguaje ni de React. React no sabe de antemano qué eventos vas a necesitar en tus propios componentes; solo entiende eventos cuando, en algún nivel de la jerarquía, la prop llega a un elemento HTML real.

----

## props.children

Cada objeto **props** de un componente tiene una propiedad llamada **children**.

`props.children` devolverá todo lo que esté entre las etiquetas JSX de apertura y cierre de un componente.

Hasta ahora, todos los componentes que has visto han sido etiquetas autocerradas, como `<MyFunctionComponent />`. ¡No tienen que serlo! Podrías escribir `<MyFunctionComponent></MyFunctionComponent>` y aún así funcionaría.

`props.children` devolvería todo lo que esté entre `<MyFunctionComponent>` y `</MyFunctionComponent>`.

Al usar **props.children**, podemos separar el componente exterior, en este caso **MyFunctionComponent**, del contenido, lo que lo hace flexible y reutilizable.

```jsx
function BigButton(props) {
  return <button>{props.children}</button>;
}
```

Con este mismo componente `BigButton`, el valor de `props.children` cambia según lo que le pases entre sus etiquetas de apertura y cierre:

```jsx
// Ejemplo 1: props.children es el string "I am a child of BigButton."
<BigButton>I am a child of BigButton.</BigButton>

// Ejemplo 2: props.children es el elemento <LilButton />
<BigButton>
  <LilButton />
</BigButton>

// Ejemplo 3: al ser autocerrado, no hay nada entre etiquetas, así que props.children es undefined
<BigButton />
```

> **En TypeScript:** `children` se tipa con `React.ReactNode`, un tipo pensado específicamente para cubrir todo lo que React puede renderizar: un string, un número, un elemento JSX, un array de elementos, o incluso `undefined`/`null` (para cuando no se pasa nada, como en el Ejemplo 3):
>
> ```tsx
> type BigButtonProps = {
>   children: React.ReactNode;
> };
>
> function BigButton({ children }: BigButtonProps) {
>   return <button>{children}</button>;
> }
> ```
>
> Si tu componente puede usarse sin hijos (como el Ejemplo 3), marcá la prop como opcional con `children?: React.ReactNode`.

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

> **En TypeScript:** los valores por defecto se combinan naturalmente con las **props opcionales**. Si `text` tiene un valor por defecto, entonces quien use `<Example />` no está obligado a pasarlo — y eso hay que reflejarlo en el tipo marcando la prop con `?`:
>
> ```tsx
> type ExampleProps = {
>   text?: string;
> };
>
> function Example({ text = 'This is default text' }: ExampleProps) {
>   return <h1>{text}</h1>;
> }
> ```
>
> Si te olvidás del `?` y dejás `text: string`, TypeScript te va a exigir la prop igual, aunque en tiempo de ejecución el valor por defecto funcione — porque, desde el punto de vista de los tipos, una prop sin `?` es obligatoria.

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