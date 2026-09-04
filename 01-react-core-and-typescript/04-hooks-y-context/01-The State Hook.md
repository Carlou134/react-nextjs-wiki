# The State Hook

## Why Use Hooks?

¿Qué deberíamos hacer si queremos añadir estado a nuestro componente de función? ¿Y si queremos que nuestra aplicación responda a los cambios en los datos?

En esta lección, aprenderemos sobre los **React Hooks** y cómo pueden ayudarnos a aprovechar de forma poderosa los componentes de función.

Los **React Hooks**, dicho de forma simple, son funciones que nos permiten gestionar el estado interno de los componentes y manejar efectos secundarios posteriores al renderizado directamente desde nuestros componentes de función. Al usar Hooks, podemos determinar qué queremos mostrar a los usuarios declarando cómo debe verse nuestra interfaz de usuario en función del estado.

React ofrece varios Hooks integrados. Algunos de ellos incluyen
**useState()**,
**useEffect()**,
**useContext()**,
**useReducer()** y
**useRef()**.
Puedes ver la lista completa en la documentación de React.

En esta lección, aprenderemos a:

* Construir un componente de función con estado.
* Usar el Hook de estado.
* Inicializar un estado y actualizarlo.
* Definir manejadores de eventos.
* Usar funciones callback del actualizador de estado.
* Usar el estado con arreglos y objetos.

-----

## ¿Cuándo usar useState?

Usá `useState()` cuando un componente necesita "recordar" un valor entre renders, y ese valor tiene que provocar que React vuelva a dibujar la pantalla cuando cambia. Es la opción por defecto para datos **simples e independientes entre sí**: un booleano, un string, un número, un array u objeto chico que cambia como una sola unidad.

No es la herramienta correcta cuando la lógica de actualización se vuelve compleja — varias acciones posibles sobre el mismo estado, o varios sub-valores que tienen que mantenerse coherentes entre sí. En ese punto, conviene escalar a `useReducer()` (lo vemos en la lección de ese mismo nombre, más adelante en esta carpeta) en lugar de forzar todo dentro de `useState()`.

-----

## Update Function Component State

Comencemos con el **State Hook**, el Hook más comúnmente usado para construir componentes de React. El **State Hook** es una exportación nombrada de la biblioteca de React, por lo que lo importamos usando **desestructuración de objetos**, así:

```javascript
import { useState } from 'react';
```

Cuando llamamos a la función **useState()**, esta devuelve un arreglo con dos valores:

1. **El estado actual**: el valor actual de este estado.
2. **El actualizador de estado**: una función que podemos usar para actualizar el valor de este estado.

Podemos usar estos dos valores para seguir el estado actual de un valor de datos o propiedad y cambiarlo cuando sea necesario. Para extraer estos dos valores del arreglo, los podemos asignar a variables locales usando **desestructuración de arreglos**. Por ejemplo:

```javascript
const [currentState, setCurrentState] = useState();
```

Veamos otro ejemplo de un componente de función que usa el **State Hook**:

```javascript
import { useState } from "react";

function Toggle() {
  const [toggle, setToggle] = useState();

  return (
    <div>
      <p>El toggle está {toggle}</p>
      <button onClick={() => setToggle("On")}>On</button>
      <button onClick={() => setToggle("Off")}>Off</button>
    </div>
  );
}
```

Fíjate cómo la función actualizadora de estado, **setToggle()**, es llamada por nuestros **event listeners** de `onClick`. Para actualizar el valor de `toggle` y **volver a renderizar** este componente con el nuevo valor, todo lo que necesitamos hacer es llamar a la función **setToggle()** pasando el siguiente valor del estado como argumento.

> **En TypeScript:** `useState()` sin argumento, como en `useState()`, deja a TypeScript sin ninguna pista sobre qué tipo de dato vas a guardar ahí — infiere `undefined`, lo cual te va a impedir asignarle luego un string como `"On"`. Cuando no tenés un valor inicial concreto para inferir el tipo, pasáselo explícitamente como **generic**: `useState<string>()`. El estado va a quedar tipado como `string | undefined` (todavía puede ser `undefined` hasta la primera actualización), lo cual es más preciso que dejar que TypeScript adivine.

Con el **State Hook**, actualizar el estado es tan simple como llamar a una función actualizadora. Llamar a esta función le indica a React que el componente necesita **volver a renderizarse**, por lo que toda la función que define el componente se ejecuta de nuevo.
La magia de **useState()** es que permite a React **mantener el seguimiento del valor actual del estado de un renderizado al siguiente**.

-----

## Initialize State

Al igual que usamos el **State Hook** para manejar una variable con valores de tipo cadena, ¡podemos usar el **State Hook** para manejar el valor de cualquier tipo de dato primitivo e incluso colecciones de datos como arreglos y objetos!

Observa el siguiente componente de función. ¿Qué tipo de dato contiene esta variable de estado?

```javascript
import { useState } from 'react';

function ToggleLoading() {
  const [isLoading, setIsLoading] = useState();

  return (
    <div>
      <p>Los datos están {isLoading ? 'Cargando' : 'No Cargando'}</p>
      <button onClick={() => setIsLoading(true)}>
        Activar carga
      </button>
      <button onClick={() => setIsLoading(false)}>
        Desactivar carga
      </button>
    </div>
  );
}
```

El componente de función **ToggleLoading()** anterior utiliza el tipo de dato más simple de todos: un **booleano**. Los booleanos se usan frecuentemente en aplicaciones de React para representar si los datos se han cargado o no. En el ejemplo anterior, vemos que los valores `true` y `false` se pasan a la función actualizadora de estado, **setIsLoading()**.

Este código funciona perfectamente tal como está, pero ¿qué pasa si queremos que nuestro componente comience con **isLoading** establecido en `true`?

Para inicializar nuestro estado con cualquier valor que queramos, simplemente pasamos el valor inicial como argumento a la función **useState()**:

```javascript
const [isLoading, setIsLoading] = useState(true);
```

Este código afecta a nuestro componente de tres maneras:

1. Durante el primer renderizado, se usa el argumento del estado inicial.
2. Cuando se llama a la función actualizadora de estado, React **ignora** el argumento del estado inicial y usa el nuevo valor.
3. Cuando el componente se vuelve a renderizar por cualquier otra razón, React continúa usando el mismo valor del renderizado anterior.

Si no pasamos un valor inicial al llamar a **useState()**, el valor actual del estado durante el primer renderizado será `undefined`. Esto suele estar bien para la computadora que ejecuta el código, pero puede ser confuso para las personas que leen nuestro código. Por eso, es preferible **inicializar explícitamente** nuestro estado. Si no tenemos el valor necesario durante el primer renderizado, podemos pasar `null` explícitamente en lugar de dejar el valor como `undefined`.

-----

## Use State Setter Outside of JSX

Veamos un ejemplo de cómo manejar el valor cambiante de una cadena mientras un usuario escribe en un campo de entrada de texto:

```javascript
import { useState } from 'react';

export default function EmailTextInput() {
  const [email, setEmail] = useState('');
  const handleChange = (event) => {
    const updatedEmail = event.target.value;
    setEmail(updatedEmail);
  }

  return (
    <input value={email} onChange={handleChange} />
  );
}
```

Aquí hay un desglose de cómo funciona el código anterior:

* Usamos **desestructuración de arreglos** para crear nuestra variable de estado local `email` y nuestra función local actualizadora `setEmail()`.
* La variable local `email` recibe el valor actual del estado en el índice 0 del arreglo devuelto por **useState()**.
* La variable local `setEmail()` recibe una referencia a la función actualizadora de estado en el índice 1 del arreglo devuelto por **useState()**.
* Es una convención nombrar la variable del actualizador usando la variable de estado actual (en este ejemplo, `email`) con “set” al principio.
* La etiqueta **input** de JSX tiene un **event listener** llamado `onChange`. Este listener llama a un manejador de eventos cada vez que el usuario escribe algo en este elemento. En el ejemplo anterior, nuestro manejador de eventos se define dentro de la definición de nuestro componente de función, pero fuera del JSX.

  * Antes en esta lección, escribimos nuestros manejadores de eventos directamente en el JSX.
  * Esos manejadores en línea funcionan perfectamente, pero cuando queremos hacer algo más complejo que simplemente llamar al actualizador de estado con un valor estático, es buena práctica separar esa lógica de nuestro JSX. Esta separación de responsabilidades hace que nuestro código sea más fácil de leer, probar y modificar.

Es común en React simplificar este código:

```javascript
const handleChange = (event) => {
  const newEmail = event.target.value;
  setEmail(newEmail);
}
```

a esto:

```javascript
const handleChange = (event) => setEmail(event.target.value);
```

o, usando **desestructuración de objetos**, así:

```javascript
const handleChange = ({target}) => setEmail(target.value);
```

Los tres fragmentos de código anteriores se comportan igual, por lo que realmente no hay una forma correcta o incorrecta entre ellos. Usaremos la última versión, la más concisa, de aquí en adelante.

----

## Set From Previous State

En el ejercicio anterior, aprendimos a actualizar el estado pasándole un nuevo valor de esta forma:

```javascript
setState(newStateValue);
```

Sin embargo, las actualizaciones de estado en React son **asíncronas**. Esto significa que hay algunos escenarios en los que partes de tu código se ejecutarán antes de que el estado termine de actualizarse.

¡Esto es algo bueno y algo malo! Agrupar las actualizaciones de estado puede mejorar el rendimiento de tu aplicación, pero también puede provocar que se usen valores de estado desactualizados. Por ello, es una **buena práctica** actualizar el estado usando una **función callback**, lo que ayuda a prevenir valores obsoletos por accidente.

Veamos el siguiente código para entender cómo se hace:

```javascript
import { useState } from 'react';
 
export default function Counter() {
  const [count, setCount] = useState(0);
 
  const increment = () => setCount(prevCount => prevCount + 1);
 
  return (
    <div>
      <p>Wow, has hecho clic en ese botón: {count} veces</p>
      <button onClick={increment}>¡Haz clic aquí!</button>
    </div>
  );
}
```

Cuando se presiona el botón, se llama al manejador de eventos `increment()`. Dentro de esta función, usamos nuestro actualizador de estado `setCount()` con una **función callback**.

Debido a que el siguiente valor de `count` depende del valor anterior de `count`, pasamos una función callback como argumento a `setCount()` en lugar de pasar un valor directamente (como hicimos en ejercicios anteriores):

```javascript
setCount(prevCount => prevCount + 1)
```

Cuando el actualizador de estado llama a la función callback, esta recibe el valor anterior de `count` como argumento. El valor que retorna esta función callback se usa como el siguiente valor de `count` (en este caso, `prevCount + 1`).

También podríamos simplemente llamar a `setCount(count + 1)` y funcionaría igual en este ejemplo, pero por razones que quedan fuera del alcance de esta lección, es más seguro usar el método con callback.

-----

## Arrays in State

Los **arreglos de JavaScript** son el mejor modelo de datos para gestionar y renderizar listas en **JSX**. Veamos el código de un sitio web para un restaurante de pizza.

```javascript
import { useState } from 'react';

// Arreglo estático de opciones de pizza disponibles.
const options = ['Bell Pepper', 'Sausage', 'Pepperoni', 'Pineapple'];

export default function PersonalPizza() {
  const [selected, setSelected] = useState([]);

  const toggleTopping = ({target}) => {
    const clickedTopping = target.value;
    setSelected((prev) => {
      // comprobar si el ingrediente seleccionado ya está elegido
      if (prev.includes(clickedTopping)) {
        // eliminar el ingrediente seleccionado del estado
        return prev.filter(t => t !== clickedTopping);
      } else {
        // agregar el ingrediente seleccionado a nuestro estado
        return [clickedTopping, ...prev];
      }
    });
  };

  return (
    <div>
      {options.map(option => (
        <button value={option} onClick={toggleTopping} key={option}>
          {selected.includes(option) ? 'Quitar ' : 'Agregar '}
          {option}
        </button>
      ))}
      <p>Pide una pizza de {selected.join(', ')}</p>
    </div>
  );
}
```

En el ejemplo anterior, estamos usando dos arreglos:

* El arreglo **options** contiene los nombres de todos los ingredientes de pizza disponibles.
* El arreglo **selected** representa los ingredientes seleccionados para nuestra pizza personalizada.

El arreglo **options** contiene datos estáticos, lo que significa que no cambian. Es una buena práctica definir los modelos de datos estáticos fuera de los componentes de función, ya que no necesitan recrearse cada vez que el componente se vuelve a renderizar. En nuestro JSX, usamos el método `.map()` de JavaScript para renderizar un botón por cada ingrediente en el arreglo **options**.

El arreglo **selected** contiene datos dinámicos, lo que significa que cambian, generalmente en función de las acciones del usuario. Inicializamos **selected** como un arreglo vacío. Cuando se hace clic en un botón, se llama al manejador de eventos `toggleTopping()`. Observa cómo este manejador usa información del objeto del evento para determinar qué ingrediente fue seleccionado.

> **En TypeScript:** `useState([])` es otro caso donde no hay nada que inferir a partir del valor inicial — TypeScript le asigna el tipo `never[]`, un arreglo que **no admite agregar ningún elemento**, así que `setSelected((prev) => [clickedTopping, ...prev])` va a marcar error de tipos. La solución es la misma que con cualquier `useState()` sin datos suficientes para inferir: pasar el tipo explícito como generic, `useState<string[]>([])`.

Al actualizar un arreglo en el estado, no simplemente agregamos nuevos datos al arreglo anterior. Reemplazamos el arreglo anterior con uno completamente nuevo. Esto significa que cualquier información que queramos conservar del arreglo anterior debe copiarse explícitamente al nuevo arreglo. Para eso usamos la **sintaxis spread**: `...prev`.

Fíjate cómo usamos los métodos `.includes()`, `.filter()` y `.map()` de los arreglos. Si estos métodos son nuevos para ti o solo quieres refrescar conceptos, tómate un momento para revisarlos. No necesitamos ser expertos absolutos en JavaScript para construir aplicaciones con React, pero invertir tiempo en fortalecer nuestras habilidades en JavaScript siempre nos ayudará a hacer más cosas, más rápido (y a divertirnos mucho más) como desarrolladores de React.

-----

## Objects in State

También podemos usar el **estado con objetos**. Cuando trabajamos con un conjunto de variables relacionadas, puede ser muy útil agruparlas dentro de un objeto. Veamos un ejemplo de esto en acción.

```javascript
export default function Login() {
  const [formState, setFormState] = useState({});
  const handleChange = ({ target }) => {
    const { name, value } = target;
    setFormState((prev) => ({
      ...prev,
      [name]: value
    }));
  };

  return (
    <form>
      <input
        value={formState.firstName}
        onChange={handleChange}
        name="firstName"
        type="text"
      />
      <input
        value={formState.password}
        onChange={handleChange}
        type="password"
        name="password"
      />
    </form>
  );
}
```

Algunas cosas a tener en cuenta:

* Usamos una **función callback** del actualizador de estado para modificar el estado basándonos en su valor anterior.
* La **sintaxis spread** es la misma para objetos que para arreglos:
  `{ ...objetoAnterior, nuevaClave: nuevoValor }`.
* Reutilizamos nuestro manejador de eventos para múltiples inputs usando el atributo `name` de la etiqueta `input` para identificar de qué input provino el evento de cambio.
* Una vez más, al actualizar el estado con `setFormState()` dentro de un componente de función, **no modificamos el mismo objeto**. Debemos copiar los valores del objeto anterior al establecer el nuevo valor del estado. Afortunadamente, la sintaxis spread hace que esto sea muy fácil de lograr.

Cada vez que se actualiza uno de los valores de los inputs, se llama a la función `handleChange()`. Dentro de este manejador de eventos, usamos **desestructuración de objetos** para extraer la propiedad `target` del objeto del evento y luego usamos nuevamente desestructuración de objetos para extraer las propiedades `name` y `value` del objeto `target`.

Dentro de la función callback del actualizador de estado, envolvemos las llaves en paréntesis de esta forma:

```javascript
setFormState((prev) => ({ ...prev }))
```

Esto le indica a JavaScript que las llaves representan un **nuevo objeto** que debe ser retornado. Usamos `...`, el operador spread, para copiar los campos correspondientes del estado anterior. Finalmente, sobrescribimos la clave adecuada con su valor actualizado.

¿Notaste los **corchetes** alrededor de `name`? Este **nombre de propiedad computado (Computed Property Name)** nos permite usar el valor de cadena almacenado en la variable `name` como clave de la propiedad.

> **En TypeScript:** al igual que con los arreglos, `useState({})` infiere el tipo `{}` — un objeto del que TypeScript no sabe qué propiedades tiene, así que `formState.firstName` va a marcar error. Acá lo correcto no es forzar un generic vacío, sino declarar primero la forma completa del objeto con un `type`, y usarlo para tipar el estado:
>
> ```tsx
> type FormState = {
>   firstName: string;
>   password: string;
> };
>
> const [formState, setFormState] = useState<FormState>({ firstName: '', password: '' });
> ```
>
> De paso, esto también obliga a inicializar el objeto con todas sus propiedades desde el principio, en vez de arrancar de un `{}` vacío e ir completándolo — una forma más segura de evitar accesos a propiedades que todavía no existen.

----

## Separate Hooks for Separate States

Aunque hay ocasiones en las que puede ser útil almacenar datos relacionados en una colección de datos, como un arreglo u objeto, también puede ser útil crear **diferentes variables de estado** para los datos que cambian de forma independiente. Gestionar datos dinámicos es mucho más fácil cuando mantenemos nuestros modelos de datos lo más simples posible.

Por ejemplo, si tuviéramos un solo objeto que almacenara el estado de una materia que estás estudiando en la escuela, podría verse algo así:

```javascript
function Subject() {
  const [state, setState] = useState({
    currentGrade: 'B',
    classmates: ['Hasan', 'Sam', 'Emma'],
    classDetails: {topic: 'Math', teacher: 'Ms. Barry', room: 201},
    exams: [{unit: 1, score: 91}, {unit: 2, score: 88}]
  })
}
```

Esto funcionaría, pero piensa en lo complicado que podría volverse copiar todos los demás valores cada vez que necesitamos actualizar algo dentro de este gran objeto de estado. Por ejemplo, para actualizar la calificación de un examen, necesitaríamos un manejador de eventos que hiciera algo como esto:

```javascript
setState((prev) => ({
  ...prev,
  exams: prev.exams.map((exam) => {
    if (exam.unit === updatedExam.unit) {
      return { 
        ...exam,
        score: updatedExam.score
      };
    } else {
      return exam;
    }
  }),
}));
```

Código complejo como este es propenso a causar errores. Es mejor crear **múltiples variables de estado** basadas en qué valores tienden a cambiar juntos.

Podemos reescribir el ejemplo anterior de la siguiente manera:

```javascript
function Subject() {
  const [currentGrade, setGrade] = useState('B');
  const [classmates, setClassmates] = useState(['Hasan', 'Sam', 'Emma']);
  const [classDetails, setClassDetails] = useState({
    topic: 'Math',
    teacher: 'Ms. Barry',
    room: 201
  });
  const [exams, setExams] = useState([
    {unit: 1, score: 91},
    {unit: 2, score: 88}
  ]);
  // ...
}
```

Gestionar datos dinámicos con variables de estado separadas tiene muchas ventajas, como hacer que nuestro código sea más sencillo de escribir, leer, probar y reutilizar entre componentes.

A menudo, nos encontramos empaquetando y organizando datos en colecciones para pasarlos entre componentes, y luego separando esos datos dentro de los componentes donde distintas partes cambian de manera independiente.
¡Lo maravilloso de trabajar con **Hooks** es que tenemos la libertad de organizar nuestros datos de la forma que tenga más sentido para nosotros!

-----

## Review

¡Ahora podemos construir **componentes de función con estado** usando el Hook de React **useState**!

Repasemos lo que aprendimos y practicamos en esta lección:

* Con React, alimentamos modelos de datos estáticos y dinámicos a **JSX** para renderizar una vista en la pantalla.
* Los **Hooks** se utilizan para “conectarse” al estado interno del componente y así gestionar datos dinámicos en componentes de función.
* Usamos el **State Hook** con el siguiente código. `currentState` hace referencia al valor actual del estado y `initialState` inicializa el valor del estado para el primer renderizado del componente:

```javascript
const [currentState, stateSetter] = useState(initialState);
```

* Los actualizadores de estado pueden llamarse dentro de manejadores de eventos.
* Podemos definir manejadores de eventos simples directamente en nuestro JSX y manejadores más complejos fuera del JSX.
* Usamos una **función callback** del actualizador de estado cuando el siguiente valor depende del valor anterior.
* Usamos arreglos y objetos para organizar y gestionar datos relacionados que tienden a cambiar juntos.
* Usamos la **sintaxis spread** en colecciones de datos dinámicos para copiar el estado anterior al nuevo estado, por ejemplo:
  `setArrayState((prev) => [ ...prev ])` y
  `setObjectState((prev) => ({ ...prev }))`.
* Es una **buena práctica** tener múltiples estados más simples en lugar de un único objeto de estado complejo.

-----

## Reglas de los Hooks

Antes de seguir avanzando hacia otros Hooks de React, es importante interiorizar las reglas que rigen su funcionamiento. No son convenciones de estilo opcionales: si no se respetan, React pierde la capacidad de asociar correctamente cada Hook con el estado o el efecto que le corresponde, y la aplicación empieza a comportarse de forma impredecible.

### Los Hooks solo funcionan en componentes de función

Los Hooks fueron diseñados exclusivamente para **componentes de función**. No pueden usarse dentro de componentes de clase, que en su lugar siguen dependiendo de `this.state` y de los métodos de ciclo de vida como `componentDidMount()`. Tampoco pueden llamarse desde funciones de JavaScript comunes que no sean componentes de React. En otras palabras, `useState()` solo tiene sentido cuando React sabe que está renderizando un componente y puede asociarle una instancia de estado.

### Los Hooks se llaman siempre en el nivel superior

La segunda regla es igual de estricta: los Hooks deben llamarse siempre en el **nivel superior** de la función del componente, nunca dentro de una condición (`if`), un bucle (`for`, `while`) ni una función anidada.

La razón tiene que ver con **cómo React realiza el seguimiento de los Hooks**. React no identifica cada Hook por su nombre, sino por el **orden** en que se llaman durante el renderizado. En el primer renderizado, React registra que la primera llamada a un Hook corresponde a un estado, la segunda a un efecto, y así sucesivamente. En cada renderizado posterior, React espera encontrar exactamente esa misma secuencia. Si un Hook se salta condicionalmente en algún renderizado, el orden se desalinea y React termina asociando el estado equivocado a la llamada equivocada.

Veamos un ejemplo incorrecto:

```javascript
function Profile({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // ❌ Hook dentro de una condición
  }

  // ...
}
```

En este componente, `useState()` solo se ejecuta cuando `isLoggedIn` es `true`. Si el valor de `isLoggedIn` cambia entre renderizados, la cantidad y el orden de los Hooks llamados cambia con él, lo que provoca errores como “Se renderizaron menos Hooks de los esperados”.

La forma correcta es llamar siempre al Hook en el nivel superior, y mover la lógica condicional **dentro** de la función que le pasamos:

```javascript
function Profile({ isLoggedIn }) {
  const [user, setUser] = useState(null); // ✅ Siempre en el nivel superior

  useEffect(() => {
    if (isLoggedIn) {
      // la condición vive dentro del efecto, no alrededor del Hook
      fetchUser().then(setUser);
    }
  }, [isLoggedIn]);

  // ...
}
```

### Los Hooks son funciones, no componentes

Por último, vale la pena aclarar una confusión común: un Hook **no es un componente**. Un componente de React es una función que recibe props y devuelve JSX para ser renderizado. Un Hook, en cambio, es una función común de JavaScript que llamamos **desde dentro** de un componente para conectarnos a capacidades internas de React, como el estado o los efectos secundarios. `useState()` no renderiza nada por sí mismo; simplemente le da a la función que define nuestro componente acceso a un valor que React recuerda entre renderizados.

Esta distinción se vuelve especialmente relevante cuando empecemos a construir nuestros propios **Hooks personalizados**: seguirán siendo funciones que empiezan con `use`, sujetas a las mismas dos reglas que acabamos de repasar, y solo podrán llamarse desde componentes de función o desde otros Hooks personalizados.

-----

## ¿Cuándo usar cada práctica de esta lección?

- **Actualizar con un valor directo** (`setCount(5)`) — cuando el siguiente valor no depende del valor anterior del estado.
- **Actualizar con función callback** (`setCount(prev => prev + 1)`) — cuando el siguiente valor **sí** depende del valor anterior. Es la opción más segura por defecto para incrementos, toggles, o cualquier "cambiá esto en base a lo que ya tenía", porque evita el problema de trabajar con un valor de estado desactualizado si React agrupa varias actualizaciones.
- **Arreglos en el estado** — cuando manejás una colección de elementos que se agregan, quitan o filtran como grupo (una lista de tareas, los ingredientes seleccionados de una pizza).
- **Objetos en el estado** — cuando varias variables están relacionadas entre sí y tiende a tener sentido leerlas o pasarlas juntas (los campos de un formulario).
- **Múltiples `useState` separados, en vez de un objeto grande** — cuando esas variables, aunque estén relacionadas conceptualmente, **cambian de forma independiente** entre sí. Es la señal contraria a la anterior: si actualizar un solo campo te obliga a hacer spread de un objeto grande con `...prev`, probablemente convenga separarlo en hooks individuales.

-----