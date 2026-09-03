# React Profiler

Aprende a usar la extensión del navegador **React Profiler** para identificar cuellos de botella en el rendimiento y así poder hacer que nuestras aplicaciones sean lo más rápidas posible.

### Introducción

React es rápido por defecto, hasta que no lo es. Es posible escribir código que provoque re-renderizados y recomputaciones innecesarias que harán que nuestras aplicaciones sean lentas. Afortunadamente, React tiene herramientas integradas para identificar estos cuellos de botella y técnicas de rendimiento que pueden ayudarnos a hacer que nuestras aplicaciones sean lo más rápidas posible. Lo único que queda por hacer es averiguar **dónde y cuándo** aplicarlas. En este artículo, aprenderemos a usar el **React Profiler** para detectar cuellos de botella en el rendimiento de nuestras aplicaciones.

Si deseas seguir el ejemplo de la aplicación, puedes descargar el código fuente. Una vez que hayas descargado el código, ejecuta los comandos `npm install` y `npm start` en la terminal raíz de la aplicación.

También hemos incluido este artículo en formato de video. ¡Puedes verlo aquí o desplazarte hacia abajo para seguir leyendo!

[Video](https://www.youtube.com/watch?v=aQT4Dw3Ynso)

## Aplicación de Ejemplo (Example App)

Usaremos una aplicación de ejemplo para demostrar cómo identificar problemas de rendimiento. Esta aplicación permite a un usuario escribir un artículo; luego, muestra un tiempo de lectura estimado al usuario.

![react-profiler-1](/Images/react-profiler-1.png)

Para asegurarse de que el usuario sepa qué es el "Tiempo de lectura" y cómo se calcula, esta aplicación tiene un botón para mostrar una explicación del tiempo de lectura. Al hacer clic, se renderiza un elemento con más información debajo de "Tiempo de lectura".

![react-profiler-2](/Images/react-profiler-2.png)

Cuando escribimos texto dentro del área de texto, la aplicación se siente rápida. Podemos escribir y hacer clic en el botón "What are these stats?" y no hay demora. Sin embargo, si pegamos una gran cantidad de texto, como el texto del artículo de Wikipedia sobre qué es un programa informático, la aplicación comenzará a ralentizarse. Escribir es lento, e incluso hacer clic en el botón para mostrar la explicación del tiempo de lectura es lento. Resulta que hay un problema de rendimiento en esta aplicación.

![react-profiler-3](/Images/react-profiler-3.gif)

## Cómo Identificar el Problema con el React Profiler

Aprendamos cómo podemos identificar el problema con el **React Profiler**.

### Descripción General de Cómo React Aplica los Cambios

Para entender cómo funciona el React Profiler, hablemos brevemente sobre cómo React aplica cambios al DOM.

![react-profiler-4](/Images/react-profiler-4.png)

React aplica los cambios en dos fases. La primera es la **fase de renderizado (render phase)**.

En la fase de renderizado, React mantiene una copia del DOM (llamada "DOM virtual" o "virtual DOM") para que, cuando React detecte que se ha producido un cambio, pueda calcular qué elementos del DOM necesitan ser modificados comparando el estado del DOM virtual antes y después del cambio.

Por ejemplo, antes de que hagamos clic en el botón "What are these stats?", el DOM virtual no tiene ningún elemento que explique lo que significa la estadística de tiempo de lectura. Una vez que hacemos clic en el botón, React detecta ese cambio y renderiza la nueva versión de la aplicación.

Una vez que React tiene tanto el estado anterior como el posterior del DOM, los compara para encontrar qué cambios necesita aplicar al DOM real. Resulta que realizar cambios en el DOM real es lento, por lo que React hace la menor cantidad de cambios posible.

Eso nos lleva a la segunda fase: la **fase de commit (commit phase)**.

Una vez que React sabe qué elementos necesitan cambiar usando su VDOM, realiza los cambios en el DOM del navegador para mostrar el nuevo estado de nuestra aplicación. Esta segunda fase es cómo el React Profiler organiza sus datos, así que ten en cuenta la "fase de commit", hablaremos de ella de nuevo pronto.

Echemos un vistazo.

### El Perfilador (Profiler)

El equipo de React crea y mantiene herramientas para desarrolladores que nos ayudan a escribir y depurar nuestras aplicaciones React. Para acceder al React Profiler, necesitaremos descargar una **extensión del navegador**.

Recomendamos usar Chrome o Firefox, ambos son compatibles con las herramientas de desarrollo de React.

![react-profiler-5](/Images/react-profiler-5.png)

Esta es la página de la extensión de Chrome Web Store para las herramientas de desarrollo de React. Aquí, podemos hacer clic en "Añadir a Chrome" para instalar las herramientas de desarrollo de React.

Una vez que hayamos instalado la extensión, podemos ejecutar una aplicación localmente y ver la herramienta **React Profiler** como una nueva pestaña en las herramientas de desarrollo. Para abrir las herramientas de desarrollo, navega al menú Ver > Desarrollador > Herramientas de desarrollador. Desde allí, haremos clic en la pestaña "Profiler".

### Grabando una Sesión y Leyendo un Gráfico de Llama (Flame Graph)

¿Recuerdas que dijimos que el React Profiler utiliza la **fase de commit** para organizar sus datos? El Profiler tiene un botón de grabación redondo y azul a la izquierda que grabará todos los **commits** realizados durante una sesión.

![react-profiler-6](/Images/react-profiler-6.png)

Después de hacer clic en él, comenzará a grabar una sesión. Durante la sesión, podemos hacer clic dentro de nuestra aplicación. Una vez que hayamos terminado, podemos hacer clic en el botón de grabación nuevamente para detener la grabación de la sesión y ver los resultados. Una vez que detengamos la sesión, el Profiler nos mostrará información sobre cómo funcionó React durante nuestra sesión.

El primer gráfico que vemos es el **gráfico de llama (flame graph)** , que son las largas barras horizontales azules y amarillas. Los gráficos de llama muestran la jerarquía de los componentes que se llamaron durante nuestra sesión y pueden ayudarnos a descubrir dónde pueden estar ocurriendo los problemas de rendimiento.

![react-profiler-7](/Images/react-profiler-7.png)

La primera barra horizontal es azul y dice "App". La etiqueta en cada barra corresponde al **nombre del componente**. Ese es el componente de nivel superior en nuestra aplicación y es lo primero que se renderiza.

Debajo de eso, está la barra "ArticleStats", que es amarilla. Se encuentra debajo de "App" en este caso porque nuestro componente "App" renderiza el componente "ArticleStats". Si el componente `ArticleStats` renderizara otros componentes, se ubicarían debajo de la barra amarilla "ArticleStats".

El **ancho de las barras** también es significativo. Cuanto más ancha sea cada barra, más tiempo tardó en renderizarse. En este caso, `App` renderizó `ArticleStats`. `App` tardó 0.1ms en renderizarse, mientras que `ArticleStats` tardó 931.8ms en renderizarse.

En el Profiler, cuanto más tiempo tarda algo en renderizarse, más amarilla será la barra. Cuanto menos tiempo tarda algo en renderizarse, más azul será la barra. Por lo tanto, cuando veamos **barras amarillas**, puede haber algunos cuellos de botella en el rendimiento que quizás queramos abordar.

![react-profiler-8](/Images/react-profiler-8.png)

Encima del gráfico de llama, hay una fila con cada **commit**. Podemos hacer clic en cada commit para ver un gráfico de llama de esa fase de renderizado/commit en particular. Si recordamos qué acciones realizamos, podemos centrarnos en qué acciones están causando los cuellos de botella en el rendimiento.

A veces, es útil grabar una sesión inmediatamente cuando la página se carga para detectar cualquier cuello de botella durante el **primer renderizado**. Podemos iniciar una sesión después de recargar haciendo clic en el icono de actualizar que está al lado del botón de grabar.

![react-profiler-9](/Images/react-profiler-9.png)

Finalmente, el Profiler tiene algunas **configuraciones (settings)** que pueden ser extremadamente útiles. Por ejemplo, podemos hacer clic en el engranaje de configuración y luego marcar la casilla para **"Highlight updates when components render"** (Resaltar actualizaciones cuando los componentes se renderizan).

![react-profiler-10](/Images/react-profiler-10.png)

Esta opción mostrará un borde resaltado alrededor de cualquier elemento que sea "commiteado" por React mientras usamos la aplicación.

A continuación, podemos hacer clic en la pestaña "Profiler" y marcar la opción **"Record why each component rendered while profiling"** (Registrar por qué se renderizó cada componente durante la creación del perfil).

![react-profiler-11](/Images/react-profiler-11.png)

Esta opción nos mostrará más información en el Profiler sobre por qué un componente en particular necesitaba cambiar.

### Solucionando un cuello de botella de rendimiento

Ahora que conocemos el funcionamiento del Profiler, volvamos a abordar el cuello de botella de rendimiento. Si hacemos clic en la barra "ArticleStats" en el gráfico de llama del React Profiler, vemos que el componente se renderizó porque **"Props changed: (showExplanation)"** (Las props cambiaron: showExplanation). Además, podemos ver que está tardando alrededor de 700ms en renderizarse.

![react-profiler-12](/Images/react-profiler-12.png)

Esto nos da una pista sobre lo que deberíamos inspeccionar en nuestra aplicación, así que saltemos al código.

Cuando hicimos clic en el botón "What are these stats?", nuestra aplicación envió una prop al componente `<ArticleStats />` para alternar un elemento que describe la estadística del tiempo de lectura.

![react-profiler-14](/Images/react-profiler-14.png)

Dado que cambiamos una prop proporcionada a `<ArticleStats />`, React asume que necesitamos volver a ejecutar toda la lógica del componente.

El problema es que el componente realiza un cálculo **costoso y de larga duración** para calcular la estadística del tiempo de lectura.

![react-profiler-15](/Images/react-profiler-15.png)

Podríamos hacer esto mucho más rápido ejecutando ese cálculo **solo si la prop `text` cambia** e ignorando ese cálculo si solo cambia la prop `showExplanation`.

Para solucionar este problema en particular, podemos aprovechar el hook **useMemo()** de React. Podemos implementarlo cambiando este código:

```jsx
const readingTime = getReadingTime(text);
```

por este código:

```jsx
const readingTime = useMemo(() => getReadingTime(text), [text]);
```

El hook `useMemo()` toma una **función** como primer argumento y una **lista de props** (array de dependencias) como segundo argumento. Si la lista de props cambia, entonces ejecutará la función. Es importante destacar que, si las props **no cambian**, `useMemo()` devolverá el último valor calculado **sin recalcular** el resultado. Esto se llama **memoización (memoization)** , de ahí el nombre de `useMemo()`.

Después de hacer este cambio, podemos grabar otra sesión y ver qué sucede.

![react-profiler-13](/Images/react-profiler-13.png)

Ahora el componente `<ArticleStats />` tarda **fracciones de milisegundo** en renderizarse en lugar de 900-1000 ms. Puedes notar que ahora ambas barras son amarillas. Esto se debe a que ambas tienen tiempos de renderizado similares ahora, así que si bien podemos centrarnos en los colores de las barras al identificar problemas, es importante **centrarse en el tiempo de renderizado** después de realizar mejoras de rendimiento. Esta es una gran mejora de rendimiento y hará que nuestra aplicación de escritura se sienta mucho más receptiva.

### Cuándo usar el React Profiler

Nuestro objetivo con todas las mejoras de rendimiento es hacer que nuestras aplicaciones sean **más rápidas para nuestros usuarios**. Cuando una interacción se siente lenta y estamos esperando a que un elemento se actualice en el DOM, entonces el React Profiler suele ser una buena herramienta para ayudar a identificar qué puede estar causando un cuello de botella en el rendimiento.

Dicho esto, el React Profiler es solo una herramienta que podemos usar para intentar identificar un problema. Algunos cuellos de botella pueden ser más fáciles de localizar con otras herramientas, como la pestaña "Performance" en las herramientas de desarrollo integradas del navegador.

### Resumen

Las herramientas de desarrollo de React proporcionan una pestaña **Profiler** en las herramientas de desarrollo de nuestro navegador, que puede grabar una sesión y mostrarnos varios datos de esa sesión. Una vez que hemos identificado un cuello de botella de rendimiento, podemos solucionar el problema y verificar nuestra solución grabando otra sesión.

Con el React Profiler, podemos medir nuestras aplicaciones **antes y después** de los cambios que realizamos, lo que elimina las conjeturas al hacer que nuestras aplicaciones sean más rápidas.

------

## Medí antes de optimizar

El React Profiler no es solo una herramienta para diagnosticar un problema de rendimiento ya detectado: también cumple la función de confirmar que ese problema existe antes de aplicar cualquier optimización. Herramientas como `useMemo()`, `useCallback()` y `React.memo()` (cubiertas en la siguiente lección) tienen un costo propio —código adicional, comparaciones de dependencias, una capa más de indirección— que solo se justifica cuando resuelven un cuello de botella real y medible.

Aplicar memoización de forma preventiva, sin haber confirmado con el Profiler que un componente se re-renderiza con más frecuencia de la necesaria o que un cálculo específico es realmente costoso, suele añadir complejidad al código sin ninguna mejora perceptible para el usuario. La disciplina correcta es siempre la misma: medir con el Profiler antes de optimizar, aplicar el cambio, y volver a medir después para confirmar que la mejora es real. Si la diferencia entre ambas mediciones no es significativa, es preferible mantener el código simple y descartar la optimización.

------

## El Profiler como componente

Todo lo visto hasta acá usa la **extensión de React DevTools** — una herramienta visual que se abre aparte del código. React también expone un componente `<Profiler>`, importable desde `'react'`, que mide el rendimiento de renderizado **programáticamente**, dentro de la propia aplicación:

```javascript
import { Profiler } from 'react';

const onRenderCallback = (
  id,              // el prop "id" del árbol de Profiler que se acaba de commitear
  phase,           // "mount" (primer render) o "update" (re-render)
  actualDuration,  // tiempo que tardó en renderizarse este commit
  baseDuration,    // tiempo estimado que tardaría sin ninguna memoización
  startTime,       // marca de tiempo en la que React empezó a renderizar
  commitTime,      // marca de tiempo en la que React confirmó este commit
  interactions     // conjunto de interacciones asociadas a esta actualización
) => {
  console.log(`${id} (${phase}) tardó ${actualDuration}ms`);
};

const App = () => (
  <Profiler id="App" onRender={onRenderCallback}>
    <Header />
    <UserList />
    <Footer />
  </Profiler>
);
```

Esto es útil en escenarios donde la extensión de DevTools no alcanza: registrar métricas de rendimiento en producción (por ejemplo, enviándolas a un servicio de monitoreo cuando `actualDuration` supera cierto umbral), automatizar alertas de regresión de performance en tests, o medir un árbol específico sin depender de que quien lo ejecute tenga la extensión instalada. Para el trabajo diario de diagnóstico durante el desarrollo, la extensión de DevTools sigue siendo la herramienta más cómoda — el componente `<Profiler>` complementa esa herramienta, no la reemplaza.

------

