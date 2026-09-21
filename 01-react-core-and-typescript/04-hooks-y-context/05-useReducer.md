# useReducer: ordenar los cambios de estado complejos

## En una frase

`useReducer` te deja juntar **toda la lógica de cambio de un estado en una sola función** (el *reducer*), y para cambiarlo solo "envías un pedido" con `dispatch`.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo guardar datos que cambian con `useState`, y por qué no se modifican arrays y objetos sino que se hacen copias: [useState](01-The%20State%20Hook.md).
* Cómo compartir datos entre componentes lejanos con Context: [React Context](04-React%20Context.md). Solo lo necesitas para la sección "Combinarlo con Context".

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Reducer:** una función que recibe el estado actual y una acción, y devuelve el estado nuevo. Es la regla que decide cómo cambia el estado.
* **Acción (action):** un objeto que describe qué quieres que pase. Por ejemplo `{ type: 'increment' }`.
* **Dispatch:** la función que usas para enviar una acción al reducer.
* **Payload:** el dato extra que viaja dentro de la acción, cuando el reducer lo necesita (por ejemplo, el producto a agregar).
* **Función pura:** una función que, con los mismos datos de entrada, siempre devuelve lo mismo, y no hace nada "por fuera" (no llama a APIs, no cambia otras variables).
* **Unión discriminada (TypeScript):** un tipo formado por varias opciones que comparten un campo (por ejemplo `type`) que dice cuál opción es.

-----

## El problema

Con `useState`, cada cambio suele tener su propia función. Cuando el mismo estado se puede cambiar de muchas maneras, el código se llena de manejadores parecidos:

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

Con tres acciones todavía se aguanta. Pero imagina diez, o un estado con varios campos que tienen que cambiar juntos: cada manejador repite su propia copia con spread (`...prev`), y es fácil olvidarte de un campo o dejar datos inconsistentes.

`useReducer` resuelve esto: toda la lógica de cambio vive en **un solo lugar**, y el resto del componente solo dice *qué pasó*.

-----

## Cómo funciona

### Las tres piezas

`useReducer` trabaja con tres piezas:

* **state:** el valor actual del estado, igual que en `useState`.
* **dispatch(acción):** la función para pedir un cambio. Es como **enviar un pedido**: tú no cambias el estado, solo avisas qué quieres que pase.
* **reducer(state, action):** la función que recibe el pedido y decide cuál es el estado nuevo. Es la **regla** que se aplica a cada pedido.

El recorrido siempre es el mismo:

```
1. Algo pasa (un clic)   ->  dispatch({ type: 'increment' })
2. React llama al reducer ->  reducer(estadoActual, { type: 'increment' })
3. El reducer devuelve el estado nuevo
4. React guarda ese estado y vuelve a renderizar el componente
```

### Paso 1: importarlo y llamarlo

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducer, initialState);
```

Fíjate en que hay dos cosas distintas en esa línea:

* Lo que **le pasas** a `useReducer`: la función `reducer` y el estado inicial `initialState`.
* Lo que **te devuelve**: un array de dos posiciones, `[state, dispatch]`.

Son dos pares de nombres que aparecen en la misma línea, pero **no son lo mismo**. No es una cosa vista dos veces.

### Paso 2: los nombres se eligen por posición

Los corchetes `[state, dispatch]` son *desestructuración de arrays*: sacan las dos posiciones del array y les ponen nombre. Se asigna **por posición**: el primero es el estado, el segundo es `dispatch`. Los nombres los eliges tú:

```jsx
const [contador, enviarAccion] = useReducer(reducer, initialState);
// contador es la posición 0 (el estado)
// enviarAccion es la posición 1 (dispatch), aunque lo hayas llamado distinto
```

La desestructuración de **objetos** (con llaves) funciona al revés: asigna **por nombre**. El orden no importa, pero el nombre tiene que coincidir con la propiedad del objeto:

```jsx
const { state, dispatch } = algunObjeto;   // busca las propiedades "state" y "dispatch"
```

### Paso 3: el reducer

El reducer es una **función pura**: recibe `(state, action)` y devuelve el estado nuevo. Usa un `switch` sobre `action.type` para decidir qué hacer:

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

Tres reglas del reducer:

* **Nunca modifica el estado que recibe.** Siempre devuelve uno **nuevo** (igual que con `useState`: copias, no cambios in situ).
* **Siempre devuelve algo.** Cada `case` termina en un `return`.
* **El `default` devuelve el estado sin cambios.** Así, si llega una acción que el reducer no conoce, no se rompe nada ni se devuelve `undefined`.

Que sea "pura" significa que no llama a APIs ni hace cosas por fuera: con el mismo estado y la misma acción, siempre da el mismo resultado. Eso la hace predecible.

### Paso 4: dispatch y payload

Para cambiar el estado, llamas a `dispatch` con un objeto de acción. Por convención, tiene al menos un `type` que dice qué pasó:

```jsx
dispatch({ type: 'increment' });
```

`dispatch` no cambia el estado en el momento: programa una llamada a `reducer(estadoActual, acción)` y le avisa a React que vuelva a renderizar con lo que el reducer devuelva.

A veces el reducer necesita un dato más. Ese dato viaja en una propiedad llamada `payload`:

```jsx
dispatch({ type: 'setCount', payload: 10 });
```

```jsx
case 'setCount':
  return { count: action.payload };
```

El punto que más confunde: **el `payload` viaja desde quien llama a `dispatch` hacia el reducer**. Nunca sale del reducer. El reducer solo lo **recibe**, como parte del segundo parámetro `action`.

```
Quien llama a dispatch  --- { type, payload } --->  reducer
                                                       |
                                                       v
                                              devuelve el estado nuevo
```

### ¿Cuándo hace falta payload?

Hace falta cuando el reducer necesita un dato que **no puede saber por sí mismo** a partir del `state`. Por ejemplo: qué producto agregar, qué `id` quitar, qué valor guardar.

No hace falta cuando alcanza con saber **qué acción fue**:

```jsx
dispatch({ type: 'clear' });                       // vaciar: no hace falta ningún dato extra
dispatch({ type: 'add', payload: item });          // agregar: el reducer no sabe cuál item
```

Ojo: el payload **no es solo para hacer cuentas**. Sirve para cualquier dato que venga de afuera, se haga una cuenta con él o no. Agregar un item no calcula nada, y aun así necesita payload, porque el reducer no puede adivinar qué item querías agregar.

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

Qué pasa, paso a paso:

1. `useReducer(reducer, initialState)` arranca con `{ count: 0 }`.
2. Al hacer clic en "+", se ejecuta `dispatch({ type: 'increment' })`.
3. React llama a `reducer({ count: 0 }, { type: 'increment' })`, que devuelve `{ count: 1 }`.
4. React guarda ese estado y vuelve a ejecutar `Counter`, que muestra `Contador: 1`.

Fíjate que el JSX no sabe **cómo** se incrementa: solo dice "incrementa". Esa lógica vive únicamente en el reducer.

-----

## Ejemplo completo 2: un carrito de compras (TypeScript)

Aquí hay tres acciones distintas, y dos de ellas necesitan payload.

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

Qué pasa, paso a paso:

1. `Action` describe **todos** los pedidos posibles. `add` lleva un `CartItem` completo; `remove` lleva solo el `id` (para quitar un producto alcanza con saber cuál es); `clear` no lleva nada.
2. `cartReducer` tiene la firma `(state: CartState, action: Action): CartState`: recibe un estado y una acción, y promete devolver un estado.
3. En `add`, se crea un array **nuevo** con el item al final. En `remove`, `filter` crea un array nuevo sin ese `id`. Nunca se toca `state.items` directamente.
4. El componente solo hace `dispatch`. No tiene ninguna lógica de cómo se agrega o se quita.

> **Sobre el `id`:** cada producto necesita un `id` **único**, porque `remove` borra todos los que tengan ese `id` y React lo usa como `key`. Por eso el botón usa `Date.now()`, que da un número distinto en cada clic. En una app real, el `id` suele venir de tu base de datos.

-----

## Combinarlo con Context

Si otros componentes lejanos también necesitan el carrito, puedes poner el `useReducer` dentro de un Provider de Context. La ventaja: los consumidores ya **no necesitan varios setters** (uno por operación). Les alcanza con `dispatch`, y toda la lógica sigue en el reducer.

Reutilizamos `CartItem`, `CartState`, `Action`, `initialState` y `cartReducer` del ejemplo anterior.

### Paso 1: el tipo del contexto y el contexto

```tsx
import { createContext, useContext, useMemo, useReducer } from 'react';

type CartContextType = {
  state: CartState;
  dispatch: React.Dispatch<Action>;
};

const CartContext = createContext<CartContextType | undefined>(undefined);
```

`React.Dispatch<Action>` es el tipo de la función `dispatch`: una función que recibe una `Action`. El contexto arranca en `undefined` porque todavía no hay Provider; el `| undefined` en el tipo lo refleja.

### Paso 2: el Provider

```tsx
function CartProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  const value = useMemo(() => ({ state, dispatch }), [state]);

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}
```

El Provider llama a `useReducer` y comparte `{ state, dispatch }`. El `useMemo` hace que el objeto `value` solo se cree de nuevo cuando `state` cambia; así los consumidores no se vuelven a renderizar por un objeto nuevo sin necesidad. (`dispatch` es siempre la misma función, por eso no hace falta ponerlo en la lista.)

### Paso 3: el hook `useCart`

```tsx
function useCart() {
  const context = useContext(CartContext);
  if (context === undefined) {
    throw new Error('useCart debe usarse dentro de un CartProvider');
  }
  return context;
}
```

Envolvemos `useContext` en un hook propio. La guardia de `undefined` avisa con un mensaje claro si te olvidaste del Provider, y de paso hace que TypeScript sepa que `context` ya no es `undefined`.

### Paso 4: usarlo en un componente

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

Fíjate en que `AddButton` está **adentro** de `<CartProvider>`, y que `App` (que crea el Provider) no llama a `useCart()`.

-----

## Lógica asíncrona

El reducer es puro: **no hace llamadas a APIs**. La petición se hace en un efecto o en un evento, y cuando termina, se hace `dispatch` con el resultado:

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

Qué pasa: el efecto pide el dato; cuando llega, envía una acción con el usuario como `payload`, y el reducer solo lo guarda en el estado. La variable `cancelled` evita hacer `dispatch` si el componente ya se fue de la pantalla. Más sobre efectos en [useEffect](02-The%20Effect%20Hook.md).

Si el manejo de datos remotos crece (reintentos, caché, invalidación), conviene mirar React Query o Zustand. Mira [Zustand, Redux y Context: cuándo usar cada uno](../../03-state-management-and-data/01-Zustand,%20Redux%20y%20Context%20-%20Cuando%20usar%20cada%20uno.md).

-----

## Errores comunes

### 1. Mutar el estado dentro del reducer

```jsx
case 'add':
  state.items.push(action.payload);   // Mal: modifica el estado recibido
  return state;
```

**Por qué pasa:** parece más corto, pero devuelves el **mismo** objeto. React compara el estado nuevo con el anterior y, al ser el mismo, no ve cambios.
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

Para ver el tipado completo de `useReducer` con Context, mira [Tipado de useReducer y Context API](../11-typescript-y-react/03-Tipado%20de%20useReducer%20y%20Context%20API.md).

-----

## Cuándo sí y cuándo no

`useState` sigue siendo la opción **por defecto**. Pasa a `useReducer` cuando veas alguna de estas señales:

* **Formularios grandes con muchos campos relacionados.** En vez de un `useState` por campo (o manejadores casi idénticos), un reducer con acciones como `updateField`, `resetForm`, `submitStart`, `submitSuccess` o `submitError` deja todo en un lugar.
* **Sub-valores que deben mantenerse coherentes.** Un ejemplo típico es una **máquina de estados**, es decir, una operación que solo puede estar en una de varias situaciones posibles (`idle`, `loading`, `success` o `error`), con los datos o el mensaje de error de cada una. Con varios `useState` sueltos podrías terminar con combinaciones imposibles (cargando y con error a la vez). Con un reducer, cada acción define el estado completo siguiente.
* **Transiciones de estado no triviales.** Cuando el estado nuevo depende del anterior de una forma más elaborada que un incremento o una copia, conviene sacar esa lógica a una función pura y testeable.

Dónde ponerlo:

* **`useReducer` solo**, dentro de un componente, si el estado es **local** a ese componente (por ejemplo, un formulario grande que vive en una sola pantalla).
* **`useReducer` con Context**, si otros componentes lejanos también necesitan leerlo o cambiarlo. Este patrón funciona muy bien para estado global de complejidad media (a veces se lo llama informalmente "Redux sin librería externa"). Si el estado y las acciones crecen mucho, suele convenir una librería dedicada como Redux Toolkit o Zustand.

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

## Siguiente lección

Con esto cierras la carpeta de Hooks y Context. Lo que sigue son los patrones de diseño de React, formas comunes de organizar componentes: [React Programming Patterns](../05-patrones-estilos-lifecycle/01-React%20Programming%20Patterns.md). Si quieres ir más a fondo con el tipado de `useReducer` y Context, mira [Tipado de useReducer y Context API](../11-typescript-y-react/03-Tipado%20de%20useReducer%20y%20Context%20API.md).
