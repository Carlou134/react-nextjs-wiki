# React Optimization

## Introduction

A las personas les encanta que sus aplicaciones sean rápidas, y las aplicaciones web no son una excepción. Sin embargo, a medida que las aplicaciones crecen, pueden empezar a sentirse lentas. En respuesta, podemos aplicar técnicas para reducir **renderizados innecesarios** y **cálculos costosos**, lo que resulta en la experiencia fluida que nuestros usuarios esperan.

React proporciona muchas técnicas para resolver problemas de rendimiento. En esta lección, veremos dos de las técnicas más comunes:

1.  **Memoización (Memoizing)** de valores, componentes y funciones.
2.  **División de código (Code splitting)** de módulos o componentes en fragmentos más pequeños.

A lo largo de esta lección, usaremos el **React Profiler** y la pestaña **Network** en las herramientas de desarrollo para medir el rendimiento de una aplicación antes y después de aplicar estas técnicas.

**Nota:** ¿Aún no has visto el video sobre el React Profiler? Te recomendamos que lo veas antes de comenzar con esta lección.

En esta lección, aprenderemos a optimizar nuestras aplicaciones React utilizando varias técnicas, incluyendo:

*   **Memoización** con `React.memo()`, `useMemo()` y `useCallback()`.
*   **División de código** con `import()`, `React.lazy()` y `<Suspense />`.

![Diagrama](/Images/exercise-1-image.png)

Echa un vistazo al diagrama proporcionado, que representa la **carga perezosa (lazy loading)** en una aplicación web de una exposición de arte. En este ejemplo, optimizamos nuestro sitio de la exposición de arte cargando primero la **sección superior** de nuestra aplicación, ya que es lo que el usuario necesitará ver de inmediato. Luego podemos cargar la ubicación y el mapa posteriormente, en segundo plano o cuando el usuario se desplace hasta ellos.

Imagina que enviamos el código para renderizar el mapa y la ubicación cuando la página se cargó por primera vez. ¿Qué tipo de efectos negativos tendría esto en alguien que usa nuestro sitio?

**Pista:**
Si el mapa y la ubicación requirieran solicitudes de red adicionales o ejecutaran cálculos costosos, entonces el usuario tendría que **descargar el código** para renderizar el mapa y la ubicación y **ejecutar ese código** antes de ver cargar la sección superior de la página. Eso haría que la página se sintiera lenta al cargar.

-----

## Memoizing Values

Cuando React carga un componente, no solo renderiza el código **JSX**, sino que también **evalúa todo el código y las funciones** dentro del componente. Algunos componentes requieren funciones que toman mucho tiempo en ejecutarse, incluyendo llamadas a API en una red lenta o consultas grandes a bases de datos. Si esto ocurre cada vez que el componente se renderiza, y si el componente se vuelve a renderizar con frecuencia, el rendimiento general de la aplicación se verá comprometido.

A menudo, los datos devueltos por las llamadas a las API **no cambian** entre re-renderizados del componente. Sería bueno si pudiéramos indicarle a React que solo llame a estas funciones costosas cuando se necesiten nuevos datos, y que devuelva un resultado almacenado en caché cuando los datos sean los mismos. Afortunadamente, React proporciona un hook que puede ayudarnos a optimizar llamadas a funciones costosas, llamado **useMemo()**.

`useMemo()` utiliza una técnica de optimización llamada **memoización (memoization)**. Esta técnica almacena en caché el resultado de una llamada a función y solo vuelve a llamar a la función cuando sus **dependencias cambian**.

El hook `useMemo()` de React toma dos argumentos. El primero es una **función**, y el segundo es un **array de dependencias**. Si alguna de las dependencias cambia, React volverá a calcular el resultado de la función. Su sintaxis se ve así:

```jsx
import { useMemo } from 'react';

function DatabaseQuerier({ query }) {

  const queryResult = useMemo(() => {
    return expensiveDatabaseQuery(query);
  }, [query]); // Solo se recalcula si 'query' cambia
  // ...
}
```

En este ejemplo, el componente `DatabaseQuerier` utiliza la función `expensiveDatabaseQuery()`, que consume mucho tiempo. Sin embargo, gracias a `useMemo()`, el `queryResult` devuelto solo se volverá a calcular si la dependencia `query` cambia.

Hay algunos comportamientos notables de `useMemo()`:

*   La función pasada a `useMemo()` se llamará cuando el componente se **monte (mount)** , por lo que siempre se llamará al menos una vez.
*   La lista de dependencias funciona igual que la lista de dependencias de **useEffect()**: si la lista está **vacía**, entonces solo se ejecutará en el primer renderizado del componente. A menudo queremos pasar una lista de dependencias que coincida con los argumentos de la función costosa ejecutada dentro del callback de `useMemo()`. Esto asegura que cualquier cambio en la entrada de la función nos dará una salida actualizada.
*   La lista de dependencias **no se pasa como argumentos** a la función, pero en la mayoría de los casos, la lista de dependencias debe ser la misma que los argumentos pasados a la función.

----

## Memoizing Components

El hook `useMemo()` memoiza valores devueltos por funciones costosas. Pero, ¿qué pasa si tenemos un componente React costoso que se vuelve a renderizar aunque sus **props** no cambien?

Cuando React detecta un cambio en un componente padre, volverá a renderizar **todos sus componentes hijos** para asegurarse de que la aplicación esté actualizada. Esto puede crear un problema de rendimiento cuando un componente hijo renderiza algo costoso, como miles de elementos o un iframe.

Para solucionar este problema, React proporciona un **componente de orden superior (higher-order component)** llamado **React.memo()**. Un componente de orden superior es un componente que toma otro componente como argumento para poder añadirle funcionalidad. En este caso, `React.memo()` solo permitirá que el componente que se le pasa se vuelva a renderizar si sus **props han cambiado**. Así es la sintaxis de `React.memo()`:

```jsx
import React from 'react';

const MemoizedListComponent = React.memo((props) => {
  const { longList } = props;
  return longList.map(item => <li>{item}</li>);
});
```

En el código anterior, pasamos un componente de función a `React.memo()` como su primer argumento. `React.memo()` comparará las props del componente de función antes y después de cada fase de renderizado y solo lo volverá a renderizar si los valores de esas props han cambiado. Luego, asignamos el resultado de `React.memo()` a `MemoizedListComponent`, que podemos usar en otros componentes.

Ten en cuenta que React solo comparará superficialmente (shallowly) las props del componente memoizado antes y después de cada renderizado. Para asegurar que valores complejos como objetos y arrays se comparen profundamente, podemos proporcionar una **función de comparación** como segundo argumento a `React.memo()`.

```jsx
const sonProfundamenteIguales = (previousProps, nextProps) => {
  return JSON.stringify(previousProps) === JSON.stringify(nextProps);
};

const ComponenteMemoizado = React.memo(ComponenteCostoso, sonProfundamenteIguales);
```

Al aplicar `React.memo()`, el componente de función **no se volverá a renderizar** si su padre cambia y sus props permanecen igual. Dependiendo del rendimiento de renderizado del componente hijo, esto puede resultar en un beneficio de rendimiento sustancial.

----

## Memoizing Functions

React nos proporciona una función de memoización más: **useCallback()**. De nuevo, React volverá a crear todo el código definido dentro de un componente cuando se vuelva a renderizar, incluyendo las funciones. `useCallback()` nos permite **memoizar una función**, evitando que React vuelva a crear esa función cuando el componente se re-renderiza.

Este hook es particularmente importante cuando se pasan funciones como **props** a componentes memoizados con `React.memo()`. Dado que `React.memo()` compara las props entre renderizados, verifica si cada prop es la misma antes y después de cada renderizado. El problema es que en JavaScript, dos funciones idénticas no serán iguales entre sí, ya que se almacenan en el lenguaje como dos **referencias separadas**. Aquí hay un ejemplo:

```jsx
const ejemploA = () => { console.log('ejemplo'); };
const ejemploB = () => { console.log('ejemplo'); };

ejemploA === ejemploB; // false
```

El ejemplo de código anterior devolverá `false`, a pesar de que `ejemploA` y `ejemploB` son funcionalmente equivalentes.

Considera un componente padre que crea una función y la pasa como prop a un componente hijo memoizado. Si el componente padre se vuelve a renderizar, volverá a crear la función y la pasará al hijo. A pesar de ser funcionalmente equivalentes, el componente hijo memoizado no podrá ver que la función antigua y la nueva función son equivalentes y no tendrá más remedio que volver a renderizarse innecesariamente.

Para solucionar esto, podemos usar el hook **useCallback()**, que acepta dos argumentos. El primer argumento es la **función** que queremos memoizar y el segundo es una **lista de dependencias**. El hook `useCallback()` solo volverá a crear la función si su lista de dependencias cambia. Similar a `useEffect()`, si se pasa un array de dependencias vacío a `useCallback()`, la función solo se creará una vez después del primer renderizado.

Aquí hay un ejemplo para demostrar su sintaxis:

```jsx
import { useState, useCallback } from 'react';
import MemoizedCounter from './Counter.js';

const CounterContainer = () => {
  const [count, setCount] = useState(0);
  const increment = useCallback(() => setCount((count) => count + 1), []);

  return <MemoizedCounter onClick={increment} />;
}
```

En este ejemplo, React solo volverá a crear la función `increment()` cuando su lista de dependencias cambie. Esto es importante, porque si `<MemoizedCounter>` está envuelto con `React.memo()`, comparará la prop `onClick` antes y después de cada renderizado para ver si `<MemoizedCounter />` debe volver a renderizarse. Dado que `useCallback()` **persistirá la función `increment()`** entre fases de renderizado, `<MemoizedCounter />` solo se renderizará al montarse (mount).

-----

## Comparativa: React.memo, useMemo y useCallback

Con las tres herramientas de memoización ya presentadas, conviene resumir en qué se diferencian y cuándo corresponde usar cada una:

| Herramienta | Qué memoiza | Se usa cuando... |
|---|---|---|
| `React.memo()` | Un **componente** completo | Un componente hijo renderiza algo costoso y sus props no cambian tan seguido |
| `useMemo()` | Un **valor** calculado | Un cálculo es costoso (bucles, consultas, transformaciones de listas grandes) |
| `useCallback()` | Una **función** | Se pasa una función como prop a un componente envuelto en `React.memo()`, o la función es dependencia de un `useEffect()` |

Las tres herramientas suelen combinarse: `React.memo()` evita que un componente se vuelva a renderizar si sus props no cambiaron, pero esa comparación de props es superficial. Si entre esas props hay un objeto, un array o una función recreados en cada render del padre —aunque su contenido sea idéntico— `React.memo()` los verá como "distintos" y el componente se volverá a renderizar de todas formas. `useMemo()` resuelve esto para valores derivados (objetos, arrays) y `useCallback()` lo resuelve para funciones, preservando la misma referencia entre renders mientras sus dependencias no cambien.

-----

## Memoización en profundidad

La idea detrás de toda técnica de memoización, independientemente de la API concreta que se use, es siempre la misma: **guardar el resultado de un cálculo y reutilizarlo mientras sus entradas no cambien, en lugar de rehacer ese cálculo en cada render**. `useMemo()` cachea un valor, `useCallback()` cachea una función y `React.memo()` cachea, en la práctica, el resultado de renderizar un componente entero; los tres siguen exactamente el mismo principio aplicado a distintos tipos de resultado.

Ese principio, sin embargo, no implica que memoizar sea gratis. Envolver un valor en `useMemo()` o una función en `useCallback()` tiene un costo propio: React debe almacenar el valor anterior, comparar el array de dependencias en cada render y decidir si recalcula o no. Cuando el cálculo que se memoiza es barato —sumar dos números, concatenar dos strings, crear una función simple que no se pasa a ningún componente memoizado— ese costo de comparar dependencias puede terminar siendo mayor que el costo de simplemente recalcular el valor. Memoizar en esos casos no mejora el rendimiento: solo agrega una capa de complejidad e indirección al código, sin ningún beneficio medible.

Por eso, antes de aplicar `useMemo()`, `useCallback()` o `React.memo()`, conviene confirmar que existe un problema real de rendimiento —usando el React Profiler— en lugar de memoizar de forma preventiva. Como criterio práctico, si una optimización de este tipo no produce una mejora claramente perceptible en el tiempo de renderizado, probablemente no vale la pena mantenerla a cambio de la complejidad adicional que introduce. Los casos donde la memoización sí suele pagar su costo son los ya mencionados: cálculos genuinamente costosos, listas o estructuras grandes derivadas de otros datos, y props (objetos, arrays o funciones) que se pasan a componentes envueltos en `React.memo()`, donde evitar una nueva referencia en cada render es precisamente lo que permite que la memoización del componente hijo funcione.

-----

## Code Splitting Modules

Hay otra categoría de técnicas para hacer nuestras aplicaciones más rápidas, que implica enviar solo las partes necesarias de nuestro código React cuando el usuario las necesita. Esto se llama **división de código (code splitting)**.

Cuando construimos una aplicación React, todo el código de nuestra aplicación se reúne en un archivo llamado **bundle (paquete)** , que los usuarios descargan cuando solicitan nuestra aplicación. Eso significa que los usuarios descargan **todo** el código de nuestra aplicación cuando cargan nuestra aplicación por primera vez. A medida que nuestras aplicaciones crecen en complejidad, también lo hará el tamaño del bundle de nuestra aplicación. Esto hará que un usuario tarde más en descargar y cargar nuestra aplicación, especialmente si tienen una conexión a internet lenta.

Una forma de hacer que nuestra aplicación sea más rápida es dividir módulos grandes del bundle de nuestra aplicación en sus propios **fragmentos (chunks)**. Un fragmento es un archivo JavaScript más pequeño que se enviará a un usuario por separado del bundle principal. Un fragmento se puede cargar después de que se cargue el bundle, lo que lleva a una mejora en el rendimiento.

Normalmente, importaríamos un módulo, como 'moment', en nuestra aplicación de esta manera:

```jsx
import moment from 'moment';

// ...

function onClick() {
  setDate(moment(new Date()).format('MM/D/YYYY'));
}
```

Cuando miramos en la pestaña **Network** de las herramientas de desarrollo, podemos ver que todo el código de nuestra aplicación se agrupó en `bundle.js` de nuestra aplicación. El archivo `bundle.js` a continuación incluye la biblioteca `moment` dentro de él.

![parte-1](/Images/excercise-5-image-1.png)

En su lugar, queremos enviar el contenido del módulo `moment` **solo cuando el usuario interactúa con una característica que requiere `moment`**. Para hacer esto, podemos usar la sintaxis **import()** para cargarlo como su propio fragmento (chunk):

```jsx
async function onClick() {
  const moment = await import('moment');
  setDate(moment.default(new Date()).format('MM/D/YYYY'));
}
```

La sintaxis `import()` se basa en **Promesas** de JavaScript, por lo que usamos `await` para esperar a que la Promesa se resuelva y marcamos la función como `async`. Además, tenemos que añadir `.default` a `moment` después de importarlo con `import()` para asegurarnos de seleccionar la **exportación por defecto (default export)** de `moment`.

Ahora, cuando miramos en la pestaña **Network**, vemos que el módulo `moment` ahora es su propio fragmento, descargado por separado del bundle principal.

![parte-1](/Images/excercise-5-image-2.png)

El último fragmento (chunk) listado en la imagen de arriba (`vendors-node_modules_moment_moment.chunk.js`) es el módulo `moment`.

----

## Code Splitting Components

A medida que nuestras aplicaciones crecen, podemos empezar a crear componentes de React que están llenos de lógica y funcionalidad complejas, como un componente para presentar un mapa con indicaciones para llegar. Este componente sería grande. En lugar de incluir un componente grande en el bundle principal de nuestra aplicación, podemos separarlo en un **fragmento (chunk)** , lo que permitirá a los usuarios descargarlo **después** del bundle principal de nuestra aplicación.

Normalmente, importaríamos un componente React de esta manera:

```jsx
import DrivingDirectionsMap from './DrivingDirectionsMap';
```

Importar un componente así lo incluirá en el bundle principal de nuestra aplicación.

Para dividir el componente `DrivingDirectionsMap` en su propio fragmento, podemos usar **React.lazy()**, que tiene la siguiente sintaxis:

```jsx
const DrivingDirectionsMap = React.lazy(() => import('./DrivingDirectionsMap'));
```

`React.lazy()` acepta una **función de callback** como argumento. Dentro del callback, usamos la función **import()**, que discutimos en el ejercicio anterior. Luego, `React.lazy()` devolverá el componente `<DrivingDirectionsMap>`. Cuando el usuario cargue la página, descargará este componente como su propio fragmento, que podremos ver en la pestaña **Network** de las herramientas de desarrollo.

Con `React.lazy()`, podemos asegurarnos de que los componentes que son **grandes y no esenciales** para el primer renderizado de nuestra aplicación se descarguen por separado del bundle principal de nuestra aplicación.

-----

## Suspense

Cuando cargamos un componente como `<DrivingDirectionsMap>` con `React.lazy()`, el componente se descargará como su propio fragmento (chunk) por separado del bundle principal. Sin embargo, React no podrá renderizar el componente `<DrivingDirectionsMap>` por separado de nuestro bundle principal. En cambio, React esperará hasta que **todos los fragmentos se hayan descargado**, después de lo cual mostrará nuestra aplicación.

Para indicarle a React que cargue primero el resto de nuestra aplicación y luego inserte nuestro componente cargado de forma perezosa, podemos usar el componente **<Suspense>** de React.

El componente `<Suspense>` instruye a React para que cargue **todas las partes de nuestra aplicación excepto los componentes importados con `React.lazy()`**. Mientras esos componentes se descargan, `<Suspense>` mostrará un **estado de carga (loading state)**. Una vez que el componente cargado de forma perezosa esté listo, `<Suspense>` insertará el componente en nuestra aplicación.

Así es como se ve la sintaxis de `<Suspense>` de React:

```jsx
import React, { Suspense } from 'react';

// ...

const DrivingDirectionsMap = React.lazy(() => import('./DrivingDirectionsMap'));

// ...

return (
  <Suspense fallback={<p>Cargando...</p>}>
    <DrivingDirectionsMap />
  </Suspense>
);
```

En el ejemplo anterior, el componente `<Suspense>` tiene al componente `<DrivingDirectionsMap>`, importado con `React.lazy()`, como su hijo.

`<Suspense>` también toma una prop llamada **`fallback`**, que es un componente de React utilizado para mostrar un estado de carga mientras se cargan sus hijos. En el ejemplo, mientras `<DrivingDirectionsMap>` se está descargando, la interfaz de usuario mostrará un párrafo con el texto "Cargando..." hasta que el componente `<DrivingDirectionsMap>` esté listo para ser mostrado.

Al importar componentes con `React.lazy()`, **siempre debemos envolver el componente cargado de forma perezosa en el componente `<Suspense>` de React**, para que el componente cargado de forma perezosa pueda renderizarse en nuestra aplicación después del resto de la aplicación. Esto permitirá que nuestras aplicaciones se carguen rápidamente en el primer renderizado y luego se llenen con componentes que son costosos de descargar.

----

## Review

Las aplicaciones React son rápidas por defecto, sin embargo, cuando se vuelven lentas, podemos **identificar cuellos de botella** en el rendimiento y aplicar optimizaciones para hacer nuestras aplicaciones más rápidas.

Para identificar cuellos de botella en el rendimiento, podemos usar el **React Profiler** para examinar qué componentes se renderizan y con qué frecuencia. También podemos usar la pestaña **Network** en las herramientas de desarrollo para inspeccionar el tamaño del bundle de JavaScript que enviamos a los usuarios.

Una vez que hemos identificado un cuello de botella en el rendimiento, podemos comenzar a aplicar **optimizaciones de rendimiento**. React proporciona herramientas que se dividen en dos categorías: **memoización (memoization)** y **división de código (code splitting)**.

### Para memoizar datos, podemos utilizar:

*   **useMemo()** - memoiza un **valor**.
*   **React.memo()** - memoiza un **componente**.
*   **useCallback()** - memoiza una **función**.

### Para dividir nuestro código en fragmentos (chunks), podemos usar:

*   **import()** - puede dividir un **módulo** en un fragmento.
*   **React.lazy()** y **<Suspense />** - pueden dividir un **componente** en su propio fragmento, al mismo tiempo que proporcionan un estado de carga y permiten que el resto de nuestra aplicación se cargue primero.

**Medir, aplicar optimizaciones de rendimiento y medir de nuevo** es la mejor manera de asegurarnos de que nuestras aplicaciones se sientan lo más rápidas posible para nuestros usuarios. Con las herramientas que aprendimos a lo largo de esta lección, estamos listos para hacer que nuestras aplicaciones sean rápidas, lo que es una gran experiencia para nuestros usuarios.

![ejercicio-final](/Images/excercise-8-image.png)

----

