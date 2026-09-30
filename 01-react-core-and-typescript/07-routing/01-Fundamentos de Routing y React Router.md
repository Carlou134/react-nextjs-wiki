# Fundamentos de routing y React Router

## En una frase

El enrutamiento (*routing*) del lado del cliente hace que la URL del navegador decida qué componentes se muestran, sin recargar la página; **React Router** es la librería que implementa esto para aplicaciones React.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un componente de función y cómo devuelve JSX: [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
* Cómo un componente recibe datos desde afuera con props: [Props](../02-componentes-y-props/03-Props.md).
* Qué es un Hook y cómo se usa `useState`: [The State Hook](../04-hooks-y-context/01-The%20State%20Hook.md).

Palabras nuevas:

* **Enrutamiento (routing):** el proceso de decidir qué contenido mostrar según la URL actual.
* **SPA (Single Page Application):** aplicación que carga un único documento HTML y usa JavaScript para actualizar la interfaz sin pedir páginas nuevas al servidor.
* **Ruta (route):** asociación entre un patrón de URL y el componente que se debe renderizar para esa URL.
* **Router:** el objeto que centraliza la configuración de rutas y decide, en cada cambio de URL, qué renderizar.

También están en el [Glosario](Glosario.md).

-----

## El problema

En un sitio tradicional, cada URL corresponde a un documento HTML distinto que el navegador pide al servidor. Al hacer clic en un enlace (`<a href="...">`), el navegador descarta la página actual, solicita una nueva y la vuelve a construir desde cero. Esto es simple, pero tiene un costo: se pierde el estado de JavaScript en memoria y hay un parpadeo visible mientras se carga el documento nuevo.

Una SPA de React funciona distinto: una sola carga inicial trae el JavaScript de toda la aplicación, y ese JavaScript decide qué mostrar. El problema es que, sin nada más, la URL del navegador queda desconectada de lo que se ve en pantalla. Si la app cambia de vista con estado local (por ejemplo, un `if` que decide qué componente renderizar), la URL no cambia, el botón "atrás" del navegador no funciona como se espera y no se puede compartir un enlace directo a una vista concreta.

**React Router** resuelve esto: sincroniza la URL del navegador con la interfaz. Escucha los cambios de URL (por ejemplo, al hacer clic en un enlace interno) y, en lugar de dejar que el navegador recargue el documento, actualiza la URL mediante la History API y renderiza el componente de React que corresponda a esa ruta. El resultado combina lo mejor de ambos mundos: URLs que se pueden compartir y usar con atrás/adelante, sin las recargas completas de la navegación tradicional.

Antes de seguir, conviene tener clara la estructura de una URL. Tomemos como ejemplo `https://ejemplo.com/articulos?buscar=react`:

* **Esquema** (`https`): el protocolo usado para acceder al recurso.
* **Dominio** (`ejemplo.com`): el sitio que aloja el recurso; es el punto de entrada de la aplicación.
* **Ruta** (`/articulos`): identifica el recurso específico. Aquí es donde actúa el enrutamiento.
* **Cadena de consulta** (`?buscar=react`): parámetros opcionales, comunes para búsquedas y filtros.

React Router trabaja principalmente sobre la parte de la ruta; los parámetros de consulta se ven en la siguiente lección.

-----

## Cómo funciona

### Una aclaración sobre versiones

React Router avanzó mucho desde la versión 6 (la más citada en tutoriales y cursos). Desde la versión 7, el paquete `react-router-dom` desapareció: todo vive en un único paquete, `react-router`. Además, la librería formalizó tres modos de uso, pensados para necesidades distintas:

* **Modo declarativo:** solo componentes JSX (`<BrowserRouter>`, `<Routes>`, `<Route>`, `<Link>`). No incluye carga de datos integrada. Es el equivalente más directo a "usar React Router como un conjunto de componentes".
* **Modo de datos:** agrega `loader`, `action` y estados de carga por ruta, mediante `createBrowserRouter` y `RouterProvider`. Es más potente, pero introduce conceptos (carga de datos por ruta) que no hacen falta para aprender los fundamentos.
* **Modo framework:** el modo de datos más un plugin propio de Vite, con convenciones de archivos, *code splitting* automático y soporte para SSR. Es, en la práctica, un framework completo (conceptualmente cercano a lo que ofrece Next.js), fuera del alcance de una app Vite simple.

Para una app creada con Vite (sin ese plugin de framework) y sin necesidad de cargar datos junto con la ruta, el **modo declarativo** es el punto de entrada recomendado hoy y es el que usa esta lección. Si más adelante necesitas `loader`/`action` por ruta, migras al modo de datos sin cambiar cómo defines los componentes `<Route>`. El modo framework se menciona solo como referencia: implica adoptar una estructura de proyecto distinta, más parecida a un framework como Next.js, y no es el tema de esta lección.

### Instalación

```bash
npm install react-router
```

Ya no se instala `react-router-dom`: ese paquete quedó absorbido por `react-router`. Si ves `react-router-dom` en código o documentación antigua, es la versión previa a esta unificación.

### La forma recomendada de proveer el router

El modo declarativo provee el router envolviendo la aplicación con el componente `BrowserRouter`:

```jsx
import { BrowserRouter } from 'react-router';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

`BrowserRouter` usa la History API del navegador para leer y modificar la URL sin provocar una recarga completa. A partir de aquí, cualquier componente dentro de `App` puede usar los componentes y Hooks de React Router, porque todos dependen de que exista un router activo más arriba en el árbol.

### Rutas básicas con `<Route>`

Dentro de `App` (o de cualquier componente hijo), se define qué se renderiza para cada ruta con `<Routes>` y `<Route>`:

```jsx
import { Routes, Route } from 'react-router';
import Home from './Home';
import About from './About';

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
    </Routes>
  );
}
```

Cada `<Route>` necesita:

* **`path`**: el patrón de URL que activa la ruta.
* **`element`**: el elemento JSX que se renderiza cuando la URL coincide con `path`.

`<Routes>` funciona como un `switch`: recorre sus `<Route>` hijos y renderiza el primero cuyo `path` coincida con la URL actual. Si ninguno coincide, no se renderiza nada (a menos que definas una ruta con `path="*"` para capturar el resto).

### `Link` y `NavLink` frente a `<a>`

Usar una etiqueta `<a href="/about">` funciona para navegar, pero el navegador interpreta cualquier clic sobre un `<a>` como una petición de un documento nuevo: recarga toda la página, descarta el estado de la aplicación en memoria y vuelve a ejecutar todo el JavaScript desde cero. Es exactamente lo que React Router busca evitar.

`Link` y `NavLink` (ambos exportados por `react-router`) resuelven esto: renderizan una etiqueta `<a>` en el DOM, pero interceptan el clic con JavaScript, actualizan la URL con la History API y dejan que React Router decida qué renderizar, sin recarga.

```jsx
import { Link, NavLink } from 'react-router';

<Link to="/about">Acerca de</Link>
<NavLink to="/about">Acerca de</NavLink>
```

Ambos aceptan una prop `to` equivalente al `href` de un `<a>`. Una ruta que empieza con `/` (como `/about`) es **absoluta**: React Router la resuelve desde la raíz del sitio, sin importar en qué ruta esté el componente que renderiza el enlace.

La diferencia entre ambos es el estado activo. `NavLink` agrega automáticamente la clase `active` (y el atributo `aria-current="page"`) cuando su `to` coincide con la URL actual, algo útil para menús de navegación:

```jsx
<NavLink
  to="/about"
  className={({ isActive }) => (isActive ? 'nav-link activo' : 'nav-link')}
>
  Acerca de
</NavLink>
```

Aquí `className` recibe una función en lugar de una cadena fija. React Router la llama con un objeto que incluye `isActive`, y el valor devuelto se usa como clase CSS. El mismo patrón funciona con la prop `style`. `Link` no tiene este comportamiento: siempre es un enlace "neutro", sin noción de estar activo.

-----

## Ejemplo completo

```jsx
// main.jsx
import { createRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router';
import App from './App';

createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

```jsx
// App.jsx
import { Routes, Route, NavLink } from 'react-router';
import Home from './Home';
import About from './About';

export default function App() {
  return (
    <div>
      <nav>
        <NavLink to="/" className={({ isActive }) => (isActive ? 'activo' : '')}>
          Inicio
        </NavLink>
        <NavLink to="/about" className={({ isActive }) => (isActive ? 'activo' : '')}>
          Acerca de
        </NavLink>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </div>
  );
}
```

```jsx
// Home.jsx
export default function Home() {
  return <h1>Inicio</h1>;
}
```

```jsx
// About.jsx
export default function About() {
  return <h1>Acerca de</h1>;
}
```

Al visitar `/`, se renderiza `Home` dentro de `App`, con el `nav` siempre visible porque vive fuera de `<Routes>`. Al hacer clic en "Acerca de", `NavLink` actualiza la URL a `/about` sin recargar la página, `Routes` cambia a renderizar `About`, y el enlace de "Acerca de" recibe la clase `activo`.

-----

## Errores comunes

### 1. Usar `<a>` en vez de `Link` para navegación interna

**Qué pasa:** al hacer clic, la página completa se recarga y se pierde cualquier estado en memoria (por ejemplo, un formulario a medio llenar en otra parte de la app).
**Por qué:** un `<a href="...">` es una instrucción nativa del navegador para pedir un documento nuevo; React Router no puede interceptarla.
**Arreglo:** usa `Link` o `NavLink` para cualquier navegación dentro de la misma aplicación. Reserva `<a>` para enlaces externos (a otro dominio).

### 2. Usar `Link`/`NavLink` fuera del router

**Qué pasa:** un error en tiempo de ejecución indicando que el componente debe usarse dentro de un `Router` (o un mensaje similar de contexto faltante).
**Por qué:** `Link`, `NavLink`, `Routes` y los Hooks de React Router dependen de un contexto que solo existe dentro de `BrowserRouter` (o el router equivalente que estés usando). Si se renderizan fuera de ese árbol, no encuentran ese contexto.
**Arreglo:** asegúrate de que `BrowserRouter` envuelva a toda la aplicación, típicamente en el punto de entrada (`main.jsx`), antes de cualquier componente que use React Router.

### 3. Definir dos `<Route>` con el mismo `path`

**Qué pasa:** solo uno de los dos se renderiza, y no siempre el que esperabas.
**Por qué:** `<Routes>` recorre sus hijos en orden y usa el primer `path` que coincide; una ruta duplicada nunca se alcanza.
**Arreglo:** revisa que cada `path` dentro de un mismo `<Routes>` sea único, y ubica las rutas más específicas antes que las genéricas.

### 4. Confundir ruta absoluta con ruta relativa

**Qué pasa:** un enlace `to="about"` (sin `/` inicial) navega a una URL distinta de la esperada si el componente vive en una ruta anidada.
**Por qué:** sin `/` inicial, `to` se resuelve de forma relativa a la ruta actual, no desde la raíz.
**Arreglo:** para destinos fijos conocidos de antemano, usa una ruta absoluta (`to="/about"`). Las rutas relativas cobran sentido recién con rutas anidadas, tema de la siguiente lección.

-----

## En TypeScript

Con TypeScript, `path` y `element` no necesitan anotación extra: `path` es un `string` y `element` acepta cualquier `ReactNode`. Donde sí conviene ser explícito es en la función de `className` o `style` de `NavLink`, si la extraes como una función aparte:

```tsx
import type { NavLinkRenderProps } from 'react-router';

function navLinkClass({ isActive }: NavLinkRenderProps): string {
  return isActive ? 'nav-link activo' : 'nav-link';
}
```

```tsx
<NavLink to="/about" className={navLinkClass}>
  Acerca de
</NavLink>
```

Tipar los parámetros dinámicos de una ruta (por ejemplo, `:id` en `/articles/:id`) requiere el Hook `useParams`, que se cubre en la siguiente lección junto con las rutas dinámicas.

-----

## Cuándo sí y cuándo no

**Usa React Router (modo declarativo) cuando** construyes una SPA con Vite (u otro bundler sin convenciones de framework) y necesitas varias vistas navegables por URL, sin requisitos de SSR ni de carga de datos integrada a la ruta.

**Considera el modo de datos de React Router cuando** cada ruta necesita cargar sus propios datos antes de renderizar (`loader`) o manejar envíos de formularios asociados a la ruta (`action`). La forma de definir `<Route>` que aprendiste aquí sigue siendo válida; solo cambia cómo se provee el router.

**No uses un router de cliente cuando** el proyecto ya es (o puede ser) un framework con enrutamiento por archivos, como Next.js: ahí la estructura de carpetas define las rutas, y mezclar React Router con ese enrutamiento genera conflictos. Consulta [Next.js Routing](../../02-nextjs-app-router/03-Next.js%20Routing.md) para ver ese enfoque.

-----

## Resumen en 5 líneas

1. El enrutamiento de cliente sincroniza la URL del navegador con lo que renderiza React, sin recargar la página.
2. Hoy se instala con `npm install react-router` (el paquete `react-router-dom` ya no existe).
3. El modo declarativo provee el router con `<BrowserRouter>` alrededor de toda la app.
4. `<Routes>` y `<Route path element>` definen qué componente se muestra según la URL.
5. `Link` y `NavLink` reemplazan a `<a>` para navegación interna sin recargar; `NavLink` además marca el enlace activo.

-----

## Para profundizar

<details>
<summary>Por qué existen tres modos en React Router</summary>

Las versiones anteriores (como la 6) mostraban una única forma "recomendada" de usar la librería, que en la práctica ya mezclaba conceptos de enrutamiento simple con capacidades de carga de datos (`createBrowserRouter`). Las versiones más recientes separaron esto explícitamente en tres modos —declarativo, de datos y framework— para que cada proyecto adopte solo la complejidad que necesita: una SPA simple puede quedarse en el modo declarativo, y escalar al modo de datos o al modo framework más adelante sin reescribir cómo definió sus rutas.

</details>

<details>
<summary>La History API, en breve</summary>

El navegador expone un objeto `history` con métodos como `pushState` y `replaceState`, que permiten cambiar la URL visible en la barra de direcciones **sin** pedir un documento nuevo al servidor. `BrowserRouter` se apoya en esta API: cuando navegas con `Link` o `useNavigate`, React Router llama a `pushState` en lugar de dejar que el navegador siga su comportamiento por defecto. También escucha el evento `popstate` para reaccionar cuando el usuario usa los botones atrás/adelante.

</details>

-----

## En entrevista

### Respuesta corta (junior)

React Router es la librería estándar para enrutamiento del lado del cliente en aplicaciones React. Permite que distintas URLs muestren distintos componentes sin recargar la página. Se instala con `npm install react-router`, se provee envolviendo la app con `BrowserRouter`, y las rutas se definen con `<Routes>` y `<Route path element>`. Para navegar entre ellas se usan `Link` y `NavLink` en vez de etiquetas `<a>`, porque estas últimas recargarían toda la aplicación.

### Respuesta ampliada (semi-senior)

* **Qué resuelve:** sincroniza la URL con la interfaz sin recargas completas, usando la History API del navegador en vez del comportamiento nativo de un `<a>`.
* **Modos:** React Router ofrece modo declarativo (`BrowserRouter`, `Routes`, `Route`, sin carga de datos integrada), modo de datos (`createBrowserRouter` + `RouterProvider`, con `loader`/`action` por ruta) y modo framework (el de datos más un plugin de Vite, con SSR y convenciones de proyecto). El modo declarativo es el punto de entrada natural para una SPA que no necesita carga de datos por ruta.
* **Cambio de paquete:** desde React Router 7, `react-router-dom` desapareció; todo se instala e importa desde `react-router`.
* **`Link` vs `NavLink`:** ambos evitan la recarga de un `<a>`; `NavLink` además expone `isActive` (y agrega `aria-current="page"`) para estilar el enlace de la ruta actual, útil en menús de navegación.
* **Coincidencia de rutas:** `<Routes>` renderiza el primer `<Route>` cuyo `path` coincide con la URL, por lo que el orden y la especificidad de los `path` importan.
* **Límite de esta capa:** el modo declarativo cubre navegación y vistas; no resuelve carga de datos, layouts anidados con `Outlet` ni parámetros dinámicos (`useParams`), que son el siguiente nivel.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué no alcanza con etiquetas `<a>` normales en una SPA?**
Porque el navegador trata cualquier clic en un `<a>` como pedido de un documento nuevo: recarga la página completa y descarta el estado de JavaScript en memoria. `Link`/`NavLink` interceptan el clic y actualizan la URL con la History API, sin recargar.

**2. ¿Qué pasa si dos `<Route>` tienen el mismo `path`?**
`<Routes>` renderiza el primero que coincide, en el orden en que aparecen; el segundo nunca se alcanza. Por eso conviene no duplicar `path` y ubicar las rutas más específicas antes que las genéricas.

**3. ¿`react-router-dom` sigue existiendo?**
No como paquete independiente desde React Router 7: se unificó en `react-router`. Si ves `react-router-dom` en un proyecto o tutorial, es código escrito para una versión anterior.

**4. ¿Cuál es la diferencia entre el modo declarativo y el modo de datos?**
El modo de datos agrega `loader` y `action` por ruta (carga y envío de datos integrados al enrutamiento) mediante `createBrowserRouter` y `RouterProvider`. El modo declarativo, con `BrowserRouter`, solo resuelve qué componente mostrar según la URL, sin ese ciclo de datos.

**5. ¿`Link` dispara un `GET` al servidor como un `<a>`?**
No. React Router previene el comportamiento por defecto del clic y actualiza la URL en el cliente; no hay petición HTTP de documento nuevo. Cualquier dato que la vista necesite se pide por separado, por ejemplo con `fetch` dentro de un efecto (o, en el modo de datos, con un `loader`).

**6. ¿Cuándo elegirías Next.js en vez de React Router en modo declarativo?**
Cuando el proyecto necesita SSR, SEO fuerte o enrutamiento por archivos integrado con obtención de datos en el servidor. Para una SPA sin esos requisitos, React Router en modo declarativo es más simple y suficiente.

-----

## Siguiente lección

Con rutas básicas y enlaces funcionando, el siguiente paso es hacerlas dinámicas: rutas con parámetros de URL, rutas anidadas con `Outlet`, redirecciones y navegación programática: [Rutas dinámicas, anidadas y navegación](02-Rutas%20Din%C3%A1micas%2C%20Anidadas%20y%20Navegaci%C3%B3n.md).
