# Learn React Router v6

## What Is Routing?

El [enrutamiento](https://www.codecademy.com/resources/docs/react/routing) es el proceso por el cual una aplicación web usa la URL actual del navegador (Localizador Uniforme de Recursos) para determinar qué contenido mostrar al usuario. Por ejemplo, un usuario que visita la página /wiki/Node.js de Wikipedia espera ver algo diferente a la página /wiki/React_(JavaScript_library).

Al organizar el contenido de una aplicación y mostrar solo lo que el usuario ha solicitado ver, el enrutamiento permite experiencias de usuario ricas, atractivas y claras.

Antes de comenzar la lección, repasemos brevemente la estructura básica de las URLs. Considera esta URL:

![Desglose de la URL](/Images/url-dark.png)

Cada URL es básicamente una solicitud de algún recurso y cada componente de la URL sirve para especificar qué recurso se desea. Las URLs constan de varios componentes, algunos obligatorios y otros opcionales:

- El esquema (por ejemplo, HTTP, HTTPS, mailto, etc.), que especifica qué protocolo se debe usar para acceder al recurso.
- El dominio (por ejemplo, codecademy.com), que especifica el sitio web que aloja el recurso. El dominio sirve como punto de entrada para tu aplicación.
- La ruta (por ejemplo, /articles), que identifica el recurso o página específica que se debe cargar y mostrar al usuario. ¡Aquí es donde comienza el enrutamiento!
- La cadena de consulta opcional (por ejemplo, ?search=node), que aparece después de un ‘?’ y asigna valores a parámetros. Los usos comunes de las cadenas de consulta incluyen parámetros de búsqueda y filtros.

Dependiendo del tipo de aplicación que construyas, hay diferentes formas de manejar las solicitudes que llegan a tu servidor. Una solución popular en el back-end es usar el framework de enrutamiento Express. En esta lección, veremos React Router, una solución de enrutamiento de front-end diseñada específicamente para aplicaciones React.

-----

## Installing React Router

Para usar React Router, necesitas incluir el paquete react-router-dom (la versión de React Router creada específicamente para navegadores web) en tu proyecto usando npm así:

```sh
npm install --save react-router-dom@6
```

Una vez que hayas agregado react-router-dom a tu proyecto, puedes importar uno de sus routers para añadir enrutamiento a tu proyecto. React Router ofrece varios routers, pero el más común es createBrowserRouter. Las otras opciones y las razones para elegir una u otra están fuera del alcance de esta lección. Si te interesa, puedes leer más sobre opciones alternativas de enrutamiento [aquí](https://reactrouter.com/start/modes).

Para agregar un router a nuestro proyecto, podemos importarlo al inicio de nuestro archivo así:

```js
import { createBrowserRouter } from 'react-router-dom';
```

Luego, inicializamos nuestro router llamando a createBrowserRouter. Este método acepta una lista de componentes JSX (hablaremos más de esto en los próximos ejercicios):

```js
import { createBrowserRouter } from 'react-router-dom';
const router = createBrowserRouter( /* aquí se definen las rutas de la aplicación */ );
```

El router sirve como la base para toda la lógica de React Router. Sin él, aparecerían errores si intentamos usar componentes o métodos de React Router. Practiquemos cómo definir un router para nuestra aplicación.

------

## Providing A Router

En el paradigma de React Router, las diferentes vistas de tu aplicación, también llamadas rutas, son simplemente componentes de React. Para incluirlas en tu aplicación, necesitas renderizarlas.

El primer paso es hacer que nuestro router esté disponible para toda la aplicación usando el RouterProvider de React Router.

```js
import { RouterProvider, createBrowserRouter } from 'react-router-dom';
const router = createBrowserRouter( /* aquí se definen las rutas de la aplicación */ );

export default function App () {
  return (
    <RouterProvider router={ router } />
  );
}
```

`createBrowserRouter` define un router que evita que los cambios en la URL recarguen la página. En su lugar, los cambios en la URL permiten que el router determine cuál de sus rutas definidas debe renderizar, pasando información sobre la ruta actual como props. Hacemos que nuestro router esté disponible en toda la aplicación usándolo con RouterProvider en la raíz de la aplicación.

En los próximos ejercicios, aprenderemos a definir los componentes de las rutas para que puedan usar esta información, pero por ahora, agreguemos un router a nuestra aplicación.

-----

## Basic Routing with `<Route>`

Con nuestro router listo, podemos empezar a definir las diferentes vistas, o rutas, que nuestra aplicación mostrará para varias rutas de URL. Por ejemplo, podríamos querer mostrar un componente About para la ruta /about y un componente SignUp para la ruta /sign-up. React Router nos da dos opciones para definir rutas: usando JSX u objetos. En esta lección, aprenderemos a implementar rutas usando la notación JSX. Si te interesa, puedes aprender más sobre la notación de objetos aquí.

El método `.createBrowserRouter` acepta un arreglo de objetos `<Route>`, así que necesitaremos usar el método `.createRoutesFromElements` de React Router para configurar nuestras rutas con JSX. También necesitamos usar el componente `<Route>` de React Router para definir nuestras rutas. Estos componentes se pueden importar del paquete react-router-dom, junto con el método `.createBrowserRouter`, así:

```js
import { RouterProvider, createBrowserRouter, createRoutesFromElements, Route } from 'react-router-dom'
```

El componente `<Route>` está diseñado para renderizar (o no) un componente según la ruta actual de la URL. Cada componente `<Route>` debe incluir:

- Una prop path que indica la ruta exacta de la URL que hará que la ruta se renderice.
- Una prop element que describe el componente que se va a renderizar.

Por ejemplo, podemos usar `.createRoutesFromElements` para configurar una `<Route>` que renderice el componente `<About>` cuando la ruta de la URL sea '/about':

```js
import About from './About.js';
import { RouterProvider, createBrowserRouter, Route } from 'react-router-dom';
const router = createBrowserRouter(createRoutesFromElements(
  <Route path='/about' element={ <About/> } />
));

export default function App () {
  return (
    <RouterProvider router={ router } />
  );
}
```

En muchos casos, puede haber ciertos componentes, como barras laterales, barras de navegación y pies de página, que queremos mostrar en cada vista. Podemos lograr esto definiendo un componente de nivel raíz que contenga los elementos de diseño que queremos mostrar siempre. Luego, podemos anidar el resto de nuestras rutas dentro de este componente raíz, así:

```js
/* imports ... */

const router = createBrowserRouter(createRoutesFromElements(
  <Route path='/' element={ <Root/> }>
    // las rutas anidadas aquí se mostrarán junto con el componente <Root/>
  </Route>
));
```

Con esta configuración de rutas, cada vez que un usuario navegue a una de las rutas anidadas, esa vista se mostrará junto con cualquier elemento que hayamos definido en nuestro componente `<Root/>`. Hablaremos más sobre componentes de nivel raíz y componentes anidados en los próximos ejercicios. Antes de continuar, practiquemos cómo agregar rutas a nuestra aplicación para que muestre el componente correcto cuando visitemos ciertas rutas.

-----

## Linking to Routes

En el último ejercicio, usaste la barra de URL para navegar a una ruta que coincidía con una de las rutas de tu aplicación. Pero ¿cómo se navega desde dentro de la propia app?

Al construir un sitio web con HTML, usamos la etiqueta de ancla (`<a>`) para crear enlaces a otras páginas. Un efecto secundario de la etiqueta `<a>` es que provoca que la página se recargue. ¡Esto puede distraer a nuestros usuarios de la experiencia inmersiva de una aplicación React fluida!

React Router ofrece dos soluciones para esto: los componentes **Link** y **NavLink**, ambos importables desde el paquete `react-router-dom`.

```js
import { createBrowserRouter, createRoutesFromElement, Route, Link, NavLink } from 'react-router-dom';
```

Tanto los componentes **Link** como **NavLink** funcionan de forma muy similar a las etiquetas de ancla:

* Tienen una prop `to` que indica la ubicación a la que se redirige al usuario, similar al atributo `href` de una etiqueta `<a>`.
* Envuelven HTML que se usa como el contenido visible del enlace.

```jsx
<Link to="/about">About</Link>
<NavLink to="/about">About</NavLink>
```

Ambos enlaces generarán etiquetas de ancla (`<a>`) con el texto “About”, que llevarán al usuario a la vista `/about` al hacer clic. Sin embargo, el comportamiento predeterminado de una etiqueta `<a>` —recargar la página— estará deshabilitado.
Ten en cuenta que si anteponemos una barra inclinada (`/`) a la ruta proporcionada en la prop `to`, como hicimos en el ejemplo anterior, esa ruta se tratará como una ruta absoluta. Es decir, React Router asumirá que la navegación comienza desde el directorio raíz. Hablaremos más adelante sobre cómo definir rutas relativas a nuestro directorio actual en los próximos ejercicios.

Entonces, ¿cuál es la diferencia entre **Link** y **NavLink**? Cuando la ruta de la URL coincide con la prop `to` de un componente **NavLink**, el enlace recibirá automáticamente una clase `'active'`. Esto es muy útil para crear menús de navegación, ya que podemos definir estilos CSS para la clase `.active` y así diferenciar entre enlaces activos e inactivos, permitiendo a los usuarios ver rápidamente qué contenido están visualizando.
También podemos pasar una función a `className` o `style` para personalizar aún más el estilo de un **NavLink** activo (o inactivo), por ejemplo:

```jsx
<NavLink 
  to="about" 
  className={ ({ isActive }) => isActive ? 'activeNavLink' : 'inactiveNavLink' }
>
  About
</NavLink>
```

En el ejemplo anterior, pasamos una función a la prop `className` que aplica la clase `activeNavLink` si el **NavLink** está activo, y `inactiveNavLink` en caso contrario.

Como hemos visto en este ejercicio, **NavLink** y **Link** son herramientas excelentes para ayudar a nuestros usuarios a navegar por nuestro sitio web. Practiquemos el uso de **Link** y **NavLink** en nuestra aplicación para que nuestros usuarios puedan moverse más fácilmente por ella.

---

## Dynamic Routes

Hasta ahora, todas las rutas que hemos visto han sido **estáticas**, lo que significa que coinciden con una única ruta específica. Esto funciona para rutas predeterminadas que muestran una vista consistente. Pero ¿qué pasa si necesitamos construir una ruta que sea más flexible?

Por ejemplo, imagina un sitio de noticias tecnológicas donde cada artículo es accesible desde la ruta `/articles/` + `algun-titulo`, donde `algun-titulo` es único para cada artículo. Especificar una ruta diferente para cada artículo no solo sería verboso y llevaría mucho tiempo, sino que también requeriría una cantidad de mantenimiento poco práctica si la estructura de la ruta llegara a cambiar o si necesitamos agregar nuevos artículos.

Sería mucho más conveniente definir una **sola ruta** que pueda coincidir con cualquier ruta con el patrón `/articles/` + `algunTitulo` y saber exactamente qué componente renderizar. **React Router** nos permite hacer esto mediante el uso de **parámetros de URL** para crear **rutas dinámicas**.

Los **parámetros de URL** son segmentos dinámicos de una URL que actúan como marcadores de posición para recursos más específicos. Aparecen en una ruta dinámica como dos puntos (`:`) seguidos de un nombre de variable, de la siguiente manera:

```jsx
const route = createBrowserRouter(createRoutesFromElement(
  <Route path='/articles/:title' element={ <Article /> }/>
))
```

Analicemos esta breve pero poderosa demostración de parámetros de URL:

* En este ejemplo, la prop `path='/articles/:title'` contiene el parámetro de URL **:title**.
* Esto significa que cuando el usuario navega a páginas como `/articles/what-is-react` o `/articles/html-and-css`, se renderizará el componente `<Article />`.
* Cuando el componente `Article` se renderiza de esta manera, puede acceder al **valor real** de ese parámetro de URL **:title** (`what-is-react` o `html-and-css`) para determinar qué artículo mostrar (más sobre esto en el próximo ejercicio). Una sola ruta puede incluso tener **múltiples parámetros** (ej. `articles/:title/comments/:commentId`) o ninguno (ej. `articles`).

Aprovechemos las rutas dinámicas usando parámetros de URL en nuestra aplicación.

-----

## useParams

Es común utilizar el valor de los **parámetros de URL** para determinar lo que se muestra en el componente que renderiza una ruta dinámica. Por ejemplo, el componente `Article` podría necesitar mostrar el título del artículo actual.

**React Router** proporciona un hook, **useParams()**, para acceder al valor de los parámetros de URL. Cuando se llama, `useParams()` devuelve un objeto que asigna los nombres de los parámetros de URL a sus valores en la URL actual.

Por ejemplo, considera el componente `Article` que se muestra a continuación, el cual es renderizado por una ruta con la URL dinámica `/articles/:title`. Observa el siguiente componente `Article`, que se renderizará cuando un usuario visite `/articles/objects`:

```jsx
import { Link, useParams } from 'react-router-dom';

export default function Article() {
  
  let { title } = useParams();
  // title será igual al string 'objects'

  // El título se renderizará en el <h1>
  return (
    <article>
      <h1>{title}</h1>
    </article>
  );
}
```

Analicemos el ejemplo anterior:

* Primero, el hook **useParams()** se importa desde `react-router-dom`.
* Luego, dentro del componente `Article`, se llama a `useParams()` que devuelve un **objeto**.
* Se utiliza la **asignación por desestructuración** para extraer el valor del parámetro de URL de ese objeto, almacenándolo en una variable llamada `title`.
* Finalmente, este valor de `title` se utiliza para mostrar el nombre del artículo en el elemento `<h1>`.

Practiquemos el uso de parámetros de URL en nuestros componentes.

----

## Nested Routes

Hasta este punto, hemos estado trabajando con enrutadores que son relativamente pequeños. A medida que nuestra aplicación crece y agregamos más funciones, es posible que deseemos que componentes adicionales se rendericen dentro de nuestras vistas definidas, dependiendo de las acciones del usuario.

Por ejemplo, supongamos que tenemos una página "Acerca de" (About) que se renderizará si accedemos a la ruta `/about`. Nos gustaría implementar una nueva función que muestre un mensaje secreto en "Acerca de" si la ruta cambia a `/about/secret`. Podríamos intentar hacer esto:

```jsx
/* imports ... */
const router = createBrowserRouter(createRoutesFromElement([
  <Route path='/about' element={ <About/> } />,
  <Route path='/about/secret' element={ <Secret/> } />
]));
```

Dado que React Router coincide exactamente con las rutas, si vamos a la ruta `/about/secret`, solo renderizará `Secret` y no `About`. Nos gustaría renderizar `About` cuando accedamos a `/about` y también renderizar `Secret` debajo de `About` cuando accedamos a `/about/secret`. Podemos hacer esto usando **rutas anidadas**.

Una **ruta anidada**, como su nombre indica, es una **Route** dentro de otra **Route**. Una **Route** que contiene una o más **Routes** anidadas dentro de ella se conoce como la **ruta padre**, y una **Route** que está contenida dentro de otra **Route** se conoce como la **ruta hija**. Al anidar **Routes**, la ruta de la **ruta hija** es relativa a la ruta de la **ruta padre**, por lo que no debemos incluir la ruta del padre en su prop `path`.

Por ejemplo, podemos crear una ruta anidada refactorizando el código anterior, de la siguiente manera:

```jsx
/* imports ... */
const router = createBrowserRouter(createRoutesFromElement(
  <Route path='/about' element={ <About/> }> {/* About se renderiza si la ruta comienza con /about */}
    <Route path='secret' element={ <Secret/> } />  {/* podemos excluir /about de esta ruta ya que es relativa a su padre */}
  </Route> 
));
```

Usando esta ruta anidada, el componente `About` se renderizará cuando la ruta comience con `/about`. Si la ruta coincide con `/about/secret`, el componente `Secret` se renderizará además del componente `About`. Recuerda que una **Route** puede ser tanto padre como hija si está anidada dentro de una ruta y contiene rutas anidadas dentro de ella. Se aplicarían las mismas propiedades de padre/hijo.

Nuestro enrutador está configurado para renderizar nuestra ruta anidada, pero si ejecutáramos este código, todavía no veríamos `Secret` renderizado debajo de `About`. Esto se debe a que no le hemos dicho a `About` dónde renderizar el elemento de su ruta hija. Para hacer esto, necesitamos hacer uso del componente **Outlet** de React Router en el componente padre `About`, de la siguiente manera:

```jsx
import { Outlet } from 'react-router-dom';

// Se renderiza cuando el usuario visita '/about'
export default function About() {
  return (
    <main>
       <h1>Lorem ipsum dolor sit amet.</h1>
       <Outlet/>  {/* renderiza el elemento hijo cuando el usuario visita /about/secret */}
    </main>   
  );
}
```

Ahora, cuando visitamos `/about/secret`, nuestro enrutador renderizará `About` y su componente de ruta hija, `Secret`, donde sea que esté definido el componente `Outlet` en el padre. Puedes pensar que es como si el enrutador reemplazara `Outlet` con nuestra ruta hija definida.

Al usar rutas anidadas, también podemos hacer uso de las **rutas índice (index routes)**. Una **ruta índice** es una **Route** que usa la prop `index` en lugar de una prop `path` y es especial porque se renderiza en la ruta de su padre. Por ejemplo:

```jsx
/* imports ... */
const router = createBrowserRouter(createRoutesFromElement(
  <Route path='/about' element={ <About/> }> {/* About se renderiza si la ruta comienza con /about */}
    <Route index element={ <IndexComponent/> } />  {/* Se renderizará cuando la ruta sea /about */}
    <Route path='secret' element={ <Secret/> } />  {/* Se renderizará cuando la ruta sea /about/secret */}
  </Route> 
));
```

Podemos pensar en una **ruta índice** como una **Route** predeterminada que se renderizará en el `Outlet` de su padre cuando la ruta coincida exactamente con la ruta del padre, para que haya algo de contenido en ese espacio.

Las **rutas anidadas** nos brindan un control preciso sobre qué, cuándo y dónde aparecen ciertos elementos dentro de su **Route** padre. Practiquemos lo que hemos aprendido agregando algunas rutas anidadas a nuestra aplicación.

----

## `<Navigate>`

Si hay algo que debes aprender de esta lección, es que **React Router trata todo como un componente**. Para sentirte completamente cómodo usando React Router en tu código, debes adoptar esta idea y el estilo de codificación declarativo que se deriva de ella. En su mayor parte, esto es bastante intuitivo, pero puede parecer un poco contradictorio cuando se trata de redirigir a los usuarios.

Para apreciar el patrón declarativo, considera un caso común para redirigir a un usuario: un usuario quiere acceder a una página `/profile` que requiere autenticación, pero no ha iniciado sesión actualmente.

El componente **Navigate** proporcionado por React Router hace que esto sea fácil! Al igual que `Link` o `NavLink`, el componente `Navigate` tiene una prop `to`. Sin embargo, mientras que `Link` y `NavLink` deben ser clickeados para navegar al usuario, una vez que el componente `Navigate` se renderiza, el usuario será llevado automáticamente a la ubicación especificada por la prop `to`:

```jsx
import { Navigate } from 'react-router-dom';

const UserProfile = ({ loggedIn }) => {
  if (!loggedIn) {
    return (
      <Navigate to='/' />
    )
  }

  return (
    // ... contenido del perfil de usuario aquí
  )  
}
```

En este ejemplo, cuando el componente `UserProfile` se renderiza, si la prop `loggedIn` es `false`, entonces el componente `Navigate` será retornado y luego renderizado, enviando al usuario a la página `/`. De lo contrario, el componente se renderizará normalmente.

Practiquemos la redirección declarativa de usuarios que no han iniciado sesión en nuestra aplicación.

----

## useNavigate

En el ejercicio anterior, aprendiste cómo redirigir de forma declarativa renderizando un componente `Navigate` que actualiza la ubicación actual del navegador. Aunque este enfoque sigue el estilo de codificación declarativo de React Router, introduce algunos pasos adicionales en el ciclo de renderizado de React:

1.  El componente `Navigate` debe ser retornado.
2.  El componente `Navigate` se renderiza.
3.  La URL se actualiza.
4.  Y finalmente, se renderiza la ruta apropiada.

React Router también proporciona un mecanismo para actualizar la ubicación del navegador de forma **imperativa** usando el hook **useNavigate**.

```jsx
import { useNavigate } from 'react-router-dom';
```

La función `useNavigate()` devuelve una función `navigate` que nos permite especificar una ruta a la que nos gustaría navegar.

Considera este ejemplo que desencadena inmediatamente una redirección de vuelta a la página `/` después de que un usuario envía exitosamente un `<form>`:

```jsx
import { useNavigate } from 'react-router-dom';

export const ExampleForm = () => {

  const navigate = useNavigate();

  const handleSubmit = e => {
    e.preventDefault();
    navigate('/');
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* elementos del formulario */}
    </form>
  );
}
```

Al permitir actualizaciones imperativas de la ubicación del navegador, la función `navigate` te permite responder inmediatamente a la entrada del usuario sin tener que esperar.

La función `useNavigate()` también nos da la capacidad de navegar programáticamente a nuestros usuarios a través de su **historial de navegación**. Escenarios como permitir a los usuarios avanzar o retroceder en una aplicación, o redirigir a los usuarios a su página anterior después de que hayan iniciado sesión, son excelentes casos de uso para esta funcionalidad. Para navegar a un usuario a través de su historial usando `useNavigate()`, pasaríamos un número entero para indicar dónde en el historial nos gustaría viajar. Un número entero positivo navega hacia adelante y un número entero negativo navega hacia atrás.

Por ejemplo:

*   `navigate(-1)` - navega a la URL anterior en el historial.
*   `navigate(1)` - navega a la siguiente URL en el historial.
*   `navigate(-3)` - navega 3 URLs hacia atrás en el historial.

A continuación, podemos ver cómo se usa la función `navigate()` para habilitar un botón "Atrás":

```jsx
import { useNavigate } from 'react-router-dom';

export const BackButton = () => {
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate(-1)}>
      Atrás
    </button>
  );
}
```

Practiquemos el uso de `useNavigate` para dar a nuestros usuarios la capacidad de navegar hacia atrás y hacia adelante sin importar dónde se encuentren en nuestra aplicación.

----

## Query Parameters

Los **parámetros de consulta** aparecen en las URL comenzando con un signo de interrogación (`?`) y van seguidos de un nombre de parámetro asignado a un valor. Son opcionales y se utilizan más a menudo para buscar, ordenar y/o filtrar recursos.

Por ejemplo, si visitaras la siguiente URL...

```
https://www.google.com/search?q=codecademy
```

... serías llevado a la página `/search` de Google mostrando resultados para el término de búsqueda `codecademy`. En este ejemplo, el nombre del parámetro de consulta es `q`.

Los parámetros de consulta pueden ser útiles para determinar qué contenido mostrar a nuestro usuario y React Router proporciona un mecanismo para obtener los valores de los parámetros de consulta con el hook **useSearchParams()**. Este hook devuelve un objeto **URLSearchParams** y una función que podemos usar para actualizarlo.

Considera este ejemplo de un componente `SortedList`:

```jsx
import { useSearchParams } from 'react-router-dom';

// Se renderiza cuando un usuario visita "/list?order=DESC"
export const SortedList = (numberList) => {
  const [ searchParams, setSearchParams ] = useSearchParams();
  const sortOrder = searchParams.get('order');
  console.log(sortOrder); // Imprime "DESC"
};
```

Analicemos este ejemplo:

* Primero, importamos **useSearchParams()** y lo llamamos dentro de `SortedList` para obtener el objeto `URLSearchParams`. Este objeto tiene un método **.get()** para recuperar los valores de los parámetros de consulta.
* Finalmente, para obtener el valor de un parámetro de consulta específico, pasamos el nombre del parámetro de consulta cuyo valor queremos obtener, como una cadena (`'order'`), al método `searchParams.get()`. El valor (`'DESC'`) se almacena entonces en la variable `sortOrder`.

Ampliemos el ejemplo de `SortedList` para que el componente use el valor `sortOrder` y renderice una lista de datos ya sea en orden ascendente, en orden descendente, o en su orden natural.

```jsx
import { useSearchParams } from 'react-router-dom';

// Se renderiza cuando un usuario visita "/list?order=DESC"
export const SortedList = (numberList) => {
  const [ searchParams, setSearchParams ] = useSearchParams();
  const sortOrder = searchParams.get('order');

  if (sortOrder === 'ASC') {
    // renderizar la numberList en orden ascendente
  } else if (sortOrder === 'DESC') {
    // renderizar la numberList en orden descendente
  } else {
    // renderizar la numberList tal como está
  }
}
```

Ahora, si el usuario visitara `/list?order=DESC`, se extraería el valor `'DESC'` y podemos renderizar el componente `SortedList` en orden descendente. Del mismo modo, visitar `/list?order=ASC` renderizará la lista en orden ascendente. Finalmente, dado que los parámetros de consulta son opcionales, si visitáramos `/list`, el componente `SortedList` se renderizaría en su orden natural.

Imagina que tenemos un componente `List` con un botón de ordenamiento que queremos usar para actualizar la URL a `/list?order=ASC` y luego renderizar `SortedList`. Podemos usar la función **setSearchParams()** para hacer esto. Por ejemplo:

```jsx
import { useSearchParams } from 'react-router-dom';

// Se renderiza cuando un usuario visita "/list"
export const List = (numberList) => {
  const [ searchParams, setSearchParams ] = useSearchParams();

  // renderizar la numberList
  return (
    <button onClick={ () => setSearchParams( {order: 'ASC'} ) }>
      Ordenar
    </button>
  );
}
```

Cuando un usuario hace clic en el botón "Ordenar", actualizaremos la ruta a `/list?order=ASC`, lo que hará que se renderice nuestro componente `SortedList` (asumiendo que la ruta está configurada para hacerlo).

`useSearchParams` funciona muy bien cuando queremos acceder a los parámetros de consulta de la ruta actual o alterarlos, pero ¿qué pasa si queremos navegar a una ruta e incluir también parámetros de consulta? Bueno, para este escenario necesitaremos usar la función de utilidad **createSearchParams()** de `react-router-dom` junto con el hook **useNavigate** que aprendimos anteriormente.

Por ejemplo, si quisiéramos navegar directamente a `/list?order=ASC` desde la ruta raíz (`/`), haríamos algo como esto:

```jsx
import { useNavigate, createSearchParams } from 'react-router-dom';

// obtener la función navigate
const navigate = useNavigate();

// definir un objeto donde la clave es el nombre del parámetro de consulta y el valor es el valor del parámetro de consulta
const searchQueryParams = {
  order: 'ASC'
};

// usar createSearchParams que toma un objeto y lo transforma a una cadena de consulta de la forma order=ASC
const searchQueryString = createSearchParams(searchQueryParams);

// forzar una navegación pasando un objeto con pathname indicando la ruta a navegar y search indicando los parámetros de consulta a añadir
navigate({
  pathname: '/list',
  search: `?${searchQueryString}`
});
```

Los puntos importantes a tener en cuenta sobre el ejemplo anterior son que:

1.  Definimos un objeto que representa los parámetros de consulta que queremos y lo llamamos `searchQueryParams`.
2.  Pasamos `searchQueryParams` a `createSearchParams`, que lo transformará de un objeto a una cadena de consulta.
3.  Llamamos a `useNavigate` y pasamos un objeto con las claves `pathname` y `search`, donde `pathname` es una cadena que indica hacia dónde navegar y `search` es una cadena que indica la cadena de consulta que se añadirá a la ruta.

Ten en cuenta que necesitamos incluir el `?`, por eso usamos una plantilla de cadena aquí.

¡Gran trabajo aprendiendo sobre los parámetros de consulta! Practiquemos ahora añadiendo algo de filtrado a nuestros artículos usándolos.

----

## Review

¡Gran trabajo! Has aprendido todo lo que necesitas saber para añadir **enrutamiento (routing)** del lado del frontend a tus aplicaciones de React usando React Router. Para recapitular, has aprendido cómo:

*   Instalar `react-router-dom` y añadirlo a una aplicación React.
*   Habilitar el enrutamiento usando `RouterProvider` y proporcionando un **router**.
*   Crear un router usando **createBrowserRouter()**.
*   Usar **createRoutesFromElements()** para configurar un router.
*   Usar el componente **Route** para añadir rutas estáticas y dinámicas a una aplicación.
*   Usar los componentes **Link** y **NavLink** para añadir enlaces a una aplicación.
*   Acceder a los valores de los **parámetros de URL** usando el hook **useParams** de React Router.
*   Crear **rutas anidadas (nested routes)** usando `Route`, `Outlet` y rutas relativas.
*   Redirigir a los usuarios de forma **declarativa** renderizando el componente **Navigate** de React Router.
*   Redirigir a los usuarios de forma **imperativa** mediante el hook **useNavigate**.
*   Acceder y establecer el valor de los **parámetros de consulta (query parameters)** usando el hook **useSearchParams** de React Router.

Gran trabajo aprendiendo sobre React Router. Has aprendido mucho sobre la funcionalidad principal de React Router. Si deseas explorar más, puedes consultar la documentación oficial [aquí](https://reactrouter.com/en/main).

Te animamos a seguir practicando lo que has aprendido y a experimentar con React Router por tu cuenta.

------