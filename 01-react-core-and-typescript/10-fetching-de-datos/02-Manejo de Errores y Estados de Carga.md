# Manejo de Errores y Estados de Carga

## Why One State Is Not Enough

En la lección anterior construimos un componente que pide datos a una API y los guarda en el estado. Ese ejemplo funciona, pero asume algo poco realista: que la petición **siempre** va a tener éxito, y que va a completarse **instantáneamente**. En una aplicación real, ninguna de las dos cosas es cierta.

Cuando un componente le pide datos a un servidor, en realidad puede encontrarse en uno de tres momentos distintos:

1. La petición está **en curso**: todavía no sabemos si va a tener éxito o no, así que no tenemos datos que mostrar.
2. La petición **tuvo éxito**: ya tenemos los datos y podemos renderizarlos.
3. La petición **falló**: quizás el servidor no respondió, quizás no hay conexión a internet, quizás la API devolvió un error. En cualquier caso, no tenemos datos, pero tampoco queremos dejar al usuario mirando una pantalla vacía sin ninguna explicación.

Si nuestro componente solo tiene una variable de estado para los datos, no tiene forma de distinguir entre "todavía no llegaron los datos" y "los datos fallaron y nunca van a llegar". Para eso necesitamos variables de estado adicionales que representen explícitamente estos tres momentos.

-----

## Tracking Loading and Error State

Vamos a ampliar nuestro componente `UserList` con dos nuevas variables de estado: `isLoading` y `error`.

```javascript
const [users, setUsers] = useState([]);
const [isLoading, setIsLoading] = useState(true);
const [error, setError] = useState(null);
```

* `isLoading` es un booleano que representa si la petición sigue en curso. Lo inicializamos en `true`, porque en cuanto el componente se monta, ya sabemos que vamos a empezar a pedir datos.
* `error` guarda el error que ocurrió, si es que ocurrió alguno. Lo inicializamos en `null` porque, al principio, no hay ningún error todavía.

Con estas dos variables, nuestro JSX puede decidir explícitamente qué mostrarle al usuario según el momento en el que se encuentra la petición:

```javascript
if (isLoading) {
  return <p>Cargando...</p>;
}

if (error) {
  return <p>Ocurrió un error: {error.message}</p>;
}

return (
  <ul>
    {users.map(user => (
      <li key={user.id}>{user.name}</li>
    ))}
  </ul>
);
```

Este patrón de retornos tempranos (*early returns*) es muy común en React: en lugar de anidar condicionales dentro del JSX principal, dejamos que el componente termine su renderizado apenas sabe qué es lo que corresponde mostrar.

-----

## Making a Safe Fetch Call

Ahora necesitamos actualizar estas dos variables de estado en el momento correcto. Para eso, en lugar de encadenar `.then()` como en la lección anterior, vamos a usar la sintaxis `async`/`await` junto con `try`/`catch`/`finally`, que hace mucho más legible el manejo de errores.

```javascript
useEffect(() => {
  const fetchUsers = async () => {
    try {
      const response = await fetch('https://api.ejemplo.com/users');

      if (!response.ok) {
        throw new Error('No se pudo obtener la lista de usuarios');
      }

      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err);
    } finally {
      setIsLoading(false);
    }
  };

  fetchUsers();
}, []);
```

Vale la pena detenerse en cada bloque:

* **`try`**: aquí va todo el código que puede fallar. Primero esperamos (`await`) a que `fetch()` resuelva la promesa y nos entregue la respuesta. Una particularidad importante de `fetch()` es que **no** rechaza la promesa automáticamente cuando el servidor responde con un código de error (por ejemplo, un 404 o un 500); solo la rechaza si la petición no pudo completarse en absoluto (por ejemplo, sin conexión). Por eso comprobamos manualmente `response.ok`, que es `false` cuando el código de estado HTTP indica un error, y lanzamos (`throw`) un error nosotros mismos en ese caso.
* **`catch`**: si algo dentro del bloque `try` lanza un error (ya sea porque `fetch()` rechazó la promesa, o porque nosotros lo lanzamos manualmente), la ejecución salta directamente aquí. Guardamos ese error en el estado con `setError(err)`.
* **`finally`**: este bloque se ejecuta siempre, sin importar si el `try` tuvo éxito o el `catch` capturó un error. Es el lugar ideal para poner `setIsLoading(false)`, porque en ambos casos la carga ya terminó.

Nota también que definimos una función `fetchUsers` **dentro** del efecto y la invocamos inmediatamente después. Esto es necesario porque la función que le pasamos a `useEffect()` no puede ser `async` directamente: React espera que esa función devuelva `undefined` o una función de limpieza, nunca una promesa. Declarar una función `async` interna y llamarla es el patrón estándar para poder usar `await` dentro de un efecto.

-----

## The Full Picture

Uniendo todas las piezas, el componente completo queda así:

```javascript
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('https://api.ejemplo.com/users');

        if (!response.ok) {
          throw new Error('No se pudo obtener la lista de usuarios');
        }

        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err);
      } finally {
        setIsLoading(false);
      }
    };

    fetchUsers();
  }, []);

  if (isLoading) {
    return <p>Cargando...</p>;
  }

  if (error) {
    return <p>Ocurrió un error: {error.message}</p>;
  }

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

El flujo completo, entonces, es el siguiente: el componente se monta con `isLoading` en `true`, por lo que lo primero que ve el usuario es el mensaje de carga. Una vez que la petición se resuelve (con éxito o con error), actualizamos el estado correspondiente y, en el bloque `finally`, marcamos `isLoading` como `false`. Ese cambio de estado dispara un nuevo renderizado, y esta vez el componente ya tiene suficiente información para mostrar la lista de usuarios o el mensaje de error, según corresponda.

Separar estos tres estados (`isLoading`, `error` y los propios datos) es lo que le permite a nuestra aplicación comunicarle claramente al usuario qué está pasando en cada momento, en lugar de dejarlo adivinando por qué la pantalla está vacía.
