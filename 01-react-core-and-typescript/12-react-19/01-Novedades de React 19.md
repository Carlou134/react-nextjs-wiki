# Novedades de React 19

## Why React 19 Matters

Todo lo que vimos hasta ahora (componentes, hooks, formularios, routing, manejo de estado) sigue siendo el corazón de React tal como lo usás hoy. React 19 no reemplaza estos conceptos, pero introduce un cambio de fondo en **dónde** se ejecuta parte de esa lógica, y agrega herramientas nuevas para simplificar patrones que hasta ahora requerían bastante código repetitivo, especialmente alrededor de formularios y datos asíncronos.

El hilo conductor de esta versión es una mejor separación entre lo que corre en el **servidor** y lo que corre en el **navegador (cliente)**, junto con una simplificación notable de cómo React maneja acciones asíncronas como el envío de un formulario.

-----

## Server Components

Hasta ahora, todos los componentes que escribimos son **componentes de cliente**: el navegador descarga su código JavaScript, lo ejecuta, y recién ahí el componente se renderiza en pantalla. Esto significa que, si ese componente necesita datos de una API, tiene que pedirlos desde el navegador (típicamente con `useEffect` y `fetch`, como vimos en la lección de fetching de datos), lo que agrega una espera visible para el usuario.

Los **Server Components** son un nuevo tipo de componente que se ejecuta **en el servidor**, no en el navegador. Un Server Component puede acceder directamente a una base de datos o a una API interna durante el renderizado, y le envía al navegador únicamente el HTML ya resuelto, sin el código JavaScript del componente en sí. Esto trae tres beneficios concretos:

* **Menos espera para el usuario**: los datos ya vienen resueltos en el HTML inicial, en lugar de pedirse después de que la página cargó.
* **Mejor SEO**: los motores de búsqueda reciben contenido ya renderizado, sin depender de que se ejecute JavaScript para verlo.
* **Menos JavaScript en el cliente**: como el componente nunca se ejecuta en el navegador, su código ni siquiera se incluye en el paquete que el navegador descarga.

-----

## "use client" and "use server"

Como una aplicación real necesita ambos tipos de componentes (algunos que solo muestran datos, y otros que responden a clics o mantienen estado local), React 19 introduce dos directivas para indicar explícitamente dónde debe ejecutarse cada pieza de código:

* `"use client"`, escrita al principio de un archivo, marca ese componente (y todo lo que importe) para que se ejecute en el navegador. Es lo que necesitás en cualquier componente que use `useState`, `useEffect`, o maneje eventos como `onClick`.
* `"use server"` marca una función para que se ejecute exclusivamente en el servidor, típicamente para acceder a una base de datos o realizar una operación que no debería exponerse al navegador.

```jsx
'use client';

function LikeButton() {
  const [liked, setLiked] = useState(false);

  return (
    <button onClick={() => setLiked(!liked)}>
      {liked ? 'Te gusta' : 'Me gusta'}
    </button>
  );
}
```

Pensalo como una forma de trazar una frontera explícita en tu código: la lógica pesada de datos vive del lado del servidor, y la interactividad (todo lo que responde directamente a las acciones del usuario) vive del lado del cliente.

-----

## Actions: A New Way to Handle Forms

Antes de React 19, manejar el envío de un formulario que interactúa con un servidor implicaba un patrón repetitivo: un manejador de `onSubmit`, un `fetch` dentro de ese manejador, y variables de estado manuales para rastrear si el envío está en curso, si tuvo éxito o si falló, exactamente como vimos en la lección de manejo de errores y estados de carga.

React 19 introduce las **Actions**: funciones (frecuentemente marcadas con `"use server"`) que podés conectar directamente al atributo `action` de un formulario, sin necesidad de escribir manualmente el `onSubmit`, el `fetch` ni el manejo de estado de carga:

```jsx
async function createUser(formData) {
  'use server';
  // lógica para guardar el usuario en la base de datos
}

function SignupForm() {
  return (
    <form action={createUser}>
      <input name="name" />
      <button type="submit">Registrarse</button>
    </form>
  );
}
```

React se encarga de recolectar los datos del formulario, invocar la función, y coordinar los estados de carga y error asociados, todo con considerablemente menos código del que este mismo flujo requería antes.

-----

## New Hooks for Forms

Junto con las Actions, React 19 agrega tres hooks nuevos que resuelven necesidades muy comunes alrededor de formularios:

* **`useActionState`**: administra el estado que resulta de ejecutar una Action, como el mensaje de error que el servidor devolvió, sin que tengas que declarar ese estado manualmente con `useState`.
* **`useFormStatus`**: permite que un componente hijo del formulario (por ejemplo, el botón de envío) sepa si el formulario padre está actualmente enviándose, para poder deshabilitarse o mostrar un indicador de carga, sin necesidad de recibir esa información por props.
* **`useOptimistic`**: te permite mostrar inmediatamente un resultado "esperado" en la interfaz mientras la Action todavía se está procesando en el servidor. Cuando la Action termina, React vuelve solo al valor real. Es la misma idea de las actualizaciones optimistas que vimos con `useMutation` en React Query, pero incorporada directamente al propio React (lo vemos en detalle en la próxima sección).

Juntos, estos tres hooks son los que permiten escribir formularios que manejan carga, error y feedback inmediato con una fracción del código que este mismo comportamiento requería con `useState` y `useEffect`.

-----

## useOptimistic en detalle

Con las actualizaciones optimistas ya vimos el problema de fondo: esperar la respuesta del servidor para reflejar una acción hace que la interfaz se sienta lenta, así que mostramos el resultado esperado de inmediato y lo corregimos solo si algo falla. `useOptimistic()` resuelve exactamente eso, sin librerías externas y con una diferencia importante en cómo maneja el "rollback".

### La firma

```jsx
const [optimisticState, setOptimistic] = useOptimistic(value, reducer?);
```

* **`value`**: el valor "real", el que se muestra cuando no hay ninguna Action pendiente. Normalmente viene de un estado o de props.
* **`reducer`** (opcional): una función pura `(currentState, action) => nextOptimisticState` que define cómo se calcula el valor optimista a partir del actual y de lo que le pasás a `setOptimistic`. Sin reducer, `setOptimistic(nuevoValor)` simplemente reemplaza el valor.
* **`optimisticState`**: lo que tenés que renderizar. Es igual a `value` mientras nada esté pendiente, y pasa a ser el valor optimista mientras una Action esté en curso.
* **`setOptimistic`**: la función con la que declarás el cambio optimista.

### Un ejemplo completo: agregar un item a una lista

```jsx
import { useOptimistic, startTransition } from 'react';

function TodoList({ todos, saveTodo }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodo) => [
      ...currentTodos,
      { ...newTodo, pending: true },
    ]
  );

  function handleAdd(text) {
    startTransition(async () => {
      addOptimisticTodo({ id: crypto.randomUUID(), text }); // aparece YA en la lista
      await saveTodo(text);                                  // mientras tanto, se guarda de verdad
    });
  }

  return (
    <ul>
      {optimisticTodos.map((todo) => (
        <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

El flujo es este: al llamar a `addOptimisticTodo(...)`, React renderiza de inmediato la lista con el item nuevo (marcado como `pending`). En segundo plano, `saveTodo()` hace el trabajo real. Cuando termina y el padre actualiza `todos` con el dato definitivo, el estado optimista y el real **convergen automáticamente en el mismo render**: no hay ningún paso adicional para "limpiar" el valor optimista.

### El rollback es automático (y esa es la diferencia con React Query)

Si la Action falla y lanza un error, React deja de mostrar el valor optimista y vuelve a renderizar lo que valga `value` en ese momento. Como el padre solo actualiza `value` cuando la operación sale bien, el efecto visible es que **el cambio optimista desaparece solo**, sin que tengas que escribir un `onError` que restaure una copia guardada.

Lo que sí sigue siendo tu responsabilidad es avisarle al usuario que algo falló:

```jsx
startTransition(async () => {
  removeOptimistic(id);
  try {
    await deleteItem(id);
  } catch (e) {
    setError(e.message); // el item reaparece solo; acá solo mostrás el mensaje
  }
});
```

Compará esto con lo que vimos en React Query, donde el rollback es explícito: `onMutate` guarda una copia del estado anterior y `onError` la restaura a mano. Con `useOptimistic()`, ese trabajo lo hace React.

### Restricciones importantes

* **`setOptimistic()` tiene que llamarse dentro de `startTransition` o dentro de una Action** (por ejemplo, la función que le pasás a `<form action={...}>`). Si la llamás fuera, React emite una advertencia y el valor optimista se ve solo por un instante.
* **No se puede llamar durante el renderizado.** Es una función para reaccionar a eventos, como cualquier setter.
* **El valor optimista es temporal por diseño**: solo existe mientras la Action está pendiente. No es un lugar donde guardar datos; para eso sigue estando `value`.
* Si usás un reducer y el `value` base cambia mientras la Action está en curso, React vuelve a ejecutar el reducer con el valor nuevo para recalcular el resultado optimista.

### ¿useOptimistic o React Query?

Son dos herramientas para el mismo problema, pero no son intercambiables:

* **`useOptimistic()`** conviene cuando el estado vive en el propio componente (o sus props) y la operación es una Action de formulario o una transición: es más simple, sin dependencias, y el rollback viene resuelto.
* **React Query** conviene cuando los datos ya viven en el **cache compartido** de tu aplicación: si varias pantallas leen la misma `queryKey`, `onMutate` puede actualizar ese cache una sola vez y todas se enteran. `useOptimistic()` es local al componente donde lo declarás; no sabe nada de un cache global.

> **En TypeScript:** el valor optimista se infiere a partir de `value`, así que en el caso simple no hay que anotar nada. Cuando usás un reducer, tipá explícitamente el segundo parámetro (`action`), porque TypeScript no tiene de dónde inferirlo:
>
> ```tsx
> type Todo = { id: string; text: string; pending?: boolean };
>
> const [optimisticTodos, addOptimisticTodo] = useOptimistic(
>   todos,
>   (currentTodos: Todo[], newTodo: Todo) => [...currentTodos, { ...newTodo, pending: true }]
> );
> ```
>
> Si el reducer tiene que soportar varios tipos de cambio optimista (agregar, quitar, marcar), la misma unión discriminada que usamos con `useReducer()` funciona igual como tipo de `action`.

-----

## The use() API

El nuevo `use()` es una función (no un hook, aunque se usa de forma parecida) que te permite leer directamente el valor resuelto de una promesa durante el renderizado, en combinación con `Suspense`:

```jsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise);

  return <p>{user.name}</p>;
}
```

Esto reemplaza, en muchos casos, el patrón que veníamos usando de `useEffect` más `useState` más un estado de `isLoading` para manejar datos asíncronos: en lugar de gestionar manualmente esos tres momentos (cargando, éxito, error), le pasás una promesa a `use()`, y dejás que un componente `Suspense` envolvente se encargue de mostrar un estado de carga mientras esa promesa se resuelve.

Una diferencia importante respecto de los hooks tradicionales: `use()` sí puede llamarse condicionalmente, dentro de un `if` o después de un retorno temprano, algo que las reglas de los hooks prohíben explícitamente para `useState` o `useEffect`.

-----

## `ref` Without `forwardRef`

Antes de React 19, si querías que un componente de función recibiera una referencia (`ref`) a uno de sus elementos internos, era obligatorio envolver ese componente en `forwardRef()`:

```jsx
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});
```

En React 19, `ref` se puede recibir como una prop común y corriente, sin necesidad de este envoltorio adicional:

```jsx
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

Es un cambio pequeño en apariencia, pero elimina una capa de indirección que generaba confusión constante, sobre todo para quienes recién estaban aprendiendo React y se topaban con `forwardRef` sin entender bien por qué era necesario.

-----

## Putting It All Together

React 19 no cambia los fundamentos que estudiamos a lo largo de esta guía (componentes, props, estado, efectos, contexto), pero sí cambia la forma en que se combinan para resolver problemas de datos y formularios: mueve parte de la lógica al servidor con los Server Components, reduce drásticamente el código necesario para manejar formularios con las Actions y sus hooks asociados, simplifica el manejo de promesas con `use()`, y elimina fricción innecesaria como la de `forwardRef`. El resultado general apunta en una misma dirección: menos código repetitivo de tu parte, y una frontera más clara entre lo que corre en el servidor y lo que corre en el navegador.
