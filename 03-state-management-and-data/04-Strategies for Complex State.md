# Strategies for Complex State

## Introduction to Strategies for Complex State

En la lección anterior, construiste una aplicación de contador simple cuyo estado del store era solo un número. Aunque la aplicación de contador ilustra cómo Redux puede gestionar el estado de una aplicación, no es un gran ejemplo de una aplicación que necesite Redux.

**Redux realmente brilla cuando se usa en aplicaciones con muchas características y una gran cantidad de datos**, donde tener un store centralizado para mantenerlo todo organizado es ventajoso. En esta lección, aprenderás estrategias para gestionar una aplicación con un **estado de store más complejo** y, en el proceso, comenzarás a construir una aplicación que crecerá a lo largo del resto de este curso.

En el navegador, puedes ver el producto final. Esta aplicación, a la que nos referiremos como la "**App de Recetas**", hace lo siguiente:

*   Muestra un conjunto de **recetas** que se obtienen de una base de datos.
*   Permite al usuario **añadir/eliminar** sus recetas favoritas de una lista separada.
*   Permite al usuario introducir un **término de búsqueda** para filtrar las recetas visibles.

Ahora, imagina que trabajas para la empresa de desarrollo de software cuyo producto principal es esta aplicación de Recetas. El gerente de producto ha determinado las características y funcionalidades deseadas, el diseñador gráfico ha definido su estilo y el ingeniero de React ha creado sus componentes. ¡Ahora te toca a ti, como **Ingeniero de Redux**, diseñar el sistema de gestión de estado que pueda unirlo todo!

En realidad, el ingeniero de Front-End implementaría tanto React como Redux.

Antes de continuar, asegúrate de estar familiarizado con los siguientes términos y conceptos relacionados con React y Redux:

### React
*   Cómo crear componentes
*   Cómo renderizar componentes usando `ReactDOM.render()`
*   Cómo anidar componentes y pasar datos a través de **props**

### Redux
*   Modelo de flujo de datos unidireccional: **Estado → Vista → Acciones → Estado → Vista ...**
*   Cómo crear una **función reductora (reducer)** : `(state, action) => nextState`
*   Cómo escribir **objetos de acción (action objects)** y **action creators**
*   Cómo crear un store usando `createStore()`
*   Cómo usar los **métodos del store** `getState()`, `dispatch()` y `subscribe()`

**Nota para los estudiantes:** La naturaleza ligeramente verbosa de Redux significa que los ejemplos en este curso pueden ser bastante extensos. Se recomienda que expandas la sección "Aprender" mientras lees los materiales de esta lección. Puedes hacerlo haciendo clic y arrastrando la línea divisoria que separa la sección "Aprender" del editor de código.

----

## Slices

Redux es más adecuado para aplicaciones complejas con muchas características, cada una de las cuales tiene algunos datos relacionados con el estado que deben gestionarse. En estos casos, los **objetos (objects)** son el tipo de datos preferido para representar el estado completo del store.

Por ejemplo, considera una aplicación de tareas (todo app) que permite a un usuario:

*   Añadir a una lista de tareas.
*   Marcar tareas individuales como completadas o incompletas.
*   Aplicar un filtro para mostrar solo las tareas completadas, solo las incompletas, o todas las tareas.

Después de añadir algunas tareas y establecer el filtro para mostrar tareas incompletas, el estado podría verse así:

```jsx
state = {
  todos: [
    {
      id: 0, 
      text: 'Completar el curso de Redux', 
      isCompleted: false
    },
    {
      id: 1, 
      text: 'Construir una app de contador', 
      isCompleted: true
    },
  ],
  visibilityFilter: 'SHOW_INCOMPLETE'
};
```

En una aplicación Redux, las propiedades de estado de nivel superior, `state.todos` y `state.visibilityFilter`, se conocen como **rebanadas (slices)**. Cada slice representa típicamente una característica diferente de la aplicación completa. Observa que un slice puede ser cualquier valor de datos, como un array de objetos (`state.todos`) o simplemente una cadena (`state.visibilityFilter`).

Como **buena práctica**, la mayoría de las aplicaciones Redux comienzan con un **initialState** que permite al programador hacer dos cosas clave:

1.  **Planificar** la estructura general del estado.
2.  Proporcionar un **valor de estado inicial** a la función reductora.

Para la aplicación de tareas, esto podría verse así:

```jsx
const initialState = {
  todos: [],
  visibilityFilter: 'SHOW_ALL'
};

const todosReducer = (state = initialState, action) => {
  // el resto de la lógica de todosReducer se omite
};
```

La aplicación de Recetas tendrá los siguientes **tres slices**:

*   **`allRecipes`**: un array de todos los objetos de recetas.
*   **`favoriteRecipes`**: un array de objetos de recetas elegidos por el usuario de `state.allRecipes`.
*   **`searchTerm`**: una cadena que filtra qué recetas se muestran.

Un ejemplo del estado del store podría verse así:

```jsx
state = {
  allRecipes: [
    {id: 0, name: 'Jjampong', img: 'img/jjampong.png' },
    {id: 2, name: 'Cheeseburguesa', img: 'img/cheeseburger.png' },
    //… más recetas omitidas
  ],
  favoriteRecipes: [
    {id: 1, name: 'Doro Wat', img: 'img/doro-wat.png' },
  ],
  searchTerm: 'Doro'
};
```

Observa que cada receta se representa como un **objeto** con una propiedad `id`, `name` e `img`.

Ahora que sabes cómo se ve la estructura del estado, el primer paso es crear un objeto **`initialState`**.

----

## Actions and Payloads For Complex State

La estructura de `initialState` ha sido definida, y sabes que el estado de esta aplicación tiene 3 slices: `allRecipes`, `favoriteRecipes` y `searchTerm`. Ahora, puedes empezar a pensar en cómo el usuario desencadenará cambios en estos slices de estado a través de **acciones (actions)**.

Recuerda, las acciones en Redux están representadas por objetos planos de JavaScript que tienen una propiedad `type` y se despachan al store usando el método `store.dispatch()`.

Cuando un estado de aplicación tiene múltiples slices, las acciones individuales típicamente cambian solo un slice a la vez. Por lo tanto, se recomienda que el `type` de cada acción siga el patrón **'sliceName/actionDescriptor'**, para aclarar qué slice de estado debe actualizarse.

Por ejemplo, en una aplicación de tareas con un slice `state.todos`, el tipo de acción para añadir una nueva tarea podría ser `'todos/addTodo'`.

Para la aplicación de Recetas, ¿qué crees que podrían ser algunas de las cadenas de tipo de acción? ¿Qué interacciones del usuario podrían desencadenar su despacho?

Aquí están las acciones que usarás:

*   **`'allRecipes/loadData'`**: Esta acción se despachará para obtener los datos necesarios de una API justo cuando la aplicación se inicia.
*   **`'favoriteRecipes/addRecipe'`**: Esta acción se despachará cada vez que el usuario haga clic en el icono ❤️ de una receta del conjunto completo de recetas.
*   **`'favoriteRecipes/removeRecipe'`**: Esta acción se despachará cada vez que el usuario haga clic en el icono 💔 de una receta de su lista de favoritas.
*   **`'searchTerm/setSearchTerm'`**: Esta acción se despachará cada vez que el usuario cambie el texto del campo de entrada de búsqueda para filtrar el conjunto completo de recetas.
*   **`'searchTerm/clearSearchTerm'`**: Esta acción se despachará cada vez que el usuario haga clic en el botón "X" junto al campo de entrada de búsqueda.

También es importante considerar cuáles de estas acciones tendrán un **payload (carga útil)** : datos adicionales pasados al reductor para llevar a cabo el cambio de estado deseado. Por ejemplo, considera las acciones para el slice `searchTerm`:

```jsx
store.dispatch({ 
  type: 'searchTerm/setSearchTerm', 
  payload: 'Espaguetis' 
});
// El estado resultante: { ..., searchTerm: 'Espaguetis' }

store.dispatch({ 
  type: 'searchTerm/clearSearchTerm' 
});
// El estado resultante: { ..., searchTerm: '' }
```

*   Cuando el usuario escribe un término de búsqueda, esos datos deben enviarse al store para que los componentes de React sepan qué recetas renderizar y cuáles ocultar.
*   Cuando el usuario limpia el campo de búsqueda, no es necesario enviar datos adicionales porque el store puede simplemente establecer el término de búsqueda como una cadena vacía de nuevo.

Una vez que tengas una idea clara de los tipos de acciones que se despacharán en tu aplicación, cuándo se despacharán y qué datos de payload llevarán, el siguiente paso es crear **action creators**.

Recuerda, los action creators son **funciones que devuelven un objeto de acción formateado**.

Los action creators permiten a los programadores de Redux reutilizar las estructuras de los objetos de acción sin tener que escribirlos a mano, y mejoran la legibilidad de su código, particularmente cuando se trata de payloads voluminosos.

Echa un vistazo a `store.js`, donde encontrarás que los action creators para las dos acciones anteriores han sido definidos para ti. Tu trabajo es crear los tres restantes: **`loadData()`**, **`addRecipe()`** y **`removeRecipe()`**.

----

## Immutable Updates & Complex State

Ahora que has definido qué cambios pueden ocurrir en el estado de tu aplicación, necesitas un **reducer** para ejecutar esos cambios.

Recuerda, la función reductora del store se llama cada vez que se despacha una acción. Se le pasa la acción y el estado actual como argumentos y devuelve el **siguiente estado** del store.

La **segunda regla de los reducers** establece que cuando el reducer está actualizando el estado, debe hacer una **copia** y devolver la copia en lugar de mutar directamente el estado entrante. Cuando el estado es un tipo de datos mutable, como un array u objeto, esto se hace típicamente usando el **operador de propagación (spread operator, ...)** .

A continuación, el `todosReducer` para una aplicación de tareas demuestra esto en acción:

```jsx
const initialState = {
  filter: 'SHOW_INCOMPLETE',
  todos: [
    { id: 0, text: 'aprender redux', completed: false },
    { id: 1, text: 'construir una app redux', completed: true },
    { id: 2, text: 'hacer un baile', completed: false },
  ]
};

const todosReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'filter/setFilter':
      return {
        ...state,
        filter: action.payload
      };
    case 'todos/addTodo': 
      return {
        ...state,
        todos: [...state.todos, action.payload]
      };
    case 'todos/toggleTodo':
      return {
        ...state,
        todos: state.todos.map(todo => {
          return (todo.id === action.payload.id) ? 
            { ...todo, completed: !todo.completed } : 
            todo;
        })
      };
    default:
      return state;
  }
};
```

*   El `todosReducer` usa el `initialState` como el valor de estado por defecto.
*   Cuando se recibe una acción `'filter/setFilter'`, propaga el contenido del estado anterior (`...state`) en un nuevo objeto antes de actualizar la propiedad `filter` con el nuevo filtro de `action.payload`.
*   Cuando se recibe una acción `'todos/addTodo'`, hace lo mismo, excepto que esta vez, dado que `state.todos` es un array mutable, su contenido también se propaga en un nuevo array, con el nuevo todo de `action.payload` añadido al final.
*   Cuando se recibe una acción `'todos/toggleTodo'`, usa el método **.map()** para crear una copia del array `state.todos`. Además, el todo que se está alternando se encuentra usando `action.payload.id`, y se propaga en un nuevo objeto y se actualiza.

Debe aclararse que el método `state.todos.map()` solo hace una copia "**superficial (shallow)**" del array, lo que significa que los objetos dentro comparten las mismas referencias que los originales. Por lo tanto, las mutaciones a los objetos dentro de la copia afectarán a los objetos dentro del original. Por ahora, podemos arreglárnoslas con esta solución, pero aprenderás cómo evitar este problema en una lección posterior sobre **Redux Toolkit**.

----

## Reducer Composition

En el ejercicio anterior, viste cómo un solo reducer era capaz de manejar la lógica para actualizar cada slice del estado del store. Aunque este enfoque funciona para estos ejemplos relativamente pequeños, a medida que el estado de la aplicación se vuelve cada vez más complejo, gestionarlo todo con un solo reducer se volverá **poco práctico**.

La solución es seguir un patrón llamado **composición de reducers (reducer composition)**. En este patrón, los **slice reducers** individuales son responsables de actualizar solo un slice del estado de la aplicación, y sus resultados se recombinan mediante un **rootReducer** para formar un solo objeto de estado.

```jsx
// Maneja solo `state.todos`.
const initialTodos = [
  { id: 0, text: 'aprender redux', completed: false },
  { id: 1, text: 'construir una app redux', completed: true },
  { id: 2, text: 'hacer un baile', completed: false },
];
const todosReducer = (todos = initialTodos, action) => {
  switch (action.type) {
    case 'todos/addTodo': 
      return [...todos, action.payload];
    case 'todos/toggleTodo':
      return todos.map(todo => {
        return (todo.id === action.payload.id) ? 
          { ...todo, completed: !todo.completed } : 
          {...todo};
      });
    default:
      return todos;
  }
};

// Maneja solo `state.filter`
const initialFilter = 'SHOW_INCOMPLETE';
const filterReducer = (filter = initialFilter, action) => {
  switch (action.type) {
    case 'filter/setFilter':
      return action.payload;
    default:
      return filter;
  }
};

const rootReducer = (state = {}, action) => {
  const nextState = {
    todos: todosReducer(state.todos, action),
    filter: filterReducer(state.filter, action)
  };
  return nextState;
};

const store = createStore(rootReducer);
```

En el patrón de composición de reducers, cuando una acción se despacha al store:

1.  El **rootReducer** llama a cada slice reducer, independientemente del `action.type`, con la acción entrante y el slice apropiado del estado como argumentos.
2.  Los slice reducers determinan individualmente si necesitan actualizar su slice de estado, o simplemente devuelven su slice de estado sin cambios.
3.  El rootReducer **reensambla** los valores de slice actualizados en un nuevo objeto de estado.

Una ventaja importante de este enfoque es que cada slice reducer solo recibe **su slice** de todo el estado de la aplicación. Por lo tanto, cada slice reducer solo necesita actualizar inmutablemente su propio slice y no le importan los demás. Esto elimina el problema de copiar objetos de estado potencialmente profundamente anidados.

Echa un vistazo a `store.js`, donde encontrarás que el reducer para la app de Recetas que escribiste en el último ejercicio (que se puede encontrar en `reducer-old.js`) ha sido parcialmente reescrito para seguir el patrón de composición de reducers:

*   El objeto `initialState` ha sido reemplazado por **variables individuales `initialSliceName`** que se utilizan como valores predeterminados para el slice de estado de cada slice reducer. Esta es otra característica común del patrón de composición de reducers.
*   Los slice reducers `allRecipesReducer` y `searchTermReducer` han sido creados para ti. Observa que cada uno tiene un caso `default`.
*   Ambos slice reducers son llamados dentro del `rootReducer` para actualizar sus respectivos slices de estado.

¡Todo lo que queda es **completar `favoriteRecipesReducer()`** e **incluirlo en `rootReducer()`**!

----

## combineReducers

En el patrón de composición de reducers, el `rootReducer` realiza los mismos pasos para cada slice reducer:

1.  Llama al slice reducer con su slice de estado y la acción como argumentos.
2.  Almacena el slice de estado devuelto en un nuevo objeto que es finalmente devuelto por `rootReducer()`.

```jsx
import { createStore } from 'redux';

// todosReducer y filterReducer omitidos

const rootReducer = (state = {}, action) => {
  const nextState = {
    todos: todosReducer(state.todos, action),
    filter: filterReducer(state.filter, action)
  };
  return nextState;
};

const store = createStore(rootReducer);
```

El paquete de Redux ayuda a facilitar este patrón proporcionando una función de utilidad llamada **combineReducers()**, que maneja este código repetitivo (boilerplate) por nosotros:

```jsx
import { createStore, combineReducers } from 'redux';

// todosReducer y filterReducer omitidos.

const reducers = {
    todos: todosReducer,
    filter: filterReducer
};
const rootReducer = combineReducers(reducers);
const store = createStore(rootReducer);
```

Analicemos este código:

*   El objeto **`reducers`** contiene los slice reducers para la aplicación. Las **claves** del objeto corresponden al nombre del slice que está siendo gestionado por el valor del reducer.
*   La función **`combineReducers()`** acepta este objeto `reducers` y devuelve una función `rootReducer`.
*   El `rootReducer` devuelto se pasa a `createStore()` para crear un objeto store.

Al igual que antes, cuando se despacha una acción al store, se ejecuta `rootReducer()`, que luego llama a cada slice reducer, pasando la acción y el slice de estado apropiado.

Las últimas 6 líneas de este ejemplo se pueden reescribir en línea:

```jsx
const store = createStore(combineReducers({
    todos: todosReducer,
    filter: filterReducer
}));
```

Echa un vistazo a `store.js`, donde encontrarás los slice reducers que creaste en el ejercicio anterior. Sin embargo, ahora falta `rootReducer()`. En lugar de crear esta función manualmente, usarás **`combineReducers()`**.

-----

## A File Structure for Redux

En este punto, puede que hayas comenzado a pensar que `store.js` se está volviendo bastante largo, ¡y la aplicación de Recetas solo tiene tres slices! Imagina intentar encajar la lógica de una aplicación con una docena o más de slices en un solo archivo. Eso no sería divertido.

En su lugar, es más común, y una mejor práctica, dividir una aplicación Redux usando el **patrón Redux Ducks**, de la siguiente manera:

```
src/
|-- index.js
|-- app/
    |-- store.js
|-- features/
    |-- featureA/
        |-- featureASlice.js
    |-- featureB/
        |-- featureBSlice.js
```

Como puedes ver en tu espacio de trabajo de codificación, esta estructura de archivos ya ha sido configurada para ti.

Toda la lógica de Redux vive dentro del directorio de nivel superior llamado `src/`. Contiene:

*   El punto de entrada para toda la aplicación, **`index.js`** (volveremos a este archivo en el próximo ejercicio).
*   Los subdirectorios **`app/`** y **`features/`**.

El directorio `src/app/` tiene solo un archivo (por ahora), **`store.js`**. Como antes, el propósito final de este archivo es crear el `rootReducer` y el store de Redux. Ahora, sin embargo, notarás que ¡el archivo está vacío! Entonces, ¿dónde están los reducers y los action creators?

El directorio `src/features/`, y sus propios subdirectorios `src/features/featureX/`, contienen **todo el código relacionado con cada slice individual** del estado del store. Por ejemplo, para el slice `state.favoriteRecipes`, su slice reducer y action creators se pueden encontrar en el archivo llamado `src/features/favoriteRecipes/favoriteRecipesSlice.js`.

Por suerte para ti, nos encargamos de gran parte del trabajo tedioso involucrado en la refactorización de este código. Además de crear nuevas carpetas y archivos, y copiar el código relevante, esta refactorización implicó **exportar** cada uno de los slice reducers y action creators, para que pudieran ser **importados** de nuevo en `store.js`.

¡Y ahí es donde entras tú!

-----

## Passing Store Data Through the Top-Level React Component

La estructura de archivos que ayudaste a implementar en el último ejercicio funciona muy bien cuando agregamos componentes de React. Echa un vistazo a la carpeta `src` en tu espacio de trabajo y encontrarás la siguiente estructura de archivos (los archivos nuevos tienen un signo (+) junto a su nombre):

```
src/
|-- index.js
|-- app/
    |-- App.js (+)
    |-- store.js
|-- components/
    |-- FavoriteButton.js (+)
    |-- Recipe.js (+)
|-- features/
    |-- allRecipes/
        |-- AllRecipes.js (+)
        |-- allRecipesSlice.js
    |-- favoriteRecipes/
        |-- FavoriteRecipes.js (+)
        |-- favoriteRecipesSlice.js
    |-- searchTerm/
        |-- SearchTerm.js (+)
        |-- searchTermSlice.js
```

Si observas la estructura de archivos real en tu editor de código, puedes notar algunos archivos/directorios no mencionados en la estructura anterior. El directorio `test/` y el archivo `index.compiled.js` se utilizan para probar tu código en Codecademy. Puedes ignorarlos.

Los nuevos componentes son:

*   **`<App />`**: El componente de nivel superior para toda la aplicación.
*   **`<AllRecipes />`**: El componente para renderizar las recetas cargadas desde la "base de datos".
*   **`<FavoriteRecipes />`**: El componente para renderizar las recetas favoritas del usuario.
*   **`<SearchTerm />`**: El componente para renderizar la barra de búsqueda que filtra las recetas visibles.
*   **`<Recipe />`** y **`<FavoriteButton />`**: Componentes genéricos utilizados por `<AllRecipes />` y `<FavoriteRecipes />`.

Aparte de los componentes genéricos, cada archivo de componente React relacionado con una característica se encuentra en el mismo directorio que el archivo **slice** que gestiona los datos renderizados por ese componente. Por ejemplo, `FavoriteRecipes.js` y `favoriteRecipesSlice.js` están ambos en el directorio `src/features/favoriteRecipes/`.

Abre el archivo `src/app/App.js` donde se almacena el componente de nivel superior, `<App />`. Como en la mayoría de las aplicaciones React, este componente de nivel superior renderizará cada componente de característica y pasará cualquier dato necesario a esos componentes como valores de **prop**. En las aplicaciones Redux, los datos pasados a cada componente de característica incluyen:

*   La **porción (slice) del estado de la store** que se va a renderizar. Por ejemplo, la porción `state.searchTerm` se pasa al componente `<SearchTerm />`.
*   El método **store.dispatch** para desencadenar cambios de estado a través de las interacciones del usuario dentro del componente. Por ejemplo, el componente `<SearchTerm />` necesitará despachar acciones `setSearchTerm()`.

Esta distribución del método `store.dispatch` y las porciones de estado a todos los componentes de características, a través del componente `<App />`, comienza en el archivo `index.js`. Abre el archivo `src/index.js` donde verás algún código React estándar para renderizar el componente `<App />` de nivel superior. ¡Notarás que la **store** falta y que el componente `<App />` no está recibiendo ninguna prop!

----

## Using Store Data Within Feature Components

Al final del último ejercicio, pudiste pasar el **estado actual de la store** y su método **store.dispatch** al componente de nivel superior, `<App />`. Esto permitió que el componente `<App />` distribuyera el método `dispatch` y las porciones (slices) del estado de la store a cada componente de característica.

Parece que has terminado, ¿verdad? No exactamente. Intenta añadir una receta favorita y ¡verás que simplemente desaparece! Echa un vistazo más de cerca a `App.js` y notarás que falta el componente `<FavoriteRecipes />`. Luego, abre `FavoriteRecipes.js` y verás que también está incompleto. Arreglemos eso.

Conectar un componente de característica a una aplicación Redux implica los siguientes pasos:

1.  **Importar** los componentes de característica de React en el archivo de nivel superior `App.js`.
2.  **Renderizar** cada componente de característica y pasarle la **porción de estado** y el método **dispatch** como props.
3.  Dentro de cada componente de característica:
    *   **Extraer** la porción de estado y `dispatch` de las props.
    *   **Renderizar** el componente utilizando los datos de la porción de estado.
    *   **Importar** cualquier **creador de acciones (action creator)** del archivo slice asociado.
    *   **Despachar acciones** en respuesta a las entradas del usuario dentro del componente.

Este proceso no es diferente de cómo implementaste una aplicación React + Redux en el pasado. Sin embargo, ahora debes considerar que las porciones del estado de la store y el método `dispatch` deben pasarse a través de **props**.

----

## Review

¡Felicitaciones! Has aprendido a construir y organizar una aplicación React Redux con múltiples porciones (slices) de estado.

Resumamos lo que has aprendido en esta lección:

*   La propiedad **`action.payload`** se utiliza para contener datos adicionales que el reducer podría necesitar para llevar a cabo una acción determinada. El nombre `payload` es simplemente una convención, ¡y su valor puede ser cualquier cosa!
*   La **sintaxis de propagación (spread syntax, `...`)** y métodos de array como **`.map()`** , **`.slice()`** y **`.filter()`** se pueden usar para actualizar **inmutably (inmutably)** el estado de una aplicación compleja.
*   La **composición de reducers (reducer composition)** es un patrón de diseño para gestionar una store de Redux con múltiples porciones.
*   El **reducer raíz (root reducer)** delega acciones a los **reducers de porción (slice reducers)** que son responsables de actualizar solo su porción asignada del estado de la store. Luego, el reducer raíz vuelve a ensamblar las porciones en un nuevo objeto de estado.
*   **`combineReducers()`** es un método proporcionado por la librería `redux` que acepta una colección de funciones reducer y devuelve un **rootReducer** que implementa el patrón de composición de reducers.
*   En una aplicación Redux, los reducers de porción a menudo se escriben en archivos separados. Este patrón se conoce como **Redux Ducks**.

En la aplicación de Recetas que completaste en el ejercicio final, la **store** se pasa desde el punto de entrada (`index.js`) a través del componente principal `<App />` como una **prop**. El componente `<App />` puede entonces pasar las porciones del estado de la store a sus subcomponentes.

Este enfoque se llama **"prop drilling"** o **"prop threading"** porque las props se "enhebran" a través del componente de nivel superior para poder llevarlas a los componentes de presentación. Esto no es ideal, considerando que el componente de nivel superior no hace uso de esas props. En la próxima lección, aprenderás cómo puedes usar **Redux Toolkit** para evitar el "prop threading" y más trucos para construir aplicaciones React Redux robustas.

-----