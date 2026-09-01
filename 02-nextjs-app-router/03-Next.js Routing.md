# Next.js Routing

## File-based Routing

Cuando exploramos una aplicación web, el contenido se nos presenta según la URL (Localizador Uniforme de Recursos). Esto se conoce como enrutamiento.

Recuerda que el enrutamiento utiliza la ruta de la URL, que viene después del dominio de la URL y está compuesta por segmentos de URL delimitados por barras inclinadas (/) para mostrar contenido. Por ejemplo, en la siguiente URL:

```
https://www.codecademy.com/catalog/subject/artificial-intelligence
```

- El dominio es `codecademy.com`.
- La ruta es `/catalog/subject/artificial-intelligence`.
- Los segmentos son `/catalog`, `/subject` y `/artificial-intelligence`.

Next.js utiliza estos mismos conceptos para ayudarnos a construir y organizar nuestra aplicación web en su sistema de enrutamiento basado en archivos. En el enrutamiento basado en archivos de Next.js, una carpeta determina un segmento de la ruta URL, y los archivos reservados determinan qué contenido se muestra.

Podemos visualizar esta estructura de enrutamiento como un árbol donde:

- Cada carpeta es un nodo en el árbol.
- La carpeta superior es el nodo raíz.
- Cada subcarpeta es el inicio de un subárbol.
- Una carpeta es un nodo hoja si no contiene subcarpetas.

Recuerda que creamos una estructura como esta cuando añadimos `page.tsx` a la carpeta `/app`, ya que Next.js crea un App Router para nosotros cuando añadimos una carpeta `/app` a nuestra aplicación, que sirve como la raíz de nuestro árbol.

A medida que continuemos con esta lección, aprenderemos más sobre los archivos reservados de Next.js y su jerarquía, así como sobre la creación de rutas anidadas, la creación de rutas dinámicas y la navegación.

------

## Basic Routes

Hasta este punto, hemos creado una carpeta `/app` y añadido un archivo `page.tsx` a la misma. `page.tsx` es uno de los muchos archivos reservados en Next.js. Recuerda que crear una carpeta no crea automáticamente una ruta que los usuarios puedan visitar. Debemos añadir un archivo `page.tsx` a la carpeta y exportar por defecto un componente React para informar al App Router que la carpeta (segmento de ruta) es accesible.

Además de `page.tsx`, Next.js utiliza un archivo llamado `layout.tsx` para definir una interfaz de usuario compartida en todos los segmentos de ruta anidados. Para definir la interfaz de usuario, exportamos por defecto un componente React que acepta una prop llamada `children` de tipo `ReactNode`, de la siguiente manera:

```tsx
import { ReactNode } from "react";

// Componente React en layout.tsx 
function MyLayout({ children } : { children: ReactNode }) {  // props desestructuradas 
  return (
    <div>  
      <p>¡Las interfaces anidadas siempre me verán arriba de ellas!</p>
      <section>
        {children}
      </section>
    </div>
  )
}

// Nota: la exportación por defecto es obligatoria
export default MyLayout;
```

Debido a que `layout.tsx` es una interfaz de usuario compartida, Next.js requiere que cada aplicación tenga al menos un `layout.tsx` en el segmento superior, conocido como layout raíz. Este layout raíz debe contener los elementos `<html>` y `<body>`. Un aspecto importante de `layout.tsx` es que mantienen su estado durante la navegación entre segmentos anidados y no se vuelven a renderizar.

Hemos hablado mucho sobre segmentos anidados, pero ¿cómo se crea uno? Para crear un segmento anidado, creamos una nueva carpeta (segmento) dentro de otra carpeta (segmento). Por ejemplo, observemos la siguiente estructura de carpetas:

```
├── app
│   ├── settings
│   │   ├── page.tsx
│   │   ├── billing
│   │   │   ├── page.tsx 
│   ├── info
│   │   ├── MyComponent.tsx
│   ├── layout.tsx
```

La estructura de carpetas anterior:

- Contiene el layout raíz compartido requerido.
- Contiene dos carpetas anidadas dentro de `/app`: `/settings` y `/info`.
- Contiene una ruta anidada accesible (`/settings/billing`) al anidar los segmentos `/settings` y `/billing`.
- Contiene una ruta accesible (`/settings`) al usar el segmento `/settings`.
- Contiene una ruta `/info` no accesible porque su carpeta no contiene un archivo `page.tsx`.

Ahora, practicaremos la creación de rutas básicas y anidadas en los siguientes puntos de control. Durante el resto de esta lección, practicarás lo que has aprendido para completar la aplicación de lectura de artículos de Codecademy.

----

## Dynamic Routes

Anteriormente, vimos cómo podemos crear rutas anidadas para definir interfaces de usuario únicas basadas en la ruta URL. ¿Qué pasa si nuestra ruta está mostrando contenido relativo a un usuario específico? Por ejemplo, si nuestra aplicación muestra la información relevante del usuario dependiendo del ID de usuario al que está destinado el contenido, ¿deberíamos definir segmentos de ruta para todos los IDs posibles como `/users/10` o `/users/20`?

La respuesta es no. Esto no solo sería tedioso, ya que la ruta para cada usuario contendrá mayormente la misma interfaz de usuario, sino que carece de escalabilidad a medida que la base de usuarios crece. Para manejar situaciones como esta, podemos usar segmentos dinámicos. Los segmentos dinámicos son porciones dinámicas de una URL que pueden cambiar (como `/users/10` y `/users/20`).

Para definir un segmento dinámico en Next.js:

1. Crea un segmento accesible (carpeta) como aprendimos en el ejercicio anterior.
2. El nombre de la carpeta para un segmento dinámico debe estar envuelto en corchetes (`[]`).
3. El nombre de la carpeta servirá como identificador que usaremos para recuperar el segmento dinámico en nuestro `page.tsx`.

Por ejemplo, la siguiente estructura de carpetas mostrará contenido para rutas como `/users/10` y `/users/20`:

```
├── app
│   ├── users
│   │   ├── [userId]
│   │   │   ├── page.tsx
```

Accedemos a los datos del segmento dinámico haciendo referencia al nombre de la carpeta como una propiedad de la prop `params` en el componente `page.tsx`. Observa el siguiente ejemplo:

```tsx
// desde Next.js 15, params es una Promise: hay que resolverla con await
export default async function MyUserPage({ params }: { params: Promise<{ userId: string }> }) {
  const { userId } = await params;
  return (
    <h1>Saludos usuario: {userId}</h1>
  )
}
```

En este ejemplo:
- Desestructuramos una prop `params`.
- Definimos el tipo de `params` como una `Promise` que resuelve a un objeto con la propiedad `userId`. Desde Next.js 15, `params` (y `searchParams`) se volvieron asíncronos para permitirle al framework empezar a renderizar la página antes de que el segmento dinámico esté resuelto, así que el componente tiene que ser `async` y hacerle `await` a `params` antes de usarlo.
- Las propiedades de `params` son siempre de tipo `string` (o `string[]` — veremos esto más adelante en la lección).
- Accedemos a los datos del segmento dinámico haciendo `await params` y luego leyendo la propiedad `userId` del objeto resuelto.

A medida que trabajamos con segmentos dinámicos, puede que queramos almacenar algunos análisis sobre el número de usuarios que han navegado a la página. Esto suena como un buen lugar para usar una interfaz de usuario compartida en un archivo `layout.tsx`, pero, debido a que los layouts no se vuelven a renderizar al navegar entre segmentos anidados, no podremos volver a ejecutar nuestra llamada a la API.

Para abordar este problema, podemos usar el archivo reservado `template.tsx`. `template.tsx` es similar a `layout.tsx` en que:

- `template.tsx` también define una interfaz de usuario compartida para sus segmentos anidados.
- Debe exportar por defecto un componente React.
- El componente exportado por defecto recibe una prop `children`.

Pero se diferencia en que se vuelve a instanciar al navegar entre segmentos anidados (aprenderemos más sobre navegación a continuación). Podemos aprovechar esta característica llamando a una API para actualizar el contador de visitas de nuestro usuario, de la siguiente manera:

```tsx
// declaraciones de importación

// se vuelve a instanciar al navegar entre segmentos anidados
export default function MyTemplate({children}: {children : ReactNode}) {
  useEffect(() => {
    updateUsersCounter()  // llamada a la API
  }, [])  // se ejecuta al montar
  // otra lógica para MyTemplate
}
```

Practiquemos agregar una ruta dinámica y usar `template.tsx` en nuestra aplicación.

---------

## Using The <Link> Component

Al explorar aplicaciones web, a menudo utilizamos enlaces para navegar.

Next.js proporciona varias formas de ofrecernos una navegación tipo Single Page Application (SPA) en el navegador. Recuerda que la navegación SPA se refiere a la idea de cambiar la ruta del navegador sin necesidad de hacer una nueva solicitud al servidor. En este ejercicio, exploraremos el componente `<Link>`.

Un componente `<Link>` se puede usar de la siguiente manera:

```jsx
const selectedUser = "25"  // id de usuario seleccionado
<section>
  <Link href="/users">Users</Link>
  <Link href={`/users/${selectedUser}`}>User: {selectedUser}</Link>  {/* ruta dinámica */}
  <Link href="/settings" replace>My Settings</Link> {/* reemplaza la ruta actual en el historial del navegador */}
  <Link href="/info">Info</Link>
</section>
```

Donde:

- La prop `href` determina la ruta a la que queremos navegar.
- La prop `href` se puede usar para enlazar a un segmento dinámico creando la ruta dinámica (en el ejemplo anterior, usamos una plantilla literal).
- El contenido de texto ("Users", "My Settings", "Info") es el texto que se muestra en la interfaz de usuario.
- La prop `replace` se usa para reemplazar la ruta URL actual con la ruta `href`.
- El componente `<Link>` es una extensión del elemento `<a>` que añade funcionalidad de prefetching. Con el prefetching, Next.js precargará los segmentos de ruta automáticamente, de modo que cuando un usuario navegue a esos segmentos, el navegador no necesite recargar.

Los componentes `<Link>` se pueden usar junto con el hook `usePathname()` del paquete `next/navigation` para aplicar estilos "activos" al `<Link>`. `usePathname()` devuelve la ruta actual en la URL como una cadena y se puede usar para aplicar estilos a un `<Link>` de la siguiente manera:

```jsx
'use client'
import Link from 'next/link'
import { usePathname } from "next/navigation";

const pathname = usePathname()  // ruta actual: /users
<section>
  <Link href="/users" className={pathname === "/users" ? styles.active : ""}>Users</Link> {/* actualmente activo */}
  <Link href="/info" className={pathname === "/info" ? styles.active : ""}>Info</Link> {/* no activo */}
</section>
```

Ten en cuenta que `usePathname()` solo se puede usar dentro de componentes cliente, por lo que utilizamos la directiva `'use client'`.

Practiquemos añadiendo enlaces de navegación a nuestro encabezado para que los usuarios tengan más facilidad para navegar entre nuestros diferentes segmentos de URL.

----

## The useRouter() Hook

Hemos aprendido a usar el componente `<Link>` para navegar entre segmentos de URL, pero a menudo nos gustaría navegar programáticamente desde un segmento de URL. Por ejemplo, si un usuario intenta acceder a un segmento de URL sin haber iniciado sesión, podemos redirigirlo automáticamente a una página de registro.

Next.js proporciona el hook `useRouter()`, que forma parte del paquete `next/navigation`, que podemos usar para realizar navegación tipo SPA de forma programática en componentes cliente. Al llamar a `useRouter()` se devuelve un objeto router que contiene los siguientes métodos:

- **push(path)**: Navega a `path` añadiendo `path` a la parte superior del historial del navegador.
- **back()**: Navega hacia atrás una entrada en el historial del navegador.
- **forward()**: Navega hacia adelante una entrada en el historial del navegador.
- **replace(path)**: Navega a `path` reemplazando la parte superior del historial del navegador con `path`.

Podemos usar estos métodos de la siguiente manera:

```jsx
'use client'  // directiva cliente requerida
// otros imports
import { useRouter } from "next/navigation";  // importación

export default function Authentication() {
  const router = useRouter();  // obtener el objeto router

  useEffect(() => {
    if(!isAuthenticated()) {
      router.replace("/sign-up")  // redirige al usuario reemplazando la ruta actual y enviándolo a /sign-up
    }
  }, [])
  return (
      // usa router.back() como callback  
      <button onClick={router.back}>Return</button>
  ) 
}
```

Para resumir, si necesitamos navegar usando la estructura de nuestra aplicación (como enlaces estáticos), usamos el componente `<Link>`, pero si necesitamos navegación programática (como redireccionamientos), usamos el hook `useRouter()`.

Practiquemos el uso de la navegación programática en nuestra aplicación.

----

## Managing Unpredictable URLs

Hemos hablado sobre cómo podemos crear segmentos dinámicos y segmentos anidados, pero ¿qué pasa si nuestra aplicación admite cualquier número de los mismos segmentos dinámicos? Por ejemplo, si nuestra aplicación tiene una función para comparar detalles de uso para sus usuarios, las rutas URL podrían verse así:

- `/users/123`: Muestra detalles de uso para el usuario con ID 123.
- `/users/123/987`: Muestra la comparación de detalles de uso para los usuarios con ID 123 y 987.

¿Cómo deberíamos manejar esto? ¿Deberíamos usar múltiples segmentos dinámicos como `/users/[userId1]/[userId2]/...`? No exactamente. Este enfoque tendría el mismo problema de escalabilidad que resolvimos usando segmentos dinámicos. En cambio, Next.js proporciona una solución de segmentos catch-all.

Los segmentos catch-all, como su nombre indica, coincidirán con todas las rutas como `/users/123` y `/users/123/987` usando un único segmento dinámico. Para crear uno, definimos nuestro segmento dinámico como hemos hecho antes (usando una carpeta y `[]`), pero también añadimos un prefijo de puntos suspensivos (`...`) al nombre del segmento dinámico, como:

```
├── app
│   ├── users
│   │   ├── [...userIds]  // segmento dinámico catch-all
│   │   │   ├── page.tsx
```

Para acceder a los datos en `userIds`, recibiremos los `params` en nuestro componente `page.tsx` de manera similar, excepto que el tipo de nuestra propiedad ya no será una cadena, sino `string[]`. Por ejemplo:

```tsx
// params sigue siendo una Promise: la resolvemos con await
export default async function MyUserPage({ params }: { params: Promise<{ userIds: string[] }> }) {
  const { userIds } = await params   // resolvemos la Promise y desestructuramos userIds
  return (
    <section>
      {/* mostramos datos para todos los userIds */}
      {userIds.map((userId) => (
        <UsageDetails key={userId} userId={userId}/>
      ))}
    </section>
  )
}
```

Este ejemplo recuperará todos los `userIds` del segmento dinámico catch-all `[...userIds]` de una propiedad `params` llamada `userIds` de tipo `string[]`.

Antes de finalizar con los segmentos dinámicos, añadamos una nueva función donde un usuario pueda seleccionar los usuarios para comparar si navega al segmento URL base `/users`. Navegar a esa ruta ahora dará un error 404.

Una forma de solucionar esto es aprovechando los segmentos catch-all opcionales de Next.js. Los segmentos catch-all opcionales, como su nombre indica, son opcionales y coincidirán con rutas como: `/users`, `/users/123` y `/users/123/987`. Para crear uno, envolvemos otro par de corchetes (`[]`) alrededor de nuestra carpeta de segmento catch-all, como `[[...userIds]]`.

Para acceder a nuestros datos, hacemos una ligera modificación de tipo donde la propiedad ahora es opcional, como:

```tsx
// params sigue siendo una Promise: la resolvemos con await
export default async function MyUserPage({ params }: { params: Promise<{ userIds?: string[] }> }) {
  const { userIds } = await params
  // Usando encadenamiento opcional
  // En la ruta /users, `userIds` no existirá en el objeto

  // ...cuerpo
}
```

Gran trabajo aprendiendo sobre los segmentos dinámicos catch-all. Practiquemos usarlos añadiendo uno a nuestra aplicación.

-----

## Reserved File Names and Component Hierarchy

Hasta ahora, hemos aprendido cómo Next.js utiliza carpetas y archivos reservados para construir segmentos de ruta e interfaces de usuario.

Observa que cada archivo reservado que hemos visto hasta ahora tiene un único propósito con funcionalidad especial, por ejemplo:

- **page.tsx**: Hace accesible un segmento URL y muestra una interfaz de usuario única.
- **layout.tsx**: Crea una interfaz de usuario compartida y envuelve cualquier interfaz de usuario anidada, mantiene el estado en la navegación de segmentos anidados.
- **template.tsx**: Crea una interfaz de usuario compartida y envuelve cualquier interfaz de usuario anidada, se reinstancia en la navegación de segmentos anidados.

Estos son solo algunos de los archivos reservados que Next.js proporciona. Exploremos tres más de los más utilizados:

- **error.tsx**: Crea un componente ErrorBoundary utilizando la interfaz de usuario definida para el segmento actual y sus segmentos anidados.
- **not-found.tsx**: Crea un componente ErrorBoundary utilizando la interfaz de usuario definida para errores 404 para el segmento actual y sus segmentos anidados.
- **loading.tsx**: Crea un componente Suspense para el segmento actual y sus segmentos anidados (exploraremos esto completamente en una lección futura).

Se definen como cualquier otro archivo reservado (crea el archivo en un segmento) y exporta un componente React. Por ejemplo:

```tsx
/* en error.tsx */

'use client' // Deben ser componentes cliente
// recibe el objeto error y la función reset como props
export default function MyErrorBoundary( { error, reset }: { error: Error; reset: () => void} ) {
  // cuerpo
}

/* en not-found.tsx */
export default function MyNotFoundUI() {
  return (
    <>
      <h1>¡Lo siento, no reconozco esta página!</h1>
      {/* resto de componentes */}
    </>
  )
}

/* en loading.tsx */
export default function Loading() {
  return <h1>Cargando contenido...</h1>
}
```

En el ejemplo, los componentes exportados de `not-found.tsx` y `loading.tsx` no reciben props. `error.tsx` recibe un objeto Error llamado `error` y un callback `reset` llamado `reset()`. Ten en cuenta que los componentes `error.tsx` deben ser componentes cliente y no capturan errores lanzados en `layout.tsx` o `template.tsx` (discutiremos esto a continuación).

Por último, probablemente te hayas preguntado cómo interactúan todos estos archivos entre sí. Por ejemplo, ¿por qué `error.tsx` no captura errores en `layout.tsx`/`template.tsx`, o puedes incluir un `layout.tsx` y un `template.tsx` en el mismo segmento? Las respuestas a ambas preguntas se encuentran en la jerarquía de componentes de Next.js.

La jerarquía de componentes establece cómo se renderizan todos los componentes exportados por defecto en los archivos reservados del segmento de ruta.

```
├── app
│   ├── layout.tsx
│   ├── template.tsx
│   ├── loading.tsx
│   ├── settings
│   │   ├── layout.tsx
│   │   ├── template.tsx
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   ├── not-found.tsx
```

Para una aplicación con la estructura de carpetas anterior,

```jsx
<Layout>
  <Template>
    <Suspense>
      <Layout>
        <Template>
          <ErrorBoundary>
            <Suspense>
              <ErrorBoundary>
                <Page/>
              </ErrorBoundary>
            </Suspense>
          </ErrorBoundary>
        </Template>
      </Layout>
    </Suspense>
  </Template>
</Layout>
```

y, en esta jerarquía:

- `<Layout>` es la raíz de un segmento.
- `<Template>` envuelve todo excepto `<Layout>`.
- `<ErrorBoundary>` envuelve `<Suspense>`, el `<ErrorBoundary>` de not-found y `<Page>`.
- `<Suspense>` envuelve `<ErrorBoundary/>` de not-found y `<Page>`.
- `<ErrorBoundary/>` de not-found envuelve `<Page>`.
- Los segmentos anidados siguen las mismas reglas de jerarquía y están envueltos completamente dentro de la jerarquía padre.

Comprender la jerarquía de componentes nos ayudará a dominar el enrutamiento de Next.js y entender dónde se necesitan ciertos archivos reservados. Practiquemos el uso de `not-found.tsx` en nuestra aplicación.

----

## Review

¡Excelente trabajo aprendiendo sobre enrutamiento en Next.js! En esta lección, has aprendido sobre:

- el App Router basado en sistema de archivos de Next.js y cómo crea una estructura de datos de tipo árbol
- los roles de las carpetas y los archivos reservados en la construcción de segmentos URL
- el archivo reservado de interfaz de usuario compartida `layout.tsx` y cómo preserva el estado en la navegación de segmentos anidados
- el archivo reservado de interfaz de usuario compartida `template.tsx` y cómo se reinstancia en la navegación de segmentos anidados
- la creación de segmentos URL anidados mediante carpetas anidadas
- la creación de segmentos URL dinámicos y cómo usar la prop `params` para recuperar los datos
- el uso de componentes `<Link>` para navegación declarativa
- el uso del hook `useRouter()` para navegación programática
- la creación de rutas dinámicas catch-all y catch-all opcionales
- los archivos reservados `error.tsx`, `loading.tsx` y `not-found.tsx` y sus casos de uso específicos
- la jerarquía de componentes de Next.js y cómo determina cómo se renderizan las cosas en la interfaz de usuario

En esta lección, cubrimos los conceptos fundamentales de enrutamiento en Next.js. Hay muchas más características que puedes aprender que se basan en los conceptos que has aprendido en esta lección.

-----