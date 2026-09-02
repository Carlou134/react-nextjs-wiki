# The Effect Hook

## Why Use useEffect?

Antes de los **Hooks**, los componentes funcionales solo se usaban para aceptar datos en forma de **props** y devolver **JSX** para ser renderizado. Sin embargo, como aprendimos en la lección anterior, el **Hook de Estado (State Hook)** nos permite gestionar datos dinámicos, en forma de estado del componente, dentro de nuestros componentes funcionales.

En esta lección, usaremos el **Hook de Efecto (Effect Hook)** para ejecutar código JavaScript después de cada renderizado con el fin de:

* obtener datos desde un servicio de back-end,
* suscribirse a un flujo de datos,
* gestionar temporizadores e intervalos,
* leer y realizar cambios en el DOM.

Los componentes se vuelven a renderizar varias veces a lo largo de su ciclo de vida. Estos momentos clave representan la oportunidad perfecta para ejecutar estos “efectos secundarios”.

Existen tres momentos clave en los que se puede utilizar el Hook de Efecto:

1. Cuando el componente se agrega por primera vez, o se monta, en el DOM y se renderiza.
2. Cuando el estado o las props cambian, provocando que el componente se vuelva a renderizar.
3. Cuando el componente se elimina, o se desmonta, del DOM.

Más adelante en esta lección, aprenderemos cómo ajustar con mayor precisión exactamente cuándo se ejecuta el Hook de Efecto.

------

## Function Component Effects

El **Hook de Efecto** le indica a nuestro componente que haga algo cada vez que se renderiza (o se vuelve a renderizar). Combinado con el estado, podemos usar el Hook de Efecto para crear cambios dinámicos interesantes en nuestras páginas web.

Supongamos que queremos permitir que un usuario cambie el título de la pestaña del navegador cada vez que escribe. Podemos implementar esto con el Hook de Efecto (**useEffect()**) de la siguiente manera:

```js
import { useState, useEffect } from 'react';
 
function PageTitle() {
  const [name, setName] = useState('');
 
  useEffect(() => {
    document.title = `Hi, ${name}`;
  });
 
  return (
    <div>
      <p>Usa el campo de entrada de abajo para renombrar esta página</p>
      <input onChange={({target}) => setName(target.value)} value={name} type='text' />
    </div>
  );
}
```

Veamos el ejemplo anterior con más detalle. Primero, importamos el Hook de Efecto desde la librería **'react'**:

```js
import { useEffect } from 'react';
```

La función **useEffect()** no tiene valor de retorno, ya que el Hook de Efecto se utiliza para llamar a otra función. Pasamos la función de callback, o efecto, que se ejecutará después de que un componente se renderice, como argumento de la función **useEffect()**. En nuestro ejemplo, el siguiente efecto se ejecuta después de cada renderizado del componente **PageTitle**:

```js
() => { document.title = `Hi, ${name}`; }
```

Aquí, asignamos **Hi, ${name}** como el valor de **document.title**.

El evento **onChange** hace que el componente **PageTitle** se vuelva a renderizar cada vez que el usuario escribe en el campo de texto. En consecuencia, esto activa **useEffect()** y cambia el título del documento.

> **En TypeScript:** un error muy común (y que a primera vista parece que "debería" funcionar) es escribir directamente `useEffect(async () => { ... })`. TypeScript lo rechaza, porque el tipo del efecto solo acepta que la función devuelva `void` o una función de limpieza — nunca una `Promise`, que es justamente lo que devuelve toda función `async`. La solución es definir la función asíncrona **adentro** del efecto y llamarla enseguida, dejando que el propio `useEffect()` no sea `async`:
>
> ```tsx
> useEffect(() => {
>   async function loadData() {
>     const data = await fetchData();
>     setData(data);
>   }
>
>   loadData();
> }, []);
> ```

Observa cómo usamos el estado actual dentro de nuestro efecto. Aunque el efecto se ejecuta después de que el componente se renderiza, ¡seguimos teniendo acceso a las variables dentro del alcance de nuestro componente funcional! Cuando React renderiza nuestro componente, actualiza el DOM como de costumbre y luego ejecuta nuestro efecto después de que el DOM ha sido actualizado. Esto ocurre en cada renderizado, incluyendo el primero y el último.

-----

## Clean Up Effects

Algunos efectos requieren **limpieza (cleanup)**. Por ejemplo, podríamos querer agregar *event listeners* a algún elemento del DOM, más allá del **JSX** de nuestro componente. Cuando añadimos *event listeners* al DOM, es importante eliminarlos cuando ya no los necesitamos para evitar **fugas de memoria**.

Consideremos el siguiente efecto:

```js
useEffect(() => {
  document.addEventListener('keydown', handleKeyPress);
  // Especificamos cómo limpiar después del efecto:
  return () => {
    document.removeEventListener('keydown', handleKeyPress);
  };
});
```

Si nuestro efecto no devolviera una función de limpieza, se agregaría un nuevo *event listener* al objeto `document` del DOM cada vez que el componente se vuelva a renderizar. Esto no solo causaría errores, sino que también podría hacer que el rendimiento de la aplicación disminuya e incluso que se bloquee.

Debido a que los efectos se ejecutan después de **cada renderizado** y no solo una vez, React llama a nuestra función de limpieza **antes de cada nuevo renderizado** y **antes de desmontar el componente**, para limpiar cada llamada al efecto.

Si nuestro efecto devuelve una función, entonces el Hook **useEffect()** siempre la trata como la función de limpieza. React ejecutará esta función de limpieza antes de que el componente se vuelva a renderizar o se desmonte. Dado que esta función es opcional, es nuestra responsabilidad devolver una función de limpieza desde nuestro efecto cuando el código del efecto pueda crear fugas de memoria.

**Nota:** Si necesitamos un callback estable dentro de un efecto sin provocar que el efecto se vuelva a ejecutar, podemos usar el nuevo Hook **useEffectEvent**. Aprenderemos más sobre **useEffectEvent** en un ejercicio posterior.

-----

## Control When Effects Are Called

La función **useEffect()** ejecuta su primer argumento (el efecto) después de cada vez que un componente se renderiza. Ya hemos aprendido cómo devolver una función de limpieza para no crear problemas de rendimiento u otros errores, pero a veces queremos **evitar que nuestro efecto se ejecute en los re-renderizados** por completo.

Es común, al definir componentes funcionales, ejecutar un efecto **solo cuando el componente se monta** (se renderiza por primera vez), pero no cuando el componente se vuelve a renderizar. ¡El Hook de Efecto hace que esto sea muy fácil! Si queremos ejecutar nuestro efecto únicamente después del primer renderizado, pasamos un **arreglo vacío** como segundo argumento a **useEffect()**. Este segundo argumento se llama **arreglo de dependencias**.

El arreglo de dependencias se utiliza para indicarle al método **useEffect()** cuándo debe ejecutar nuestro efecto y cuándo debe omitirlo. Nuestro efecto siempre se ejecuta después del primer renderizado, pero solo se volverá a ejecutar si algo dentro del arreglo de dependencias ha cambiado de valor entre renderizados.

Seguiremos aprendiendo más sobre este segundo argumento en los próximos ejercicios, pero por ahora nos enfocaremos en usar un arreglo de dependencias vacío para ejecutar un efecto cuando un componente se monta por primera vez y, si nuestro efecto devuelve una función de limpieza, ejecutar esa función cuando el componente se desmonta.

```js
useEffect(() => {
  alert("el componente se renderizó por primera vez");
  return () => {
    alert("el componente está siendo eliminado del DOM");
  };
}, []);
```

Si no pasáramos un arreglo vacío como segundo argumento al **useEffect()** anterior, esas alertas se mostrarían antes y después de **cada renderizado** de nuestro componente, lo cual claramente no es el momento adecuado para mostrar esos mensajes. ¡Simplemente pasar `[]` a la función **useEffect()** es suficiente para configurar cuándo se ejecutan el efecto y la función de limpieza!

-----

## Fetch Data

Al desarrollar software, a menudo comenzamos con comportamientos predeterminados y luego los modificamos para mejorar el rendimiento.

Hemos aprendido que el comportamiento predeterminado del **Hook de Efecto** es ejecutar la función del efecto **después de cada renderizado**.

Luego, aprendimos que podemos pasar un **arreglo vacío** como segundo argumento a **useEffect()** si solo queremos que nuestro efecto se ejecute después del **primer renderizado** del componente.

En este ejercicio, aprenderemos a usar el **arreglo de dependencias** para configurar con mayor precisión exactamente **cuándo** queremos que se ejecute nuestro efecto.

Cuando nuestro efecto es responsable de **obtener datos desde un servidor**, prestamos especial atención a cuándo se ejecuta. Los viajes innecesarios de ida y vuelta entre nuestros componentes de React y el servidor pueden ser costosos en términos de:

* Procesamiento
* Rendimiento
* Uso de datos para usuarios móviles
* Costos de servicios de API

Cuando los datos que nuestros componentes necesitan para renderizar **no cambian**, podemos pasar un arreglo de dependencias vacío para que los datos se obtengan después del primer renderizado. Cuando se recibe la respuesta del servidor, podemos usar un *setter* del **Hook de Estado** para almacenar los datos de la respuesta del servidor en el estado local del componente para renderizados futuros. Usar el Hook de Estado y el Hook de Efecto juntos de esta manera es un patrón poderoso que evita que nuestros componentes obtengan datos nuevos innecesariamente después de cada renderizado.

Un arreglo de dependencias vacío le indica al Hook de Efecto que nuestro efecto **nunca necesita volver a ejecutarse**, es decir, que no depende de nada. Especificar cero dependencias significa que el resultado de ejecutar ese efecto no cambiará y que ejecutarlo una sola vez es suficiente.

Un arreglo de dependencias **no vacío** le indica al Hook de Efecto que puede omitir la ejecución del efecto después de los re-renderizados **a menos que** el valor de alguna de las variables en el arreglo de dependencias haya cambiado. Si el valor de una dependencia cambia, entonces el Hook de Efecto ejecutará el efecto nuevamente.

Aquí tienes un buen ejemplo tomado de la documentación oficial de React:

```js
useEffect(() => {
  document.title = `Hiciste clic ${count} veces`;
}, [count]); // Solo vuelve a ejecutar el efecto si cambia el valor almacenado en count
```

-----

## Rules of Hooks

Hay **dos reglas principales** que debemos tener en cuenta al usar Hooks:

1. **Solo llamar a Hooks en el nivel superior.**
2. **Solo llamar a Hooks desde funciones de React.**

Como hemos estado practicando con el **Hook de Estado** y el **Hook de Efecto**, hemos seguido estas reglas con facilidad, pero es útil tenerlas siempre presentes a medida que llevas tu nuevo entendimiento de los Hooks al mundo real y comienzas a usar más Hooks en tus aplicaciones de React.

Cuando React construye el **DOM Virtual**, la librería llama una y otra vez a las funciones que definen nuestros componentes a medida que el usuario interactúa con la interfaz. React realiza un seguimiento de los datos y funciones que gestionamos con Hooks basándose en **el orden en que aparecen dentro de la definición del componente funcional**. Por esta razón, siempre llamamos a nuestros Hooks en el nivel superior; **nunca** los llamamos dentro de bucles, condiciones o funciones anidadas.

En lugar de confundir a React con código como este:

```js
if (userName !== '') {
  useEffect(() => {
    localStorage.setItem('savedUserName', userName);
  });
}
```

Podemos lograr el mismo objetivo asegurándonos de llamar al Hook de forma consistente en cada renderizado:

```js
useEffect(() => {
  if (userName !== '') {
    localStorage.setItem('savedUserName', userName);
  }
});
```

En segundo lugar, los Hooks solo pueden usarse en **funciones de React**. Hemos estado trabajando con **useState()** y **useEffect()** dentro de componentes funcionales, y este es el uso más común. El único otro lugar donde se pueden usar Hooks es dentro de **Hooks personalizados (custom hooks)**. Los Hooks personalizados son increíblemente útiles para organizar y reutilizar lógica con estado entre componentes funcionales.

----

## Separate Hooks for Separate Effects

Cuando varios valores están **estrechamente relacionados** y cambian al mismo tiempo, puede tener sentido agruparlos en una colección como un **objeto** o un **arreglo**. Sin embargo, empaquetar datos juntos también puede añadir **complejidad** al código encargado de gestionarlos. Por lo tanto, es una buena idea **separar responsabilidades** gestionando diferentes datos con distintos **Hooks**.

Compara la complejidad de este ejemplo, donde los datos están agrupados en un solo objeto:

```js
// Maneja tanto position como menuItems con un solo hook useEffect.
const [data, setData] = useState({ position: { x: 0, y: 0 } });
useEffect(() => {
  get('/menu').then((response) => {
    setData((prev) => ({ ...prev, menuItems: response.data }));
  });
  const handleMove = (event) =>
    setData((prev) => ({
      ...prev,
      position: { x: event.clientX, y: event.clientY }
    }));
  window.addEventListener('mousemove', handleMove);
  return () => window.removeEventListener('mousemove', handleMove);
}, []);
```

Con la simplicidad de este otro enfoque, donde hemos separado las responsabilidades:

```js
// Maneja menuItems con un hook useEffect.
const [menuItems, setMenuItems] = useState(null);
useEffect(() => {
  get('/menu').then((response) => setMenuItems(response.data));
}, []);

// Maneja position con un hook useEffect separado.
const [position, setPosition] = useState({ x: 0, y: 0 });
useEffect(() => {
  const handleMove = (event) =>
    setPosition({ x: event.clientX, y: event.clientY });
  window.addEventListener('mousemove', handleMove);
  return () => window.removeEventListener('mousemove', handleMove);
}, []);
```

No siempre es obvio si conviene **agrupar los datos** o **separarlos**, pero con la práctica nos volvemos mejores organizando nuestro código para que sea más fácil de **entender**, **extender**, **reutilizar** y **probar**.

-----

## Using useEffectEvent

A veces, un efecto necesita el **valor más reciente de una variable de estado**, pero no queremos incluir esa variable en el **arreglo de dependencias**. Agregarla obligaría al efecto a volver a ejecutarse y a recrear *listeners* o temporizadores que deberían mantenerse estables.

React proporciona **useEffectEvent** para resolver este problema. Este Hook nos permite crear un **callback especial** que siempre tiene acceso al estado o a las **props** más recientes, sin provocar que el propio efecto se vuelva a ejecutar.

Primero, lo importamos:

```js
import { useState, useEffect, useEffectEvent } from "react";
```

Creamos algo de estado:

```js
const [count, setCount] = useState(0);
```

Ahora envolvemos la lógica que siempre debe usar el valor más reciente de `count`:

```js
const logCount = useEffectEvent(() => {
  console.log("Latest:", count);
});
```

`logCount` no es una función normal. Es un **callback estable** que React actualiza internamente para que, cuando se ejecute, siempre reciba el valor más reciente de `count`. Esto evita valores obsoletos (*stale values*) mientras mantiene el efecto estable.

Luego, instalamos un *listener* que solo debe configurarse una vez:

```js
useEffect(() => {
  function handleClick() {
    logCount();
  }
  window.addEventListener("click", handleClick);
  return () => window.removeEventListener("click", handleClick);
}, []);
```

El efecto se ejecuta una sola vez gracias al `[]`, pero `logCount` siempre tiene el estado actualizado. Este patrón es útil para efectos de larga duración, como *event listeners*, intervalos y suscripciones, que deben permanecer consistentes mientras siguen reaccionando a valores que cambian.

----

## Review

En esta lección, aprendimos cómo escribir **efectos** que gestionan temporizadores, manipulan el DOM y obtienen datos desde un servidor. Con el **Hook de Efecto**, podemos realizar este tipo de acciones en componentes funcionales con mucha facilidad.

Repasemos los conceptos principales de esta lección:

* Podemos importar la función **useEffect()** desde la librería **'react'** y llamarla dentro de nuestros componentes funcionales.
* **Efecto** se refiere a la función que pasamos como primer argumento de **useEffect()**. Por defecto, el Hook de Efecto ejecuta este efecto **después de cada renderizado**.
* La **función de limpieza (cleanup)** es devuelta de forma opcional por el efecto. Si el efecto hace algo que necesita limpiarse para evitar fugas de memoria, entonces el efecto devuelve una función de limpieza, y el Hook de Efecto llamará a esta función **antes de volver a ejecutar el efecto** y **cuando el componente se desmonte**.
* El **arreglo de dependencias** es el segundo argumento opcional que se puede pasar a **useEffect()** para evitar ejecutar el efecto repetidamente cuando no es necesario. Este arreglo debe incluir **todas las variables de las que depende el efecto**.
* El Hook de Efecto se centra en **programar cuándo se ejecuta el código del efecto**. Podemos usar el arreglo de dependencias para configurar cuándo se ejecuta nuestro efecto de las siguientes maneras:

| Arreglo de dependencias | El efecto se ejecuta después del primer renderizado y… |
| ----------------------- | ------------------------------------------------------ |
| `undefined`             | en cada re-renderizado                                 |
| Arreglo vacío           | no se vuelve a ejecutar                                |
| Arreglo no vacío        | cuando cambia cualquier valor del arreglo              |

Los Hooks nos brindan la flexibilidad para organizar nuestro código de distintas maneras, **agrupando datos relacionados** y **separando responsabilidades**, para mantener el código **simple**, **libre de errores**, **reutilizable** y **fácil de probar**.

----

## useLayoutEffect

React expone otro Hook de la misma familia que **useEffect()**, con una firma idéntica —recibe una función de efecto y, opcionalmente, un arreglo de dependencias— pero con una diferencia crucial en **cuándo** se ejecuta: **useLayoutEffect()**.

```js
import { useLayoutEffect } from 'react';
```

Como aprendimos antes, **useEffect()** ejecuta su función de efecto de forma **asíncrona**, después de que React actualiza el DOM y **después de que el navegador ha pintado la pantalla**. Esto es deliberado: React difiere el efecto para no bloquear el pintado, manteniendo la interfaz fluida.

**useLayoutEffect()**, en cambio, ejecuta su función de efecto de forma **síncrona**, inmediatamente después de que React realiza las mutaciones del DOM, pero **antes de que el navegador pinte** esos cambios en pantalla. En la práctica, esto significa que el código dentro de un **useLayoutEffect()** puede leer y modificar el DOM, y el usuario nunca llegará a ver un estado intermedio: solo verá el resultado final, ya corregido.

### Cuándo usarlo

El caso de uso típico de **useLayoutEffect()** es cuando necesitamos **medir algo del DOM** —por ejemplo, el tamaño o la posición de un elemento con `getBoundingClientRect()`— y, a partir de esa medición, **ajustar sincrónicamente** algún valor antes de que el usuario vea el resultado. Un ejemplo clásico es posicionar un *tooltip* para que no se salga de la pantalla: si esa lógica se hiciera con **useEffect()**, el navegador podría pintar el tooltip en una posición incorrecta durante una fracción de segundo y luego "saltar" a la posición correcta, generando un parpadeo visual (*flicker*) perceptible por el usuario.

```js
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [height, setHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setHeight(height);
  }, []);

  return <div ref={ref}>Contenido del tooltip</div>;
}
```

Fuera de este tipo de escenarios —mediciones de layout, animaciones que dependen de una medición previa, o evitar parpadeos visuales al sincronizar el DOM con datos externos— **useEffect()** sigue siendo la opción correcta por defecto. Como **useLayoutEffect()** bloquea el pintado del navegador hasta que termina de ejecutarse, abusar de él puede degradar el rendimiento percibido de la aplicación. Las reglas de dependencias y la función de limpieza funcionan exactamente igual que en **useEffect()**; lo único que cambia es el momento en el que React decide ejecutar el efecto.

----