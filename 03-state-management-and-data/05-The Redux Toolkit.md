# The Redux Toolkit

## Intro to Redux Toolkit

Probablemente hayas notado que trabajar con Redux puede volverse bastante verboso y complejo. Si te sientes abrumado por todas las partes móviles y detalles que recordar, no estás solo.

Los problemas y quejas comunes que la gente encuentra al trabajar con Redux incluyen:

*   "Configurar una store de Redux es demasiado complejo."
*   "Tengo que añadir muchos paquetes para que Redux haga algo útil."
*   "Redux requiere demasiado código boilerplate."
*   "Escribir actualizaciones inmutables es demasiado propenso a errores."

¡Afortunadamente, el equipo de Redux creó **Redux Toolkit** para abordar estos desafíos!

Redux Toolkit contiene paquetes y funciones adaptados para construir una aplicación Redux. Incorpora **mejores prácticas**, simplifica la mayoría de las tareas de Redux, previene errores comunes y facilita la escritura de aplicaciones Redux.

Debido a lo efectivo que ha demostrado ser para abordar las preocupaciones de la lógica verbosa "escrita a mano" del pasado, Redux Toolkit se ha convertido en la **forma preferida** de escribir la lógica de las aplicaciones Redux.

En esta lección, nos sumergiremos en las poderosas características de Redux Toolkit que te permitirán **refactorizar y simplificar** tu lógica Redux existente. Probarás dos métodos esenciales, **`createSlice()`** y **`configureStore()`**, y aprenderás cómo integrarlos en tu aplicación. Para explorar la gama completa de métodos que ofrece Redux Toolkit, consulta la [documentación de Redux Toolkit](https://redux-toolkit.js.org/).

Si quieres probar el código localmente mientras trabajas en esta lección, usa este comando para instalar el paquete Redux Toolkit en tu aplicación:

```bash
npm install @reduxjs/toolkit
```

----

## "Slices" of State

Antes de profundizar en esta lección, refresquemos nuestra memoria sobre lo que queremos decir cuando hablamos de una **"porción" (slice)** de estado. Una **"porción" de estado** es un segmento del estado global que se centra en una característica particular. Abarca los datos relacionados, junto con sus reducers, acciones y selectores asociados. Piénsalo como una unidad autocontenida dedicada a gestionar una parte específica de la funcionalidad de tu aplicación.

En el siguiente ejemplo, `state.todos` y `state.visibilityFilter` representan porciones distintas.

```jsx
const state = {
  todos: [
    {
      id: 0,
      text: "Aprender Redux-React",
      completed: true,
    },
    {
      id: 1,
      text: "Aprender Redux Toolkit",
      completed: false,
    }
  ], 
  visibilityFilter: "SHOW_ALL"
}
```

Para cada porción del estado, normalmente definimos un **reducer** correspondiente. Estos se conocen como **"slice reducers"** (reducers de porción). Cada reducer es similar a un trabajador únicamente responsable de gestionar el estado de su respectiva porción. Este enfoque modular simplifica las aplicaciones complejas y facilita la depuración.

Exploremos el reducer de porción para la porción `state.todos`:

```jsx
/* todosSlice.js */
const addTodo = (todo) => {
  return {
    type: 'todos/addTodo',
    payload: todo
  }
}

const toggleTodo = (todo) => {
  return {
    type: 'todos/toggleTodo',
    payload: todo
  }
}

const todos = (state = [], action) => {
  switch (action.type) {
    case 'todos/addTodo':
      return [
        ...state,
        {
          id: action.payload.id,
          text: action.payload.text,
          completed: false
        }
      ]
    case 'todos/toggleTodo':
      return state.map(todo =>
        todo.id === action.payload.id ? { ...todo, completed: !todo.completed } : todo
      )
    default:
      return state
  }
}
```

Observa que este archivo maneja exclusivamente los datos de `state.todos` y no interactúa con la porción `state.visibilityFilter`. Gestionar el estado **una porción a la vez** nos permite manejar de manera más efectiva la lógica distinta de cada parte individual de nuestra aplicación.

Si bien este ejemplo muestra la lógica del reducer y los creadores de acciones juntos, en proyectos más grandes, a menudo dividimos estas partes en archivos separados.

Hay mucho código escrito solo para tener algunos reducers y creadores de acciones. Visitamos ahora la función **`createSlice()`** de Redux Toolkit y veamos cómo agiliza este proceso.

----

## Refactoring with createSlice()

Ahora que hemos visto una forma de definir un reducer de porción y los creadores de acciones asociados, podemos ver cómo usar **`createSlice()`** para agilizar el proceso. Así es como se veía nuestro código antes:

```jsx
/* todosSlice.js */
const addTodo = (todo) => {
  // lógica omitida...
}

const toggleTodo = (todo) => {
  // lógica omitida...
}

const todos = (state = [], action) => {
  // lógica omitida...
}
```

A partir de esto, podemos ver que Redux tradicional requiere escribir **tipos de acción, creadores de acción y reducers** por separado. `createSlice()` agiliza este proceso generando todo esto basándose en un **único objeto de configuración**.

`createSlice` tiene un parámetro: un **objeto de configuración**. El objeto tiene las siguientes propiedades:

*   **`name`**: Una cadena que identifica el nombre de la porción. `createSlice()` usa esto para generar los tipos de acción y los creadores de acción.
*   **`initialState`**: El valor del estado inicial para el reducer.
*   **`reducers`**: Un objeto donde cada clave representa un **tipo de acción**, un identificador de cadena para la acción. El método asociado, conocido como **"case reducer"**, describe cómo debe actualizarse el estado cuando se desencadena esa acción. Estos reducers funcionan como conjuntos de instrucciones, dirigiendo los cambios de estado basándose en el tipo de acción despachada.

¡Veamos esto en acción! Revisa el siguiente ejemplo:

```jsx
/* todosSlice.js */
// Objeto de configuración para createSlice()
const options = {
  name: 'todos', // Nombre de la porción
  initialState: [], // Estado inicial de la porción
  reducers: {
    // Reducer para la acción "addTodo"
    addTodo: (state, action) => {
      return [
        ...state,
        {
          id: action.payload.id,
          text: action.payload.text,
          completed: false
        }
      ];
    },
    // Reducer para la acción "toggleTodo"
    toggleTodo: (state, action) => {
      return state.map(todo =>
        (todo.id === action.payload.id) ? { ...todo, completed: !todo.completed } : todo
      );
    }
  }
};

const todosSlice = createSlice(options);
```

En este ejemplo, se crea un objeto de configuración con el nombre `options`. Se pasa a `createSlice()` para generar componentes para gestionar una porción. Usando `options`, `createSlice()` genera un **slice**:

*   llamado `todos`,
*   inicializado con un array vacío como su estado inicial,
*   equipado con reducers asociados a dos nombres de acción: `addTodo` y `toggleTodo`,
*   equipado con **creadores de acción generados automáticamente** para cada función reducer definida en el objeto `reducers`. Estos creadores de acción generados se nombran según las claves del reducer.

¡Podemos ver aquí que usar `createSlice()` reduce drásticamente la cantidad de código boilerplate que necesitas escribir!

En los próximos ejercicios, exploraremos `createSlice()` aún más profundamente. Por ahora, practiquemos llamando a `createSlice()`.

---

## Writing "Mutable" Code with Immer

Una de las reglas más cruciales para los reducers de Redux es **evitar cambiar el estado directamente**. Esto significa que necesitamos hacer copias de cada nivel de anidación que se va a actualizar. Normalmente logramos esto usando los **operadores de propagación (spread operators)** de arrays y objetos de JavaScript, así como otras funciones que crean copias de los valores originales.

¡Adherirse a este principio puede volverse bastante complejo, por lo que el error más común cometido por los usuarios de Redux es modificar accidentalmente el estado dentro de los reducers!

¡Redux Toolkit tiene una solución para este enigma! `createSlice()` utiliza una librería llamada **Immer** para ayudar a evitar este error.

Immer utiliza un objeto especial de JS llamado **Proxy** para envolver los datos que proporcionas y te permite escribir código que "muta" esos datos envueltos. Immer hace esto rastreando todos los cambios que has realizado y luego usa esa lista de cambios para devolver un valor actualizado de forma inmutable, como si hubieras escrito toda la lógica de actualización inmutable a mano.

Immer se utiliza internamente de forma automática, ¡así que no hay nada que debas hacer por tu parte para asegurarte de que se actualice de forma inmutable!

Por lo tanto, en lugar de esto:

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      return [
        ...state,
        {
          ...action.payload,
          completed: false
        }
      ];
    },
    toggleTodo: (state, action) => {
      return state.map(todo =>
        todo.id === action.payload.id ? { ...todo, completed: !todo.completed } : todo
      );
    }
  }
});
```

Puedes escribir código que se vea así:

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push({ 
        ...action.payload, 
        completed: false 
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.find(todo => todo.id === action.payload.id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    }
  }
});
```

`addTodo` está llamando a `state.push()` aquí, lo que normalmente es malo porque la función `array.push()` **muta** el array existente. Del mismo modo, `toggleTodo` simplemente encuentra el objeto `todo` coincidente y luego lo muta reasignando su valor.

**Sin embargo, gracias a Immer, este código funcionará perfectamente.**

No necesitas aprender la librería Immer. Todo lo que necesitas saber es que `createSlice()` la aprovecha, permitiéndonos **"mutar" nuestro estado de forma segura**. Puede que te resulte útil revisar algunos de los [patrones de actualización comunes utilizados con Immer](https://immerjs.github.io/immer/update-patterns/).

----

## Returned Objects and Auto-Generated Actions

Hasta ahora, hemos cubierto el objeto que se pasa a `createSlice()`. Ahora, profundicemos en lo que esta función realmente **devuelve**.

Tomemos el ejemplo `todosSlice` con el que hemos estado trabajando. Cuando aplicas `createSlice()`, te devuelve un objeto como este:

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo(state, action) {
      const { id, text } = action.payload;
      state.push({ id, text, completed: false });
    },
    toggleTodo(state, action) {
      const todo = state.find(todo => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    }
  }
});

/* Objeto devuelto por todosSlice */
{
  name: 'todos',
  reducer: (state, action) => newState,
  actions: {
    addTodo: (payload) => ({type: 'todos/addTodo', payload}),
    toggleTodo: (payload) => ({type: 'todos/toggleTodo', payload})
  },
  // el campo de los case reducers se omite aquí
}
```

Desglosemos esto:

*   **`name`**: Contiene una cadena utilizada como **prefijo** para los tipos de acción generados.
*   **`reducer`**: Esta es la **función reducer completa**.
*   **`actions`**: Estos son los **creadores de acción generados automáticamente**.

Entonces, ¿cómo son estos objetos de acción generados automáticamente?

Por defecto, cada creador de acción acepta **un argumento**, que se convierte en `action.payload`. La cadena `action.type` se forma combinando el `name` de la porción con el nombre de la función del reducer de caso.

Por ejemplo:

```jsx
console.log(todosSlice.actions.addTodo('walk dog'));
// {type: 'todos/addTodo', payload: 'walk dog'}
```

Con estos creadores de acción generados automáticamente, podemos exportarlos y usarlos en otros archivos. En teoría, podrías exportar todo el objeto `slice` devuelto por `createSlice()`. Pero, siguiendo el patrón **"ducks"** de la comunidad Redux, sugerimos exportar los **creadores de acción con nombre por separado** del reducer:

```jsx
export const { addTodo, toggleTodo } = todosSlice.actions;
```

Una vez que exportamos los creadores de acción, podemos usarlos para despachar acciones de manera estructurada en toda nuestra aplicación.

----

## Returned Objects and Reducers

Examinemos ahora más de cerca el **reducer** dentro del objeto devuelto por `createSlice()`.

```jsx
const options = {
  // campos de opciones omitidos.
};
const todosSlice = createSlice(options);

/* Objeto devuelto por todosSlice */
{
  name: 'todos',
  reducer: (state, action) => newState,
  actions: {
    addTodo: (payload) => ({type: 'todos/addTodo', payload}),
    toggleTodo: (payload) => ({type: 'todos/toggleTodo', payload})
  },
  // el campo de los case reducers se omite aquí
}
```

**`todosSlice.reducer` es la función reducer integral** que representa la colección de **case reducers**, cada uno asociado con diferentes acciones que se espera que maneje tu porción. Efectivamente, combina los case reducers en uno solo. Esto se conoce comúnmente como el **"slice reducer"** (reducer de la porción).

Cuando se despacha una acción con el tipo `'todos/addTodo'`, `todosSlice` emplea `todosSlice.reducer()` para comprobar si el tipo de la acción despachada se alinea con alguno de los case reducers en `todos.actions`. Si se encuentra una coincidencia, se ejecuta la función del case reducer correspondiente; si no, se devuelve el estado actual. ¡Esto refleja el patrón que empleamos anteriormente con las sentencias `switch/case`!

Una vez autogenerado, **`todosSlice.reducer` necesita ser exportado** para que pueda integrarse en la store global y utilizarse como la porción de estado `todos`. Según el patrón "ducks", exportamos `todosSlice.reducer` como **exportación por defecto**.

```jsx
export const { addTodo, toggleTodo } = todosSlice.actions;
export default todosSlice.reducer;
```

Exportar el reducer de una porción es como darle a cada parte de tu aplicación su propia **caja especial** con instrucciones sobre cómo manejar sus datos. ¡Esta caja se guarda en un lugar central, la **store**, donde la gestión de datos de tu aplicación se unifica!

---

## Converting the Store to Use `configureStore()`

Además de simplificar la lógica para acciones y reducers, Redux Toolkit tiene un método **`configureStore()`** que simplifica el proceso de configuración de la store. `configureStore()` envuelve los métodos `createStore()` y `combineReducers()` de la librería Redux, y maneja la mayor parte de la configuración de la store automáticamente.

Por ejemplo, echa un vistazo a este archivo, que crea y exporta un **rootReducer**:

```jsx
// rootReducer.js
import { combineReducers } from 'redux';
import todosReducer from './features/todos/todosSlice';
import filtersReducer from './features/filters/filtersSlice';

const rootReducer = combineReducers({
  // Define un campo de estado de nivel superior llamado 'todos', manejado por todosReducer
  todos: todosReducer,
  visibilityFilter: filtersReducer
});

export default rootReducer;
```

...y este archivo, que crea y exporta la store:

```jsx
// store.js
import { createStore } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';
import rootReducer from './reducer';
import { fetchTodos } from './actions'; 

const store = createStore(rootReducer, composeWithDevTools());
store.dispatch(fetchTodos());
export default store;
```

Ahora, echemos un vistazo a cómo podemos **refactorizar estos dos archivos usando `configureStore()`**.

`configureStore()` acepta un único parámetro: un **objeto de configuración**. El objeto de entrada debe tener una propiedad `reducer` que defina:

*   Una función para ser usada como **reducer raíz**, o
*   Un objeto de **reducers de porción (slice reducers)** , que se combinarán para crear un reducer raíz.

Hay muchas propiedades disponibles en este objeto, pero para los propósitos de esta lección, solo la propiedad `reducer` será suficiente.

```jsx
import { configureStore } from '@reduxjs/toolkit';
import todosReducer from './features/todos/todosSlice';
import filtersReducer from './features/filters/filtersSlice';

const store = configureStore({
  reducer: {
    // Define un campo de estado de nivel superior llamado 'todos', manejado por todosReducer
    todos: todosReducer,
    filters: filtersReducer
  }
});

export default store;
```

Observa todo el trabajo que esta única llamada a `configureStore()` hace por nosotros:

*   **Reducer**: Combina `todosReducer` y `filtersReducer` en una función de reducer raíz que manejará un estado raíz con la forma `{todos, filters}`, eliminando la necesidad de llamar a `combineReducers()`. Esto reduce la cantidad de código boilerplate que necesitamos escribir.
*   **Store**: Crea una store de Redux usando ese reducer raíz, eliminando la necesidad de llamar a `createStore()`.
*   **Middleware**: Añade automáticamente middleware para comprobar errores comunes como la mutación accidental del estado. En la forma manual tradicional, necesitaríamos configurar esto nosotros mismos.
*   **DevTools**: Configura automáticamente la conexión con la extensión Redux DevTools. En la forma manual tradicional, también necesitaríamos configurar esto nosotros mismos.

Debido a la cantidad de código boilerplate que podemos omitir con `configureStore()`, podemos simplemente importar los reducers de porción individuales directamente en este archivo, en lugar de crear un archivo separado para el reducer raíz y tener que exportarlo/importarlo.

Como esto es tan simple como cambiar el código de configuración de la store, **todo el código de características existente de la aplicación funcionará perfectamente**.

¡Probemos esto por nosotros mismos!

----

## Review

¡Felicitaciones! Has aprendido mucho sobre Redux Toolkit y los métodos esenciales para refactorizar y simplificar la lógica Redux existente.

*   **Redux Toolkit (RTK)** contiene paquetes y funciones que incorporan las mejores prácticas sugeridas, simplifican la mayoría de las tareas de Redux, previenen errores comunes y facilitan la escritura de aplicaciones Redux.
*   RTK tiene una función **`createSlice()`** que nos ayudará a simplificar nuestra lógica de reducers y acciones de Redux.
*   `createSlice()` tiene un parámetro, un **objeto de configuración**, que llamamos `options`. En esta lección, cubrimos tres propiedades del objeto: `name`, `initialState` y `reducers`. El objeto de configuración tiene más propiedades que se cubrirán en lecciones siguientes.
*   Un **case reducer** es un método que puede actualizar el estado y se ejecutará cuando se despache el tipo de acción correspondiente. Esto es similar a un `case` en una sentencia `switch`.
*   Puedes escribir código que **"mute" (mute)** el estado dentro de los case reducers pasados a `createSlice()`, e **Immer** devolverá de forma segura y precisa un estado actualizado de forma inmutable.
*   `createSlice()` devuelve un objeto con las siguientes propiedades: `name`, `reducer`, `actions` y `caseReducers`.
*   Normalmente usamos una convención de código de la comunidad Redux llamada **patrón "ducks"** al exportar los creadores de acción y el reducer.
*   RTK tiene una función **`configureStore()`** que simplifica el proceso de configuración de la store. `configureStore()` envuelve las funciones `createStore()` y `combineReducers()` del núcleo de Redux, y maneja la mayor parte de la configuración de la store automáticamente.

----