# Fetch de Datos con useEffect

## Why Fetch Data Inside useEffect?

Hasta ahora hemos trabajado con datos que ya existían dentro de nuestro componente, ya fuera como estado inicial o como props recibidas de un componente padre. Pero en una aplicación real, la mayoría de los datos que mostramos no nacen en el propio componente: viven en un servidor, y nuestro componente necesita **pedirlos** antes de poder mostrarlos.

Esta acción de pedir datos a una API se conoce como **fetching**, y en JavaScript la realizamos con la función `fetch()`. El problema es que `fetch()` es una operación **asíncrona**: no se completa instantáneamente durante el renderizado, sino que toma un tiempo (a veces milisegundos, a veces más) en ir hasta el servidor, esperar la respuesta y traerla de vuelta.

Aquí es donde entra el **Effect Hook**, `useEffect()`. Un componente de función existe para una sola cosa: describir cómo se ve la interfaz según el estado actual. Pedir datos a un servidor no es parte de ese renderizado, es un **efecto secundario**: algo que sucede como consecuencia de que el componente se haya mostrado en pantalla, pero que no forma parte del cálculo del JSX en sí. Por eso, React nos pide explícitamente que separemos esta lógica usando `useEffect()`, en lugar de intentar hacer el fetch directamente en el cuerpo del componente.

-----

## Building a User List Component

Vamos a construir un componente que muestre una lista de usuarios obtenidos desde una API. El patrón que usaremos aquí es uno de los más comunes en aplicaciones React, y lo verás una y otra vez: **estado vacío al inicio, efecto que pide los datos, estado actualizado cuando llegan, y renderizado que refleja ese estado**.

Empezamos declarando el estado que va a contener nuestra lista de usuarios, inicializado como un arreglo vacío:

```javascript
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);

  // ...
}
```

Inicializar `users` como `[]` (y no como `undefined` o `null`) es una decisión deliberada: nuestro JSX va a usar `.map()` sobre este valor para renderizar la lista, y `.map()` solo funciona sobre arreglos. Si dejáramos `users` sin inicializar, el primer renderizado fallaría al intentar iterar sobre `undefined`.

A continuación, usamos `useEffect()` para disparar la petición a la API en cuanto el componente se monta:

```javascript
useEffect(() => {
  fetch('https://api.ejemplo.com/users')
    .then(response => response.json())
    .then(data => setUsers(data));
}, []);
```

Repasemos lo que ocurre dentro de esta función de efecto:

* `fetch('https://api.ejemplo.com/users')` inicia la petición HTTP hacia el servidor y devuelve una **promesa**.
* Cuando esa promesa se resuelve, recibimos un objeto `response`. Este objeto todavía no contiene los datos en un formato utilizable: contiene la respuesta cruda del servidor. Por eso llamamos a `response.json()`, que también devuelve una promesa y se encarga de **parsear** el cuerpo de la respuesta a un objeto JavaScript.
* Cuando esa segunda promesa se resuelve, tenemos finalmente nuestros datos (`data`), y los guardamos en el estado con `setUsers(data)`.

Llamar a `setUsers()` le indica a React que el estado cambió, lo que dispara un nuevo renderizado del componente. Esta vez, `users` ya no es un arreglo vacío, sino la lista que vino del servidor.

Finalmente, usamos ese estado en el JSX para renderizar la lista:

```javascript
return (
  <ul>
    {users.map(user => (
      <li key={user.id}>{user.name}</li>
    ))}
  </ul>
);
```

-----

## Why an Empty Dependency Array?

El segundo argumento de `useEffect()` es un **arreglo de dependencias**, y es la pieza más importante para entender cuándo se ejecuta nuestro efecto. En el ejemplo anterior, pasamos un arreglo vacío: `[]`.

Un arreglo de dependencias vacío le dice a React: *"este efecto no depende de ningún valor que cambie entre renderizados, así que ejecútalo una sola vez, inmediatamente después de que el componente se monte por primera vez"*. Esto es exactamente lo que queremos para una petición de datos que solo necesita ocurrir al cargar el componente: no tendría sentido volver a pedir la lista completa de usuarios cada vez que el componente se vuelve a renderizar por cualquier otro motivo.

Si **omitiéramos** el segundo argumento por completo, el efecto se ejecutaría después de **cada** renderizado, lo que en este caso dispararía una petición nueva a la API en un ciclo infinito: el fetch actualiza el estado, el estado dispara un nuevo renderizado, el nuevo renderizado vuelve a ejecutar el efecto (porque no tiene arreglo de dependencias), y el fetch se vuelve a disparar. Es un error muy común al empezar con `useEffect()`, así que vale la pena tenerlo presente.

En resumen, el patrón completo se ve así:

```javascript
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('https://api.ejemplo.com/users')
      .then(response => response.json())
      .then(data => setUsers(data));
  }, []);

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default UserList;
```

Este patrón (estado inicial vacío, efecto con arreglo de dependencias vacío, actualización de estado al recibir los datos) es la base sobre la que construiremos técnicas más avanzadas, como el manejo de errores y estados de carga, y la re-ejecución del fetch cuando alguna dependencia cambia.
