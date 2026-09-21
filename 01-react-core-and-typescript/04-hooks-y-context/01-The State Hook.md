# useState: guardar datos que cambian

## En una frase

`useState` le da **memoria** a tu componente: guarda un dato y, cuando lo cambias, React vuelve a dibujar la pantalla con el dato nuevo.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un componente de función y cómo devuelve JSX: [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
* Cómo un componente recibe datos desde afuera con props: [Props](../02-componentes-y-props/03-Props.md).

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Estado (state):** un dato que el componente recuerda y que, al cambiar, cambia lo que se ve en pantalla.
* **Hook:** una función especial de React, con un nombre que empieza con `use`, que te deja usar herramientas de React (como la memoria) dentro de un componente.
* **Renderizar:** que React ejecute tu componente y dibuje en pantalla lo que devuelve. "Volver a renderizar" es ejecutarlo de nuevo, con datos nuevos.
* **Setter:** la función que usas para cambiar el estado.

-----

## El problema

Imagina que quieres un botón que cuente cuántas veces lo tocaste. Lo primero que se te ocurre es una variable común:

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

Esto **no funciona**, por dos motivos:

1. React no se entera de que `count` cambió, así que no vuelve a dibujar la pantalla. El botón sigue mostrando `0`.
2. Aunque React redibujara, tu componente es una función y se ejecuta **de cero** cada vez. La línea `let count = 0` volvería a poner el contador en `0`.

Necesitas un lugar donde guardar el dato que **sobreviva** entre renders y que además **avise a React** cuando cambia. Ese lugar es `useState`.

-----

## Cómo funciona

### Paso 1: importarlo

```jsx
import { useState } from 'react';
```

### Paso 2: llamarlo y recibir dos cosas

```jsx
const [count, setCount] = useState(0);
```

Vamos por partes:

* `useState(0)`: el `0` es el **valor inicial**, con el que arranca el dato.
* `useState` te devuelve un **array de dos posiciones**: primero el **valor actual**, después la **función para cambiarlo** (el setter).
* Los corchetes `[count, setCount]` son una forma corta de sacar esas dos cosas del array y ponerles nombre. Se llama *desestructuración de arrays*. Como se asigna **por posición**, los nombres los eliges tú; la costumbre es `algo` y `setAlgo`.

### Paso 3: mostrar el valor

Usas `count` como cualquier variable, dentro de llaves en el JSX:

```jsx
<p>Clics: {count}</p>
```

### Paso 4: cambiarlo con el setter

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

Cuando haces clic, pasa esto, en orden:

```
1. Clic  ->  setCount(1)
2. React se entera de que el estado cambió
3. React vuelve a ejecutar la función Counter
4. Esta vez useState devuelve 1 (no 0)
5. La pantalla se actualiza y muestra "Clics: 1"
```

La clave está en el paso 4: en cada ejecución del componente, `useState` te devuelve **el valor más reciente**, y el valor inicial (`0`) solo se usa la primera vez.

### El valor inicial

Puede ser de cualquier tipo: un número, un texto, un booleano, un array o un objeto.

```jsx
const [name, setName] = useState('');          // texto
const [isOpen, setIsOpen] = useState(false);   // verdadero o falso
const [tasks, setTasks] = useState([]);        // lista
const [user, setUser] = useState(null);        // "todavía no hay nada"
```

Si no le pasas nada, el valor inicial es `undefined`. Funciona, pero es confuso para quien lee el código. Es mejor ser explícito: si todavía no tienes el dato, pon `null`.

-----

## Cambiar el estado según su valor anterior

Mira este botón, que quiere sumar 2 en cada clic:

```jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
}
```

Uno esperaría que sume 2, pero **suma 1**. En esas dos líneas `count` vale lo mismo (por ejemplo `0`), así que las dos hacen `setCount(0 + 1)`.

La solución es pasarle al setter **una función**, en lugar de un valor:

```jsx
function handleClick() {
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
}
```

Ahora sí suma 2. React llama a la función y le entrega como `prev` el valor **más fresco**, sin importar cuántas actualizaciones haya en fila.

La regla para acordarte:

* Si el valor nuevo **depende del anterior** (contar, alternar un verdadero/falso, agregar a una lista): pasa una **función** (`setCount((prev) => prev + 1)`).
* Si el valor nuevo **no depende del anterior** (guardar lo que escribió el usuario): pasa el valor directo (`setName('Ana')`).

-----

## Arrays y objetos: nunca los modifiques, haz una copia

Este es el error más común al empezar, así que vale la pena entender el motivo.

React decide si tiene que volver a dibujar preguntándose: *"¿el valor nuevo es un objeto distinto del anterior?"*. Si le pasas el **mismo** array, aunque le hayas agregado cosas por dentro, para React no cambió nada y no redibuja.

```jsx
const [tasks, setTasks] = useState(['Estudiar']);

// Mal: modifica el array que ya existe, y React no se entera
tasks.push('Practicar');
setTasks(tasks);

// Bien: crea un array nuevo
setTasks((prev) => [...prev, 'Practicar']);
```

Los tres puntos `...` se llaman *spread* y significan "copia todo lo que había". Entonces `[...prev, 'Practicar']` se lee: "un array nuevo con todo lo que ya había, más 'Practicar' al final".

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

Fíjate en los paréntesis alrededor de las llaves: `({ ... })`. Le dicen a JavaScript que esas llaves son **un objeto** que estás devolviendo, y no el cuerpo de la función.

-----

## ¿Un solo objeto grande o varios useState?

Puedes guardar datos relacionados en un objeto, pero no siempre conviene. Una regla práctica:

* Si los datos **cambian juntos** (por ejemplo, las coordenadas `x` e `y` de un punto): un objeto.
* Si los datos **cambian por separado** (el nombre de un curso, la lista de alumnos, la nota de un examen): varios `useState`.

```jsx
// Cada dato cambia por su cuenta: mejor separados
const [grade, setGrade] = useState('B');
const [classmates, setClassmates] = useState(['Hasan', 'Sam']);
const [exams, setExams] = useState([{ unit: 1, score: 91 }]);
```

Con un solo objeto grande, cada cambio te obliga a copiar todo lo demás con el spread, y es fácil olvidarte de algo. Con varios `useState`, cada uno se cambia sin tocar a los otros.

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

Qué pasa, paso a paso:

1. Hay **dos estados**: `text` (lo que se está escribiendo) y `tasks` (la lista). Cambian por separado, así que van en dos `useState`.
2. Cada vez que escribes una letra, `onChange` llama a `setText` con el texto nuevo. React redibuja y el campo muestra lo escrito.
3. Al tocar "Agregar", `handleAdd` crea un array **nuevo** con la tarea al final (`[...prev, text]`) y vacía el campo.
4. React redibuja y la lista muestra la tarea nueva.

> El `key={index}` alcanza para una lista que solo crece. Si la lista se puede reordenar o se pueden borrar elementos del medio, conviene darle a cada tarea un `id` propio y usarlo como `key`.

-----

## Errores comunes

### 1. Cambiar el estado directamente

```jsx
count = count + 1;     // no hace nada visible
tasks.push('Nueva');   // React no se entera
```

**Por qué pasa:** React solo se entera de un cambio cuando llamas al setter.
**Cómo se arregla:** usa siempre el setter, y con arrays y objetos, pásale una copia nueva.

### 2. Llamar al setter en lugar de pasarlo

```jsx
<button onClick={setCount(1)}>Reiniciar</button>   // error
```

React muestra un error que dice `Too many re-renders`. **Por qué pasa:** `setCount(1)` con paréntesis **se ejecuta al dibujar**, no al hacer clic; eso cambia el estado, React vuelve a dibujar, se ejecuta otra vez... y así sin fin.
**Cómo se arregla:** pásale una función que se ejecute recién en el clic.

```jsx
<button onClick={() => setCount(1)}>Reiniciar</button>
```

### 3. Leer el estado justo después de cambiarlo

```jsx
setCount(5);
console.log(count);   // muestra el valor VIEJO
```

**Por qué pasa:** el nuevo valor no está disponible en esa misma ejecución. Recién aparece en el **próximo** renderizado, cuando React vuelve a ejecutar el componente.
**Cómo se arregla:** si necesitas el valor nuevo, guardalo en una variable antes (`const next = 5; setCount(next);`) y usa esa variable.

### 4. Usar `useState` dentro de un `if` o un bucle

**Por qué pasa:** los Hooks tienen reglas de dónde se pueden llamar (mira "Las reglas de los Hooks", abajo).
**Cómo se arregla:** llamalos siempre al principio de tu componente, y pon la condición **adentro** de lo que hagas con el valor.

-----

## En TypeScript

TypeScript deduce el tipo del estado a partir del valor inicial. El problema aparece cuando ese valor no le da suficiente información:

```tsx
const [name, setName] = useState<string>();          // sin valor inicial: el tipo es string | undefined
const [tasks, setTasks] = useState<string[]>([]);    // lista vacía: hay que decir de qué es la lista

type FormState = { firstName: string; password: string };
const [form, setForm] = useState<FormState>({ firstName: '', password: '' });   // objeto: define antes su forma
```

Por qué cada uno:

* **Sin valor inicial:** TypeScript no sabe qué vas a guardar. Con `<string>()` se lo dices.
* **Lista vacía:** sin `<string[]>`, TypeScript entiende que es una lista que **no puede contener nada** (`never[]`), y te marca error apenas intentas agregar algo.
* **Objeto:** empezar con `{}` vacío te impide después leer `form.firstName`. Definir el `type` primero y usarlo evita el problema.

-----

## Cuándo sí y cuándo no

**Usa `useState` para** datos que cambian y que afectan lo que se ve: un contador, el texto de un campo, una lista, si un menú está abierto o cerrado.

**No lo uses para:**

* **Datos que se pueden calcular a partir de otro estado o de props.** Si tienes `price` y `quantity`, el total (`price * quantity`) se calcula en cada renderizado; no hace falta guardarlo aparte. Guardar de más lleva a datos que se desincronizan.
* **Datos que no cambian lo que se ve.** Si no afecta la pantalla, una variable común alcanza.
* **Lógica de actualización muy complicada**, con muchas acciones distintas sobre el mismo estado. Para eso existe [useReducer](05-useReducer.md).

-----

## Resumen en 5 líneas

1. `useState(valorInicial)` te devuelve `[valor, setter]`: el dato actual y la función para cambiarlo.
2. Cambiar el estado con el setter hace que React **vuelva a ejecutar** tu componente con el valor nuevo.
3. Si el valor nuevo depende del anterior, pasa una **función** al setter: `setCount((prev) => prev + 1)`.
4. Con arrays y objetos, **nunca los modifiques**: crea una copia nueva con el spread (`...`).
5. Datos que cambian juntos van en un objeto; datos que cambian por separado, en varios `useState`.

-----

## Para profundizar

<details>
<summary>Las reglas de los Hooks</summary>

Los Hooks (`useState` y todos los que vienen) tienen dos reglas. No son un estilo opcional: si las rompes, React se confunde y tu app se comporta raro.

**Regla 1: solo en componentes de función (o en otros Hooks propios).** No funcionan en componentes de clase ni en funciones comunes de JavaScript.

**Regla 2: siempre en el nivel de arriba del componente.** Nunca dentro de un `if`, un bucle (`for`, `while`) ni una función anidada.

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

**Por qué existe la regla 2:** React no identifica cada Hook por su nombre, sino por el **orden** en que se llaman. En el primer renderizado registra "el primer Hook es un estado, el segundo es un efecto...", y en los siguientes espera exactamente la misma secuencia. Si un `if` hace que un Hook a veces se salte, el orden se desarma y React le entrega a cada llamada el dato equivocado. El error típico es "Rendered fewer hooks than expected".

</details>

<details>
<summary>Los Hooks son funciones, no componentes</summary>

Un componente es una función que recibe props y devuelve JSX para dibujar. Un Hook, en cambio, es una función común que llamas **desde adentro** de un componente para usar una herramienta de React. `useState` no dibuja nada: solo le da a tu componente acceso a un valor que React recuerda. Más adelante vas a crear tus propios Hooks (los vemos en la lección de Custom Hooks), y siguen siendo funciones cuyo nombre empieza con `use`.

</details>

<details>
<summary>Valor inicial calculado (inicialización perezosa)</summary>

Si el valor inicial es costoso de calcular, pásale a `useState` una **función** en lugar del valor:

```jsx
const [data, setData] = useState(() => calcularAlgoCostoso());
```

React ejecuta esa función **solo la primera vez**. Si escribieras `useState(calcularAlgoCostoso())`, el cálculo se repetiría en cada renderizado, aunque el resultado se ignore después del primero. Lo vas a ver, por ejemplo, para leer un valor de `localStorage` al arrancar.

</details>

<details>
<summary>Por qué el estado no cambia "al instante"</summary>

Cuando llamas a un setter, React no cambia el valor en ese mismo momento: **junta** los cambios pedidos dentro del mismo evento y actualiza todo junto al final, con un solo redibujo. Es una optimización (menos trabajo), y es la razón por la que `console.log(count)` justo después de `setCount(5)` todavía muestra el valor viejo, y por la que conviene usar la forma con función (`(prev) => ...`) cuando el cambio depende del valor anterior.

</details>

-----

## Siguiente lección

Ahora que tu componente puede recordar datos, el paso que sigue es hacer que **reaccione** a cambios y se conecte con cosas de afuera (una API, un temporizador, el título de la página): [useEffect](02-The%20Effect%20Hook.md).
