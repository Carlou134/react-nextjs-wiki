# Rutas Dinámicas, Anidadas y Navegación

## En una frase

Con parámetros de URL, rutas anidadas y los hooks de navegación, React Router muestra contenido distinto según la URL, comparte un layout entre varias vistas y cambia de página desde código en vez de depender solo de clics del usuario.

-----

## Antes de empezar

Esta lección da por sabido lo que cubre [Fundamentos de Routing y React Router](01-Fundamentos%20de%20Routing%20y%20React%20Router.md): qué es el enrutamiento, cómo instalar React Router, cómo proveerlo con un router y cómo declarar rutas estáticas con `<Route>`, `Link` y `NavLink`. Si no la leíste todavía, empieza por ahí.

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Parámetro de URL:** segmento variable de una ruta, escrito como `:nombre` (por ejemplo, `:id` en `/productos/:id`).
* **Ruta dinámica:** ruta que incluye uno o más parámetros de URL y coincide con muchas URLs distintas, no con una sola.
* **Ruta anidada:** una `<Route>` declarada dentro de otra. La de afuera es la ruta padre; la de adentro, la ruta hija.
* **Ruta índice:** ruta hija especial, con la prop `index` en vez de `path`, que se renderiza cuando la URL coincide exactamente con la ruta del padre.
* **`Outlet`:** componente que marca, dentro del layout padre, en qué punto se debe renderizar la ruta hija activa.
* **Parámetro de consulta (query param):** par `clave=valor` que aparece en la URL después de `?`, opcional, usado típicamente para filtros, orden o paginación.
* **Historial de navegación:** la pila de URLs que el navegador recuerda para permitir avanzar y retroceder.

**Nota de versión.** React Router avanzó bastante desde la v6 con la que se suele aprender esto. Hoy el paquete es `react-router` (ya no hace falta `react-router-dom` para los hooks y componentes; el `RouterProvider` para el modo con `createBrowserRouter` vive en `react-router/dom`). La API de `useParams`, `Outlet`, `useNavigate` y `useSearchParams` se mantiene igual en lo esencial. El cambio más importante de nombre o de forma recomendada es `<Navigate>`: sigue existiendo, pero la documentación oficial ahora la desaconseja a favor de `useNavigate`. Cada sección de abajo aclara qué sigue igual y qué cambió.

-----

## El problema

Con rutas estáticas alcanza para páginas fijas, pero una aplicación real necesita tres cosas más:

* **Layouts compartidos.** Una barra de navegación o un menú lateral que se vea en varias vistas sin copiar el mismo JSX en cada una.
* **Navegación programática.** Redirigir después de que un formulario se envía con éxito, o sacar a un usuario no autenticado de una página protegida, sin esperar a que haga clic en algo.
* **Leer datos de la URL.** Mostrar el detalle de un recurso según un identificador en la ruta (`/productos/42`), o filtrar una lista según un parámetro de consulta (`/productos?categoria=libros`).

Rutas dinámicas, rutas anidadas y los hooks de navegación resuelven cada uno de estos puntos.

-----

## Cómo funciona

### Rutas dinámicas y `useParams`

Un parámetro de URL convierte una ruta fija en una ruta dinámica. Se declara con dos puntos seguidos de un nombre:

```jsx
<Route path="/productos/:id" element={<ProductDetail />} />
```

Esta ruta coincide con `/productos/1`, `/productos/42` o cualquier otro valor en esa posición. Una ruta puede tener varios parámetros (`/productos/:id/reseñas/:reviewId`) o ninguno.

Dentro del componente que renderiza esa ruta, el hook `useParams` devuelve un objeto con los valores actuales:

```jsx
import { useParams } from 'react-router';

function ProductDetail() {
  const { id } = useParams();
  // en /productos/42, id vale "42"

  return <h1>Producto {id}</h1>;
}
```

Los valores de `useParams` siempre son texto, incluso si en la URL parecen números. Si necesitas operar con ellos como número, convertirlos es responsabilidad tuya (`Number(id)`).

### Rutas anidadas con `Outlet`

Una ruta anidada es una `<Route>` declarada dentro de otra. La de afuera es la **ruta padre**; la de adentro, la **ruta hija**. La ruta de la hija es relativa a la del padre, así que no repite ese prefijo:

```jsx
<Route path="/productos" element={<ProductsLayout />}>
  <Route index element={<ProductList />} />
  <Route path=":id" element={<ProductDetail />} />
</Route>
```

Con esta configuración, `/productos` renderiza `ProductsLayout` y, adentro, la **ruta índice** `ProductList` (se activa cuando la URL coincide exactamente con la del padre). `/productos/42` renderiza `ProductsLayout` y, adentro, `ProductDetail`.

Declarar la ruta anidada no alcanza: el layout padre necesita indicar dónde va la ruta hija. Para eso existe `Outlet`:

```jsx
import { Outlet } from 'react-router';

function ProductsLayout() {
  return (
    <section>
      <h1>Catálogo</h1>
      <Outlet /> {/* aquí se renderiza ProductList o ProductDetail, según la URL */}
    </section>
  );
}
```

`Outlet` no navega ni decide nada por sí mismo: solo marca el punto de inserción. React Router reemplaza ese `Outlet` por el elemento de la ruta hija que coincida con la URL actual.

### Redirecciones: de `<Navigate>` a `useNavigate`

`<Navigate>` es un componente: al renderizarse, redirige a la ruta indicada en su prop `to`.

```jsx
import { Navigate } from 'react-router';

function UserProfile({ loggedIn }) {
  if (!loggedIn) {
    return <Navigate to="/" />;
  }

  return <p>Perfil del usuario</p>;
}
```

Esta forma sigue funcionando, pero la documentación actual de React Router la desaconseja para componentes de función: recomienda usar `useNavigate` en su lugar. El caso donde `<Navigate>` todavía tiene sentido es un componente de clase, donde no se pueden llamar hooks. La siguiente subsección muestra la forma vigente de lograr lo mismo con `useNavigate`.

### `useNavigate`: navegación imperativa

`useNavigate` devuelve una función para cambiar la URL desde código, en respuesta a un evento o dentro de un efecto:

```jsx
import { useNavigate } from 'react-router';

function ExampleForm() {
  const navigate = useNavigate();

  function handleSubmit(e) {
    e.preventDefault();
    navigate('/');
  }

  return <form onSubmit={handleSubmit}>{/* campos */}</form>;
}
```

Para el caso que antes se resolvía con `<Navigate>` durante el render (redirigir si no hay sesión), la forma vigente es llamar a `navigate` dentro de un efecto, no directamente en el cuerpo del componente:

```jsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router';

function UserProfile({ loggedIn }) {
  const navigate = useNavigate();

  useEffect(() => {
    if (!loggedIn) navigate('/');
  }, [loggedIn, navigate]);

  if (!loggedIn) return null;
  return <p>Perfil del usuario</p>;
}
```

`navigate` también acepta un número entero para moverse por el historial: `navigate(-1)` retrocede una entrada, `navigate(1)` avanza una. Úsalo con cuidado: si el usuario llegó directo a esa página (por ejemplo, por un enlace externo), puede no haber una entrada anterior en el historial, o puede terminar en un sitio inesperado.

```jsx
function BackButton() {
  const navigate = useNavigate();
  return <button onClick={() => navigate(-1)}>Atrás</button>;
}
```

### Query params con `useSearchParams`

Los parámetros de consulta viven después del `?` en la URL (`/lista?orden=DESC`) y sirven para datos opcionales como filtros u orden. El hook `useSearchParams` devuelve una tupla: el objeto `URLSearchParams` actual y una función para actualizarlo.

```jsx
import { useSearchParams } from 'react-router';

// se renderiza en "/lista?orden=DESC"
function SortedList() {
  const [searchParams] = useSearchParams();
  const order = searchParams.get('orden'); // "DESC", o null si no está en la URL
  // ...
}
```

`searchParams.get()` siempre devuelve `string | null`: `null` cuando ese parámetro no está presente en la URL.

Para actualizarlos, se usa la función que devuelve el hook, no una mutación directa del objeto:

```jsx
function List() {
  const [searchParams, setSearchParams] = useSearchParams();

  return (
    <button onClick={() => setSearchParams({ orden: 'ASC' })}>
      Ordenar
    </button>
  );
}
```

Llamar a `setSearchParams` actualiza la URL y provoca un nuevo render con los parámetros nuevos. Si el cambio depende del valor anterior (por ejemplo, agregar un filtro sin perder los que ya estaban), conviene pasarle una función:

```jsx
setSearchParams((prev) => {
  prev.set('pagina', '2');
  return prev;
});
```

-----

## Ejemplo completo

Una mini tienda: un layout con navegación, un listado filtrable por categoría (query param) y una vista de detalle por id (ruta dinámica).

```jsx
// router.jsx
import { createBrowserRouter, createRoutesFromElements, Route } from 'react-router';
import { RouterProvider } from 'react-router/dom';
import StoreLayout from './StoreLayout';
import ProductList from './ProductList';
import ProductDetail from './ProductDetail';

const router = createBrowserRouter(
  createRoutesFromElements(
    <Route path="/" element={<StoreLayout />}>
      <Route index element={<ProductList />} />
      <Route path="productos/:id" element={<ProductDetail />} />
    </Route>
  )
);

export default function App() {
  return <RouterProvider router={router} />;
}
```

```jsx
// StoreLayout.jsx
import { Outlet, NavLink } from 'react-router';

export default function StoreLayout() {
  return (
    <div>
      <nav>
        <NavLink to="/">Catálogo</NavLink>
      </nav>
      <Outlet />
    </div>
  );
}
```

```jsx
// ProductList.jsx
import { Link, useSearchParams } from 'react-router';

const PRODUCTS = [
  { id: '1', name: 'Teclado', category: 'accesorios' },
  { id: '2', name: 'Monitor', category: 'pantallas' },
  { id: '3', name: 'Mouse', category: 'accesorios' },
];

export default function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get('categoria');

  const visible = category
    ? PRODUCTS.filter((p) => p.category === category)
    : PRODUCTS;

  return (
    <div>
      <button onClick={() => setSearchParams({ categoria: 'accesorios' })}>
        Solo accesorios
      </button>
      <button onClick={() => setSearchParams({})}>Ver todo</button>

      <ul>
        {visible.map((p) => (
          <li key={p.id}>
            <Link to={`/productos/${p.id}`}>{p.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

```jsx
// ProductDetail.jsx
import { useParams, useNavigate } from 'react-router';

export default function ProductDetail() {
  const { id } = useParams();
  const navigate = useNavigate();

  return (
    <div>
      <h1>Producto {id}</h1>
      <button onClick={() => navigate(-1)}>Volver</button>
    </div>
  );
}
```

Qué hace cada pieza:

1. `StoreLayout` es la ruta padre: pone la navegación una sola vez y usa `Outlet` para mostrar `ProductList` o `ProductDetail` según la URL.
2. `ProductList` es la ruta índice (`/`). Lee el query param `categoria` con `useSearchParams` para filtrar, y los botones lo actualizan sin recargar la página.
3. `ProductDetail` es la ruta dinámica (`/productos/:id`). Lee `id` con `useParams` y usa `useNavigate(-1)` para volver al listado sin necesitar un enlace fijo.

-----

## Errores comunes

### 1. Olvidar `<Outlet />` en el layout

**Qué pasa:** la URL cambia, la ruta hija está bien declarada, pero nada nuevo aparece en pantalla.
**Por qué:** React Router sabe qué ruta hija coincide, pero no tiene dónde insertarla dentro del componente padre.
**Arreglo:** agregar `<Outlet />` en el punto del layout donde debe aparecer el contenido hijo.

### 2. `useParams` devuelve `string | undefined` en TypeScript

**Qué pasa:** al tipar `useParams` con un genérico, el editor marca error al pasar `id` a una función que espera `string`.
**Por qué:** React Router no puede verificar en tiempo de compilación que la ruta realmente incluye ese parámetro; el tipo refleja esa incertidumbre.
**Arreglo:** validar el valor antes de usarlo, o asumir el riesgo explícitamente solo cuando la ruta lo garantiza (ver la sección de TypeScript).

### 3. Navegar con `<a>` en vez de `useNavigate` o `Link`

**Qué pasa:** el clic recarga toda la página en vez de solo cambiar la vista.
**Por qué:** `<a>` es una etiqueta del navegador; dispara una navegación de documento completo, ajena a React Router.
**Arreglo:** usar `Link` o `NavLink` para navegación declarativa, y `useNavigate` para la imperativa.

### 4. Mutar `URLSearchParams` directamente

**Qué pasa:** el filtro parece aplicarse, pero la URL no cambia, y el valor se puede perder en el siguiente render.
**Por qué:** `searchParams` es el objeto derivado del estado del router en ese render. Modificarlo con `.set()` sin llamar a `setSearchParams` no notifica al router.
**Arreglo:** llamar siempre a `setSearchParams`, con una función *updater* cuando el cambio depende del valor anterior.

-----

## En TypeScript

```tsx
import { useParams, useSearchParams } from 'react-router';

function ProductDetail() {
  const { id } = useParams<{ id: string }>();
  // id: string | undefined

  const [searchParams] = useSearchParams();
  const category: string | null = searchParams.get('categoria');
}
```

* **`useParams<{ id: string }>()`:** el genérico describe qué parámetros esperas. Aun así, el valor resultante puede incluir `undefined`, porque TypeScript confía en tu anotación, no en la definición real de la ruta. Si tu código exige que `id` exista siempre, valídalo antes de usarlo (o lánzale un error si falta) en vez de forzar el tipo con `as string`.
* **`useSearchParams()`:** devuelve `[URLSearchParams, SetURLSearchParams]`. `searchParams.get('clave')` siempre tipa como `string | null`; hay que manejar el `null` antes de usar el valor.

-----

## Cuándo sí y cuándo no

**Usa rutas dinámicas y anidadas cuando** necesitas mostrar contenido según un identificador de la URL, compartir un layout entre varias vistas, o que un filtro, un orden o una página puedan compartirse por URL y sobrevivan a recargar o a usar atrás/adelante del navegador.

**No las uses para:**

* **Estado que no tiene sentido en la URL,** como el texto que el usuario está escribiendo en un campo antes de enviarlo. Eso es trabajo de `useState` local.
* **Guardas de autenticación complejas en apps grandes con Data Routers.** Ahí conviene resolverlas con `loader` y la utilidad `redirect()` de React Router, que corta la navegación antes de renderizar nada, en vez de redirigir desde un efecto una vez que el componente ya se montó.

-----

## Resumen en 5 líneas

1. Un parámetro de URL (`:id`) crea una ruta dinámica; `useParams` lee su valor actual.
2. Las rutas anidadas comparten un layout: `Outlet` marca dónde se renderiza la ruta hija activa.
3. `<Navigate>` sigue existiendo, pero la forma vigente de redirigir en componentes de función es `useNavigate`, normalmente dentro de un efecto.
4. `useSearchParams` lee y actualiza los parámetros de consulta sin recargar la página; nunca se debe mutar el objeto directamente.
5. Desde la v6, el paquete se unificó en `react-router` (`RouterProvider` vive en `react-router/dom`), pero el comportamiento de estos hooks no cambió en lo esencial.

-----

## Para profundizar

<details>
<summary>Rutas protegidas con <code>loader</code> y <code>redirect()</code></summary>

Cuando el router se crea con `createBrowserRouter`, cada ruta puede declarar un `loader`: una función que corre antes de renderizar la ruta. Ahí se puede cortar la navegación con la utilidad `redirect()`, sin llegar a montar el componente:

```jsx
import { redirect } from 'react-router';

function profileLoader() {
  const loggedIn = checkSession();
  if (!loggedIn) {
    return redirect('/');
  }
  return null;
}

<Route path="/perfil" element={<UserProfile />} loader={profileLoader} />
```

Esto evita el parpadeo de "se monta el componente, después un efecto lo redirige" que tiene el patrón con `useNavigate` en un `useEffect`. Es más avanzado porque exige entender `loader`, algo fuera del alcance de esta lección.

</details>

<details>
<summary>Múltiples niveles de rutas anidadas</summary>

Una ruta puede ser hija y padre a la vez. Nada impide anidar varios niveles:

```jsx
<Route path="/tienda" element={<StoreLayout />}>
  <Route path="productos" element={<ProductsLayout />}>
    <Route index element={<ProductList />} />
    <Route path=":id" element={<ProductDetail />} />
  </Route>
</Route>
```

Cada nivel necesita su propio `<Outlet />` en su layout para que el nivel siguiente tenga dónde renderizarse. La ruta final (`/tienda/productos/42`) atraviesa dos `Outlet` anidados antes de llegar a `ProductDetail`.

</details>

<details>
<summary>Rutas relativas en <code>Link</code> y <code>navigate</code></summary>

Una ruta que no empieza con `/` es relativa a la ruta actual, no a la raíz. Desde `/productos/42`, `<Link to="../">` sube al nivel del padre (`/productos`), y `navigate('hermana')` navega a `/productos/hermana`. Esto es útil dentro de rutas anidadas para no repetir el prefijo completo en cada enlace.

</details>

-----

## En entrevista

### Respuesta corta (junior)

React Router permite armar rutas dinámicas con parámetros de URL (`:id`, leídos con `useParams`) y rutas anidadas que comparten un layout gracias a `Outlet`. Para navegar desde código se usa `useNavigate`, y para leer filtros u orden en la URL se usa `useSearchParams`. El componente `<Navigate>` existe, pero hoy se recomienda `useNavigate` en su lugar para componentes de función.

### Respuesta ampliada (semi-senior)

* **Rutas dinámicas:** un segmento con `:nombre` en el `path` crea un parámetro de URL; `useParams()` lo expone como texto, nunca como número.
* **Rutas anidadas:** la ruta hija hereda el prefijo del padre y se renderiza dentro de su `Outlet`. Las rutas índice (`index`) cubren el caso "coincide exactamente con el padre".
* **Navegación declarativa vs. imperativa:** `Link`/`NavLink` para que el usuario haga clic; `useNavigate` para disparar la navegación desde código (envío de formulario, redirección condicional, historial con `navigate(-1)`).
* **`<Navigate>` como caso especial:** sigue existiendo para componentes de clase, donde no hay hooks, pero la documentación actual lo desaconseja en componentes de función a favor de `useNavigate` dentro de un efecto.
* **Query params:** `useSearchParams` da un `URLSearchParams` (inmutable en la práctica: mutar y no llamar a `setSearchParams` no actualiza la URL) y una función para reemplazarlo, con soporte para pasar un objeto o una función *updater*.
* **Cuándo escalar:** para lógica de protección de rutas más seria, `loader` + `redirect()` en Data Routers evita el parpadeo de redirigir después de montar el componente.

### Preguntas frecuentes de seguimiento

**1. ¿Cuál es la diferencia entre rutas anidadas y anidar componentes a mano?**
Anidar componentes a mano significa importar uno dentro de otro directamente en el JSX; siempre se renderizan juntos. Las rutas anidadas dependen de la URL: el padre siempre se renderiza, pero cuál hijo aparece en su `Outlet` (o si aparece alguno) lo decide la ruta activa.

**2. ¿Cuándo usar `<Navigate>` y cuándo `useNavigate`?**
En componentes de función, `useNavigate` es la forma recomendada, tanto para navegación disparada por eventos como para redirecciones dentro de un efecto. `<Navigate>` queda para componentes de clase, donde no se pueden usar hooks.

**3. ¿Cómo se comparten datos entre una ruta padre y su ruta hija?**
El padre puede pasarle datos a sus hijos a través de la prop `context` de `Outlet`, y el hijo los lee con `useOutletContext`. Es la vía pensada para esto, distinta de pasar props porque la ruta hija no es un elemento hijo directo en el JSX.

**4. ¿Qué diferencia hay entre `useParams` y `useSearchParams`?**
`useParams` lee segmentos fijos del *path* que la ruta declaró como dinámicos (`:id`); siempre están definidos por la estructura de la ruta. `useSearchParams` lee la parte opcional después del `?`, pensada para datos que no cambian qué componente se renderiza, solo cómo se comporta (filtro, orden, página).

**5. ¿Cómo se protege una ruta que requiere sesión iniciada?**
La forma simple es leer el estado de sesión en el componente y redirigir con `useNavigate` dentro de un efecto si no hay sesión. La forma más robusta, cuando el router usa `createBrowserRouter`, es un `loader` que llama a `redirect()` antes de renderizar, evitando montar el componente protegido aunque sea por un instante.

**6. ¿Por qué `navigate(-1)` puede fallar o hacer algo inesperado?**
Porque depende del historial real del navegador, no de la estructura de rutas de la app. Si el usuario llegó directo a esa página (enlace externo, URL escrita a mano), puede no haber una entrada anterior, o esa entrada puede pertenecer a otro sitio.

-----

## Siguiente lección

Con el enrutamiento cubierto, el siguiente tema es qué hacer cuando algo se rompe durante el render: [React Error Boundaries](../08-manejo-de-errores/01-React%20Error%20Boundaries.md).
