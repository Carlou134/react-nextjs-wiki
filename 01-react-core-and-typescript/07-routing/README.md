# Routing con React Router

En esta carpeta aprendes a sincronizar la **URL** del navegador con lo que React renderiza, a declarar **rutas dinámicas y anidadas** que comparten un layout, y a **navegar** desde código en vez de depender solo de clics del usuario. Es el paso que convierte una serie de vistas sueltas en una aplicación con varias pantallas navegables por URL, con soporte para atrás/adelante y enlaces compartibles.

-----

## Antes de empezar

Conviene que ya hayas leído:

* [Componentes y Props](../02-componentes-y-props/README.md): cómo se define un componente de función y cómo recibe datos con props.
* [Hooks y Context](../04-hooks-y-context/README.md): sobre todo `useState` y `useEffect`, que las lecciones de esta carpeta dan por conocidos.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

**Sobre la versión de React Router.** Ambas lecciones usan la versión actual de la librería (a partir de la v7): el paquete se llama `react-router` (ya no existe `react-router-dom` como paquete aparte) y el punto de entrada es el **modo declarativo** (`BrowserRouter`, `Routes`, `Route`, `Link`, `NavLink`). La segunda lección también muestra, en su ejemplo completo, el **modo de datos** (`createBrowserRouter` + `RouterProvider`, este último importado desde `react-router/dom`). Si aprendiste con la v6 —la versión más citada en tutoriales— la lógica de `Routes`/`Route`/`Link` es la misma; lo que cambió es el nombre del paquete y la separación explícita en modos (declarativo, de datos y framework), que las lecciones explican antes de usarlos.

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Fundamentos de Routing y React Router](01-Fundamentos%20de%20Routing%20y%20React%20Router.md) | Qué problema resuelve el enrutamiento de cliente, cómo instalar React Router, cómo proveerlo con `BrowserRouter` y cómo declarar rutas estáticas con `Routes`, `Route`, `Link` y `NavLink` | Componentes y props, `useState` |
| [2. Rutas Dinámicas, Anidadas y Navegación](02-Rutas%20Din%C3%A1micas%2C%20Anidadas%20y%20Navegaci%C3%B3n.md) | Parámetros de URL con `useParams`, layouts compartidos con rutas anidadas y `Outlet`, navegación programática con `useNavigate`, y query params con `useSearchParams` | Fundamentos de Routing y React Router |

-----

## El mapa completo en una mirada

```
URL del navegador   <->   Router (BrowserRouter)                    ->   componente renderizado

/                   ->    <Route path="/" element={<Home />} />     ->   Home
/about              ->    <Route path="/about" element={<About />}/> ->  About

Rutas anidadas: el padre siempre se renderiza; el Outlet decide qué hijo aparece

/productos          ->    <Route path="/productos" element={<ProductsLayout />}>
                             <Route index element={<ProductList />} />
                             <Route path=":id" element={<ProductDetail />} />
                           </Route>

/productos            ->  ProductsLayout
                             └── <Outlet />  ->  ProductList   (ruta índice)

/productos/42         ->  ProductsLayout
                             └── <Outlet />  ->  ProductDetail (useParams().id === "42")
```

-----

## Cómo está armada cada lección

Todas las lecciones tienen la misma estructura, así sabes dónde buscar cada cosa:

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El problema:** una situación concreta que muestra por qué existe la herramienta.
4. **Cómo funciona:** el paso a paso, con código chico y explicado.
5. **Ejemplo completo:** todo junto en un caso real.
6. **Errores comunes:** lo que suele salir mal, por qué pasa y cómo se arregla.
7. **En TypeScript:** lo que cambia si tu proyecto usa TypeScript.
8. **Cuándo sí y cuándo no:** para decidir si es la herramienta correcta.
9. **Resumen en 5 líneas:** para repasar.
10. **Para profundizar:** detalles avanzados, plegados. Puedes omitirlos en la primera lectura.
11. **En entrevista:** una respuesta corta (junior), una ampliada (semi-senior) y preguntas de seguimiento frecuentes.

-----

## Después de esta carpeta

Con el enrutamiento cubierto, el siguiente paso es qué hacer cuando algo se rompe durante el render, en la carpeta de manejo de errores: [React Error Boundaries](../08-manejo-de-errores/01-React%20Error%20Boundaries.md).
