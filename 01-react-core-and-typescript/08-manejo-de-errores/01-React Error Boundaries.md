# React Error Boundaries

## Introduction

En este punto de tu viaje de aprendizaje, es probable que te hayas encontrado con el bloque **try/catch**. Como sugieren los nombres de las palabras clave, los bloques `try` permiten a los programadores intentar ejecutar algún código. Si se lanza un error en tiempo de ejecución, los bloques `catch` interceptan y manejan el error.

```jsx
try {
  estoPodriaLanzarUnError();
} 
catch (error) {
  esoEstaBienPodemosManejarlo(error);
}
```

Los planes de respaldo como los bloques `try/catch` permiten a los programadores responder a los errores y, con suerte, solucionarlos, en lugar de dejar que estos bloqueen toda la aplicación. React utiliza una técnica similar para manejar errores dentro de su árbol de componentes: los **componentes de límite de errores (error boundaries)**.

Los **límites de errores** son componentes de React que capturan errores en tiempo de ejecución en cualquier parte de su árbol de componentes hijos. El límite de errores puede registrar esos errores y mostrar una interfaz de usuario de respaldo (una "UI de respaldo" o "fallback UI") en lugar de una pantalla en blanco. Los límites de errores se utilizan comúnmente alrededor de nuevas características que no han sido probadas exhaustivamente, ya que permiten recuperarse en caso de un error.

Profundizaremos en la sintaxis más adelante, pero aquí hay un ejemplo rápido de un componente de límite de errores protegiendo un componente propenso a errores:

```jsx
<ErrorBoundary FallbackComponent={FallbackUI}>
  <MiComponentePropensoAErrores />
</ErrorBoundary>
```

En esta lección, aprenderemos cómo construir un componente `ErrorBoundary` simple para proteger nuestra aplicación. Luego, veremos una implementación popular que puedes usar en tus proyectos en lugar de tener que crearla desde cero cada vez. Al final de esta lección, podrás:

*   Crear un componente de clase `ErrorBoundary` que envuelva otros componentes.
*   Usar el método estático **getDerivedStateFromError()** para renderizar una UI de respaldo después de que se haya lanzado un error.
*   Usar **componentDidCatch()** para registrar información del error.
*   Restablecer los componentes rotos a un buen estado.
*   Enumerar ejemplos de errores para los que se pueden usar los límites de errores (y aquellos para los que no se pueden usar).
*   Usar el componente `ErrorBoundary` del paquete **react-error-boundary**.

¡Comencemos!

---

## Creating an Error Boundary

Como aprendimos en la introducción, los **límites de errores** son componentes de React que hacen algunas cosas clave:

*   Envuelven a otros componentes.
*   Renderizan sus hijos si no se detectan errores.
*   Renderizan una **UI de respaldo (fallback UI)** si se detecta un error.
*   Registran el error de alguna manera si se detecta un error.

En los próximos ejercicios, implementaremos cada una de estas características. Para este ejercicio, comencemos de a poco y centrémonos en esos dos primeros pasos. Podemos crear un `ErrorBoundary` que envuelva otros componentes y renderice sus hijos cuando no haya errores presentes, de la siguiente manera:

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return this.props.children;
  }
}
```

En este ejemplo, creamos un componente de clase `ErrorBoundary` cuya función `render()` devuelve `this.props.children`: los descendientes que el `ErrorBoundary` envuelve.

Luego podríamos comenzar a usar este `ErrorBoundary` en una aplicación de la siguiente manera:

```jsx
<ErrorBoundary>
  <MiComponentePropensoAErrores />
</ErrorBoundary>
```

En este punto, nuestro componente `ErrorBoundary` actúa simplemente como un componente de paso para `MiComponentePropensoAErrores`. ¡Esto es ideal! Cuando no hay errores presentes, queremos que nuestro `ErrorBoundary` se mantenga fuera del camino. Pero ¿qué sucede si ocurre un error? ¡Manejaremos eso en el próximo ejercicio!

Ten en cuenta que estamos creando este `ErrorBoundary` como un **componente de clase**, no como un componente de función. Esto es intencional. De hecho, los límites de errores deben crearse como componentes de clase, y actualmente no hay forma de escribir un límite de errores como un componente de función.

Ahora es tu turno de intentar hacer tu propio `ErrorBoundary`. Echa un vistazo al código en `App.js` en el editor proporcionado. Es una aplicación de interruptores de luz con cuatro interruptores. Cada interruptor tiene un botón de encendido/apagado y un botón etiquetado como "Bad Switch" que lanzará un error. Confirma que cada interruptor controla su propia sección de la aplicación y que presionar el botón de error bloqueará la aplicación. Actualiza la aplicación para probar cada botón de error.

---

## Rendering a Fallback UI

En este punto, nuestro componente `ErrorBoundary` no es realmente un límite de errores. Todavía deja pasar errores que provocan que toda la aplicación se bloquee. Para que nuestro límite de errores proteja realmente nuestra aplicación, necesitamos:

1.  **Identificar** cuándo ha ocurrido un error.
2.  **Almacenar** el error internamente.
3.  **Renderizar** una UI de respaldo.

Primero, podemos realizar un seguimiento de si hay un error presente usando el **estado interno** del componente `ErrorBoundary`. En el `constructor()`, añadiremos algo de estado:

```jsx
constructor(props) {
   super(props);
   // Nuevo código
   this.state = { error: null };
}
```

Observa que nuestro estado comienza con `error` teniendo un valor `null`, es decir, no hay ningún error.

A continuación, para actualizar este valor cuando ocurra un error, definimos el método estático **getDerivedStateFromError()**:

```jsx
constructor(props) { //… }
 
// Nuevo código
static getDerivedStateFromError(error) {
   return { error }; 
}
 
render() { //… }
```

Este método se invoca después de que un componente hijo haya lanzado un error. Recibe el error que se lanzó como parámetro y debe devolver el **siguiente estado** del límite de errores. En este caso, queremos devolver el error que recibió el componente.

**Nota:** Aquí usamos la abreviatura de ES6. `{ error }` es equivalente a `{ error: error }`.

Finalmente, podemos usar el valor `this.state.error` del límite de errores para determinar qué renderizar: el árbol de componentes envuelto o la UI de respaldo.

```jsx
render() {
  // Nuevo código
  if (this.state.error) {  // Renderizar una UI de respaldo si ocurrió un error
    return (
      <div>
        <h2>¡Se detectó un error!</h2>
      </div>
    );
  }
  return this.props.children;  // De lo contrario, ¡renderizar los hijos!
}
```

Ahora, nuestro límite de errores primero verifica su propio valor interno `this.state.error` antes de renderizar sus hijos. Si se encuentra un error, se muestra la UI de respaldo en su lugar.

¡Probémoslo!

----

## Logging Errors

Renderizar una UI de respaldo es una mejora valiosa para la experiencia del usuario. Sin embargo, cuando ocurre un error, es importante que los desarrolladores (y esos valientes usuarios que envían informes de errores) puedan ver la información del error y hacer un plan para evitar que esos errores vuelvan a ocurrir.

El **registro de errores (error logging)** se implementa en un componente `ErrorBoundary` usando el método de ciclo de vida **componentDidCatch()**:

```jsx
componentDidCatch(error, errorInfo) {
  console.log(error);
  console.log(errorInfo.componentStack); 
}
```

Este método es llamado por React después de que uno de sus componentes hijos haya lanzado un error. Cuando se llama, recibe un objeto `error` que representa el error lanzado. También recibe un objeto `errorInfo` con una propiedad `.componentStack`. Esta propiedad es increíblemente útil, ya que contiene un **seguimiento de pila (stack trace)** que muestra el historial de componentes renderizados que condujeron al error.

```
Error: ¿Por qué siquiera tenemos este interruptor?
    in LightSwitch (creado por App)
    in ErrorBoundary (creado por App)
    in div (creado por App)
    in App
```

Al registrar este seguimiento de pila, podemos localizar con mayor precisión la fuente del error.

La forma en que tu equipo utiliza el método `componentDidCatch()` para registrar la información del error puede variar ampliamente. Al probar esta característica, puedes decidir simplemente imprimir el error en la consola. Sin embargo, en aplicaciones de producción, esta información a menudo se envía a un servidor para monitoreo, alertas y depuración.

¡Ahora es tu turno de intentarlo!

----

## Resetting the Component

Hasta ahora, hemos podido capturar errores, registrarlos en la consola y mostrar una UI de respaldo. Esto hace posible que el resto de nuestra aplicación continúe ejecutándose, pero ¿cómo recuperamos el componente roto más allá de actualizar la página?

Nuestro componente `<ErrorBoundary>` determina si renderizar el componente hijo o la UI de respaldo basándose en su propio valor interno `this.state.error`. Si cambiamos este valor de estado de error de vuelta a `null`, el componente `ErrorBoundary` se volverá a renderizar junto con el componente hijo envuelto por el límite de errores.

Si bien hay muchas maneras de hacer esto, en este ejercicio, repasaremos un enfoque bastante básico: agregar un botón que el usuario pueda presionar para restablecer el componente roto. Observa el ejemplo a continuación:

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
    this.reset = this.reset.bind(this);
  }
 
  reset() {
    this.setState({ error: null });
  }

  static getDerivedStateFromError(error) {
    return { error };
  }
 
  componentDidCatch(error, errorInfo) {
    console.log(error);
    console.log(errorInfo.componentStack);
  }
 
  render() {
    if (this.state.error) {
      return (
        <div>
          <h2>Algo salió mal.</h2>
          <button onClick={this.reset}>
            Restablecer
          </button>
        </div>
      );
    }
    return this.props.children;
  } 
}
```

En este ejemplo:

*   Modificamos la función `constructor()` para **vincular (bind)** el método `reset()` al valor `this` del componente de clase. Este paso es necesario para asegurar que la llamada a `this.setState()` dentro de `reset()` tenga la referencia `this` adecuada. Puedes leer más sobre por qué esto es necesario en la [documentación de React](https://es.react.dev/docs/faq-functions.html#bind-in-constructor-why).
*   Creamos una nueva función **manejadora de eventos**, `reset()`. Cuando se llama, establecerá el valor del estado `error` a `null`.
*   Creamos un nuevo elemento **botón** en la UI de respaldo con `this.reset` asignado a la prop `onClick`.

Cuando el usuario hace clic en el elemento botón de la UI de respaldo, se llamará a la función `reset()` y el valor del estado `error` se restablecerá a `null`. Como resultado, el componente `ErrorBoundary` se volverá a renderizar y renderizará `this.props.children`.

En aplicaciones de producción, la lógica de `reset()` puede ser más compleja. Podríamos intentar recuperar datos no guardados, hacer una llamada a una API para restablecer el estado en un servidor o base de datos, o cerrar la sesión de un usuario para evitar el acceso no deseado a funciones protegidas. Este ejemplo ofrece una visión general de alto nivel de lo que se necesita hacer para que nuestra aplicación vuelva a un estado de funcionamiento.

¡Intentémoslo nosotros mismos!

-----

## Implementing react-error-boundary

Normalmente, los componentes de límite de errores se crean una vez y luego se usan en toda la aplicación donde sea necesario. Una vez que el componente de límite de errores ha sido configurado, no hay mucha necesidad de modificar su código. Como resultado, no es común que los programadores creen sus propios componentes `ErrorBoundary` desde cero. En su lugar, a menudo recurren a implementaciones como el popular paquete **react-error-boundary**.

Este paquete exporta un componente `ErrorBoundary` que hace todo lo que nuestro componente `ErrorBoundary` puede hacer ¡y más! Para replicar el comportamiento básico de UI de respaldo de nuestro propio componente `ErrorBoundary` personalizado, podríamos escribir:

```jsx
import { ErrorBoundary } from 'react-error-boundary';
 
function ErrorFallback() {
  return (
    <div>
      <h2>¡¡Se detectó un error!!</h2>
    </div>
  );
}
 
function App() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <MiComponente />
    </ErrorBoundary>
  );
}
```

En este ejemplo:

1.  **Importamos** el componente `ErrorBoundary` desde `'react-error-boundary'`.
2.  **Definimos** un componente `ErrorFallback()` con la UI que se renderizará si el `ErrorBoundary` captura un error.
3.  **Renderizamos** el `ErrorBoundary` con nuestro componente anidado dentro.
4.  **Pasamos** una prop `FallbackComponent={ErrorFallback}` al `ErrorBoundary`. Cuando el `ErrorBoundary` captura un error, ¡el componente `ErrorFallback` se renderiza en su lugar!

En este punto, este `ErrorBoundary` no tiene ningún comportamiento de registro o restablecimiento, pero llegaremos a eso en el próximo ejercicio. Por ahora, ¡añadamos el `ErrorBoundary` básico a la aplicación de los interruptores de luz!

----

## Logging and Resetting with react-error-boundary

Ahora podemos reemplazar nuestro `ErrorBoundary` personalizado con el componente `ErrorBoundary` de `react-error-boundary`, pero hay algunas características que perdimos. Específicamente, ¡no tenemos registro de errores ni la función de restablecimiento del componente!

Afortunadamente, `react-error-boundary` tiene implementaciones integradas para añadir estas importantes características. Echemos un vistazo.

Primero, podemos ver cómo añadir el **registro de errores** con la prop `onError`:

```jsx
<ErrorBoundary onError={logError} FallbackComponent={ErrorFallback}>
  <MiComponente/>
</ErrorBoundary>
```

En este ejemplo, añadimos la prop `onError` con nuestra función de registro `logError` como valor. `onError` enviará dos valores familiares a la función proporcionada: `error` y `errorInfo`. Observa que estos son los mismos argumentos que se pasan al método de ciclo de vida `componentDidCatch()`.

A continuación, podemos ver cómo **restablecer el ErrorBoundary** con la prop `resetErrorBoundary`:

```jsx
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <h2>¡¡Ocurrió un error en la aplicación!!</h2>
      <p>Error: {error.message}</p>
      <button onClick={resetErrorBoundary}>
        Restablecer
      </button>
    </div>
  );
}
```

Cuando nuestro `ErrorBoundary` renderiza un componente de respaldo, le pasará dos valores como props: `error` y `resetErrorBoundary`.

*   `error` es el objeto `Error` en sí mismo, que puede ser útil si queremos mostrar `error.message`.
*   `resetErrorBoundary` es una función de callback que restablecerá el estado de error interno del propio `ErrorBoundary`. Este callback se puede asignar a la prop `onClick` del elemento botón en nuestra UI de respaldo.

Con este código implementado, cuando el usuario haga clic en el botón "Restablecer" en nuestra UI de respaldo, el `ErrorBoundary` se restablecerá e intentará volver a renderizar el componente hijo roto.

**Nota:** Si echas un vistazo al código fuente de esta característica, ¡puedes ver que es básicamente la misma que nuestra propia implementación!

-----

## Passing Props to Fallback Components

En el ejercicio anterior, practicamos la renderización de una UI de respaldo usando la prop `FallbackComponent` asignada a un componente `<ErrorFallback>`. El componente `<ErrorFallback>` recibió dos props en sí mismo: `error` y `resetErrorBoundary`:

```jsx
function ErrorFallback({ error, resetErrorBoundary }) {
  // Manejar la lógica de error / resetErrorBoundary...
}

// Más tarde en algún JSX renderizado...
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <ComponenteProtegido />
</ErrorBoundary>
```

Los dos valores, `error` y `resetErrorBoundary`, se pasan automáticamente como props al componente que se proporcione como `FallbackComponent`. Pero si necesitáramos pasar props adicionales a `<ErrorFallback>`, ¿cómo podríamos hacerlo?

Pasar props adicionales a nuestra UI de respaldo nos permite una **flexibilidad mucho mayor** en cómo renderizamos ese respaldo. Es posible que queramos proporcionar más detalles sobre el error lanzado o cambiar entre diferentes UIs dependiendo del error. Para cualquiera de estos escenarios, ¡la solución se puede lograr usando los conceptos básicos de React!

En lugar de proporcionar `ErrorFallback` como el valor de la prop `FallbackComponent`, podemos proporcionar un componente de función anónima en línea (inline) de la siguiente manera:

```jsx
function ErrorFallback({ error, resetErrorBoundary, newProp }) {
  // Manejar la lógica de error / resetErrorBoundary...
  // ¡Pero ahora también tenemos el valor newProp!
}

// Más tarde en algún JSX renderizado...
<ErrorBoundary
  FallbackComponent={(props) => (
    <ErrorFallback {...props} newProp={"foo"} />
  )}
>
  <ComponenteProtegido />
</ErrorBoundary>
```

Analicémoslo:

*   El componente `<ErrorBoundary>` se sigue renderizando con la prop `FallbackComponent`.
*   El componente pasado como `FallbackComponent` es un componente de función anónima en línea con un argumento `props`. Sabemos que este argumento `props` es un objeto con dos valores: `error` y `resetErrorBoundary`.
*   El componente de función anónima en línea devuelve el componente `<ErrorFallback>` con los valores originales de `props` (`error` y `resetErrorBoundary`) junto con un valor `newProp` (en este caso `"foo"`).

Pasar un valor como `"foo"` es bastante trivial, pero ilustra el punto: ¡podemos pasar cualquier prop adicional a nuestra UI de respaldo usando este enfoque! Veamos cómo esto puede ser más útil en nuestra aplicación de ejemplo.

-----

## When & Where to Use Error Boundaries

Antes de terminar, debemos discutir **cuándo y dónde** usar los límites de errores para maximizar su impacto potencial, así como sus **limitaciones**.

### Manténlo Enfocado (Keep It Focused)

Los límites de errores aíslan un árbol de componentes del resto de la aplicación y renderizan una UI de respaldo cuando ocurre un error. Sin embargo, si nuestro límite de errores está demasiado lejos de la fuente del error, entonces otros componentes cercanos serán "tragados" por el error.

Por ejemplo, imagina si pusiéramos un solo límite de errores envuelto alrededor del **componente raíz**. Se lanza un error en algún lugar de la aplicación, y el límite de errores reemplaza el componente raíz con una UI de respaldo. Si bien la aplicación no se ha bloqueado, el usuario solo ve la UI de respaldo y nada más. Esta no es una buena experiencia.

En cambio, queremos intentar poner nuestros límites de errores lo más cerca posible de la **fuente de posibles errores**. A menudo, esto significa colocar nuestros límites de errores alrededor de **nuevas características** que aún no se han probado exhaustivamente. A medida que nuestros usuarios interactúan con estas nuevas características, los errores serán capturados y, con un registro detallado, podemos concentrarnos en la ubicación de nuestros límites de errores con el tiempo.

### Limitaciones (Limitations)

Sin embargo, hay algunas limitaciones clave de los límites de errores que deben tenerse en cuenta. Según la documentación de React:

> Los límites de errores pueden capturar errores que ocurren durante el renderizado, en **métodos de ciclo de vida (lifecycle methods)** y en constructores de todo el árbol debajo de ellos. Los límites de errores **no** capturan errores para **manejadores de eventos (event handlers)** , **código asíncrono (asynchronous code)** , **renderizado del lado del servidor (server-side rendering)** , o errores lanzados en el propio límite de errores (en lugar de sus hijos).

![error-step-1](/Images/error-boundaries-ex8-1.svg)
![error-step-2](/Images/error-boundaries-ex8-2.svg)
![error-step-3](/Images/error-boundaries-ex8-3.svg)
![error-step-4](/Images/error-boundaries-ex8-4.svg)
![error-step-5](/Images/error-boundaries-ex8-5.svg)


----

## Review

## Resumen de Límites de Errores (Error Boundaries)

¡Bien hecho! Has creado con éxito tus propios componentes `ErrorBoundary` y has utilizado el popular paquete `react-error-boundary` para mantener tu aplicación en funcionamiento frente a los errores. Los límites de errores son defensores esenciales de nuestras aplicaciones React que permiten a nuestros usuarios tener la experiencia más fluida posible.

Recapitulemos lo que hemos aprendido hasta ahora:

*   Los **límites de errores** son componentes de React que envuelven secciones de una aplicación React, capturando errores en tiempo de ejecución en cualquier parte de su árbol de componentes hijos. Pueden renderizar una UI de respaldo y registrar errores en lugar de simplemente mostrar una pantalla en blanco. Los límites de errores te permiten recuperarte en caso de un error, lo que conduce a una mejor experiencia de usuario.
*   Los componentes de React se convierten en límites de errores una vez que implementan uno (o ambos) de los métodos `static getDerivedStateFromError()` y/o `componentDidCatch()`. Para usar estos métodos, el límite de errores debe ser un **componente de clase**.
*   El método **`static getDerivedStateFromError()`** recibe el valor del error lanzado y debe devolver el siguiente `this.state` del límite de errores. El nuevo valor de `this.state` del límite de errores puede usarse para determinar qué renderizar: la UI de respaldo o el árbol de componentes envuelto. El valor de `this.state` puede restablecerse para reiniciar el límite de errores y sus hijos.
*   El método **`componentDidCatch()`** se utiliza para registrar el error lanzado. Recibe el error lanzado y un objeto `errorInfo` como parámetros. `errorInfo` tiene una propiedad `.componentStack` que contiene el historial de componentes renderizados que condujeron al error.
*   Normalmente, los límites de errores se crean una vez y se usan múltiples veces en toda la aplicación. Es común usar implementaciones de límites de errores de terceros como **react-error-boundary**.
*   El paquete `react-error-boundary` exporta un componente `<ErrorBoundary>` que puede recibir las props `onError` (que puede asignarse a una función para registrar errores) y `FallbackComponent` (que puede asignarse a un componente de UI de respaldo).
*   Podemos proporcionar una **función en línea (inline function)** para la prop `FallbackComponent` a fin de pasar props adicionales a la UI de respaldo.
*   Los límites de errores deben mantenerse lo más cerca posible de la **fuente del error**. Los lugares comunes para usar límites de errores son alrededor de nuevas características que no han sido probadas exhaustivamente.
*   Los límites de errores **no** capturan errores en manejadores de eventos, código asíncrono, renderizado del lado del servidor, o errores lanzados en el propio límite de errores.

¡Felicitaciones de nuevo por terminar esta lección!

-----