# useEffect: ejecutar código cuando algo pasa

## En una frase

`useEffect` es el Hook que **sincroniza** un componente con un sistema externo a React (una API, el DOM, un temporizador, el teclado). Su código se ejecuta **después** de que React actualiza la pantalla (después del render y el commit), no durante el cálculo del JSX.

-----

## Requisitos previos

Conviene dominar:

* Qué es un componente y cómo devuelve JSX: [Tu primer componente](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
* Cómo guardar datos que cambian con estado: [useState](01-The%20State%20Hook.md).

Términos (también en el [glosario](Glosario.md)):

* **Efecto secundario (side effect):** cualquier operación del componente que no consiste en calcular el JSX. Por ejemplo: pedir datos, cambiar `document.title`, arrancar un temporizador o escuchar el teclado.
* **Efecto:** la función que se pasa a `useEffect`. Contiene el efecto secundario.
* **Montar:** que el componente aparezca por primera vez en pantalla. **Desmontar:** que se elimine de ella.
* **Array de dependencias:** lista de valores que indica a React cuándo volver a ejecutar el efecto.
* **Función de limpieza (cleanup):** función que devuelve el efecto para revertir lo que hizo (por ejemplo, quitar un listener).

-----

## El problema

Un ejemplo típico: mantener `document.title` sincronizado con el estado. Hacerlo directamente en el cuerpo del componente es incorrecto:

```jsx
function PageTitle() {
  const [name, setName] = useState('');

  // Mal: se toca algo de afuera mientras React todavía está dibujando
  document.title = `Hola, ${name}`;

  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

El cuerpo del componente debe ser una función **pura**: dado el mismo estado y las mismas props, devuelve el mismo JSX y no produce efectos observables. Pedir datos, arrancar temporizadores o modificar el DOM ahí rompe esa regla. React puede ejecutar el render varias veces (por ejemplo, en StrictMode) o descartarlo, y cada vez repetiría el efecto en un momento que no controlas.

`useEffect` declara ese trabajo aparte: "cuando la pantalla ya esté actualizada, ejecuta esto".

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

`useEffect` recibe una función (el efecto). React **no la ejecuta durante el render**: la ejecuta después de haber actualizado el DOM (commit). En general eso ocurre después de que el navegador pinta; si el render lo provocó un evento discreto como un clic, React puede ejecutarla antes del pintado.

El efecto es un closure: accede a los valores de `name`, props y demás variables **del render en el que se creó**.

### Paso 2: cuándo corre

Por defecto, el efecto corre así:

```
1. React ejecuta tu componente y dibuja la pantalla
2. React ejecuta el efecto
3. El usuario escribe una letra -> cambia el estado
4. React vuelve a dibujar
5. React vuelve a ejecutar el efecto
```

Es decir: corre después del **primer** render y después de **cada** render siguiente. Al escribir, `name` cambia, React vuelve a renderizar y el efecto actualiza el título.

### Paso 3: el array de dependencias

Normalmente el efecto no debe correr en cada render. El **segundo argumento**, el array de dependencias, controla cuándo se vuelve a ejecutar:

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

React compara cada dependencia con su valor del render anterior usando `Object.is` (comparación por referencia en objetos, arrays y funciones). Si ninguna cambió, omite el efecto. Por eso un objeto o función creado en cada render cuenta como "nuevo" y dispara el efecto siempre.

Ejemplo: volver a pedir datos cuando cambia un valor:

```jsx
useEffect(() => {
  fetch(`/api/usuarios/${userId}`)
    .then((res) => res.json())
    .then((data) => setUser(data));
}, [userId]);   // cada vez que cambia userId, se pide el usuario nuevo
```

La regla: el array debe incluir **todos los valores reactivos que el efecto lee** (props, estado y variables o funciones declaradas en el componente), aquí `userId`. Las dependencias no se "eligen": las determina el código del efecto.

### Paso 4: la función de limpieza

Si el efecto **devuelve una función**, React la usa como función de limpieza para revertir lo que el efecto creó.

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

1. **Antes de volver a ejecutar el efecto** (con los valores del render anterior).
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

Regla: cada efecto se limpia **antes** de ser reemplazado por el siguiente, y una última vez al desmontar.

Sin limpieza, cada ejecución agregaría **otro** listener y los anteriores seguirían activos: comportamiento duplicado y *fuga de memoria*.

La limpieza es opcional: se necesita cuando el efecto crea algo que puede duplicarse o quedar vivo (un listener, un `setInterval`, una suscripción). Si el efecto solo asigna `document.title`, no hace falta.

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

Sin el `[]`, la limpieza y el efecto correrían tras **cada** render.

> **Si en desarrollo ves los mensajes duplicados, no es un bug.** Con `<StrictMode>` (activado por defecto en las plantillas de Vite y en Next.js con App Router), React 18 y 19 ejecutan en desarrollo un ciclo extra al montar: efecto, limpieza y efecto otra vez. Sirve para detectar limpiezas ausentes o incorrectas. En producción no ocurre.

-----

## Separar en varios efectos

Cada efecto debe representar **un solo proceso de sincronización**. Si el componente sincroniza dos cosas sin relación, usa un `useEffect` por cada una, cada uno con sus dependencias y su limpieza.

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

Juntarlos mezclaría ciclos de vida distintos: el listener necesita limpieza y el pedido no. Separados, cada efecto se entiende y se modifica de forma independiente.

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

Flujo:

1. El primer render muestra "Cargando..." (`loading` empieza en `true`).
2. Tras el commit, corre el efecto y llama a `loadUser`.
3. Al llegar la respuesta, `setUser` y `setLoading(false)` provocan un nuevo render con el nombre.
4. Si el padre pasa otro `userId`, el efecto vuelve a correr y pide el usuario nuevo.

La función `async` está **dentro** del efecto y el efecto no es `async` (motivo en "En TypeScript"). Este patrón con manejo de errores está en [Fetch de datos con useEffect](../10-fetching-de-datos/01-Fetch%20de%20Datos%20con%20useEffect.md).

Este ejemplo es deliberadamente mínimo. Le faltan dos cosas necesarias en una app real:

* **Manejo de errores:** si el pedido falla, `user` sigue en `null` y `user.name` lanza un error. Ver [Manejo de errores y estados de carga](../10-fetching-de-datos/02-Manejo%20de%20Errores%20y%20Estados%20de%20Carga.md).
* **Condición de carrera (race condition):** si `userId` cambia antes de que llegue la respuesta anterior, la respuesta vieja puede llegar después y sobrescribir a la nueva. Se resuelve ignorando o cancelando la respuesta obsoleta con la función de limpieza (una bandera `ignore` o un `AbortController`). Ver [Optimización de fetch con dependencias](../10-fetching-de-datos/03-Optimizaci%C3%B3n%20de%20Fetch%20con%20Dependencias.md).

-----

## Errores comunes

### 1. Bucle infinito

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1);   // cambia el estado en cada ejecución
});                      // sin array: corre en cada renderizado
```

**Por qué pasa:** el efecto cambia el estado, el estado provoca un render y, sin array, el render vuelve a ejecutar el efecto: ciclo infinito.
**Cómo se arregla:** primero evalúa si el efecto es necesario (a menudo el valor puede derivarse en el render). Si lo es, ajusta las dependencias para que solo los valores relevantes lo disparen. Un estado que el efecto modifica y también lista como dependencia genera el mismo ciclo, salvo que exista una condición que lo corte.

### 2. Olvidar la limpieza

```jsx
useEffect(() => {
  const id = setInterval(() => console.log('tick'), 1000);
  // Mal: nadie lo detiene
}, []);
```

**Por qué pasa:** el intervalo sigue activo tras desmontar el componente, y cada nueva ejecución del efecto crea otro.
**Cómo se arregla:** devuelve una función de limpieza que lo detenga.

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

**Por qué pasa:** una función `async` siempre devuelve una `Promise`, y React solo admite que el efecto devuelva `undefined` o una función de limpieza.
**Cómo se arregla:** define la función `async` dentro del efecto y llámala:

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

**Por qué pasa:** con `[]` el efecto corre solo al montar y su closure conserva el `query` de ese render. Aunque `query` cambie, el efecto no se ejecuta de nuevo (valor "stale", desactualizado).
**Cómo se arregla:** incluye todo lo que el efecto lee: `[query]`. No silencies la regla `react-hooks/exhaustive-deps`; corrige el código.

-----

## En TypeScript

`useEffect(async () => { ... })` produce un error de tipos: el efecto debe devolver `void` o una función de limpieza, y una función `async` devuelve una `Promise`.

La solución es la misma: función `async` interna y efecto sin `async`.

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

**Usa `useEffect` para** sincronizar el componente con algo **externo a React**: peticiones de red, `document.title` o manipulación directa del DOM, temporizadores, suscripciones, `localStorage`, librerías no React.

**No lo uses para:**

* **Derivar valores de props o estado.** Calcúlalos en el render (con `useMemo` solo si el cálculo es costoso y lo has medido). Un efecto que copia estado a otro estado agrega un render extra innecesario.
* **Responder a acciones del usuario** (clic, envío de formulario). Eso va en el manejador del evento.

**Qué array usar:**

* Sin array: corre tras **cada** render. Es poco frecuente que sea lo correcto.
* `[]`: el efecto no lee ningún valor reactivo (una suscripción o una configuración inicial).
* `[x]`: el efecto debe repetirse cuando cambia `x` (nuevo pedido al cambiar un id o un texto de búsqueda).

**Usa limpieza** cuando el efecto crea un listener, un intervalo o una suscripción.

-----

## Resumen en 5 líneas

1. `useEffect(efecto, dependencias)` ejecuta código **después** del commit, para sincronizar con un sistema externo.
2. Sin array corre en cada render; con `[]` solo al montar; con `[x]` cuando cambia `x` (comparación con `Object.is`).
3. Si el efecto devuelve una función, es la **limpieza**: corre antes de cada re-ejecución y al desmontar.
4. El array debe incluir todos los valores reactivos que el efecto lee; el efecto no puede ser `async`, pero puede llamar a una función `async` interna.
5. Para derivar valores o responder a eventos, no uses un efecto.

-----

## Para profundizar

<details>
<summary>useLayoutEffect: cuando necesitas medir antes de pintar</summary>

`useLayoutEffect` tiene la misma firma que `useEffect`, pero corre en otro momento:

* `useEffect` corre, en general, **después** de que el navegador pintó.
* `useLayoutEffect` corre justo después de que React actualiza el DOM y **antes del pintado**, de forma síncrona. Si actualiza estado, React vuelve a renderizar antes de pintar y el usuario no ve el estado intermedio.

Se usa para **medir el DOM** (tamaño o posición de un elemento) y ajustar el resultado sin parpadeo. Por ejemplo, posicionar un *tooltip*:

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

Para todo lo demás, usa `useEffect`: `useLayoutEffect` bloquea el pintado hasta terminar y abusar de él degrada el rendimiento. Las reglas de dependencias y limpieza son las mismas. En SSR no se ejecuta en el servidor y React puede advertirlo.

</details>

<details>
<summary>useEffectEvent: leer el valor más reciente sin que sea dependencia</summary>

A veces un efecto necesita leer el valor **más reciente** de estado o props sin que ese valor sea una dependencia: incluirlo obligaría a re-ejecutar el efecto y a recrear listeners o temporizadores que deberían ser estables.

`useEffectEvent` separa esa lógica "no reactiva": devuelve una función que siempre ve las props y el estado más recientes, y que no se declara en las dependencias. Solo debe llamarse desde dentro de efectos.

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

El efecto corre una sola vez por el `[]`, pero `logCount` siempre ve el `count` actual. Es útil en efectos de larga duración (listeners, intervalos, suscripciones). Es estable desde React 19.2; en versiones anteriores no está disponible.

</details>

<details>
<summary>Las reglas de los Hooks (recordatorio)</summary>

`useEffect` sigue las mismas dos reglas que `useState`: se llama solo desde componentes de función (o Hooks propios) y siempre en el nivel superior, nunca dentro de un `if`, un bucle o una función anidada. Si necesitas una condición, ponla **dentro** del efecto:

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

## En entrevista

### Respuesta corta (junior)

`useEffect` es un Hook que ejecuta código después de que React renderiza y actualiza el DOM. Sirve para sincronizar el componente con algo externo a React: pedir datos, suscribirse a eventos, manejar temporizadores o modificar el DOM. Recibe el efecto y un array de dependencias que define cuándo se vuelve a ejecutar. Si el efecto devuelve una función, esa función limpia lo que creó.

### Respuesta ampliada (semi-senior)

* **Cuándo corre:** después del render y el commit (normalmente tras el pintado), nunca durante el render. El render sigue siendo puro.
* **Dependencias:** React compara cada una con `Object.is` contra el render anterior. Objetos, arrays y funciones creados en el render son "nuevos" cada vez; se estabilizan moviéndolos dentro del efecto, o con `useMemo` / `useCallback` cuando hace falta.
* **Closures:** el efecto captura los valores de su render. Omitir dependencias produce valores obsoletos (stale).
* **Limpieza:** corre antes de cada re-ejecución (con los valores del render anterior) y al desmontar. Debe revertir la configuración del efecto.
* **StrictMode (desarrollo):** monta, limpia y vuelve a montar para comprobar que el efecto es simétrico. Si rompe algo, falta la limpieza.
* **Race conditions:** una petición vieja puede resolver después de una nueva. Se ignora o cancela en la limpieza (`ignore` o `AbortController`).
* **No siempre se necesita un efecto:** los valores derivados se calculan en el render; la lógica ligada a una acción del usuario va en el manejador de eventos; reiniciar estado al cambiar una prop se resuelve con `key`. Para datos remotos en apps reales suelen preferirse librerías como TanStack Query o el fetching del framework, que ya resuelven caché, cancelación y errores.

### Preguntas frecuentes de seguimiento

**1. ¿Qué diferencia hay entre sin array, `[]` y con dependencias?**
Sin array, el efecto corre tras cada render. Con `[]`, solo tras el montaje (y su limpieza al desmontar). Con `[a, b]`, tras el montaje y cada vez que `a` o `b` cambian según `Object.is`.

**2. ¿Por qué mi efecto se ejecuta dos veces en desarrollo?**
Por `<StrictMode>`: en desarrollo React monta, ejecuta la limpieza y vuelve a montar para detectar efectos sin limpieza o no idempotentes. No ocurre en producción. La solución es escribir bien la limpieza, no quitar StrictMode.

**3. ¿Cómo evitas una race condition al pedir datos?**
Se marca en la limpieza que la respuesta ya no interesa, o se cancela la petición:

```jsx
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/users/${id}`, { signal: controller.signal })
    .then((res) => res.json())
    .then(setUser)
    .catch((err) => {
      if (err.name !== 'AbortError') setError(err);
    });
  return () => controller.abort();
}, [id]);
```

**4. ¿Qué diferencia hay entre `useEffect` y `useLayoutEffect`?**
`useLayoutEffect` corre de forma síncrona tras actualizar el DOM y antes del pintado; `useEffect` corre normalmente después. El primero sirve para medir el layout y ajustar sin parpadeo; bloquea el pintado, así que se usa solo cuando hace falta.

**5. ¿Cuándo NO usarías `useEffect`?**
Para derivar valores de props o estado (se calculan en el render), para reaccionar a eventos del usuario (van en el manejador) y para reiniciar estado cuando cambia una prop (se usa `key`). Un efecto que solo copia un estado en otro suele indicar un diseño mejorable.

**6. ¿Es correcto ignorar el aviso de `exhaustive-deps`?**
Casi nunca. El aviso indica que el efecto lee valores reactivos que no declaró, con riesgo de datos obsoletos. Se corrige moviendo lógica dentro del efecto, estabilizando el valor o usando `useEffectEvent` (React 19.2+) si el valor no debe disparar el efecto.

-----

## Siguiente lección

Cuando la misma lógica con `useState` y `useEffect` se repite en varios componentes, puedes guardarla en una función propia y reutilizarla: [Custom Hooks](03-Custom%20Hooks.md).
