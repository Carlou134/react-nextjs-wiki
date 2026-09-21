# useReducer: ordenar los cambios de estado complejos

## En una frase

`useReducer` centraliza **toda la lógica de cambio de un estado en una sola función** (el *reducer*). Los componentes no modifican el estado: describen lo que ocurrió con `dispatch`.

-----

## Antes de empezar

Requisitos:

* `useState` y la inmutabilidad (copiar en vez de mutar arrays y objetos): [useState](01-The%20State%20Hook.md).
* Context, solo para la sección "Combinarlo con Context": [React Context](04-React%20Context.md).

Términos (también en el [glosario](Glosario.md)):

* **Reducer:** función `(state, action) => newState`. Define cómo cambia el estado.
* **Acción (action):** objeto que describe lo que ocurrió. Por ejemplo `{ type: 'increment' }`.
* **Dispatch:** función que envía una acción al reducer.
* **Payload:** dato adicional dentro de la acción, cuando el reducer lo necesita (por ejemplo, el producto a agregar).
* **Función pura:** con los mismos argumentos devuelve siempre el mismo resultado y no produce efectos secundarios (no llama a APIs ni modifica variables externas).
* **Unión discriminada (TypeScript):** unión de tipos que comparten un campo literal (por ejemplo `type`) que identifica cada variante.

-----

## El problema

Con `useState`, cada operación suele tener su propio manejador. Si un mismo estado admite muchas operaciones, la lógica se dispersa en el componente:

```jsx
const [items, setItems] = useState([]);

function addItem(item) {
  setItems((prev) => [...prev, item]);
}

function removeItem(id) {
  setItems((prev) => prev.filter((item) => item.id !== id));
}

function clearItems() {
  setItems([]);
}
```

Con tres operaciones es manejable. Con diez, o con un estado de varios campos que deben cambiar juntos, cada manejador repite su copia con spread (`...prev`) y crece el riesgo de olvidar un campo o dejar datos inconsistentes.

`useReducer` concentra la lógica de cambio en **un solo lugar**. El resto del componente solo indica *qué ocurrió*.

-----

## Cómo funciona

### Las tres piezas

`useReducer` trabaja con tres piezas:

* **state:** el valor actual del estado, igual que en `useState`.
* **dispatch(acción):** solicita un cambio. El componente no modifica el estado: solo informa qué ocurrió.
* **reducer(state, action):** recibe el estado actual y la acción, y calcula el estado nuevo.

El recorrido siempre es el mismo:

```
1. Algo pasa (un clic)   ->  dispatch({ type: 'increment' })
2. React llama al reducer ->  reducer(estadoActual, { type: 'increment' })
3. El reducer devuelve el estado nuevo
4. React guarda ese estado y vuelve a renderizar el componente
```

### Firma

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducer, initialState);
```

La línea tiene dos pares de nombres distintos:

* Argumentos: la función `reducer` y el estado inicial `initialState`.
* Valor devuelto: un array de dos posiciones, `[state, dispatch]`.

### Desestructuración por posición

Los corchetes `[state, dispatch]` son *desestructuración de arrays*: asignan por **posición** (primero el estado, luego `dispatch`). Los nombres son libres:

```jsx
const [contador, enviarAccion] = useReducer(reducer, initialState);
// contador es la posición 0 (el estado)
// enviarAccion es la posición 1 (dispatch), aunque lo hayas llamado distinto
```

La desestructuración de **objetos** (con llaves) asigna **por nombre**: el orden no importa, pero el nombre debe coincidir con la propiedad.

```jsx
const { state, dispatch } = algunObjeto;   // busca las propiedades "state" y "dispatch"
```

### El reducer

El reducer es una **función pura** `(state, action) => newState`. Suele usar un `switch` sobre `action.type`:

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}
```

Reglas del reducer:

* **No muta el estado recibido.** Devuelve un objeto **nuevo** (igual que con `useState`).
* **Siempre devuelve un estado.** Cada `case` termina en `return`.
* **El `default` devuelve el estado sin cambios**, para que una acción desconocida no produzca `undefined`.
* **Es pura:** mismo estado y misma acción, mismo resultado. Sin llamadas a APIs, sin `Date.now()` ni `Math.random()` dentro.

### dispatch y payload

Para cambiar el estado se llama a `dispatch` con un objeto de acción. Por convención incluye un `type` que identifica lo ocurrido:

```jsx
dispatch({ type: 'increment' });
```

`dispatch` no cambia el estado de inmediato: encola la acción, React ejecuta `reducer(estadoActual, acción)` y renderiza con el resultado. En el mismo manejador, `state` conserva el valor del render actual.

Si el reducer necesita un dato adicional, este viaja en una propiedad que por convención se llama `payload`:

```jsx
dispatch({ type: 'setCount', payload: 10 });
```

```jsx
case 'setCount':
  return { count: action.payload };
```

El `payload` viaja **desde quien llama a `dispatch` hacia el reducer**. Nunca sale de él: el reducer lo recibe como parte del segundo parámetro `action`.

```
Quien llama a dispatch  --- { type, payload } --->  reducer
                                                       |
                                                       v
                                              devuelve el estado nuevo
```

### ¿Cuándo hace falta payload?

Hace falta cuando el reducer necesita un dato que **no puede deducir del `state`**: qué producto agregar, qué `id` quitar, qué valor guardar.

No hace falta cuando el `type` basta:

```jsx
dispatch({ type: 'clear' });                       // vaciar: no requiere datos extra
dispatch({ type: 'add', payload: item });          // agregar: el reducer no sabe cuál item
```

El payload no sirve solo para cálculos: es cualquier dato externo que la acción necesite.

-----

## Ejemplo completo 1: un contador

```jsx
import { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Contador: {state.count}</p>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reiniciar</button>
    </div>
  );
}
```

Flujo:

1. `useReducer(reducer, initialState)` arranca con `{ count: 0 }`.
2. El clic en "+" ejecuta `dispatch({ type: 'increment' })`.
3. React llama a `reducer({ count: 0 }, { type: 'increment' })`, que devuelve `{ count: 1 }`.
4. React guarda ese estado y vuelve a renderizar `Counter`, que muestra `Contador: 1`.

El JSX no sabe **cómo** se incrementa: solo emite la acción. Esa lógica vive únicamente en el reducer.

-----

## Ejemplo completo 2: un carrito de compras (TypeScript)

Tres acciones; dos llevan payload.

```tsx
import { useReducer } from 'react';

type CartItem = { id: number; name: string; price: number };
type CartState = { items: CartItem[] };

type Action =
  | { type: 'add'; payload: CartItem }
  | { type: 'remove'; payload: { id: number } }
  | { type: 'clear' };

const initialState: CartState = { items: [] };

function cartReducer(state: CartState, action: Action): CartState {
  switch (action.type) {
    case 'add':
      return { items: [...state.items, action.payload] };
    case 'remove':
      return { items: state.items.filter((item) => item.id !== action.payload.id) };
    case 'clear':
      return { items: [] };
    default:
      return state;
  }
}

function Cart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  return (
    <div>
      <button onClick={() => dispatch({ type: 'add', payload: { id: Date.now(), name: 'Teclado', price: 50 } })}>
        Agregar teclado
      </button>
      <button onClick={() => dispatch({ type: 'clear' })}>Vaciar</button>

      <ul>
        {state.items.map((item) => (
          <li key={item.id}>
            {item.name} - ${item.price}
            <button onClick={() => dispatch({ type: 'remove', payload: { id: item.id } })}>Quitar</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Puntos clave:

1. `Action` enumera **todas** las acciones posibles. `add` lleva un `CartItem`; `remove` solo el `id`; `clear` no lleva payload.
2. `cartReducer` tiene la firma `(state: CartState, action: Action): CartState`.
3. `add` crea un array **nuevo** con el item al final; `remove` usa `filter`, que también devuelve un array nuevo. Nunca se modifica `state.items`.
4. El componente solo hace `dispatch`; no contiene la lógica de agregar o quitar.

> **Sobre el `id`:** debe ser **único**, porque `remove` elimina todos los items con ese `id` y React lo usa como `key`. `Date.now()` sirve en el ejemplo, ya que se llama en el manejador de eventos y no dentro del reducer, pero puede repetirse si hay dos clics en el mismo milisegundo. En una app real, el `id` viene de la base de datos o de `crypto.randomUUID()`.

-----

## Combinarlo con Context

Si componentes lejanos necesitan el carrito, se coloca el `useReducer` dentro de un Provider. Los consumidores no reciben un setter por operación: les basta `dispatch`, y la lógica sigue en el reducer.

Reutilizamos `CartItem`, `CartState`, `Action`, `initialState` y `cartReducer` del ejemplo anterior.

### 1. El tipo del contexto y el contexto

```tsx
import { createContext, useContext, useMemo, useReducer } from 'react';

type CartContextType = {
  state: CartState;
  dispatch: React.Dispatch<Action>;
};

const CartContext = createContext<CartContextType | undefined>(undefined);
```

`React.Dispatch<Action>` es el tipo de `dispatch`: una función que recibe una `Action`. El valor por defecto es `undefined` (sin Provider), y el `| undefined` del tipo lo refleja.

### 2. El Provider

```tsx
function CartProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  const value = useMemo(() => ({ state, dispatch }), [state]);

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}
```

El Provider llama a `useReducer` y comparte `{ state, dispatch }`. `useMemo` recrea el objeto `value` solo cuando `state` cambia, para que los consumidores no se re-rendericen por un objeto nuevo en cada render del Provider. `dispatch` tiene identidad estable, por eso no va en las dependencias.

### 3. El hook `useCart`

```tsx
function useCart() {
  const context = useContext(CartContext);
  if (context === undefined) {
    throw new Error('useCart debe usarse dentro de un CartProvider');
  }
  return context;
}
```

`useContext` se envuelve en un hook propio. La guardia de `undefined` lanza un error claro si falta el Provider y, además, estrecha el tipo: TypeScript sabe que `context` ya no es `undefined`.

### 4. Uso en un componente

```tsx
function AddButton() {
  const { state, dispatch } = useCart();

  return (
    <button onClick={() => dispatch({ type: 'add', payload: { id: Date.now(), name: 'Mouse', price: 20 } })}>
      Agregar mouse ({state.items.length} en el carrito)
    </button>
  );
}

function App() {
  return (
    <CartProvider>
      <AddButton />
    </CartProvider>
  );
}
```

`AddButton` está **dentro** de `<CartProvider>`; `App`, que crea el Provider, no llama a `useCart()`.

### Optimización: separar state y dispatch

Con un solo contexto, todo consumidor se re-renderiza cuando cambia `state`, aunque solo use `dispatch`. Una alternativa es usar dos contextos: `CartStateContext` para `state` y `CartDispatchContext` para `dispatch`. Como `dispatch` es estable, los componentes que solo despachan no se re-renderizan por cambios del estado.

```tsx
const CartStateContext = createContext<CartState | undefined>(undefined);
const CartDispatchContext = createContext<React.Dispatch<Action> | undefined>(undefined);

function CartProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  return (
    <CartDispatchContext.Provider value={dispatch}>
      <CartStateContext.Provider value={state}>{children}</CartStateContext.Provider>
    </CartDispatchContext.Provider>
  );
}
```

(Cada contexto necesita su propio hook con guardia de `undefined`, como `useCart`.)

-----

## Lógica asíncrona

El reducer es puro: **no hace llamadas a APIs**. La petición se hace en un efecto o en un manejador de eventos, y al terminar se hace `dispatch` con el resultado:

```jsx
useEffect(() => {
  let cancelled = false;

  fetchUser().then((user) => {
    if (!cancelled) {
      dispatch({ type: 'userLoaded', payload: user });
    }
  });

  return () => {
    cancelled = true;
  };
}, []);
```

El efecto pide el dato y, al llegar, envía una acción con el usuario como `payload`; el reducer solo lo guarda. La variable `cancelled` evita despachar resultados obsoletos si el efecto se limpió (desmontaje o cambio de dependencias). Más sobre efectos en [useEffect](02-The%20Effect%20Hook.md).

Si el manejo de datos remotos crece (reintentos, caché, invalidación), conviene mirar React Query o Zustand. Mira [Zustand, Redux y Context: cuándo usar cada uno](../../03-state-management-and-data/01-Zustand,%20Redux%20y%20Context%20-%20Cuando%20usar%20cada%20uno.md).

-----

## Errores comunes

### 1. Mutar el estado dentro del reducer

```jsx
case 'add':
  state.items.push(action.payload);   // Mal: modifica el estado recibido
  return state;
```

**Por qué pasa:** se devuelve el **mismo** objeto. React compara con `Object.is`, no detecta cambio y puede omitir el re-render. Además, StrictMode ejecuta el reducer dos veces en desarrollo, así que `push` agregaría el item duplicado.
**Cómo se arregla:** devuelve siempre un objeto nuevo.

```jsx
case 'add':
  return { items: [...state.items, action.payload] };   // Bien
```

### 2. Olvidar el `default` o el `return`

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    // Mal: sin default, una acción desconocida devuelve undefined
  }
}
```

**Por qué pasa:** si ningún `case` coincide, la función termina sin devolver nada, y tu estado pasa a ser `undefined`. Lo mismo ocurre si un `case` no tiene `return`.
**Cómo se arregla:** cada `case` termina en `return`, y agrega un `default: return state;`.

### 3. Reutilizar nombres en la desestructuración

```jsx
function reducer(state, action) { /* ... */ }   // afuera, a nivel del archivo
const initialState = { count: 0 };

function Counter() {
  // Mal: colisión de nombres, dentro del componente
  const [reducer, initialState] = useReducer(reducer, initialState);
}
```

Da el error `Cannot access 'reducer' before initialization`.

**Por qué pasa:** con `const`, el nombre queda **reservado en todo el bloque** (aquí, el cuerpo de `Counter`), aunque todavía no tenga valor. Entonces ese `const` crea un `reducer` nuevo y vacío dentro de `Counter`, que tapa al de afuera. El `reducer` de la derecha ya no apunta a tu función: apunta a esa variable nueva, que todavía no se inicializó.
**Cómo se arregla:** lo que te devuelve `useReducer` se llama `state` y `dispatch`; deja `reducer` e `initialState` para lo que le pasas.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);   // Bien
```

### 4. Desestructurar `useCart()` con corchetes

```tsx
const [state, dispatch] = useCart();   // Mal
```

**Por qué pasa:** `useCart()` devuelve un **objeto** (`{ state, dispatch }`), no un array. Los corchetes son para arrays, como los que devuelven `useState` y `useReducer`.
**Cómo se arregla:** usa llaves.

```tsx
const { state, dispatch } = useCart();   // Bien
```

### 5. Llamar a `useCart()` fuera del Provider

```tsx
function App() {
  const { state } = useCart();   // Mal: App crea el Provider, no está adentro
  return (
    <CartProvider>
      <p>{state.items.length}</p>
    </CartProvider>
  );
}
```

**Por qué pasa:** el componente que crea el Provider **no puede consumirlo**: un componente solo recibe el valor de un Provider que tenga **por encima**. Aquí `useCart()` corre antes de que exista un Provider y lanza el error de la guardia.
**Cómo se arregla:** separa en dos componentes: uno que renderiza el Provider y otro, hijo, que llama a `useCart()`. Mira el paso 7 de la receta en [React Context](04-React%20Context.md).

### 6. Despachar una acción sin el payload que necesita

```tsx
dispatch({ type: 'add' });   // Mal: falta el payload
```

**Por qué pasa:** el reducer va a intentar guardar `action.payload`, que es `undefined`.
**Cómo se arregla:** envía el payload (`dispatch({ type: 'add', payload: item })`). Si tipaste `Action`, TypeScript te marca el error antes de ejecutar nada.

-----

## En TypeScript

La forma correcta de tipar las acciones es una **unión discriminada**: varias opciones que comparten el campo `type`, que dice cuál es cuál.

```tsx
type Action =
  | { type: 'add'; payload: CartItem }
  | { type: 'remove'; payload: { id: number } }
  | { type: 'clear' };
```

Dentro del `switch`, TypeScript **estrecha** el tipo de `action` en cada `case`: mira el valor de `type` y sabe qué opción quedó.

```tsx
switch (action.type) {
  case 'add':
    action.payload;   // TypeScript sabe que es un CartItem
    break;
  case 'clear':
    action.payload;   // Error: la opción 'clear' no tiene payload
    break;
}
```

Esto atrapa, antes de ejecutar el código, dos errores comunes: escribir mal un `type` (`'ad'` en vez de `'add'`) y despachar una acción sin el payload que necesita.

Para exigir **exhaustividad**, el `default` puede asignar `action` a `never`. Si se agrega una variante a `Action` y no se maneja, TypeScript marca error:

```tsx
default: {
  const _exhaustive: never = action;
  return state;
}
```

Para ver el tipado completo de `useReducer` con Context, mira [Tipado de useReducer y Context API](../11-typescript-y-react/03-Tipado%20de%20useReducer%20y%20Context%20API.md).

-----

## Cuándo sí y cuándo no

`useState` sigue siendo la opción **por defecto**. Pasa a `useReducer` cuando veas alguna de estas señales:

* **Formularios grandes con muchos campos relacionados.** En vez de un `useState` por campo (o manejadores casi idénticos), un reducer con acciones como `updateField`, `resetForm`, `submitStart`, `submitSuccess` o `submitError` deja todo en un lugar.
* **Sub-valores que deben mantenerse coherentes.** Un ejemplo típico es una **máquina de estados**, es decir, una operación que solo puede estar en una de varias situaciones posibles (`idle`, `loading`, `success` o `error`), con los datos o el mensaje de error de cada una. Con varios `useState` sueltos podrías terminar con combinaciones imposibles (cargando y con error a la vez). Con un reducer, cada acción define el estado completo siguiente.
* **Transiciones de estado no triviales.** Cuando el estado nuevo depende del anterior de una forma más elaborada que un incremento o una copia, conviene sacar esa lógica a una función pura y testeable.

Dónde ponerlo:

* **`useReducer` solo**, dentro de un componente, si el estado es **local** a ese componente (por ejemplo, un formulario grande que vive en una sola pantalla).
* **`useReducer` con Context**, si otros componentes lejanos también necesitan leerlo o cambiarlo. Es adecuado para estado compartido de complejidad media. Si el estado y las acciones crecen mucho, o hay problemas de re-renders, suele convenir una librería dedicada como Redux Toolkit o Zustand.

-----

## Resumen en 5 líneas

1. `const [state, dispatch] = useReducer(reducer, initialState)`: lo que le pasas (`reducer`, `initialState`) es distinto de lo que te devuelve (`[state, dispatch]`).
2. El reducer es una función pura `(state, action) => nuevoEstado`: nunca muta el estado recibido, y su `default` devuelve el estado sin cambios.
3. Para cambiar el estado envías una acción con `dispatch({ type, payload })`; el `payload` viaja de quien llama a `dispatch` hacia el reducer, y solo se usa cuando el reducer necesita un dato externo.
4. Con Context, los consumidores solo necesitan `dispatch`; `useCart()` devuelve un objeto y se desestructura con llaves.
5. `useState` es la opción por defecto; `useReducer` conviene con muchas acciones, estados coherentes entre sí o transiciones no triviales.

-----

## Para profundizar

<details>
<summary>La "zona muerta temporal" (temporal dead zone)</summary>

Cuando declaras una variable con `const` o `let`, JavaScript la "reserva" para todo el bloque desde el principio, antes de ejecutar la línea donde se le asigna valor. Entre el comienzo del bloque y esa línea, la variable **existe pero no se puede leer**: ese tramo se llama zona muerta temporal. Si intentas leerla ahí, JavaScript lanza un error como `Cannot access 'x' before initialization`.

```jsx
// A nivel del archivo
function reducer(state, action) { /* ... */ }
const initialState = { count: 0 };

function Counter() {
  // Este const declara un "reducer" nuevo dentro de Counter.
  // Ese nombre queda reservado en todo Counter desde el principio, pero sin valor (zona muerta)...
  // ...y el lado derecho lee "reducer": ahora es el nuevo, todavía sin valor.
  const [reducer, initialState] = useReducer(reducer, initialState);
}
```

El error aparece porque el `const` nuevo está en un nivel **más interno** (dentro de `Counter`) que la función original (a nivel del archivo), así que la tapa. Si las dos declaraciones estuvieran en el mismo nivel, JavaScript daría otro error distinto, `Identifier 'reducer' has already been declared`.

Es una regla general de `const` y `let`, que no tiene nada de especial con `useReducer`: pasa con cualquier variable que se lea en su propia declaración.

</details>

<details>
<summary>Por qué un reducer puro es fácil de testear</summary>

Como el reducer es una función pura, para probarlo no necesitas renderizar ningún componente: lo llamas directo con un estado y una acción, y comparas el resultado.

```jsx
test('increment suma 1', () => {
  const result = reducer({ count: 0 }, { type: 'increment' });
  expect(result).toEqual({ count: 1 });
});
```

Con el mismo estado y la misma acción siempre obtienes el mismo resultado, así que no hace falta simular nada.

</details>

<details>
<summary>Modelar estados como máquina de estados</summary>

Una máquina de estados describe una operación con un conjunto **cerrado** de situaciones posibles y las transiciones permitidas entre ellas. Por ejemplo, una petición puede estar en `idle` (sin empezar), `loading`, `success` o `error`:

```tsx
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: string[] }
  | { status: 'error'; message: string };
```

Cada acción del reducer devuelve **uno** de esos estados completos. Así no puede existir una combinación imposible, como `loading` con un mensaje de error a la vez, algo que sí se podría armar con varios `useState` independientes.

</details>

-----

## En entrevista

### Respuesta corta (junior)

`useReducer` es un hook para manejar estado con una función reducer `(state, action) => newState`, que concentra todas las transiciones del estado. Los componentes no modifican el estado: llaman a `dispatch` con una acción que describe lo ocurrido. Se usa en lugar de `useState` cuando el estado tiene varios campos relacionados o muchas operaciones, porque la lógica queda en un solo lugar y es más fácil de mantener y probar.

### Respuesta ampliada (semi-senior)

* **Reducer puro:** `(state, action) => newState`, sin efectos secundarios ni mutación. Mismos argumentos, mismo resultado.
* **`dispatch` estable:** conserva su identidad entre renders, por lo que no hace falta ponerlo en dependencias de `useMemo` o `useCallback`, y pasarlo como prop o por Context no rompe la memoización.
* **Lógica centralizada y testeable:** al ser una función pura, se prueba sin renderizar componentes.
* **Acciones como unión discriminada:** el `type` permite que TypeScript estreche `action` en cada `case`. Con `const _x: never = action` en el `default` se obtiene comprobación de exhaustividad.
* **Inmutabilidad:** el reducer devuelve un objeto nuevo. React compara con `Object.is`; si devuelves la misma referencia, puede omitir el re-render.
* **Con Context:** el Provider expone `state` y `dispatch`. Separarlos en dos contextos evita que los componentes que solo despachan se re-rendericen cuando cambia el estado.
* **Relación con Redux:** comparten el patrón reducer/acción/dispatch. Redux añade un store externo a React, middleware, DevTools y suscripción por selectores.
* **Cuándo `useState` basta:** estado simple e independiente, con pocas operaciones triviales.
* **StrictMode:** en desarrollo React invoca el reducer dos veces para detectar impurezas. Un reducer con efectos o mutaciones produce resultados incorrectos.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre `useState` y `useReducer`?**
`useState` expone un setter y la lógica de cambio queda en los manejadores. `useReducer` mueve esa lógica a un reducer y los componentes solo despachan acciones. Internamente son equivalentes en capacidad; cambia la organización del código.

**2. ¿Qué es un reducer puro y por qué importa?**
Es una función que depende solo de sus argumentos y no tiene efectos secundarios. Importa porque React puede ejecutarla más de una vez (StrictMode) y porque la hace predecible y testeable.

**3. ¿Para qué sirve `payload`?**
Para enviar al reducer datos que no puede deducir del estado, como el item a agregar o el `id` a quitar. Es una convención de nombre, no una regla de React: la forma de la acción la defines tú.

**4. ¿`dispatch` cambia entre renders?**
No. React garantiza su identidad estable, así que puede omitirse de las dependencias de `useEffect` sin riesgo de ejecuciones extra.

**5. ¿Cómo se tipan `Action` y el reducer en TypeScript?**
`Action` es una unión discriminada y el reducer se firma con estado y acción explícitos:

```tsx
type Action = { type: 'add'; payload: CartItem } | { type: 'clear' };
function cartReducer(state: CartState, action: Action): CartState { /* ... */ }
```

**6. ¿`useReducer` + Context reemplaza a Redux?**
Cubre estado compartido de complejidad media. Context re-renderiza a todos los consumidores cuando cambia su valor y no ofrece selectores, middleware ni DevTools de serie. Para estado global grande o con muchas actualizaciones, Redux Toolkit o Zustand son más adecuados.

-----

## Siguiente lección

Con esto cierras la carpeta de Hooks y Context. Lo que sigue son los patrones de diseño de React, formas comunes de organizar componentes: [React Programming Patterns](../05-patrones-estilos-lifecycle/01-React%20Programming%20Patterns.md). Si quieres ir más a fondo con el tipado de `useReducer` y Context, mira [Tipado de useReducer y Context API](../11-typescript-y-react/03-Tipado%20de%20useReducer%20y%20Context%20API.md).
