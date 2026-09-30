# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**BrowserRouter.** Componente de React Router que provee el router usando la History API del navegador. Envuelve toda la aplicación, normalmente en el punto de entrada, para que cualquier componente hijo pueda usar los componentes y Hooks de enrutamiento.

**Enrutamiento (routing).** Proceso de decidir qué contenido mostrar según la URL actual. En una SPA lo resuelve una librería como React Router, del lado del cliente y sin pedir un documento nuevo al servidor.

**Historial de navegación.** Pila de URLs que el navegador recuerda para permitir avanzar y retroceder. `navigate(-1)` y `navigate(1)` se mueven por esta pila, no por la estructura de rutas de la aplicación.

**History API.** API del navegador, con métodos como `pushState` y `replaceState`, que permite cambiar la URL visible sin pedir un documento nuevo al servidor. `BrowserRouter` la usa para actualizar la URL y escucha el evento `popstate` para reaccionar a los botones atrás/adelante.

**Link.** Componente de React Router que reemplaza a `<a>` para navegación interna. Renderiza una etiqueta `<a>` en el DOM, pero intercepta el clic, actualiza la URL con la History API y evita la recarga completa de la página.

**Modo declarativo.** Forma de usar React Router basada solo en componentes JSX (`BrowserRouter`, `Routes`, `Route`, `Link`), sin carga de datos integrada por ruta. Es el punto de entrada recomendado para una SPA simple hecha con Vite.

**Modo de datos.** Forma de usar React Router que agrega `loader`, `action` y estados de carga por ruta, mediante `createBrowserRouter` y `RouterProvider`. Se usa cuando cada ruta necesita cargar o enviar sus propios datos.

**Modo framework.** Forma de usar React Router que suma al modo de datos un plugin propio de Vite, con convenciones de archivos, *code splitting* automático y soporte para SSR; en la práctica funciona como un framework completo.

**Navegación programática.** Cambiar la URL desde código, en respuesta a un evento o dentro de un efecto, en vez de esperar a que el usuario haga clic en un enlace. Se logra con el Hook `useNavigate`.

**NavLink.** Componente de React Router similar a `Link`, que además marca automáticamente el enlace activo (clase `active` y atributo `aria-current="page"`) cuando su destino coincide con la URL actual. Útil para menús de navegación.

**Outlet.** Componente que marca, dentro del layout de una ruta padre, el punto donde debe renderizarse la ruta hija activa. No decide nada por sí mismo: React Router lo reemplaza por el elemento de la ruta hija que coincide con la URL.

**Parámetro de consulta (query param).** Par `clave=valor` que aparece en la URL después de `?`, opcional, usado típicamente para filtros, orden o paginación. Se lee y actualiza con el Hook `useSearchParams`.

**Parámetro de URL.** Segmento variable de una ruta, escrito como `:nombre` (por ejemplo, `:id` en `/productos/:id`). Convierte una ruta fija en una ruta dinámica y se lee con el Hook `useParams`.

**Redirección.** Cambio de la URL disparado por la aplicación en vez de por un clic del usuario. Puede hacerse con el componente `<Navigate>` (desaconsejado hoy en componentes de función) o, de forma vigente, llamando a `useNavigate` dentro de un efecto.

**Route.** Elemento de configuración que asocia un patrón de URL (`path`) con el componente que debe renderizarse (`element`) para esa URL. Puede anidarse dentro de otro `Route` para compartir un layout.

**Router.** Objeto que centraliza la configuración de rutas y decide, en cada cambio de URL, qué renderizar. React Router lo provee mediante componentes como `BrowserRouter`.

**Routes.** Componente que recorre sus `Route` hijos y renderiza el primero cuyo `path` coincide con la URL actual, de forma similar a un `switch`. Si ninguno coincide, no renderiza nada, salvo que exista una ruta con `path="*"`.

**Ruta (route).** Asociación entre un patrón de URL y el componente que debe renderizarse para esa URL.

**Ruta absoluta.** Ruta que empieza con `/` (por ejemplo `/about`). React Router la resuelve siempre desde la raíz del sitio, sin importar en qué ruta esté el componente que renderiza el enlace.

**Ruta anidada.** `Route` declarada dentro de otra. La de afuera es la ruta padre; la de adentro, la ruta hija, y su `path` es relativo al de la ruta padre (no repite ese prefijo).

**Ruta dinámica.** Ruta que incluye uno o más parámetros de URL y coincide con muchas URLs distintas, no con una sola.

**Ruta índice.** Ruta hija especial, declarada con la prop `index` en vez de `path`, que se renderiza cuando la URL coincide exactamente con la ruta del padre.

**Ruta relativa.** Ruta que no empieza con `/`. React Router la resuelve a partir de la ruta actual, no desde la raíz; solo tiene sentido dentro de rutas anidadas.

**SPA (Single Page Application).** Aplicación que carga un único documento HTML y usa JavaScript para actualizar la interfaz sin pedir páginas nuevas al servidor.

**useNavigate.** Hook que devuelve una función para cambiar la URL desde código. Se usa dentro de manejadores de eventos o de efectos, y también acepta un número entero (`navigate(-1)`) para moverse por el historial de navegación.

**useParams.** Hook que devuelve un objeto con los valores actuales de los parámetros de URL de la ruta activa. Los valores siempre son texto, aunque en la URL parezcan números.

**useSearchParams.** Hook que devuelve una tupla con el `URLSearchParams` actual de la URL y una función para actualizarlo. Nunca se debe mutar el objeto directamente: hay que llamar siempre a la función que devuelve el Hook.
