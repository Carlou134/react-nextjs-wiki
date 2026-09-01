# Component Lifecycle Methods

## The Component Lifecycle

Hemos visto que los componentes de React pueden ser muy dinámicos. Se crean, se renderizan, se agregan al DOM, se actualizan y se eliminan. Todos estos pasos son parte del ciclo de vida de un componente.

El ciclo de vida de un componente tiene tres partes principales:

- Montaje, cuando el componente se inicializa y se coloca en el DOM por primera vez.
- Actualización, cuando el componente se actualiza como resultado de un cambio en el estado o en las props.
- Desmontaje, cuando el componente se elimina del DOM.

Cada componente de React con el que has interactuado realiza al menos el primer paso. Si un componente nunca se monta, nunca lo verías.

La mayoría de los componentes interesantes se actualizan en algún momento. Un componente puramente estático—como, por ejemplo, un logo—puede que nunca se actualice. Pero si el estado de un componente cambia, se actualiza. O si se le pasan diferentes props a un componente, también se actualiza.

Finalmente, un componente se desmonta cuando se elimina del DOM. Por ejemplo, si tienes un botón que oculta un componente, es probable que ese componente se desmonte. Si tu aplicación tiene varias pantallas, es probable que cada pantalla (y todos sus componentes hijos) se desmonten. Si un componente está "vivo" durante toda la vida de tu aplicación (por ejemplo, un componente `<App />` de nivel superior o una barra de navegación persistente), no se desmontará. Pero la mayoría de los componentes pueden desmontarse de una forma u otra.

Vale la pena mencionar que cada instancia de componente tiene su propio ciclo de vida. Por ejemplo, si tienes 3 botones en una página, entonces hay 3 instancias de componentes, cada una con su propio ciclo de vida. Sin embargo, una vez que una instancia de componente se desmonta, eso es todo: nunca se volverá a montar, actualizar o desmontar.

![react-lifecycle](/Images/react_diagram-lifecycle-flow.png)

-----

## Introduction to Lifecycle Methods

Los componentes de React tienen varios métodos, llamados métodos de ciclo de vida, que se llaman en diferentes partes del ciclo de vida de un componente. Así es como el programador maneja el ciclo de vida de un componente.

Quizás no lo sabías, pero ya has usado dos de los métodos de ciclo de vida más comunes: `constructor()` y `render()`! `constructor()` es el primer método que se llama durante la fase de montaje. `render()` se llama después, durante la fase de montaje, para renderizar el componente por primera vez, y también durante la fase de actualización, para volver a renderizar el componente.

Observa que los métodos de ciclo de vida no siempre corresponden uno a uno con una parte del ciclo de vida. `constructor()` solo se ejecuta durante la fase de montaje, pero `render()` se ejecuta tanto en la fase de montaje como en la de actualización.

Con este nuevo conocimiento, vamos a construir un componente de reloj simple.

------

## componentDidMount

Hemos creado un componente de reloj, pero es estático. ¿No sería mejor si se actualizara?

En términos generales, queremos actualizar `this.state.date` con una nueva fecha una vez por segundo.

JavaScript tiene una función útil, `setInterval()`, que nos ayudará a hacer esto. Nos permite ejecutar una función en un intervalo definido. En nuestro caso, haremos una función que actualice `this.state.date` y la llamaremos cada segundo.

Queremos ejecutar un código como este:

```js
// NOTA: Este código no se limpia correctamente.
// Veremos esto en el siguiente ejercicio.
const oneSecond = 1000;
setInterval(() => {
  this.setState({ date: new Date() });
}, oneSecond);
```

Ya tenemos el código que queremos ejecutar, eso es genial. Pero, ¿dónde debemos poner este código? Es decir, ¿en qué parte del ciclo de vida del componente debe ir?

Recuerda, el ciclo de vida del componente tiene tres partes principales:

- Montaje, cuando el componente se inicializa y se coloca en el DOM por primera vez.
- Actualización, cuando el componente se actualiza como resultado de un cambio en el estado o en las props.
- Desmontaje, cuando el componente se elimina del DOM.

Definitivamente no debe ir en la fase de desmontaje—¡no queremos iniciar el intervalo cuando el reloj desaparece de la pantalla! Tampoco es útil durante la fase de actualización—queremos que el intervalo comience tan pronto como aparezca el reloj, y no queremos esperar a una actualización. Probablemente tenga sentido poner este código en la fase de montaje.

Hemos visto dos funciones: `render()` y el `constructor`. ¿Podemos poner este código en alguno de esos lugares?

`render()` no es una buena opción. Por un lado, se ejecuta durante la fase de montaje y la de actualización—demasiado a menudo para nosotros. Además, generalmente es una mala idea establecer efectos secundarios como este en `render()`, ya que puede crear errores sutiles en el futuro.

`constructor()` tampoco es ideal. Solo se ejecuta durante la fase de montaje, lo cual es bueno, pero generalmente debes evitar efectos secundarios como este en los constructores porque viola algo llamado el Principio de Responsabilidad Única. En resumen, no es responsabilidad del constructor iniciar efectos secundarios. (Puedes leer más sobre el principio en Wikipedia.)

Si no es en `render()` ni en el constructor, ¿entonces dónde? Aquí entra un nuevo método de ciclo de vida, `componentDidMount()`.

`componentDidMount()` es el último método que se llama durante la fase de montaje. El orden es:

1. El constructor
2. `render()`
3. `componentDidMount()`

En otras palabras, se llama después de que el componente se renderiza. Aquí es donde queremos iniciar nuestro temporizador.

(A otro método, `getDerivedStateFromProps()`, se le llama entre el constructor y `render()`, pero se usa muy rara vez y normalmente no es la mejor manera de lograr tus objetivos. No hablaremos de él en esta lección.)

------

## componentWillUnmount

Nuestro reloj funciona, pero tiene un problema importante. Nunca le dijimos al intervalo que se detuviera, así que seguirá ejecutando esa función para siempre (o al menos, hasta que el usuario abandone o recargue la página).

Cuando el componente se desmonta (es decir, se elimina de la página), ese temporizador seguirá funcionando, intentando actualizar el estado de un componente que ya no existe. Esto significa que tus usuarios tendrán código JavaScript ejecutándose innecesariamente, lo que afectará el rendimiento de tu aplicación.

React mostrará una advertencia como esta:

    Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method.

Imagina si el reloj se monta y desmonta cientos de veces; eventualmente, esto hará que tu página se vuelva lenta por todo el trabajo innecesario. También verás advertencias en la consola de tu navegador. Aún peor, esto puede causar errores sutiles y molestos.

Todo esto puede pasar si no limpiamos los efectos secundarios de un componente. En este caso, es una llamada a setInterval(), pero los componentes pueden tener muchos otros efectos secundarios: cargar datos externos con AJAX, modificar el DOM manualmente, establecer un valor global, y más. Tratamos de limitar los efectos secundarios, pero es difícil construir una aplicación interesante sin ninguno.

En general, cuando un componente produce un efecto secundario, debes recordar limpiarlo.

JavaScript nos da la función clearInterval(). setInterval() puede devolver un ID, que luego puedes pasar a clearInterval() para detenerlo. Este es el código que queremos usar:

```js
const oneSecond = 1000;
this.intervalID = setInterval(() => {
  this.setState({ date: new Date() });
}, oneSecond);

// Más tarde...
clearInterval(this.intervalID);
```

En resumen, queremos seguir configurando nuestro setInterval() en componentDidMount(), pero luego queremos limpiar ese intervalo cuando el reloj se desmonte.

Vamos a introducir un nuevo método del ciclo de vida: componentWillUnmount(). componentWillUnmount() se llama en la fase de desmontaje, justo antes de que el componente sea destruido por completo. Es un buen momento para limpiar cualquier cosa que haya dejado el componente.

En este caso, lo usaremos para limpiar el intervalo del reloj.

-----

## componentDidUpdate

Recuerda las tres partes del ciclo de vida de un componente:

- Montaje (Mounting): cuando el componente se inicializa y se coloca en el DOM por primera vez.
- Actualización (Updating): cuando el componente se actualiza como resultado de un cambio en el estado o en las props.
- Desmontaje (Unmounting): cuando el componente se elimina del DOM.

Ya hemos visto el montaje (`constructor()`, `render()`, y `componentDidMount()`). También hemos visto el desmontaje (`componentWillUnmount()`). Ahora vamos a ver la fase de actualización.

Una actualización ocurre cuando cambian las props o el estado. Esto ya lo has visto varias veces. Cada vez que llamas a `setState()` con nuevos datos, provocas una actualización. Cada vez que cambias las props que recibe un componente, también lo actualizas.

Cuando un componente se actualiza, llama a varios métodos, pero solo dos se usan comúnmente.

El primero es `render()`, que hemos visto en todos los componentes de React. Cuando cambian las props o el estado de un componente, se llama a `render()`.

El segundo, que aún no hemos visto, es `componentDidUpdate()`. Así como `componentDidMount()` es un buen lugar para la configuración inicial, `componentDidUpdate()` es un buen lugar para trabajar después de una actualización.

-----

## Review

Hemos llegado al final de la lección. Aprendimos sobre las tres fases principales del ciclo de vida de un componente:

- Montaje, cuando el componente se inicializa y se coloca en el DOM por primera vez. Vimos que el constructor, render() y componentDidMount() se llaman en esta fase.  
- Actualización, cuando el componente se actualiza por un cambio en el estado o en las props. Vimos que render() y componentDidUpdate() se llaman en esta fase.  
- Desmontaje, cuando el componente se elimina del DOM. Vimos que componentWillUnmount() se llama aquí, lo cual es buen momento para limpiar recursos.

También aprendimos a configurar efectos secundarios y a eliminarlos. Ahora sabemos cómo crear componentes más robustos y complejos.

A la derecha tienes una referencia que puedes usar. Muestra las tres fases del ciclo de vida de un componente y qué métodos se llaman en cada fase. También puedes consultar este diagrama interactivo.

[Imagen](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

Para más información, puedes leer la documentación oficial de React. Consulta “State and Lifecycle” y la documentación de React.Component.

-----

