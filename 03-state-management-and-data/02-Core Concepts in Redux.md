# Core Concepts in Redux

## Introduction to Redux

Imagina que estás creando una aplicación de calendario con varias funcionalidades. Una parte de la aplicación muestra eventos; otra permite a los usuarios filtrar qué tipos de **eventos** se muestran; una tercera parte de la aplicación **establece** la zona horaria actual; y una cuarta parte crea nuevos eventos. Para hacer posibles estas funcionalidades, tendrías que gestionar datos compartidos y actualizaciones entre estos componentes. Sin una gestión adecuada, la complejidad de estas tareas puede escalar rápidamente.

Ahí es donde entra **Redux**. Redux es una librería de gestión de estado que sigue un patrón conocido como la **arquitectura Flux**. En Flux y Redux, la información compartida se consolida dentro de un solo objeto en lugar de estar dispersa entre componentes individuales. Los componentes reciben datos para renderizar y pueden solicitar cambios usando **acciones (actions)** , que son eventos desencadenados por interacciones del usuario u otros eventos. El **estado (state)** está disponible en toda la aplicación, y las actualizaciones se manejan de manera predecible, notificando a los componentes cada vez que ocurre un cambio.

Dicho de otra manera, aquí está la descripción de la documentación de Redux: "Los patrones y herramientas proporcionados por Redux facilitan la comprensión de **cuándo, dónde, por qué y cómo** se está actualizando el estado en tu aplicación, y cómo se comportará la lógica de tu aplicación cuando ocurran esos cambios. Redux te guía hacia la escritura de código que es **predecible y comprobable (testeable)** , lo que ayuda a darte confianza de que tu aplicación funcionará como se espera".

Existen herramientas similares como Recoil, MobX y Apollo Client, pero Redux es la herramienta probada y confiable para la gestión de estado en aplicaciones React. Es más popular en la comunidad de desarrolladores y está bien respaldada con documentación y tutoriales en línea.

Esta lección cubrirá los conceptos básicos de Redux: **cómo funciona Redux** y la **terminología básica** utilizada para describir una aplicación Redux. Asume que conoces **funciones**, **arrays** y **objetos** de JavaScript. Si necesitas repasarlos, consulta las unidades correspondientes en nuestro curso "Learn JavaScript".

----

## One-Way Data Flow

En la mayoría de las aplicaciones, hay tres partes:

*   **Estado (State)** – los datos actuales utilizados en la aplicación.
*   **Vista (View)** – la interfaz de usuario mostrada a los usuarios.
*   **Acciones (Actions)** – **eventos** que un usuario puede realizar para cambiar el estado.

El flujo de información sería el siguiente:

1.  El **estado** contiene los datos actuales utilizados por los componentes de la aplicación.
2.  Los componentes de la **vista** muestran esos datos del estado.
3.  Cuando un usuario interactúa con la vista, como hacer clic en un botón, el **estado** se actualizará de alguna manera.
4.  La **vista** se actualiza para mostrar el nuevo estado.

Con React simple, estas tres partes se superponen bastante. Los componentes no solo renderizan la interfaz de usuario, sino que también pueden gestionar su propio estado. Cuando ocurren acciones que pueden cambiar el estado, los componentes necesitan comunicar estos cambios directamente entre sí.

**Redux ayuda a separar el estado, la vista y las acciones** al requerir que el estado sea gestionado por una **única fuente**. Las **solicitudes** para cambiar el estado son enviadas a esta única fuente por los componentes de la vista en forma de una **acción**. Cualquier componente de la vista que se vea afectado por estos cambios es informado por esta única fuente. Al imponer esta estructura, Redux hace que nuestro código sea más **legible**, **confiable** y **mantenible**.

![redux](/Images/One-way-data-flow-v2-transparent-bg.png)

----

## State

El **estado (state)** en una aplicación web representa la información actual que impulsa el comportamiento y la apariencia de la aplicación. Actúa como una **fuente centralizada de datos**, almacenando los detalles esenciales de la aplicación en un momento dado.

Por ejemplo, en una aplicación de calendario, el estado incluiría **eventos** (nombre, fecha, etiqueta), la zona horaria actual y los filtros de visualización. En una aplicación de tareas pendientes (to-do), el estado consistiría en los elementos de la lista (descripción, completado/no completado), el orden existente de los elementos y los filtros de visualización. En un editor de texto, el estado abarcaría el contenido del documento, la configuración de impresión y los **comentarios**.

Las aplicaciones complejas tienen una multitud de estados para realizar un seguimiento, y pasar estados hacia abajo en el árbol de componentes puede volverse tedioso e ineficiente. Redux, como una herramienta valiosa, mejora los frameworks y librerías de JavaScript al ofrecer una solución **consistente y predecible** para la gestión del estado.

Con Redux, el estado puede ser **cualquier tipo de JavaScript**, incluyendo número, string, booleano, array y objeto.

Aquí hay un ejemplo de estado para una aplicación de tareas pendientes:

```jsx
const state = [ 'Imprimir mapa del sendero', 'Empacar bocadillos', 'Cumbre de la montaña' ];
```

Cada pieza de información en este estado (un array en este caso) informaría alguna parte de la interfaz de usuario.

----

## Actions

La mayoría de las aplicaciones bien diseñadas tendrán componentes separados que necesitan **comunicarse y compartir datos**.

Una lista de tareas pendientes podría tener un campo de entrada donde el usuario puede escribir un nuevo elemento. La aplicación podría transferir estos datos desde el campo de entrada, agregarlos a un array de todas las tareas pendientes y renderizarlos como texto en la pantalla. Esta interacción completa se puede definir como una **acción (action)** . Las acciones describen un evento o una acción que ha ocurrido y proporcionan información sobre lo que necesita ser actualizado en el estado de la aplicación. En resumen, **las acciones son cómo Redux gestiona y actualiza el estado**.

En Redux, las acciones se representan como objetos planos de JavaScript. Aquí hay un ejemplo de cómo podría verse esa acción:

```jsx
const action = {
  type: 'todos/addTodo',
  payload: 'Tomar selfies'
};
```

*   Cada acción debe tener una propiedad **`type`** con un valor de **string**. Esto describe la acción.
*   Típicamente, una acción tiene una propiedad **`payload`** con un valor de **objeto**. Esto incluye cualquier información relacionada con la acción. En este caso, el `payload` es el texto de la tarea pendiente.

Cuando se genera una acción y notifica a otras partes de la aplicación, decimos que la acción es **despachada (dispatched)** .

Aquí hay dos ejemplos más de acciones:

*   **"Eliminar todas las tareas pendientes"**. Esto no requiere `payload` porque no se necesita información adicional:

```jsx
const action = {
  type: 'todos/removeAll'
};
```

*   **"Eliminar la tarea 'Empacar bocadillos'"**:

```jsx
const action = {
  type: 'todos/removeTodo',
  payload: 'Empacar bocadillos'
};
```

-----

## Reducers

Hasta ahora, hemos definido el estado de nuestra aplicación y las acciones que representan **solicitudes** para cambiar ese estado, pero no hemos visto cómo se llevan a cabo estos cambios en JavaScript. La respuesta es un **reductor (reducer)** .

Un **reductor**, o función reductora, es una función plana de JavaScript que define cómo el **estado actual** y una **acción** se utilizan en combinación para crear el **nuevo estado**.

Aquí hay un ejemplo de una función reductora para una aplicación de tareas pendientes:

```jsx
const initialState = [ 'Imprimir mapa del sendero', 'Empacar bocadillos', 'Cumbre de la montaña' ];

const todoReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'todos/addTodo': {
      return [ ...state, action.payload];
    }
    case 'todos/removeAll': {
      return [];
    }
    default: {
      return state;
    }
  }
};
```

Hay algunas cosas sobre este reductor que son ciertas para **todos los reductores**:

*   Es una función plana de JavaScript.
*   Define el **siguiente estado** de la aplicación dado un estado actual y una acción específica.
*   Devuelve un estado inicial predeterminado si no se proporciona ninguna acción.
*   Devuelve el estado actual si la acción **no es reconocida**.

Hay dos sintaxis de JavaScript intermedias utilizadas aquí:

*   Usamos el signo igual `=` para proporcionar un **valor por defecto** para el parámetro `state`.
*   Usamos el **operador de propagación (spread operator)** `...` para copiar el estado actual y cualquier valor cambiado en un **nuevo objeto**, no en el argumento `state` existente. Explicaremos por qué en el próximo ejercicio.

-----

## Rules of Reducers

En el ejercicio anterior, escribimos reductores que devolvían una **nueva copia del estado** en lugar de editarlo directamente. Hicimos esto para cumplir con las **reglas de los reductores** proporcionadas por la documentación de Redux:

1.  **Solo deben calcular el nuevo valor del estado basándose en los argumentos `state` y `action`**.
2.  **No se les permite modificar el estado existente**. En su lugar, deben copiar el estado existente y hacer cambios en los valores copiados.
3.  **No deben realizar ninguna lógica asíncrona ni tener otros "efectos secundarios (side effects)"**.

Por **lógica asíncrona o "efectos secundarios"** nos referimos a cualquier cosa que la función haga además de devolver un valor, por ejemplo: registrar en la consola, guardar un archivo, establecer un temporizador, hacer una solicitud HTTP y generar números aleatorios.

Al adherirse a estas reglas, Redux promueve una **separación clara de responsabilidades**, mejora la **mantenibilidad** de la base de código y permite una **depuración y prueba eficientes**.

----

## Rules of Reducers

En el ejercicio anterior, escribimos reductores que devolvían una **nueva copia del estado** en lugar de editarlo directamente. Hicimos esto para cumplir con las **reglas de los reductores** proporcionadas por la documentación de Redux:

1.  **Solo deben calcular el nuevo valor del estado basándose en los argumentos `state` y `action`**.
2.  **No se les permite modificar el estado existente**. En su lugar, deben copiar el estado existente y hacer cambios en los valores copiados.
3.  **No deben realizar ninguna lógica asíncrona ni tener otros "efectos secundarios (side effects)"**.

Por **lógica asíncrona o "efectos secundarios"** nos referimos a cualquier cosa que la función haga además de devolver un valor, por ejemplo: registrar en la consola, guardar un archivo, establecer un temporizador, hacer una solicitud HTTP y generar números aleatorios.

Al adherirse a estas reglas, Redux promueve una **separación clara de responsabilidades**, mejora la **mantenibilidad** de la base de código y permite una **depuración y prueba eficientes**.

----

## Immutable Updates and Pure Functions

En programación, las tres reglas de los **reducers** en Redux pueden describirse de forma más amplia. Estas reglas establecen que los reducers deben realizar **actualizaciones inmutables (immutable updates)** y ser **funciones puras (pure functions)**.

### Actualizaciones Inmutables (Immutable Updates)

Cuando una función realiza actualizaciones inmutables a sus argumentos, **no modifica directamente** el argumento original. En su lugar, crea una **copia** y modifica esa copia. Este proceso se conoce como actualización inmutable porque la función no altera o muta los argumentos originales.

**Esta función muta su argumento:**

```jsx
const mutableUpdater = (obj) => {
  obj.completed = !obj.completed;
  return obj;
}

const task = { text: 'do dishes', completed: false };
const updatedTask = mutableUpdater(task);
console.log(updatedTask); 
// Imprime { text: 'do dishes', completed: true };

console.log(task); 
// Imprime { text: 'do dishes', completed: true }; (¡El original también cambió!)
```

Mientras tanto, **esta función "actualiza inmutablente" su argumento**:

```jsx
const immutableUpdater = (obj) => {
  return {
    ...obj, // Copia todas las propiedades del objeto original
    completed: !obj.completed // Actualiza la propiedad en la copia
  }
}

const task = { text: 'iron clothes', completed: false };
const updatedTask = immutableUpdater(task);
console.log(updatedTask); 
// Imprime { text: 'iron clothes', completed: true };

console.log(task); 
// Imprime { text: 'iron clothes', completed: false }; (¡El original no cambia!)
```

Al copiar el contenido del argumento `obj` en un nuevo objeto (`{...obj}`) y actualizar la propiedad `completed` de la copia, el argumento `obj` original permanecerá sin cambios.

Ten en cuenta que los **strings**, números y booleanos simples son **inmutables** en JavaScript, por lo que podemos simplemente devolverlos sin hacer una copia:

```jsx
const immutator = (num) => num + 1;
const x = 5;
const updatedX = immutator(x);

console.log(x, updatedX); // Imprime 5, 6 (el original no cambia)
```

### Funciones Puras (Pure Functions)

Si una función es **pura**, entonces siempre tendrá las mismas salidas dadas las mismas entradas.

Esta es una combinación de las reglas 1 y 3 de Redux:

*   Los reducers solo deben calcular el nuevo valor del estado basándose en los argumentos `state` y `action`.
*   Los reducers **no deben** realizar ninguna lógica asíncrona u otros "efectos secundarios (side effects)".

En este ejemplo, la función **no es una función pura** porque su valor devuelto depende del estado de un endpoint remoto:

```jsx
const addItemToList = (list) => {
  let item;
  fetch('https://anything.com/endpoint')
    .then(response => {
      if (!response.ok) {
        item = {};
      }
      
      item = response.json();
   });

   return [...list, item];  
};
```

La función se puede hacer pura **sacando la declaración `fetch()` fuera de la función**:

```jsx
let item;
fetch('https://anything.com/endpoint')
  .then(response => {
    if (!response.ok) {
      item = {};
    }
    
    item = response.json();
  });

const addItemToList = (list, item) => {
    return [...list, item];
};
```

En esta versión modificada, `addItemToList` ahora toma `item` como argumento, por lo que su salida depende únicamente de sus entradas (`list` e `item`), convirtiéndola en una función pura.

---

## Store

Hasta ahora hemos cubierto el **estado (state)** , las **acciones (actions)** , los **reducers** y cómo participan en el flujo de datos unidireccional (one-way data flow). ¿Dónde tiene lugar todo esto?

Redux utiliza un objeto especial llamado **store**. El store sirve como **contenedor para el estado** y es la pieza central de tu aplicación y la **única fuente de verdad (single source of truth)** . El store se encarga de facilitar el **despacho (dispatching)** de acciones y de activar el **reducer** cuando se despachan acciones. En la mayoría de las aplicaciones Redux, normalmente solo hay **un store**.

Reformulemos el flujo de datos usando el nuevo término:

1.  El **store** inicializa el **estado** con un valor por defecto.
2.  La **vista (view)** muestra ese estado al usuario.
3.  Cuando un usuario interactúa con la vista, como haciendo clic en un botón, se **despacha (dispatch)** una **acción (action)** al store.
4.  El **reducer** del store combina la acción despachada y el estado actual para determinar el **siguiente estado (next state)** .
5.  La vista se **actualiza** para mostrar el nuevo estado.

Si bien no escribiremos ningún código para el store durante esta lección, es esencial que comprendas el estado, las acciones, los reducers y su papel en el flujo de datos unidireccional.

-----

## Review

¡Felicitaciones! En esta lección, has construido una base conceptual sólida de Redux y, en el proceso, has construido un objeto de estado (state), algunas acciones (actions) y un reducer. Esto es lo que más has aprendido:

*   **Redux** es una librería para gestionar y actualizar el estado de la aplicación basada en la arquitectura **Flux**.
*   Redux hace que el código sea más **predecible**, **comprobable** y **mantenible** al consolidar el estado en un solo objeto. A los componentes se les dan datos para renderizar y pueden solicitar cambios utilizando eventos llamados **acciones (actions)** .
*   En una aplicación Redux, los datos fluyen en **una dirección (one-way data flow)** : desde el **estado (state)** a la **vista (view)** , a la **acción (action)** , de vuelta al estado, y así sucesivamente.
*   El **estado (state)** es la información actual que respalda una aplicación web.
*   Una **acción (action)** es un objeto que describe un evento en la aplicación. Debe tener una propiedad `type` y, típicamente, también tiene una propiedad `payload`.
*   Un **reducer** es una función que determina el siguiente estado de la aplicación dado un estado actual y una acción específica. Devuelve un estado inicial por defecto si no se proporciona ninguno y devuelve el estado actual si la acción no es reconocida.
*   Un **reducer debe seguir estas tres reglas**:
    1.  Solo debe calcular el nuevo valor del estado basándose en el estado existente y la acción.
    2.  No se le permite modificar el estado existente. En su lugar, debe **copiar el estado existente** y hacer cambios en los valores copiados.
    3.  No debe realizar ninguna lógica asíncrona u otros "efectos secundarios (side effects)".
*   En otras palabras, un **reducer debe ser una función pura (pure function)** y debe **actualizar el estado de forma inmutable (immutably)** .
*   El **store** es un contenedor para el estado. Proporciona una forma de despachar acciones y llama al reducer cuando se despachan acciones. Normalmente, solo hay **un store** en una aplicación Redux.

-----