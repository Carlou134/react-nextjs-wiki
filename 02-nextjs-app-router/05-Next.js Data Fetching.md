# Next.js Data Fetching

## Data Fetching with Next.js

En una aplicación Next.js, podemos obtener datos tanto en el servidor como en el cliente. Habrá circunstancias en las que obtener datos en el cliente sea inevitable, pero idealmente, los datos deberían obtenerse en el servidor. Dado que el servidor tiene acceso directo a las bases de datos, los waterfalls cliente-servidor se pueden reducir obteniendo datos en el servidor. Además, cuando se utilizan componentes de servidor, los datos se pueden obtener y renderizar en el mismo entorno.

Obtener datos en el servidor también significa que podemos evitar exponer datos sensibles al cliente. Sin embargo, esto también puede crear una falsa sensación de seguridad. Los componentes de servidor pueden renderizarse en el cliente si se utilizan dentro de una plantilla de componente cliente. Aunque esta flexibilidad es parte del diseño, puede llevar a la exposición de datos sensibles si, por ejemplo, las credenciales de API utilizadas al obtener datos en un componente de servidor se incluyen en el paquete del cliente. Para prevenir estos escenarios críticos, veremos cómo usar el paquete 'server-only' para asegurar que ciertos módulos de código específicos del servidor nunca sean importados en componentes cliente.

En esta lección, aprenderemos cómo obtener datos tanto en el servidor como en el cliente, y conoceremos diversas técnicas para crear una experiencia de usuario fluida.

----

## Data Fetching on the Client

Podemos usar Route Handlers (Manejadores de Rutas) para definir nuestros propios manejadores de solicitudes personalizados, que se ejecutan en el servidor para obtener datos y enviar respuestas de vuelta al cliente.

Para definir Route Handlers, creamos un archivo `route.ts` dentro del directorio `/app`. Ten en cuenta que los Route Handlers solo están disponibles dentro del directorio `/app`, y un archivo `route.ts` tampoco puede existir en el mismo nivel que el archivo `page.tsx`. Por lo tanto, necesitaremos crear un directorio dentro de nuestro directorio `/app` para alojar nuestros Route Handlers. Es una práctica común nombrar este directorio `api`.

En `route.ts`, ubicado en nuestro directorio `app/api`, podemos definir nuestros manejadores de solicitudes para cualquiera de los métodos HTTP compatibles con Next.js, como GET, POST, PUT, PATCH, DELETE, HEAD y OPTIONS.

Podemos definir nuestro manejador de solicitudes GET usando la siguiente estructura:

```ts
export async function GET() {
  const response = await fetch('https://api.com/some/route')
  
  if(!response.ok){
    throw new Error('Failed to fetch data.')
  }
  
  const result = await response.json()
  return Response.json(result)
}
```

Desde Next.js 15, las peticiones `fetch()` **no se cachean por defecto** — cada llamada obtiene datos frescos, a menos que se lo indiquemos explícitamente. (Antes de la versión 15, era al revés: `fetch()` cacheaba por defecto y había que pasar `cache: 'no-store'` para desactivarlo — si trabajás con un proyecto en una versión anterior a Next.js 15, vas a encontrarte con ese comportamiento). Para optar por el cacheo, pasamos `cache: 'force-cache'` en el objeto de opciones:

```ts
const response = await fetch('https://api.com/some/route', {
  cache: 'force-cache',
})
```

También podemos construir rutas anidadas creando una carpeta dentro del directorio `/api` y definiendo Route Handlers para la ruta dentro de un archivo `route.ts` separado. Por ejemplo, si quisiéramos definir una API para obtener usuarios, podemos crear una carpeta llamada `user` dentro de la carpeta `/app/api` y crear un archivo `route.ts` separado dentro de `/app/api/user`.

----

## Data Fetching on the Server

También podemos obtener datos directamente en nuestro componente de servidor. Por ejemplo, podemos agregar la llamada `fetch` dentro de nuestro componente `<Home>` de la siguiente manera:

```tsx
// app/page.tsx
export default async function Home() {
  const response = await fetch('https://api.com/some/route', { cache: 'no-store' });
  
  if(!response.ok){
    throw new Error('Failed to fetch data.');
  }
  const result = await response.json();
  
  // otra lógica para el componente
}
```

Recuerda que obtener datos en un componente de servidor puede llevar a la exposición de datos sensibles, como claves de API. Para prevenir esto, podemos obtener datos en un archivo separado usando el paquete 'server-only'. Por ejemplo, podemos crear una carpeta en el directorio raíz llamada `utils` y crear un archivo llamado `getData.ts`. Dentro del archivo, importaremos el paquete 'server-only' para evitar que este archivo sea incluido en el paquete del cliente.

```ts
// utils/getData.ts
import 'server-only';

export default async function getData(){
  const response = await fetch('https://api.com/some/route', {cache: 'no-store'});

  if(!response.ok){
    throw new Error('Failed to fetch data.');
  }

  return response.json();
}
```

Luego podemos importar la función `getData()` desde `utils/getData` en cualquier componente de servidor.

-----

## Parallel vs Sequential Data Fetching

Hay dos patrones de obtención de datos cuando se trabaja con componentes React: paralelo y secuencial.

![Parallel-data-fetching](/Images/parallel-data-fetching.svg)

Cuando se utiliza un **patrón de obtención de datos en paralelo**, las solicitudes en una ruta ocurren simultáneamente. Ya hemos visto un ejemplo de un patrón de obtención de datos en paralelo donde nuestro componente `<Activity>` y el componente `<FriendActivity>` obtenían datos simultáneamente cuando se cargaba nuestra ruta `/`.

![Sequential-data-fetching](/Images/sequential-data-fetching.svg)

Cuando se utiliza el **patrón de obtención de datos secuencial**, las solicitudes crean waterfalls a medida que ocurren una después de la otra. Podríamos optar por cargar datos secuencialmente cuando una obtención de datos necesita depender del resultado de otra obtención.

Practiquemos cómo utilizar el patrón de obtención de datos secuencial.

-----

## Preloading Data

Podemos optimizar aún más la obtención de datos en paralelo mediante la precarga de datos.

Podemos crear una función, típicamente llamada `preload()`, para obtener y almacenar en caché datos de manera anticipada antes de que necesiten ser renderizados. Cuando se obtienen datos en el servidor, podemos definir la función `preload` dentro del archivo que utiliza la directiva `'server-only'`. Luego, podemos usar la función `cache` de React para almacenar en caché los datos obtenidos.

```tsx
// en utils/getPosts.ts

import { cache } from 'react'
import 'server-only'

export const preload = () => {
  void getPosts();
}

export const getPosts = cache(async() => {
  const response = await fetch('https://api.com/some/route');
  // más lógica de fetch
})
```

En el ejemplo de código anterior, la función `getPosts()` se llama dentro de la función `preload()` con el operador `void`, que evalúa la función `getPosts()` y devuelve `undefined`, lo que significa que los datos se obtienen y se almacenan en caché para su uso posterior.

Podemos llamar a la función `preload()` antes de que se active la llamada a `getPosts()` en un componente.

```tsx
import { preload } from '../utils/getPosts'
import Posts from '../components/Posts/Posts'

export default function Home(){
  preload();
  
  return (
    <Posts />
  )
}
```

Aquí, dado que la función `getPosts()` se llama dentro del componente `<Posts>`, la función `preload()` activará la obtención y tendrá los datos almacenados en caché antes de que necesiten ser utilizados dentro del componente `<Posts>`.

----

## Revalidating Data

Recuerda que, al pasar `cache: 'force-cache'`, le pedimos a Next.js que almacene esos datos en caché. Para controlar cómo se purgan los datos cacheados y se vuelven a obtener los datos más recientes, podemos revalidar nuestros datos.

Podemos revalidar nuestros datos almacenados en caché de dos formas: **revalidación basada en tiempo** y **revalidación bajo demanda**.

Usando la revalidación basada en tiempo, podemos especificar cada cuánto tiempo deben revalidarse nuestros datos. Con la revalidación bajo demanda, agrupamos datos dentro de una ruta o una etiqueta que deben actualizarse simultáneamente cuando se activa un evento particular.

Para usar la **revalidación basada en tiempo**, añadimos la opción `next.revalidate` a nuestra llamada `fetch` para especificar la vida útil de nuestros datos en segundos:

```ts
const response = await fetch('https://api.com/some/route', {
  next: { revalidate: 60 }
});
```

En el ejemplo de código anterior, la opción `next.revalidate` tiene el valor de 60 segundos, lo que significa que los datos se revalidarán cada minuto.

Podemos usar la **revalidación bajo demanda** mediante etiquetas de caché o por ruta. Para usar etiquetas, necesitaremos añadir la opción `next.tags` en nuestra llamada `fetch`:

```ts
const response = await fetch('https://api.com/some/route', {
  next: { tags: ['posts'] }
});
```

Para usar la revalidación bajo demanda, crearemos una **Server Action**, una función asíncrona que se ejecuta en el servidor. Para crear una server action, usamos la directiva `'use server'`. Luego, podemos importar la función `revalidateTag()` desde `'next/cache'` para usarla dentro de una server action.

```ts
'use server'

import { revalidateTag } from 'next/cache'

export async function updatePost(){
  revalidateTag('posts')
}
```

En el ejemplo anterior, la server action `updatePost()` llama a la función `revalidateTag()` en los datos almacenados en caché con la etiqueta `posts`.

De manera similar, también podemos revalidar por ruta usando la función `revalidatePath()` desde `'next/cache'`.

```ts
'use server'

import { revalidatePath } from 'next/cache'

export async function updatePost(){
  revalidatePath('/posts')
}
```

En el ejemplo anterior, la server action `updatePost()` llama a la función `revalidatePath()` en la ruta `/posts`.

Esta server action se puede usar en el envío de formularios en componentes cliente y servidor:

```tsx
import { updatePost } from './actions';

export default async function EditPost(){
  // lógica para el componente
  
  return(
    // devolver otros elementos
    <form action={updatePost}>
      <button type="submit">Update</button>
    </form>
  )
}
```

En el código anterior, proporcionamos la server action `updatePost()` como el valor del atributo `action` del formulario.

-----

## Loading UI

La obtención de grandes cantidades de datos puede llevar tiempo. Aquí es donde las interfaces de usuario de carga serán útiles, ya que se puede renderizar un estado de carga instantáneo en lugar de un segmento mientras se cargan los datos. Las interfaces de usuario de carga proporcionan retroalimentación visual a los usuarios de que los datos se están obteniendo actualmente. Una interfaz de usuario de carga puede ser un spinner, un esqueleto, un indicador de progreso o un mensaje de carga.

![loading-Ui](/Images/loading-ui.svg)

Recuerda que para crear un estado de carga instantáneo, creamos un archivo llamado `loading.tsx` en la carpeta que contiene el componente que usará la interfaz de usuario de carga. Dentro, definimos el componente `<Loading>` que contiene la interfaz de usuario de carga.

```tsx
export default function Loading(){
  return (
      <p>Cargando datos...</p>
  )
}
```

En el ejemplo de código anterior, tenemos nuestro componente `<Loading>` que muestra "Cargando datos..." como nuestro mensaje de carga.

Luego podemos usar el componente `<Loading>` como el fallback de un límite `<Suspense>`, que veremos más de cerca en el próximo ejercicio.

-----

## Streaming

En comparación con cómo el renderizado del lado del servidor puede ser secuencial y bloqueante, el uso de técnicas de streaming permite que una aplicación web renderice progresivamente y transmita incrementalmente partes de la interfaz de usuario al cliente.

![suspense](/Images/suspense.svg)

El streaming se puede lograr con el componente `<Suspense>`:

```jsx
return (
  <main>
    <Suspense>
      <UserProfile />
    </Suspense>
    <Suspense>
      <UserPosts />
    </Suspense>
  </main>
)
```

En el ejemplo de código anterior, cada uno de los componentes `<UserProfile>` y `<UserPosts>` está envuelto dentro de bloques `<Suspense>`. Esto permitirá que `<UserProfile>` y `<UserPosts>` se carguen independientemente uno del otro y se rendericen en el cliente tan pronto como cada componente esté listo.

Podemos proporcionar una interfaz de usuario de carga como fallback de cada límite de `<Suspense>`. La interfaz de usuario de carga se renderizará en lugar del componente dentro del `<Suspense>` hasta que todas las acciones dentro del componente se completen y esté listo para ser renderizado.

```jsx
<Suspense fallback={<Loading />} >
   <UserPosts />
</Suspense>
```

Aquí, hasta que nuestro componente `<UserPosts>` esté listo para ser renderizado, la interfaz de usuario de carga definida en el componente `<Loading>` se mostrará en su lugar.

----

## Server-Only Forms

Hemos visto cómo obtener y usar datos en nuestras aplicaciones Next.js. Ahora, veamos cómo podemos obtener, validar y procesar datos del usuario.

Para hacer esto, crearemos un formulario exclusivo del servidor mediante un componente de servidor y Server Actions.

Primero, crearemos un componente de servidor:

```tsx
// components/FeedbackForm/FeedbackForm.tsx
export default function FeedbackForm(){
  return(
    <form>
      <label>
        Nombre:
        <input type="text" name="name" required />
      </label>
      <label>
        Correo electrónico:
        <input type="email" name="email" />
      </label>
      <label>
        Comentario:
        <textarea name="feedback" required />
      </label>
      <button type="submit">Enviar</button>
    </form>
  )
}
```

En el código anterior, tenemos un formulario con tres campos:

- Un campo de entrada de nombre requerido de tipo texto.
- Un campo de entrada de correo electrónico de tipo email.
- Un área de texto de comentario requerida.

Aquí, usamos la validación de formulario de HTML estableciendo el tipo apropiado para cada campo y usando la validación `required` donde sea necesario.

Para manejar el formulario, crearemos una Server Action. Recuerda que podemos crear una Server Action usando la directiva `'use server'`.

```ts
// components/FeedbackForm/actions.ts
'use server'

export async function handleFeedback( formData: FormData ){
  const rawFormData = {
    name: formData.get('name') as string || '',
    email: formData.get('email') as string || '',
    feedback: formData.get('feedback') as string || '',
  }
  
  // hacer más cosas con los datos del formulario
}
```

En el código anterior, creamos un archivo llamado `actions.ts` en la misma carpeta donde se encuentra nuestro componente `<FeedbackForm>`. La función `handleFeedback()` aceptará y procesará los datos del formulario.

Ahora que hemos definido la Server Action, podemos importarla y llamarla como el atributo `action` de `<form>`.

```tsx
// components/FeedbackForm/FeedbackForm.tsx
import { handleFeedback } from './actions'

export default function FeedbackForm(){
  return(
    <form action={handleFeedback}>
      {/* más campos de formulario */}
    </form>
  )
}
```

Aquí, hemos importado la función `handleFeedback()` desde `actions.ts` y la hemos pasado como el valor del atributo `action` del elemento `<form>`.

Una vez que hayamos terminado de procesar los datos del formulario en nuestra server action `handleFeedback()`, podemos redirigir a los usuarios a una nueva ruta usando la función `redirect()` de `'next/navigation'`.

```ts
// components/FeedbackForm/actions.ts
'use server'

import { redirect } from 'next/navigation'

export async function handleFeedback( formData: FormData ){
  // procesar los datos del formulario
  
  redirect('/feedback/thankyou');
}
```

En nuestro ejemplo de código anterior, después de que se hayan procesado los datos del formulario, llamamos a la función `redirect()` para redirigir a los usuarios a la ruta `/feedback/thankyou`.

-----

## Review

En esta lección, aprendimos cómo obtener datos tanto en el servidor como en el cliente. Repasemos:

- Idealmente, los datos deberían obtenerse en el servidor, ya que el servidor tiene acceso directo a la base de datos, se pueden reducir los waterfalls entre cliente y servidor, los datos se pueden obtener y renderizar en el mismo entorno, y aumenta la seguridad al no exponer datos sensibles al cliente.
- Los **Route Handlers** se utilizan para definir manejadores de solicitudes personalizados para obtener datos en el cliente.
- `fetch()` se puede llamar directamente dentro de un componente de servidor.
- El paquete **'server-only'** se utiliza para evitar que un componente de servidor sea enviado al cliente.
- Desde Next.js 15, las peticiones `fetch()` no se cachean por defecto. Para optar por el cacheo, se pasa `cache: 'force-cache'` en el objeto de opciones de una llamada `fetch` (en versiones anteriores a Next.js 15 el default era el opuesto, y `cache: 'no-store'` era lo que había que pasar para desactivarlo).
- Cuando se utiliza el **patrón de obtención de datos secuencial**, las solicitudes crean waterfalls a medida que ocurren una después de la otra.
- Cuando se utiliza el **patrón de obtención de datos en paralelo**, las solicitudes en una ruta ocurren al mismo tiempo.
- Los patrones de obtención de datos en paralelo se pueden optimizar **precargando datos**.
- Los datos almacenados en caché se pueden revalidar de dos formas: **revalidación basada en tiempo** y **revalidación bajo demanda**.
- La **revalidación basada en tiempo** revalida los datos después de una duración específica en segundos.
- Una **Server Action** es una función asíncrona que se ejecuta en el servidor. Para crear una server action, usa la directiva `'use server'`.
- La **revalidación bajo demanda** revalida los datos basándose en una ruta o una etiqueta cuando se activa un evento particular.
- Las **interfaces de usuario de carga (Loading UIs)** se renderizan en lugar de un segmento mientras se cargan los datos.
- El **streaming** usando límites `<Suspense>` permite que una aplicación web renderice progresivamente y transmita incrementalmente partes de la interfaz de usuario al cliente.
- Los **formularios exclusivos del servidor** se pueden crear en un componente de servidor utilizando Server Actions.

----