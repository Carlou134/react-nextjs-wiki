# useState: guardar datos que cambian

## En una frase

`useState` le da **memoria** a un componente: React conserva un valor entre renders y, cuando lo actualizas con el setter, programa un nuevo render con el valor nuevo.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un componente de función y cómo devuelve JSX: [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
* Cómo un componente recibe datos desde afuera con props: [Props](../02-componentes-y-props/03-Props.md).

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Estado (state):** dato que React conserva entre renders y cuyo cambio provoca un nuevo render.
* **Hook:** función de React, con nombre que empieza por `use`, que se llama dentro de un componente de función (o de otro Hook) para acceder a capacidades de React, como el estado.
* **Renderizar:** que React ejecute el componente para calcular qué UI debe mostrar. Un re-render es una nueva ejecución con datos actualizados.
* **Setter:** la función que devuelve `useState` para actualizar el estado.

-----

## El problema

Un contador implementado con una variable local no funciona:

```jsx
function Counter() {
  let count = 0;

  return (
    <button onClick={() => { count = count + 1; }}>
      Clics: {count}
    </button>
  );
}
```

Hay dos motivos:

1. Mutar una variable local no notifica a React, así que no hay nuevo render. El botón sigue mostrando `0`.
2. Aunque hubiera un nuevo render, el componente es una función que se ejecuta **de cero** en cada render. `let count = 0` reiniciaría el valor.

Se necesita un almacenamiento que **persista** entre renders y que **notifique a React** cuando cambia. Eso resuelve `useState`.

-----

## Cómo funciona

### Importación y firma

```jsx
import { useState } from 'react';

const [count, setCount] = useState(0);
```

* `useState(0)`: el argumento es el **valor inicial**. Solo se usa en el primer render.
* Devuelve un **array de dos posiciones**: el **valor actual** y el **setter**.
* `[count, setCount]` es *desestructuración de arrays*. Se asigna **por posición**, así que los nombres son libres; la convención es `algo` y `setAlgo`.

El valor se usa como cualquier variable, entre llaves en el JSX (`<p>Clics: {count}</p>`), y se actualiza con el setter:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clics: {count}
    </button>
  );
}
```

Secuencia al hacer clic:

```
1. Clic  ->  setCount(1)
2. React encola la actualización y programa un render
3. React vuelve a ejecutar la función Counter
4. useState devuelve 1 (no 0)
5. React actualiza el DOM y muestra "Clics: 1"
```

En cada render, `useState` devuelve el valor más reciente del estado. El valor inicial solo se usa en el primer render.

### El valor inicial

Puede ser de cualquier tipo: número, texto, booleano, array u objeto.

```jsx
const [name, setName] = useState('');          // texto
const [isOpen, setIsOpen] = useState(false);   // verdadero o falso
const [tasks, setTasks] = useState([]);        // lista
const [user, setUser] = useState(null);        // "todavía no hay nada"
```

Sin argumento, el valor inicial es `undefined`. Es válido, pero ambiguo al leerlo. Si el dato aún no existe, es más claro usar `null`.

-----

## Cambiar el estado según su valor anterior

Este handler pretende sumar 2, pero **suma 1**:

```jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
}
```

El estado es una *instantánea* por render: dentro del handler, `count` tiene un valor fijo (por ejemplo `0`). Ambas líneas ejecutan `setCount(0 + 1)`.

La solución es pasar al setter una **función actualizadora** (*updater*) en lugar de un valor:

```jsx
function handleClick() {
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
}
```

Ahora suma 2. React encola los updaters y los aplica en orden en el siguiente render: cada uno recibe como `prev` el resultado del anterior. Los updaters deben ser funciones puras.

Regla:

* Si el valor nuevo **depende del anterior** (contador, alternar un booleano, agregar a una lista): usa un updater (`setCount((prev) => prev + 1)`).
* Si **no depende** del anterior (guardar lo que escribió el usuario): pasa el valor directo (`setName('Ana')`).

-----

## Arrays y objetos: nunca los modifiques, haz una copia

Es uno de los errores más frecuentes.

React compara el valor nuevo con el anterior usando `Object.is`, es decir, **por referencia** en objetos y arrays. Si pasas la **misma** referencia, aunque hayas mutado su contenido, React considera que no hubo cambio y omite el re-render.

```jsx
const [tasks, setTasks] = useState(['Estudiar']);

// Mal: modifica el array que ya existe, y React no se entera
tasks.push('Practicar');
setTasks(tasks);

// Bien: crea un array nuevo
setTasks((prev) => [...prev, 'Practicar']);
```

El operador *spread* (`...`) copia los elementos de un array (o las propiedades de un objeto) en uno nuevo. `[...prev, 'Practicar']` crea un array nuevo con los elementos previos más `'Practicar'`. La copia es **superficial**: los objetos anidados siguen compartiendo referencia.

Otras operaciones comunes con arrays, siempre devolviendo uno nuevo:

```jsx
// Quitar un elemento
setTasks((prev) => prev.filter((task) => task !== 'Estudiar'));

// Cambiar un elemento
setTasks((prev) => prev.map((task) => (task === 'Estudiar' ? 'Estudiar React' : task)));
```

Con los objetos pasa lo mismo:

```jsx
const [user, setUser] = useState({ name: 'Ana', age: 30 });

// Copia todo lo del usuario y cambia solo la edad
setUser((prev) => ({ ...prev, age: 31 }));
```

Los paréntesis en `({ ... })` indican que las llaves son un literal de objeto devuelto, no el cuerpo de la función.

-----

## ¿Un solo objeto grande o varios useState?

Criterio práctico:

* Datos que **cambian juntos** (coordenadas `x` e `y`): un objeto.
* Datos que **cambian por separado** (nombre de un curso, lista de alumnos, nota de un examen): varios `useState`.

```jsx
// Cada dato cambia por su cuenta: mejor separados
const [grade, setGrade] = useState('B');
const [classmates, setClassmates] = useState(['Hasan', 'Sam']);
const [exams, setExams] = useState([{ unit: 1, score: 91 }]);
```

Con un objeto grande, cada actualización exige copiar el resto con spread, y omitirlo pierde datos: el setter **reemplaza** el estado, no lo fusiona (a diferencia de `setState` en clases). Con varios `useState`, cada valor se actualiza de forma independiente.

-----

## Ejemplo completo: una lista de tareas

```jsx
import { useState } from 'react';

export default function TodoList() {
  const [text, setText] = useState('');
  const [tasks, setTasks] = useState([]);

  function handleAdd() {
    if (text.trim() === '') return;          // no agregar tareas vacías
    setTasks((prev) => [...prev, text]);     // array nuevo con la tarea al final
    setText('');                             // vaciar el campo
  }

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAdd}>Agregar</button>

      <ul>
        {tasks.map((task, index) => (
          <li key={index}>{task}</li>
        ))}
      </ul>
    </div>
  );
}
```

Puntos clave:

1. Hay **dos estados** independientes: `text` (el campo) y `tasks` (la lista).
2. El input es *controlado*: su `value` viene del estado y `onChange` lo actualiza con `setText`.
3. `handleAdd` crea un array **nuevo** (`[...prev, text]`) y vacía el campo. Ambas actualizaciones se procesan en un solo re-render.

> El `key={index}` alcanza para una lista que solo crece. Si la lista se puede reordenar o se pueden borrar elementos del medio, conviene darle a cada tarea un `id` propio y usarlo como `key`.

-----

## Errores comunes

### 1. Cambiar el estado directamente

```jsx
count = count + 1;     // no hace nada visible
tasks.push('Nueva');   // React no se entera
```

**Por qué pasa:** React solo detecta cambios cuando se llama al setter con un valor distinto (`Object.is`).
**Solución:** usa siempre el setter y, con arrays y objetos, pásale una copia nueva.

### 2. Llamar al setter en lugar de pasarlo

```jsx
<button onClick={setCount(1)}>Reiniciar</button>   // error
```

React lanza `Too many re-renders`. **Por qué pasa:** `setCount(1)` con paréntesis **se ejecuta durante el render**, no en el clic; actualiza el estado, provoca otro render, y se repite en bucle.
**Solución:** pasa una función que se ejecute en el clic.

```jsx
<button onClick={() => setCount(1)}>Reiniciar</button>
```

### 3. Leer el estado justo después de cambiarlo

```jsx
setCount(5);
console.log(count);   // muestra el valor VIEJO
```

**Por qué pasa:** el estado es una instantánea del render actual. `count` no cambia dentro de esa ejecución; el valor nuevo llega en el **siguiente** render.
**Solución:** si necesitas el valor nuevo en ese mismo handler, calcúlalo en una variable (`const next = 5; setCount(next);`) y usa `next`. Para reaccionar a un cambio de estado, usa el valor en el render o en un efecto.

### 4. Usar `useState` dentro de un `if` o un bucle

**Por qué pasa:** React identifica los Hooks por su orden de llamada (ver "Las reglas de los Hooks", abajo).
**Solución:** llámalos siempre en el nivel superior del componente y pon la condición **dentro** de lo que hagas con el valor.

-----

## En TypeScript

TypeScript infiere el tipo del estado a partir del valor inicial. Hace falta anotarlo cuando ese valor no aporta suficiente información:

```tsx
const [name, setName] = useState<string>();          // sin valor inicial: el tipo es string | undefined
const [tasks, setTasks] = useState<string[]>([]);    // lista vacía: hay que decir de qué es la lista

type FormState = { firstName: string; password: string };
const [form, setForm] = useState<FormState>({ firstName: '', password: '' });   // objeto: define antes su forma
```

Motivos:

* **Sin valor inicial:** el tipo inferido sería `undefined`; con `<string>()` se declara `string | undefined`.
* **Lista vacía:** con `strict`, `useState([])` infiere `never[]` y rechaza cualquier elemento. Se anota `<string[]>`.
* **Objeto:** iniciar con `{}` infiere un tipo sin propiedades y `form.firstName` falla. Define el `type` y pásalo como genérico.

-----

## Cuándo sí y cuándo no

**Usa `useState` para** datos que cambian con el tiempo y afectan lo que se renderiza: un contador, el texto de un campo, una lista, un menú abierto o cerrado.

**No lo uses para:**

* **Datos derivables de otro estado o de props.** Con `price` y `quantity`, el total (`price * quantity`) se calcula durante el render. Duplicarlo en estado genera inconsistencias.
* **Datos que no afectan el render** (por ejemplo, un id de temporizador). Para eso sirve `useRef`, que no dispara re-renders.
* **Lógica de actualización muy complicada**, con muchas acciones distintas sobre el mismo estado. Para eso existe [useReducer](05-useReducer.md).

-----

## Resumen en 5 líneas

1. `useState(valorInicial)` devuelve `[valor, setter]`: el valor actual y la función para actualizarlo.
2. Actualizar el estado con el setter programa un **nuevo render** del componente con el valor nuevo.
3. Si el valor nuevo depende del anterior, usa un **updater**: `setCount((prev) => prev + 1)`.
4. Con arrays y objetos, **no los mutes**: crea una copia nueva con spread (`...`).
5. Datos que cambian juntos van en un objeto; datos que cambian por separado, en varios `useState`.

-----

## Para profundizar

<details>
<summary>Las reglas de los Hooks</summary>

Los Hooks tienen dos reglas. No son de estilo: si se rompen, React asocia mal los valores y la app falla.

**Regla 1: solo en componentes de función o en Hooks propios.** No funcionan en componentes de clase ni en funciones comunes.

**Regla 2: solo en el nivel superior.** Nunca dentro de condicionales, bucles ni funciones anidadas.

```jsx
function Profile({ isLoggedIn }) {
  // Mal: el Hook solo se ejecuta a veces
  if (isLoggedIn) {
    const [user, setUser] = useState(null);
  }
}

function Profile({ isLoggedIn }) {
  // Bien: el Hook siempre se ejecuta; la condición va adentro de lo que hagas con él
  const [user, setUser] = useState(null);

  if (!isLoggedIn) return <p>Inicia sesión</p>;
  return <p>Hola, {user?.name}</p>;
}
```

**Por qué existe la regla 2:** React no identifica cada Hook por nombre, sino por el **orden** de las llamadas. En cada render espera la misma secuencia que en el anterior. Si un Hook se omite condicionalmente, el orden se desalinea y cada llamada recibiría el valor de otra. Un error típico es "Rendered fewer hooks than expected".

</details>

<details>
<summary>Los Hooks son funciones, no componentes</summary>

Un componente es una función que recibe props y devuelve JSX. Un Hook es una función que se llama **desde dentro** de un componente (u otro Hook) para acceder a una capacidad de React. `useState` no devuelve UI: da acceso a un valor que React conserva. Los Hooks propios (lección de Custom Hooks) también son funciones cuyo nombre empieza por `use`.

</details>

<details>
<summary>Valor inicial calculado (inicialización perezosa)</summary>

Si el valor inicial es costoso de calcular, pasa a `useState` una **función inicializadora** en lugar del valor:

```jsx
const [data, setData] = useState(() => calcularAlgoCostoso());
```

React la ejecuta **solo en el primer render**. Con `useState(calcularAlgoCostoso())`, la función se invoca en cada render, aunque el resultado se descarte después del primero. Es común al leer `localStorage` al montar. En desarrollo con `StrictMode`, React la ejecuta dos veces para detectar impurezas, por lo que debe ser pura.

</details>

<details>
<summary>Por qué el estado no cambia "al instante"</summary>

Un setter no modifica la variable `count` del render actual: **encola** la actualización. React agrupa (*batching*) las actualizaciones del mismo evento y procesa todas en un solo re-render. Desde React 18 esto ocurre también en promesas, `setTimeout` y handlers nativos, no solo en eventos de React. Por eso `console.log(count)` tras `setCount(5)` muestra el valor viejo, y por eso conviene el updater (`(prev) => ...`) cuando el cambio depende del valor anterior.

</details>

-----

## En entrevista

### Respuesta corta (junior)

`useState` es un Hook de React que permite a un componente de función conservar un valor entre renders. Devuelve el valor actual y un setter. Al llamar al setter, React vuelve a renderizar el componente con el valor nuevo. Sirve para datos que cambian con el tiempo y afectan la UI, como el texto de un input o un contador.

### Respuesta ampliada (semi-senior)

* **Instantánea por render:** el estado es constante dentro de un render. El setter no muta la variable; encola una actualización que se aplica en el siguiente render.
* **Batching:** React agrupa varias actualizaciones en un solo re-render. Desde React 18 es automático también en promesas, timeouts y handlers nativos.
* **Updater:** `setX(prev => ...)` recibe el resultado de la actualización anterior en la cola. Es la forma correcta cuando el valor nuevo depende del anterior, y evita leer valores obsoletos (*stale closures*).
* **Comparación:** React usa `Object.is`. Con la misma referencia omite el render de los hijos y los efectos; mutar un objeto o array y volver a pasarlo no dispara la actualización. Se debe crear una copia.
* **Reemplazo, no fusión:** el setter sustituye el estado completo; con objetos hay que copiar el resto con spread. La copia es superficial.
* **Inicialización perezosa:** `useState(() => fn())` ejecuta `fn` solo en el primer render.
* **Trade-offs:** no guardes en estado lo derivable de props u otro estado; usa `useRef` para valores mutables que no afectan el render; usa `useReducer` cuando la lógica de actualización crece.
* **Errores típicos:** mutar estado, leer el valor "nuevo" justo tras el setter, llamar al setter durante el render (`onClick={setCount(1)}`) y usar Hooks condicionalmente.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué `setCount(count + 1)` dos veces seguidas suma solo 1?**
Porque `count` es la misma instantánea en ambas líneas: las dos ejecutan `setCount(0 + 1)`. Con `setCount(prev => prev + 1)` cada updater recibe el resultado del anterior y suma 2.

**2. ¿Por qué no se puede hacer `tasks.push(x); setTasks(tasks)`?**
La referencia del array no cambia, y React compara con `Object.is`, así que ve el mismo valor y omite el re-render. Además, mutar el estado rompe la inmutabilidad en la que se apoyan optimizaciones como `React.memo`. Se usa `setTasks(prev => [...prev, x])`.

**3. ¿El setter es síncrono o asíncrono?**
Ni una cosa ni la otra en sentido estricto: no devuelve promesa ni bloquea. Encola la actualización y el nuevo valor solo se ve en el siguiente render; por eso un `console.log(count)` posterior muestra el valor previo.

**4. ¿Qué pasa si llamas al setter con el mismo valor?**
React lo detecta con `Object.is` y evita renderizar los hijos y ejecutar efectos. En algunos casos puede ejecutar el componente una vez más antes de descartar la actualización, así que el render en sí debe ser puro.

**5. ¿Cuándo usar la inicialización perezosa?**
Cuando el valor inicial es costoso (por ejemplo, leer y parsear `localStorage`). `useState(() => leer())` lo calcula una sola vez; `useState(leer())` lo ejecuta en cada render, aunque el resultado se ignore.

**6. ¿Cómo se reinicia el estado de un componente?**
Cambiando su `key`: React desmonta la instancia anterior y monta una nueva con estado inicial.

```jsx
<Form key={userId} />
```

-----

## Siguiente lección

Ahora que tu componente puede recordar datos, el paso que sigue es hacer que **reaccione** a cambios y se conecte con cosas de afuera (una API, un temporizador, el título de la página): [useEffect](02-The%20Effect%20Hook.md).
