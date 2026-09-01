# React Query

## Why Server State Is Different

En la lección de fetching de datos con `useEffect`, construimos manualmente el manejo de estados de carga, error y datos, y vimos que incluso algo aparentemente simple, como un buscador, esconde problemas sutiles como las condiciones de carrera. Ese esfuerzo no fue en vano: entender qué está pasando "por debajo" es lo que te permite apreciar realmente lo que una librería como **React Query** (también conocida por su paquete `@tanstack/react-query`) resuelve por vos.

La idea central de React Query parte de una distinción importante: el estado de tu aplicación no es todo igual. Hay **estado de cliente** (por ejemplo, si un modal está abierto o cerrado, o qué pestaña está seleccionada), que le pertenece completamente a tu aplicación. Y hay **estado de servidor** (los datos que vienen de una API): usuarios, posts, productos. Este segundo tipo de estado tiene características muy distintas: no es tuyo, puede quedar desactualizado en cualquier momento porque otra persona lo modificó, y necesita ser sincronizado, cacheado y refrescado. React Query es una librería especializada exclusivamente en gestionar ese segundo tipo de estado, para que dejes de reimplementar manualmente `isLoading`, `error` y la lógica de cache con `useState` y `useEffect` en cada componente.

-----

## Automatic Caching with useQuery

El hook central de React Query es `useQuery()`. Con él, describimos **qué datos necesitamos** y **cómo obtenerlos**, y la librería se encarga del resto:

```tsx
import { useQuery } from '@tanstack/react-query';

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });

  if (isLoading) return <p>Cargando...</p>;
  if (error) return <p>Ocurrió un error</p>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

Compará esto con el componente que construimos manualmente en la lección de manejo de errores y carga: allí declaramos tres variables de estado (`users`, `isLoading`, `error`), un efecto con `try`/`catch`/`finally`, y una función `async` interna. Aquí, `useQuery()` nos devuelve directamente `data`, `isLoading` y `error`, ya gestionados internamente.

Pero la diferencia más importante no es la que se ve, sino la que React Query hace por debajo: **cachea** el resultado. Si otro componente en cualquier parte de la aplicación vuelve a llamar `useQuery({ queryKey: ['users'], queryFn: getUsers })`, React Query no dispara una nueva petición de red: le devuelve inmediatamente los datos que ya tiene guardados en su cache. Esto evita peticiones duplicadas y hace que la interfaz se sienta instantánea al navegar entre pantallas que muestran los mismos datos.

-----

## Query Keys: Identifying What You Cache

El arreglo `['users']` que vimos en el ejemplo anterior es la **query key**: un identificador único que React Query usa para saber a qué dato corresponde cada entrada de su cache. Cuando querés cachear datos que dependen de un parámetro, como el detalle de un usuario específico, la query key también incluye ese parámetro:

```tsx
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => getUserById(userId),
});
```

Esto es importante porque React Query no solo usa la query key para decidir si ya tiene el dato en cache, sino también para saber **qué invalidar** cuando algo cambia. Si actualizás al usuario con id `5`, podés decirle a React Query que invalide específicamente la entrada `['user', 5]`, sin afectar el cache de ningún otro usuario. Pensá en la query key como la dirección exacta de un dato dentro del cache.

-----

## Refetching to Stay in Sync

Como el estado de servidor puede quedar desactualizado en cualquier momento (otra persona, o incluso otra pestaña del mismo usuario, pudo haber modificado el dato), React Query vuelve a pedir los datos automáticamente en ciertas situaciones: cuando el usuario vuelve a enfocar la ventana del navegador, cuando se reconecta a internet después de haber estado offline, o después de que pasa un tiempo configurable desde la última petición. Todo esto ocurre sin que tengas que escribir un solo `useEffect` adicional, y es lo que permite que la interfaz muestre datos frescos sin sacrificar la velocidad que da el cache.

-----

## Mutating Data with useMutation

`useQuery()` está pensado para **leer** datos (operaciones GET). Cuando necesitamos **modificar** datos en el servidor (crear, actualizar o eliminar), usamos el hook complementario `useMutation()`:

```tsx
import { useMutation } from '@tanstack/react-query';

const { mutate } = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    console.log('Usuario creado con éxito');
  },
  onError: (error) => {
    console.log('Ocurrió un error al crear el usuario', error);
  },
});
```

A diferencia de `useQuery()`, que se dispara automáticamente cuando el componente se renderiza, una mutación se ejecuta explícitamente llamando a la función `mutate()` que el hook nos devuelve, típicamente dentro de un manejador de eventos como el envío de un formulario.

Las funciones de callback `onSuccess` y `onError` nos permiten reaccionar al resultado de la mutación: mostrar una notificación de éxito, redirigir al usuario, registrar el error, o (como veremos a continuación) invalidar datos que quedaron desactualizados en el cache como consecuencia de la mutación.

-----

## Optimistic Updates and Rollback

Una de las técnicas más interesantes que React Query facilita son las **actualizaciones optimistas** (*optimistic updates*). La idea es simple de enunciar pero poderosa en su efecto: en lugar de esperar a que el servidor confirme que una operación tuvo éxito para reflejarla en la interfaz, actualizamos la interfaz **inmediatamente**, asumiendo que la operación va a tener éxito, y solo revertimos ese cambio si el servidor termina respondiendo con un error.

El ejemplo clásico es un botón de "me gusta" en una publicación. Sin actualizaciones optimistas, el flujo sería: el usuario hace clic, la interfaz espera la respuesta del servidor, y recién ahí el corazón se pinta de rojo. Esa espera, aunque sea de unos pocos cientos de milisegundos, se siente lenta. Con actualizaciones optimistas, el corazón se pinta de rojo en el instante en que el usuario hace clic, y la petición al servidor viaja en segundo plano.

¿Y si la petición finalmente falla? Aquí es donde entra el **rollback**: React Query nos da los hooks necesarios (`onMutate`, para guardar una copia del estado anterior antes de aplicar el cambio optimista, y `onError`, para restaurar esa copia si la petición falla) para devolver la interfaz al estado en el que estaba antes del clic, como si nunca hubiera ocurrido.

Esta técnica no es apropiada para cualquier operación. Tiene sentido para acciones rápidas y de bajo riesgo, donde la probabilidad de fallo es baja y las consecuencias de revertir son mínimas: un "me gusta", agregar un producto al carrito, marcar una tarea como completada. No tiene sentido, en cambio, para operaciones donde la precisión es crítica y una reversión visible podría confundir o generar desconfianza en el usuario, como un pago, una transferencia bancaria, o cualquier operación que requiera trazabilidad exacta. En esos casos, es preferible esperar la confirmación real del servidor antes de actualizar la interfaz, aunque eso signifique una experiencia ligeramente menos inmediata.

-----

## When to Reach for React Query

Vale la pena remarcar que React Query no reemplaza a Zustand o Redux: resuelve un problema distinto. Zustand y Redux gestionan estado de **cliente** (datos que le pertenecen a tu aplicación y que vos controlás por completo). React Query gestiona estado de **servidor** (datos que no te pertenecen, que pueden cambiar por fuera de tu aplicación, y que necesitan sincronizarse, cachearse y refrescarse). En una aplicación real, es común usar ambas herramientas juntas: Zustand para el estado de la interfaz, y React Query para todo lo que involucre comunicación con una API.
