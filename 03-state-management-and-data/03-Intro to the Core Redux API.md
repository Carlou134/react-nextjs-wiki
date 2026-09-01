# Intro to the Core Redux API

## What is the Redux API?

En esta lección, aprenderás cómo aplicar los conceptos fundamentales de **Redux** a una aplicación Redux real.

Recuerda, las aplicaciones de Redux se construyen sobre un modelo de **flujo de datos unidireccional** y son gestionadas por el **store (almacén)** :

1.  El **estado (state)** es el conjunto de valores de datos que describe la aplicación. Se utiliza para renderizar la interfaz de usuario (UI).
2.  Los usuarios interactúan con la UI, lo que **despacha acciones (dispatches actions)** al store. Una **acción (action)** es un objeto que expresa un cambio deseado en el estado.
3.  El store genera su siguiente estado utilizando una **función reductora (reducer function)** , que recibe la acción más reciente y el estado actual como entradas.
4.  Finalmente, la UI se **vuelve a renderizar** basándose en el nuevo estado del store, y todo el proceso puede comenzar de nuevo.

Construir una aplicación que siga los principios fundamentales de Redux se puede hacer sin bibliotecas externas. Sin embargo, la biblioteca dedicada de Redux proporciona algunas herramientas muy útiles para manejar los aspectos más comunes de la construcción de una aplicación Redux y ayuda a garantizar que se cumplan los principios fundamentales de Redux.

Esta lección se centrará en la creación de una aplicación Redux básica con el método **createStore()** de la API de Redux y los siguientes métodos relacionados con el store:

*   **store.getState()**
*   **store.dispatch(action)**
*   **store.subscribe(listener)**

**Nota:** El método del store `store.replaceReducer(nextReducer)` es un método avanzado y no se cubrirá en este curso.

----

## Create a Redux Store

Toda aplicación de Redux utiliza una **función reductora (reducer function)** que describe qué acciones pueden actualizar el estado y cómo esas acciones conducen al siguiente estado.

Por ejemplo, supón que quisieras construir una aplicación para un **interruptor de luz (light switch)**. Su reductor podría verse así:

```jsx
const initialState = 'on';
const lightSwitchReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'toggle':
      return state === 'on' ? 'off' : 'on';
    default:
      return state;
  }
}
```

Este reductor maneja un solo tipo de acción, `'toggle'`, y devuelve el siguiente estado del store: `'on'` si había sido `'off'` y viceversa. Si se recibe una acción no reconocida, se devuelve el estado actual del store.

El programador podría ejecutar manualmente el reductor con el estado actual del store y la acción deseada para realizar, de la siguiente manera:

```jsx
let state = 'on';
state = lightSwitchReducer(state, { type: 'toggle' });
console.log(state); // Imprime 'off'
```

Sin embargo, esta es la **responsabilidad principal del store (almacén)** . El store es un objeto que aplica el modelo de flujo de datos unidireccional sobre el que se construye Redux. Contiene el estado actual en su interior, recibe los despachos de acciones, ejecuta el reductor para obtener el siguiente estado y proporciona acceso al estado actual para que la UI se vuelva a renderizar.

Redux exporta una valiosa función auxiliar para crear este objeto store llamada **createStore()**. La función auxiliar `createStore()` tiene un solo argumento: una **función reductora**.

Para crear un store con `lightSwitchReducer`, podrías escribir:

```jsx
import { createStore } from 'redux';

const initialState = 'on';
const lightSwitchReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'toggle':
      return state === 'on' ? 'off' : 'on';
    default:
      return state;
  }
}

const store = createStore(lightSwitchReducer);
```

Por el resto de esta lección, estarás usando Redux para construir una **aplicación de contador simple** en la que el estado es un solo número.

En el editor de código, encontrarás el valor `initialState`, así como `countReducer`, que describe cómo se puede actualizar el estado en respuesta a una acción `'increment'`.

----

## Dispatch Actions to the Store

El objeto `store` devuelto por `createStore()` proporciona una serie de **métodos (methods)** útiles para interactuar con su estado, así como con la función reductora con la que fue creado.

El método más utilizado, **store.dispatch()**, se puede utilizar para **despachar una acción (dispatch an action)** al store, indicando que deseas actualizar el estado. Su único argumento es un objeto de acción (action object), que debe tener una propiedad `type` que describa el cambio de estado deseado.

```jsx
const action = { type: 'actionDescriptor' }; 
store.dispatch(action);
```

Cada vez que se llama a `store.dispatch()` con un objeto de acción, la función reductora del store se ejecutará con el mismo objeto de acción. Suponiendo que el `action.type` es reconocido por el reductor, el estado se actualizará y se devolverá.

Veamos cómo funciona esto en la aplicación del interruptor de luz del ejercicio anterior:

```jsx
import { createStore } from 'redux';

const initialState = 'on';
const lightSwitchReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'toggle':
      return state === 'on' ? 'off' : 'on';
    default:
      return state;
  }
}

const store = createStore(lightSwitchReducer);

console.log(store.getState()); // Imprime 'on'

store.dispatch({ type: 'toggle' }); 
console.log(store.getState()); // Imprime 'off'

store.dispatch({ type: 'toggle' });
console.log(store.getState()); // Imprime 'on'
```

En este ejemplo, también puedes ver otro método del store, **store.getState()**, que devuelve el valor actual del estado del store. Imprimir su valor entre cada acción despachada nos permite ver cómo cambia el estado del store.

Internamente, cuando el store ejecuta su reductor, utiliza `store.getState()` como argumento `state`. Aunque no lo verás, puedes imaginar que cuando se despacha una acción como esta...

```jsx
store.dispatch({ type: 'toggle' });
```

...el store llama al reductor de esta manera:

```jsx
lightSwitchReducer(store.getState(), { type: 'toggle' });
```

----

## Action Creators

Como viste en el ejercicio anterior, es probable que despaches acciones del mismo tipo múltiples veces o desde múltiples lugares. Escribir todo el objeto de acción puede ser tedioso y crea oportunidades para cometer un error.

Por ejemplo, en la aplicación del interruptor de luz, cuyo reductor acepta acciones `'toggle'` para encender o apagar la luz, podrías escribir:

```jsx
store.dispatch({Type:'toggle'});
store.dispatch({type:'toggel'});
store.dispatch({typo:'toggle'});
```

¿Detectaste los errores?

En la mayoría de las aplicaciones Redux, se utilizan **action creators (creadores de acciones)** para reducir esta repetición y proporcionar consistencia. Un action creator es simplemente una **función que devuelve un objeto de acción** con una propiedad `type`. Típicamente se les llama y se pasan directamente al método `store.dispatch()`, lo que resulta en menos errores y una declaración de despacho más fácil de leer.

El código anterior podría reescribirse usando un action creator llamado `toggle()` de la siguiente manera:

```jsx
const toggle = () => {
  return { type: "toggle" };
}
store.dispatch(toggle()); // Cambia la luz a 'off'
store.dispatch(toggle()); // Cambia la luz de vuelta a 'on'
store.dispatch(toggle()); // Cambia la luz de vuelta a 'off'
```

Aunque no son necesarios en una aplicación Redux, los action creators nos ahorran el tiempo necesario para escribir todo el objeto de acción, reducen las posibilidades de cometer un error tipográfico y mejoran la legibilidad de nuestra aplicación.

A menudo, incluso antes de escribir el reductor de una aplicación, los programadores de Redux escribirán action creators como una forma de planificar qué acciones estarán disponibles para despachar al store.

-----

## Respond to State Changes

En una aplicación web típica, las interacciones del usuario que desencadenan eventos del DOM ("click", "keydown", etc.) pueden ser escuchadas y respondidas utilizando un **event listener (escuchador de eventos)**.

De manera similar, en Redux, las acciones despachadas al store pueden ser escuchadas y respondidas utilizando el método **store.subscribe()**. Este método acepta un argumento: una **función**, a menudo llamada **listener (escuchador)** , que se ejecuta en respuesta a cambios en el estado del store.

```jsx
const reactToChange = () => console.log('¡cambio detectado!');
store.subscribe(reactToChange);
```

En este ejemplo, cada vez que se despacha una acción al store y ocurre un cambio en el estado, el listener suscrito, `reactToChange()`, se ejecutará.

A veces es útil detener la respuesta del listener a los cambios en el store, por lo que `store.subscribe()` devuelve una función **unsubscribe (cancelar suscripción)**.

Podemos ver esto en acción en la aplicación del interruptor de luz:

```jsx
// lightSwitchReducer(), toggle(), y store omitidos...

const reactToChange = () => {
  console.log(`¡La luz se cambió a ${store.getState()}!`);
}
const unsubscribe = store.subscribe(reactToChange);

store.dispatch(toggle());
// Se llama a reactToChange(), imprimiendo:
// '¡La luz se cambió a off!'

store.dispatch(toggle());
// Se llama a reactToChange(), imprimiendo:
// '¡La luz se cambió a on!'

unsubscribe(); 
// reactToChange() ahora está desuscrito

store.dispatch(toggle());
// ¡no hay declaración de impresión!

console.log(store.getState()); // Imprime 'off'
```

En este ejemplo:

*   La función listener `reactToChange()` está suscrita al store.
*   Cada vez que se despacha una acción, se llama a `reactToChange()` e imprime el valor actual del interruptor de luz. Es común que los **callbacks** suscritos al store utilicen `store.getState()` dentro de ellos.
*   Después de las dos primeras acciones despachadas, se llama a `unsubscribe()`, lo que provoca que `reactToChange()` ya no se ejecute en respuesta a despachos posteriores realizados al store.

**Nota:** No siempre es obligatorio usar la función `unsubscribe()` devuelta por `store.subscribe()`, aunque es útil saber que existe.

Ahora, echa un vistazo a `main.js` en el editor de código. Verás que se han despachado algunas acciones al store de la aplicación de contador. Supongamos que quisieras imprimir el valor actual de `store.getState()` cada vez que el estado cambie. Si bien podrías escribir algo como esto:

```jsx
store.dispatch(decrement());
console.log(`El conteo es ${store.getState()}`);
store.dispatch(increment());
console.log(`El conteo es ${store.getState()}`);
store.dispatch(increment());
console.log(`El conteo es ${store.getState()}`);
```

...sabemos que este enfoque es repetitivo. En su lugar, puedes **suscribir un listener de cambio** para imprimir el estado actual en respuesta a los cambios de estado automáticamente.

-----

## Connect the Redux Store to a UI

Hasta ahora, has construido una aplicación de contador funcional usando Redux que carece de una interfaz de usuario (UI) adecuada. ¡Cambiemos eso!

**Redux y React se pueden usar juntos** para crear una aplicación altamente interactiva. Aunque estaremos usando React, Redux no se limita solo a React; puede ser usado dentro del contexto de cualquier framework de UI. Sin embargo, Redux se combina más comúnmente con React. Esto tiene sentido, considerando que Redux nació como un proyecto independiente de Dan Abramov y Andrew Clark, inspirado explícitamente en el patrón **Flux** que Facebook había popularizado para gestionar el estado de aplicaciones React — de ahí el parentesco conceptual entre ambas librerías, aunque Redux nunca fue un proyecto desarrollado por Facebook.

La UI para nuestra aplicación de contador debería mostrar el número de conteo actual y permitir al usuario incrementar o decrementar este valor usando los botones proporcionados. Echa un vistazo a la ventana del navegador web conectada, y podrás ver que los elementos para dicha interfaz están presentes, pero **aún no han sido conectados al store de Redux**.

Conectar un store de Redux con cualquier UI requiere algunos pasos consistentes, independientemente de cómo se implemente la UI:

1.  Crear un store de Redux.
2.  Renderizar el estado inicial de la aplicación.
3.  Suscribirse a las actualizaciones. Dentro del callback de suscripción:
    *   Obtener el estado actual del store.
    *   Seleccionar los datos necesarios para esta pieza de UI.
    *   Actualizar la UI con los datos.
4.  Responder a los eventos de la UI despachando acciones de Redux.

Estos mismos pasos se siguen al construir una interfaz usando React, Angular o jQuery. Para este ejercicio, hemos configurado una aplicación React simple. Nuestro enfoque será actualizar una UI usando un store de Redux, por lo que no cubriremos algunas de las interacciones entre React y Redux en este ejercicio (profundizaremos en el próximo ejercicio).

Ahora, abre `store.js`, donde encontrarás las piezas de código de Redux que has construido a lo largo de esta lección: los **action creators** `increment()` y `decrement()`, el **reducer** `countReducer`, y el **store** que lo une todo.

En `App.js`, notarás:

*   Hemos importado los action creators, `increment` y `decrement`.
*   Un componente `App` que espera dos props, `state` y `dispatch`, que se pasarán desde el store.
*   El componente `App` renderiza un elemento `<p>` y dos elementos `<button>`. También contiene un par de **manejadores de clic (click handlers)** : `incrementorClicked` y `decrementorClicked`.

Finalmente, en `index.js`:

*   El store es importado.
*   Una función `render` está definida y llamada.

Usar el componente `App` y la función `render` nos permitirá conectar el store de Redux a la UI. Comencemos.

----

## Review

¡Felicitaciones! Pudiste aplicar los conceptos fundamentales del framework Redux implementando una aplicación usando la biblioteca Redux.

Al completar esta lección, ahora eres capaz de:

*   **Importar** la función auxiliar `createStore()` desde la biblioteca `'redux'`.
*   **Crear un objeto store** que contenga todo el estado de tu aplicación Redux usando `createStore()`.
*   **Obtener el estado actual** del store usando `store.getState()`.
*   **Despachar acciones** al store usando `store.dispatch(action)`.
*   **Crear action creators** para reducir la creación repetitiva de objetos de acción.
*   **Registrar una función listener de cambio** para responder a cambios en el store usando `store.subscribe(listener)`.
*   **Reconocer el patrón** para conectar Redux a cualquier interfaz de usuario.

A lo largo de esta lección, puede que hayas pensado que Redux añade mucha complejidad innecesaria a estas aplicaciones simples. Implementamos Redux de una manera muy básica, lo cual es útil para aprender, pero no es cómo se hace en el mundo real.

**Redux brilla cuando se usa en aplicaciones con muchas características y una gran cantidad de datos**, donde tener un store centralizado para mantenerlo todo organizado es ventajoso. En la próxima lección, aprenderás cómo construir y organizar aplicaciones Redux con **estado complejo**.

----