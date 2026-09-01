# Optimización de Fetch con Dependencias

## Why Fetch Again When Something Changes?

Hasta ahora, nuestros efectos de fetch se ejecutaban una única vez, al montar el componente, gracias a un arreglo de dependencias vacío (`[]`). Pero hay un escenario extremadamente común en el que esto no es suficiente: un buscador. Pensemos en un input de búsqueda donde el usuario escribe un término, y la lista de resultados en pantalla debe actualizarse automáticamente a medida que escribe, sin necesidad de un botón de "Buscar".

Para lograr esto, necesitamos que nuestro efecto de fetch se vuelva a ejecutar cada vez que cambie el valor que el usuario escribió. Esto es exactamente para lo que existe el **arreglo de dependencias** de `useEffect()`: en lugar de dejarlo vacío, le pasamos una lista de valores, y React se encarga de comparar esos valores entre un renderizado y el siguiente. Si alguno cambió, React vuelve a ejecutar la función de efecto.

-----

## Building a Search Component

Empecemos por guardar el texto que el usuario escribe en una variable de estado:

```javascript
const [query, setQuery] = useState('');
```

Conectamos esta variable a un input controlado, de forma que cada tecla que el usuario presiona actualice `query`:

```javascript
<input
  value={query}
  onChange={(event) => setQuery(event.target.value)}
/>
```

Ahora viene la parte clave: un efecto que dispara la búsqueda, pero que declara `query` como dependencia:

```javascript
useEffect(() => {
  const fetchPosts = async () => {
    const response = await fetch(`https://api.ejemplo.com/posts?title=${query}`);
    const data = await response.json();
    setPosts(data);
  };

  fetchPosts();
}, [query]);
```

Al escribir `[query]` en lugar de `[]`, le estamos diciendo a React: *"ejecuta este efecto después del primer renderizado, y luego vuelve a ejecutarlo cada vez que el valor de `query` sea distinto al que tenía en el renderizado anterior"*. Esto es exactamente el comportamiento que buscamos: cada letra que el usuario escribe actualiza `query`, ese cambio de estado dispara un nuevo renderizado, y como `query` cambió, React vuelve a correr el efecto y dispara una nueva petición con el término de búsqueda actualizado.

El resto del componente se completa renderizando los resultados a partir del estado `posts`:

```javascript
import { useState, useEffect } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    const fetchPosts = async () => {
      const response = await fetch(`https://api.ejemplo.com/posts?title=${query}`);
      const data = await response.json();
      setPosts(data);
    };

    fetchPosts();
  }, [query]);

  return (
    <div>
      <input
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

export default Search;
```

-----

## The Race Condition Problem

Este componente funciona, pero esconde un problema sutil que vale la pena entender antes de llevarlo a producción: las **condiciones de carrera** (*race conditions*).

Pensemos en lo que ocurre cuando un usuario escribe rápido, por ejemplo la palabra "react". Cada letra dispara una nueva petición:

* Se dispara una petición para `"r"`.
* Se dispara una petición para `"re"`.
* Se dispara una petición para `"rea"`.
* Se dispara una petición para `"reac"`.
* Se dispara una petición para `"react"`.

El problema es que estas cinco peticiones viajan por la red de forma independiente, y nada garantiza que **lleguen en el mismo orden en que se enviaron**. Es perfectamente posible que la petición para `"react"` (la que realmente nos interesa) llegue antes que la petición para `"re"`, simplemente porque el servidor tardó un poco más en resolver esa consulta en particular. Si eso ocurre, el `setPosts(data)` de la petición para `"re"` se ejecutará **después**, sobrescribiendo los resultados correctos con resultados obsoletos. El usuario terminaría viendo en pantalla resultados que no corresponden a lo que realmente escribió.

-----

## Cleaning Up Stale Requests

La forma estándar de resolver esto es usar la **función de limpieza** que `useEffect()` permite retornar. React ejecuta esta función de limpieza justo antes de volver a correr el efecto (o al desmontar el componente), lo que nos da la oportunidad de "cancelar" o invalidar el trabajo del efecto anterior antes de empezar uno nuevo.

Una forma sencilla de aplicar esto, sin recurrir a APIs más avanzadas como `AbortController`, es usar una bandera local que marque si el efecto sigue siendo el vigente:

```javascript
useEffect(() => {
  let isActive = true;

  const fetchPosts = async () => {
    const response = await fetch(`https://api.ejemplo.com/posts?title=${query}`);
    const data = await response.json();

    if (isActive) {
      setPosts(data);
    }
  };

  fetchPosts();

  return () => {
    isActive = false;
  };
}, [query]);
```

Analicemos qué cambió:

* Declaramos una variable local `isActive`, inicializada en `true`, dentro del cuerpo del efecto.
* Antes de llamar a `setPosts(data)`, comprobamos que `isActive` siga siendo `true`. Solo actualizamos el estado si este efecto en particular sigue siendo el "actual".
* Retornamos una función de limpieza que pone `isActive` en `false`.

Cuando `query` cambia, React va a ejecutar la función de limpieza del efecto **anterior** antes de correr el nuevo efecto. Eso significa que la variable `isActive` de la petición vieja (por ejemplo, la de `"re"`) queda en `false` antes de que su respuesta llegue del servidor. Cuando esa respuesta finalmente llega, la comprobación `if (isActive)` evita que sus datos, ya obsoletos, sobrescriban el estado.

Este mismo mecanismo de limpieza es el que usaríamos para cancelar suscripciones, temporizadores o cualquier otro efecto que no deba seguir vigente una vez que sus dependencias cambiaron. En el caso de peticiones de red, existen herramientas más específicas para cancelar la petición en sí (como `AbortController`), pero el patrón de la bandera `isActive` es suficiente para entender el problema y evitarlo en la gran mayoría de los casos, y es una base sólida antes de explorar soluciones más completas como React Query, que resuelve este mismo problema de forma automática.
