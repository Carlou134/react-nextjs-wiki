# useEffect: ejecutar código cuando algo pasa

## En una frase

`useEffect` te deja ejecutar código **después** de que React dibujó la pantalla, para conectar tu componente con algo de afuera: una API, el título de la pestaña, un temporizador o el teclado.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un componente y cómo devuelve JSX: [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
* Cómo guardar datos que cambian con estado: [useState](01-The%20State%20Hook.md).

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Efecto secundario (side effect):** algo que tu componente hace y que no es "dibujar la pantalla". Por ejemplo: pedir datos a una API, cambiar el título de la pestaña, arrancar un temporizador o escuchar el teclado.
* **Efecto:** la función que le pasas a `useEffect`. Ahí adentro va el efecto secundario.
* **Montar:** que el componente aparezca en pantalla por primera vez. **Desmontar** es lo contrario: que el componente se saque de la pantalla.
* **Array de dependencias:** una lista de valores que le indica a React cuándo tiene que volver a ejecutar el efecto.
* **Función de limpieza (cleanup):** una función que devuelve el efecto para deshacer lo que hizo (por ejemplo, dejar de escuchar el teclado).

-----

## El problema

Quieres que el título de la pestaña del navegador diga `Hola, Ana` mientras el usuario escribe su nombre. Lo primero que se te puede ocurrir es cambiarlo directo en el cuerpo del componente:

```jsx
function PageTitle() {
  const [name, setName] = useState('');

  // Mal: se toca algo de afuera mientras React todavía está dibujando
  document.title = `Hola, ${name}`;

  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

El cuerpo del componente tiene un solo trabajo: **calcular qué se ve en pantalla**. Si además ahí adentro pides datos, arrancas temporizadores o tocas el DOM (la estructura de la página), mezclas dos cosas distintas. El componente se ejecuta muchas veces, y cada vez repetiría ese trabajo extra en un momento poco controlado.

Necesitas un lugar para decir: "cuando la pantalla ya esté actualizada, haz esto". Ese lugar es `useEffect`.

-----

## Cómo funciona

### Paso 1: importarlo y darle una función

```jsx
import { useState, useEffect } from 'react';

function PageTitle() {
  const [name, setName] = useState('');

  useEffect(() => {
    document.title = `Hola, ${name}`;
  });

  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

`useEffect` recibe una función (el efecto). React **no la ejecuta mientras dibuja**: espera a actualizar la pantalla y recién después la ejecuta.

Fíjate que el efecto puede usar `name` sin problema: tiene acceso a las variables de tu componente (estado, props, etc.).

### Paso 2: cuándo corre

Por defecto, el efecto corre así:

```
1. React ejecuta tu componente y dibuja la pantalla
2. React ejecuta el efecto
3. El usuario escribe una letra -> cambia el estado
4. React vuelve a dibujar
5. React vuelve a ejecutar el efecto
```

Es decir: corre después del **primer** renderizado y después de **cada** renderizado siguiente. Al escribir, `name` cambia, React redibuja, y el efecto actualiza el título.

### Paso 3: el array de dependencias

Muchas veces no quieres que el efecto corra en cada renderizado. Para eso existe el **segundo argumento**, el array de dependencias:

```jsx
useEffect(() => {
  document.title = `Hiciste clic ${count} veces`;
}, [count]);   // solo se vuelve a ejecutar si cambia count
```

Hay tres casos:

| Lo que pasas | Cuándo corre el efecto |
| --- | --- |
| Nada (sin array) | Después del primer renderizado y después de **cada** renderizado |
| `[]` (array vacío) | Solo después del **primer** renderizado (al montar) |
| `[x, y]` | Después del primer renderizado y **cada vez que cambie** `x` o `y` |

React compara el valor de cada dependencia con el del renderizado anterior. Si ninguna cambió, se saltea el efecto.

Un ejemplo típico es volver a pedir datos cuando cambia un valor:

```jsx
useEffect(() => {
  fetch(`/api/usuarios/${userId}`)
    .then((res) => res.json())
    .then((data) => setUser(data));
}, [userId]);   // cada vez que cambia userId, se pide el usuario nuevo
```

La regla: el array tiene que incluir **todas las variables del componente que el efecto usa** (aquí, `userId`).

### Paso 4: la función de limpieza

Algunos efectos crean algo que hay que deshacer. Si tu efecto **devuelve una función**, React la trata como la función de limpieza.

```jsx
useEffect(() => {
  function handleKey(event) {
    console.log('Tecla:', event.key);
  }

  document.addEventListener('keydown', handleKey);

  // Limpieza: deshace lo que hizo el efecto
  return () => {
    document.removeEventListener('keydown', handleKey);
  };
}, []);
```

React ejecuta la limpieza en dos momentos:

1. **Antes de volver a ejecutar el efecto** (si el efecto corre de nuevo, primero se limpia el anterior).
2. **Cuando el componente se desmonta.**

El orden completo, en las tres situaciones posibles:

```
El componente aparece (montar)
  1. React dibuja la pantalla
  2. Corre el efecto #1

Cambia una dependencia (actualizar)
  1. React dibuja la pantalla con los datos nuevos
  2. Corre la LIMPIEZA del efecto #1
  3. Corre el efecto #2

El componente desaparece (desmontar)
  1. Corre la LIMPIEZA del último efecto (#2)
```

La regla que se ve en el dibujo: cada efecto se limpia siempre **antes** de ser reemplazado por el siguiente, y una última vez cuando el componente se va.

Si no limpiaras, cada ejecución del efecto sumaría **otro** listener (un "escuchador" de eventos), y los viejos seguirían activos. Eso genera errores, hace la app más lenta y consume memoria de más (una *fuga de memoria*).

La limpieza es opcional: solo la necesitas cuando el efecto crea algo que puede duplicarse o quedar vivo (un listener, un `setInterval`, una suscripción). Si el efecto solo asigna `document.title`, no hace falta.

### Paso 5: efecto solo al montar

Con `[]`, el efecto corre una sola vez tras el primer renderizado, y su limpieza corre al desmontar:

```jsx
useEffect(() => {
  console.log('El componente apareció en pantalla');

  return () => {
    console.log('El componente se sacó de la pantalla');
  };
}, []);
```

Sin el `[]`, esos dos mensajes saldrían antes y después de **cada** renderizado.

> **Si en desarrollo ves los mensajes duplicados, no es un bug.** Muchos proyectos (los creados con Vite o con Next.js suelen traerlo activado) usan el modo estricto de React, `<StrictMode>`. En desarrollo, ese modo ejecuta el efecto, su limpieza y el efecto otra vez, a propósito, para ayudarte a descubrir limpiezas que faltan. En producción el efecto corre una sola vez.

-----

## Separar en varios efectos

Si tu componente hace dos cosas que no tienen relación, usa **un `useEffect` por cada una**. Cada efecto tiene su propio array de dependencias y su propia limpieza.

```jsx
// Efecto 1: trae el menú (una sola vez)
const [menuItems, setMenuItems] = useState(null);
useEffect(() => {
  fetch('/api/menu')
    .then((res) => res.json())
    .then((data) => setMenuItems(data));
}, []);

// Efecto 2: sigue la posición del mouse (con limpieza)
const [position, setPosition] = useState({ x: 0, y: 0 });
useEffect(() => {
  function handleMove(event) {
    setPosition({ x: event.clientX, y: event.clientY });
  }
  window.addEventListener('mousemove', handleMove);
  return () => window.removeEventListener('mousemove', handleMove);
}, []);
```

Juntarlos en un solo efecto obligaría a mezclar en el mismo lugar el pedido de datos y el listener, y a manejar todo en un único objeto de estado. Separados, cada efecto se entiende, se cambia y se prueba por su cuenta.

-----

## Ejemplo completo: buscar un usuario según un id

```jsx
import { useState, useEffect } from 'react';

export default function UserCard({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Función async definida ADENTRO del efecto
    async function loadUser() {
      setLoading(true);
      const res = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
      const data = await res.json();
      setUser(data);
      setLoading(false);
    }

    loadUser();
  }, [userId]);   // se vuelve a pedir cuando cambia el id

  if (loading) return <p>Cargando...</p>;
  return <p>{user.name}</p>;
}
```

Qué pasa, paso a paso:

1. Al aparecer, el componente dibuja "Cargando..." (porque `loading` empieza en `true`).
2. Después de dibujar, corre el efecto y llama a `loadUser`.
3. Cuando llega la respuesta, `setUser` y `setLoading(false)` cambian el estado y React redibuja con el nombre.
4. Si el padre le pasa otro `userId`, el efecto vuelve a correr y trae el usuario nuevo.

Fíjate que la función `async` está **adentro** del efecto y el efecto en sí no es `async`. Por qué, lo ves en "En TypeScript". Para ver este patrón con manejo de errores, mira [Fetch de datos con useEffect](../10-fetching-de-datos/01-Fetch%20de%20Datos%20con%20useEffect.md).

Este ejemplo es simple a propósito, y le faltan dos cosas que en una app real hacen falta:

* **Manejo de errores:** si el pedido falla, `user` queda en `null` y `user.name` rompería la pantalla. Lo vemos en [Manejo de errores y estados de carga](../10-fetching-de-datos/02-Manejo%20de%20Errores%20y%20Estados%20de%20Carga.md).
* **Condición de carrera:** si `userId` cambia antes de que llegue la respuesta anterior, una respuesta vieja podría llegar después y pisar a la nueva. Se resuelve cancelando o ignorando la respuesta vieja, y está en [Optimización de fetch con dependencias](../10-fetching-de-datos/03-Optimizaci%C3%B3n%20de%20Fetch%20con%20Dependencias.md).

-----

## Errores comunes

### 1. Bucle infinito

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1);   // cambia el estado en cada ejecución
});                      // sin array: corre en cada renderizado
```

**Por qué pasa:** el efecto cambia el estado, el estado cambia el renderizado, el renderizado vuelve a disparar el efecto (no hay array que lo frene), que vuelve a cambiar el estado... y así sin fin.
**Cómo se arregla:** pregúntate si de verdad necesitas ese efecto. Si sí, pon un array de dependencias que frene el ciclo (por ejemplo `[]`, o solo los valores que deberían dispararlo). Nunca pongas como dependencia un estado que el propio efecto cambia sin una condición que lo corte.

### 2. Olvidar la limpieza

```jsx
useEffect(() => {
  const id = setInterval(() => console.log('tick'), 1000);
  // Mal: nadie lo detiene
}, []);
```

**Por qué pasa:** el intervalo sigue corriendo aunque el componente ya no esté en pantalla, y si el efecto vuelve a correr, se crea otro más.
**Cómo se arregla:** devuelve una función que lo detenga.

```jsx
useEffect(() => {
  const id = setInterval(() => console.log('tick'), 1000);
  return () => clearInterval(id);   // Bien
}, []);
```

### 3. Poner `async` directo en el efecto

```jsx
useEffect(async () => {        // error
  const data = await fetchData();
  setData(data);
}, []);
```

**Por qué pasa:** una función `async` siempre devuelve una `Promise`. Pero lo único que React espera que devuelva el efecto es nada o una función de limpieza.
**Cómo se arregla:** define la función `async` adentro y llamala:

```jsx
useEffect(() => {
  async function load() {
    const data = await fetchData();
    setData(data);
  }
  load();
}, []);
```

### 4. Dependencias incompletas

```jsx
useEffect(() => {
  console.log(`Buscando: ${query}`);
}, []);   // Mal: usa query pero no está en la lista
```

**Por qué pasa:** con `[]` el efecto corre una sola vez y se queda con el valor de `query` de ese momento. Aunque `query` cambie después, el efecto sigue viendo el valor viejo (un valor "desactualizado").
**Cómo se arregla:** incluye todo lo que el efecto usa: `[query]`.

-----

## En TypeScript

Si escribes `useEffect(async () => { ... })`, TypeScript te marca error. El tipo del efecto solo acepta que la función devuelva `void` (nada) o una función de limpieza. Una función `async` devuelve una `Promise`, y eso no encaja.

La solución es la misma que vimos: una función `async` adentro, y el efecto sin `async`.

```tsx
useEffect(() => {
  async function loadData() {
    const data = await fetchData();
    setData(data);
  }

  loadData();
}, []);
```

-----

## Cuándo sí y cuándo no

**Usa `useEffect` para** sincronizar tu componente con algo que vive **fuera de React**: pedir datos a una API, cambiar el título de la pestaña o tocar el DOM directamente, temporizadores e intervalos, suscripciones, `localStorage`.

**No lo uses para:**

* **Calcular algo a partir de props o estado para mostrarlo.** Alcanza con una variable común dentro del componente (o con `useMemo` si el cálculo es costoso). Un efecto ahí sobra y agrega un renderizado extra.
* **Reaccionar a un clic o a un envío de formulario.** Eso va en el manejador del evento.

**Qué array usar:**

* Sin array: casi nunca es lo que quieres; el efecto corre en **cada** renderizado. Si no pasas el segundo argumento, revisa si en realidad necesitabas `[]` o dependencias concretas.
* `[]`: cuando debe correr una sola vez al montar (una suscripción, un pedido inicial que no depende de nada).
* `[x]`: cuando debe volver a correr solo si cambia `x` (el caso típico del nuevo pedido cuando cambia un id o un texto de búsqueda).

**Usa limpieza** cuando el efecto crea un listener, un intervalo o una suscripción.

-----

## Resumen en 5 líneas

1. `useEffect(efecto, dependencias)` ejecuta código **después** de que React actualiza la pantalla, para sincronizar con algo de afuera.
2. Sin array corre en cada renderizado; con `[]` solo tras el primer renderizado; con `[x]` cuando cambia `x`.
3. Si el efecto devuelve una función, es la **limpieza**: corre antes de cada nueva ejecución del efecto y al desmontar.
4. El array debe incluir todas las variables del componente que el efecto usa; el efecto no puede ser `async`, pero sí puede definir y llamar una función `async` adentro.
5. Si solo necesitas calcular algo para mostrarlo, no uses un efecto.

-----

## Para profundizar

<details>
<summary>useLayoutEffect: cuando necesitas medir antes de pintar</summary>

`useLayoutEffect` se escribe igual que `useEffect` (una función y, opcionalmente, un array de dependencias) pero corre en otro momento:

* `useEffect` corre **después** de que el navegador pintó la pantalla.
* `useLayoutEffect` corre justo después de que React actualiza el DOM, pero **antes de que el navegador pinte**. Así, el usuario nunca ve un estado intermedio.

Sirve para **medir algo del DOM** (el tamaño o la posición de un elemento) y ajustar algo enseguida, para evitar un parpadeo. Por ejemplo, posicionar un *tooltip* para que no se salga de la pantalla:

```jsx
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [height, setHeight] = useState(0);

  useLayoutEffect(() => {
    const rect = ref.current.getBoundingClientRect();
    setHeight(rect.height);
  }, []);

  return <div ref={ref}>Contenido del tooltip</div>;
}
```

Con `useEffect`, el navegador podría pintar el tooltip en una posición incorrecta por un instante y después "saltar" a la correcta.

Para todo lo demás, usa `useEffect`: `useLayoutEffect` frena el pintado del navegador hasta que termina, y abusar de él puede hacer que la app se sienta más lenta. Las reglas de dependencias y de limpieza son las mismas.

</details>

<details>
<summary>useEffectEvent: leer el valor más reciente sin que sea dependencia</summary>

A veces un efecto necesita el valor **más reciente** de una variable de estado, pero no quieres ponerla en el array de dependencias: hacerlo obligaría al efecto a volver a correr y a recrear listeners o temporizadores que deberían quedarse estables.

`useEffectEvent` resuelve eso. Crea un callback especial que siempre tiene acceso al estado o a las props más recientes, sin que el efecto tenga que volver a ejecutarse.

```jsx
import { useState, useEffect, useEffectEvent } from 'react';

function Demo() {
  const [count, setCount] = useState(0);

  // Siempre lee el count más reciente
  const logCount = useEffectEvent(() => {
    console.log('Último valor:', count);
  });

  useEffect(() => {
    function handleClick() {
      logCount();
    }
    window.addEventListener('click', handleClick);
    return () => window.removeEventListener('click', handleClick);
  }, []);   // el listener se instala una sola vez

  return <button onClick={() => setCount(count + 1)}>Sumar</button>;
}
```

El efecto corre una sola vez gracias al `[]`, pero `logCount` siempre ve el `count` actual. Es útil para efectos de larga duración (listeners, intervalos, suscripciones) que deben mantenerse estables mientras reaccionan a valores que cambian. Es un Hook nuevo de React; si tu proyecto usa una versión anterior, puede no estar disponible.

</details>

<details>
<summary>Las reglas de los Hooks (recordatorio)</summary>

`useEffect` es un Hook, así que sigue las mismas dos reglas que `useState`: se llama solo desde componentes de función (o Hooks propios) y siempre en el nivel de arriba del componente, nunca dentro de un `if`, un bucle o una función anidada. Si necesitas una condición, ponela **adentro** del efecto:

```jsx
// Bien: el Hook siempre se ejecuta; la condición va adentro
useEffect(() => {
  if (userName !== '') {
    localStorage.setItem('savedUserName', userName);
  }
}, [userName]);
```

La explicación completa está en la sección "Las reglas de los Hooks" de la lección [useState](01-The%20State%20Hook.md).

</details>

-----

## Siguiente lección

Cuando la misma lógica con `useState` y `useEffect` se repite en varios componentes, puedes guardarla en una función propia y reutilizarla: [Custom Hooks](03-Custom%20Hooks.md).
