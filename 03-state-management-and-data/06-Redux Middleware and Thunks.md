# Redux Middleware and Thunks

## Introduction

Hasta ahora, podemos crear una aplicación con gestión de estado de Redux. Pero no hemos cubierto uno de los desafíos más comunes en el desarrollo de aplicaciones: realizar **solicitudes asíncronas (asynchronous requests)**. Con una store básica de Redux, solo podemos hacer **actualizaciones síncronas (synchronous updates)**. Cuando se despacha una acción, es procesada inmediatamente por un reducer, que actualiza la store en consecuencia. Pero al desarrollar aplicaciones, a menudo queremos realizar operaciones asíncronas (como hacer llamadas a una API) y actualizar el estado basándonos en los resultados.

En esta lección, obtendrás las herramientas necesarias para escribir **lógica asíncrona** que interactúe con tu store de Redux.

*   Primero, aprenderás sobre dos conceptos generales en computación: **middleware** y **thunks**, y las formas en que se relacionan con Redux.
*   A continuación, aprenderás sobre el **ciclo de vida de una promesa (promise lifecycle)** y cómo puedes usarlo para proporcionar una experiencia de usuario satisfactoria.
*   Finalmente, practicarás añadiendo lógica asíncrona a tus aplicaciones Redux utilizando las herramientas proporcionadas por `@reduxjs/toolkit`.

**Nota:** Esta lección utiliza **Mock Service Worker (MSW)** para replicar la funcionalidad de una API externa. Para usar MSW, querrás usar Google Chrome y habilitar las cookies de terceros.

-------

## Middleware in Redux

Desde el principio, Redux puede satisfacer la mayoría de las necesidades de gestión de estado de tu aplicación. Pero cada proyecto es diferente, por lo que Redux proporciona algunas formas de personalizar su comportamiento. Una de las formas en que podemos personalizar Redux es añadiendo **middleware**.

Puede que estés familiarizado con el middleware por experiencias trabajando con otros frameworks. Como su nombre indica, el middleware es el código que se ejecuta **en el medio**, generalmente entre que un framework recibe una solicitud y produce una respuesta. El middleware es una herramienta poderosa para extender, modificar o personalizar el comportamiento predeterminado de un framework o librería para satisfacer las necesidades específicas de una aplicación.

En Redux, el middleware se ejecuta **entre el momento en que se despacha una acción y el momento en que esa acción se pasa al reducer**. En este punto, ya conoces la forma en que los datos fluyen a través de Redux: las acciones se despachan a la store, donde son procesadas por los reducers que producen un nuevo estado; ese nuevo estado se vuelve accesible para todos los componentes que lo referencian, provocando que esos componentes se actualicen. Hemos representado ese flujo en el diagrama incluido y hemos añadido el middleware para ayudarte a ver dónde y cómo entra en juego.

El middleware intercepta las acciones **después de que se despachan y antes de que se pasen al reducer**. Algunas tareas comunes que realiza el middleware incluyen: registro (logging), almacenamiento en caché (caching), añadir tokens de autenticación a los encabezados de las solicitudes, informes de fallos (crash reporting), enrutamiento (routing) y realización de **solicitudes asíncronas (asynchronous requests)** para obtener datos. Puedes añadir cualquiera de estas funcionalidades a tus aplicaciones utilizando middleware popular de código abierto. Por supuesto, también puedes escribir tu propio middleware para resolver problemas específicos de tu aplicación y su arquitectura.

Para hacer solicitudes asíncronas en nuestra aplicación de recetas, estamos utilizando una función de utilidad de Redux Toolkit llamada **`createAsyncThunk()`** y la opción **`extraReducers`** que puedes pasar a la función `createSlice`. En ejercicios posteriores, veremos cómo `createAsyncThunk()` utiliza middleware y **thunks** (discutiremos esto más en el próximo ejercicio) para hacer posibles las solicitudes asíncronas; por ahora, solo debes entender **dónde se sitúa el middleware en el flujo de datos de Redux**.

**Nota:** En Redux Toolkit, el middleware se añade automáticamente cuando usamos `configureStore`, pero el concepto sigue siendo el mismo: el middleware se ejecuta entre el despacho de una acción y el reducer que la maneja.

----

## Introduction to Thunks

Recuerda que nuestro objetivo general en esta lección es darte las herramientas que necesitas para añadir funcionalidad asíncrona a tus aplicaciones Redux. Una de las formas más flexibles y populares de añadir funcionalidad asíncrona a Redux implica el uso de **thunks**. Un **thunk** es una **función de orden superior** que envuelve el cálculo que queremos realizar más tarde.

Por ejemplo, esta función `add()` devuelve un thunk que realizará `x + y` cuando se llame.

```jsx
const add = (x, y) => {
  return () => {
    return x + y; 
  } 
}
```

Los thunks son útiles porque nos permiten **empaquetar fragmentos de cálculo** que queremos retrasar en paquetes que pueden ser pasados en el código. Considera estas dos llamadas a funciones, que dependen de la función `add()` de arriba:

```jsx
const delayedAddition = add(2, 2);
delayedAddition(); // => 4
```

Observa que **llamar a `add()` no hace que la suma ocurra** – simplemente devuelve una función que realizará la suma cuando sea llamada. Para realizar la suma, debemos llamar a `delayedAddition()`.

----

## Promise Lifecycle Actions

En un mundo perfecto, cada solicitud de red que hagamos produciría una respuesta inmediata y exitosa. Pero las **solicitudes de red (network requests)** pueden ser lentas y, a veces, fallar. Como desarrolladores, debemos tener en cuenta estas realidades para crear la mejor experiencia posible para nuestros usuarios. Si sabemos que una solicitud está pendiente, podemos hacer que nuestra aplicación sea más fácil de usar mostrando un **estado de carga (loading state)**. Del mismo modo, si sabemos que una solicitud ha fallado, podemos mostrar un **estado de error (error state)** apropiado.

Para crear estas experiencias de usuario satisfactorias, necesitamos realizar un seguimiento del **estado en el que se encuentran nuestras solicitudes asíncronas** en cualquier momento dado, para poder reflejar esos estados para el usuario. Es común despachar una acción **"pending" (pendiente)** justo antes de realizar una operación asíncrona, y acciones **"fulfilled" (cumplida)** o **"rejected" (rechazada)** dependiendo de los resultados de la operación completada. Toma este ejemplo ilustrativo `fetchUserById`:

```jsx
import { fetchUser } from './api';

const fetchUserById = async (id) => {
  const payload = await fetchUser(id);
  // actualizar los datos del usuario en la store con `payload`
}
```

Reescrito para incluir acciones pending y rejected, podría verse así:

```jsx
import { fetchUser } from './api';

const fetchUserById = async (id) => {
  // actualizar la store para mostrar que se están solicitando los datos del usuario -> estado "pending"
  try {
    const payload = await fetchUser(id);
    // actualizar los datos del usuario en la store con `payload` -> estado "fulfilled"
  } catch(err) {
    // notificar a la store que falló la obtención de los datos del usuario -> estado "rejected"
  }
}
```

Llamamos a estas acciones **pending/fulfilled/rejected** acciones del ciclo de vida de la promesa (**promise lifecycle actions**). Este patrón es tan común que Redux Toolkit proporciona una abstracción útil, **`createAsyncThunk`**, para incluir acciones del ciclo de vida de la **promesa (promise)** en tus aplicaciones Redux. Exploraremos ese método en los siguientes ejercicios.

![promises](/Images/Promises_transparent.png)

----

# createAsyncThunk()

**`createAsyncThunk()`** es una función con dos parámetros: una cadena de **tipo de acción** y un **callback asíncrono**, que genera un **creador de acción thunk** que ejecutará el callback proporcionado y despachará automáticamente las acciones del **ciclo de vida de la promesa (promise lifecycle actions)** según corresponda, para que no tengas que despachar las acciones pending/fulfilled/rejected manualmente.

Para usar `createAsyncThunk()`, primero necesitarás importarlo desde Redux Toolkit de la siguiente manera:

```jsx
import { createAsyncThunk } from '@reduxjs/toolkit';
```

A continuación, necesitarás llamar a `createAsyncThunk()`, pasando dos argumentos. El primero es una **cadena que representa el tipo de la acción asíncrona**. Convencionalmente, las cadenas de tipo toman la forma `"resourceType/actionName"`. En este caso, dado que estamos obteniendo un usuario individual por su id, nuestro tipo de acción será `'users/fetchUserById'`. El segundo argumento de `createAsyncThunk` es el **creador de carga útil (payload creator)** : una **función asíncrona** que devuelve una promesa que se resuelve en el resultado de una operación asíncrona. Aquí está `fetchUserById` reescrito usando `createAsyncThunk`:

```jsx
import { createAsyncThunk } from '@reduxjs/toolkit';
import { fetchUser } from './api';

const fetchUserById = createAsyncThunk(
  'users/fetchUserById', // tipo de acción
  async (arg, thunkAPI) => { // creador de carga útil (payload creator)
    const response = await fetchUser(arg);
    return response.json();
  }
);
```

Hay algunas cosas que vale la pena destacar aquí:

1.  Observa que el **payload creator** recibe dos argumentos: **`arg`** y **`thunkAPI`**. Los elaboraremos en el próximo ejercicio.
2.  Nota que el payload creator que proporcionamos **no despacha ninguna acción**. Simplemente devuelve el resultado de una operación asíncrona.

Como puedes ver, `createAsyncThunk()` hace que la definición de creadores de acción thunk sea concisa. Todo lo que tienes que escribir es una **función thunk asíncrona**; `createAsyncThunk()` se encarga del resto, devolviendo un creador de acción que despachará las acciones pending/fulfilled/rejected según corresponda.

----

## Passing Arguments to Thunks

