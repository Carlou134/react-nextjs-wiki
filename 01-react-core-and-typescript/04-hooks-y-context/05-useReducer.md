# useReducer

## ¿Por Qué Usar useReducer?

Hasta ahora hemos gestionado el estado de nuestros componentes con el **State Hook** (`useState()`). Funciona muy bien mientras el estado es simple: un booleano, una cadena, un número, o incluso un objeto con pocas propiedades relacionadas entre sí.

Sin embargo, a medida que la lógica de actualización de un componente crece —varias acciones posibles, múltiples piezas de estado que deben cambiar de forma coordinada, actualizaciones que dependen fuertemente del estado anterior—, mantener todo con `useState()` empieza a ensuciar el código. Terminamos escribiendo varios manejadores de eventos parecidos entre sí, cada uno repitiendo su propia lógica de copiar el estado anterior con el operador spread, y resulta fácil olvidar algún campo o introducir inconsistencias entre actualizaciones que deberían mantenerse sincronizadas.

React ofrece el Hook **useReducer()** para estos casos. En lugar de modificar el estado directamente a través de múltiples funciones actualizadoras, `useReducer()` centraliza toda la lógica de actualización en una única función llamada **reducer**, y expone una función `dispatch()` para "anunciar" qué tipo de actualización queremos realizar.

En esta lección aprenderemos:

* Qué problema resuelve `useReducer()` frente a `useState()`.
* La firma de `useReducer()` y el significado de cada uno de sus valores.
* Cómo funciona la función reducer.
* El patrón de `dispatch()` con objetos de acción.
* Un ejemplo completo de un contador construido con `useReducer()`.
* Cuándo conviene preferir `useReducer()` en lugar de `useState()`.

-----

## La Firma de useReducer

El Hook `useReducer()` es una exportación nombrada de la librería de React, igual que `useState()`:

```javascript
import { useReducer } from 'react';
```

Se invoca así:

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

Esta línea tiene tres partes que conviene distinguir con claridad:

1. **`reducer`**: una función que nosotros definimos, con la firma `(state, action) => newState`. Es la única responsable de calcular cómo cambia el estado.
2. **`initialState`**: el valor con el que se inicializa el estado durante el primer renderizado, del mismo modo que el argumento que le pasamos a `useState()`.
3. El valor de retorno es un arreglo de dos elementos que extraemos mediante **desestructuración de arreglos**, siguiendo la misma convención que ya conocemos de `useState()`:
   * **`state`**: el valor actual del estado.
   * **`dispatch`**: una función que usamos para enviar una acción al reducer y así solicitar una actualización de estado.

Al igual que con cualquier otro Hook, `useReducer()` solo puede llamarse en el nivel superior de un componente de función (o de un Hook personalizado), nunca dentro de condicionales ni bucles.

-----

## Cómo Funciona La Función Reducer

El **reducer** es una función **pura**: recibe el estado actual y un objeto de **acción**, y devuelve el siguiente estado. Nunca modifica el estado que recibe; siempre calcula y devuelve un valor nuevo.

Que sea una función pura —sin llamadas a APIs, sin efectos secundarios, sin depender de datos externos que puedan cambiar— es lo que la hace predecible: dado el mismo estado y la misma acción, un reducer siempre produce el mismo resultado. Esta pureza también la hace muy fácil de probar de forma aislada, ya que podemos llamar a `reducer(state, action)` directamente en un test, sin necesidad de renderizar ningún componente.

Un reducer típico usa una sentencia `switch` sobre la propiedad `type` de la acción para decidir qué transformación aplicar:

```javascript
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

El caso `default`, que simplemente devuelve el estado sin modificar, es importante: evita que el reducer produzca un resultado inesperado (como `undefined`) si en algún momento se despacha una acción con un `type` que no reconoce.

-----

## El Patrón de Dispatch

Nunca actualizamos el estado directamente. En su lugar, llamamos a `dispatch()` con un **objeto de acción**, que por convención incluye al menos una propiedad `type` describiendo qué ocurrió:

```javascript
dispatch({ type: 'increment' });
```

Llamar a `dispatch()` no actualiza el estado de forma sincrónica por sí mismo: lo que hace es programar una llamada a `reducer(currentState, action)` y decirle a React que debe volver a renderizar el componente con el valor que ese reducer devuelva, de forma muy similar a como la función actualizadora que devuelve `useState()` programa un nuevo renderizado.

Cuando una acción necesita transportar información adicional además de su tipo, es común incluir una propiedad `payload`:

```javascript
dispatch({ type: 'setCount', payload: 10 });
```

Y el reducer la usa para calcular el nuevo estado:

```javascript
case 'setCount':
  return { count: action.payload };
```

-----

## Ejemplo Completo: Un Contador

Juntemos todas las piezas en un componente `Counter` completo:

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

El flujo completo, cada vez que el usuario hace clic en alguno de los botones, es el siguiente:

1. Se llama a `dispatch()` con el objeto de acción correspondiente (por ejemplo, `{ type: 'increment' }`).
2. React invoca a `reducer()`, pasándole el `state` actual y esa acción.
3. `reducer()` calcula y devuelve el siguiente estado.
4. React actualiza `state` con ese valor y vuelve a renderizar el componente `Counter`.

Nótese que, desde el punto de vista del componente, la lógica de "qué significa incrementar" o "qué significa reiniciar" no vive en el JSX ni en manejadores de eventos dispersos: vive enteramente dentro del reducer, en un único lugar.

-----

## ¿Cuándo Usar useReducer en Lugar de useState?

`useState()` sigue siendo la opción más simple y la que deberíamos usar por defecto para la mayoría de los componentes. `useReducer()` empieza a justificar su complejidad adicional cuando aparecen algunas de estas señales.

La primera es un **formulario con muchos campos relacionados** que deben validarse o transformarse de forma consistente. En lugar de tener un `useState()` por cada campo —o un único objeto de estado actualizado por muchos manejadores casi idénticos—, un reducer con acciones como `updateField`, `resetForm`, `submitStart`, `submitSuccess` o `submitError` mantiene toda esa lógica en un solo lugar, en vez de dispersarla por el componente.

La segunda señal es tener **múltiples sub-valores que deben permanecer coherentes entre sí**. Un ejemplo típico es una especie de máquina de estados que describe si una operación está en `idle`, `loading`, `success` o `error`, junto con los datos o el mensaje de error asociados a cada uno de esos estados. Modelar esto con varios `useState()` independientes abre la puerta a combinaciones inconsistentes (por ejemplo, `loading` en `true` y, al mismo tiempo, un error ya establecido); con un reducer, cada acción decide explícitamente la forma completa del siguiente estado.

La tercera señal es cuando **el siguiente estado depende del anterior de una manera no trivial**, más allá de un simple incremento o de copiar un objeto con el operador spread. Cuanto más elaborada es esa lógica, más beneficio se obtiene al extraerla a una función pura y testeable por separado del componente.

Por último, `useReducer()` combina naturalmente con **Context** para compartir estado más estructurado entre muchos componentes, exponiendo `state` y `dispatch` a través de un Provider en lugar de múltiples funciones actualizadoras sueltas. Ese patrón se explica con más detalle en la sección "Context con useReducer" de la lección de **React Context**.

-----

## Resumen

* `useReducer()` centraliza la lógica de actualización de estado en una función **reducer**, en lugar de repartirla entre múltiples manejadores de eventos.
* Su firma es `const [state, dispatch] = useReducer(reducer, initialState)`.
* El **reducer** es una función pura con la forma `(state, action) => newState`; nunca modifica el estado directamente, siempre devuelve uno nuevo.
* Actualizamos el estado llamando a `dispatch()` con un **objeto de acción**, típicamente con una propiedad `type` y, opcionalmente, un `payload`.
* `useReducer()` conviene especialmente en formularios complejos, estados con múltiples sub-valores que deben mantenerse coherentes, y transiciones de estado no triviales.
* Para la mayoría de los componentes, `useState()` sigue siendo la opción más simple y adecuada; `useReducer()` es una herramienta para cuando esa simplicidad deja de ser suficiente.

-----
