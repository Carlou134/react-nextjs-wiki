# Custom Hooks: reutilizar lógica con estado

## En una frase

Un **custom hook** es una función tuya, con nombre que empieza con `use`, que junta en un solo lugar lógica hecha con otros Hooks (como `useState` y `useEffect`) para que la reutilices en varios componentes.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo guardar datos que cambian con [useState](01-The%20State%20Hook.md).
* Cómo ejecutar código cuando pasa algo con [useEffect](02-The%20Effect%20Hook.md).

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Custom hook (hook personalizado):** una función común que empieza con `use` y que llama a otros Hooks adentro.
* **Lógica con estado:** código que usa `useState` (y a veces `useEffect`) para recordar datos y reaccionar a cambios.
* **Convención:** un acuerdo entre programadores. No lo exige React como una función nueva: lo seguimos porque ayuda a que todos entiendan el código.
* **Tupla:** un arreglo con una cantidad fija de posiciones, donde cada posición tiene su propio tipo (por ejemplo, "primero un booleano, segundo una función").

-----

## El problema

Imagina que en tu app varios componentes necesitan un valor de "encendido / apagado": un botón de modo oscuro, un menú que se abre y se cierra, un panel que se muestra u oculta. En cada uno escribes lo mismo:

```jsx
function DarkModeButton() {
  const [isOn, setIsOn] = useState(false);
  const toggle = () => setIsOn((prev) => !prev);

  return <button onClick={toggle}>{isOn ? 'On' : 'Off'}</button>;
}

function MenuButton() {
  // Mal: la misma lógica copiada y pegada
  const [isOn, setIsOn] = useState(false);
  const toggle = () => setIsOn((prev) => !prev);

  return <button onClick={toggle}>{isOn ? 'Cerrar' : 'Abrir'}</button>;
}
```

Con dos componentes no parece grave. Pero si la lógica crece (por ejemplo, con un `useEffect` adentro), tendrías que copiar y pegar todo cada vez, y si encuentras un error, corregirlo en cada copia.

Lo que quieres es escribir esa lógica **una sola vez** y usarla donde haga falta.

-----

## Cómo funciona

### Qué es un custom hook

Es una función común de JavaScript. Lo único "especial" es que:

1. Su nombre **empieza con `use`** (`useToggle`, `useLocalStorage`...).
2. **Llama a otros Hooks** adentro.

Es una **convención**, no una función nueva de React. No hay nada que importar ni registrar: escribes la función y ya está.

Además tiene que **cumplir las reglas de los Hooks**, igual que `useState` o `useEffect`: llamarse solo desde componentes de función (o desde otros Hooks) y siempre en el nivel de arriba, nunca dentro de un `if` o un bucle. Las explicamos en "Las reglas de los Hooks", dentro de "Para profundizar" de [useState](01-The%20State%20Hook.md).

Fuera de eso, no tiene un formato obligatorio: tú decides qué argumentos recibe y qué devuelve.

### Paso 1: escribir el hook

Sacamos la lógica repetida a una función. Se acostumbra ponerla en su propio archivo:

```jsx
// useToggle.js
import { useState } from 'react';

export const useToggle = (initialState = false) => {
  // El argumento sirve como valor inicial del estado
  const [state, setState] = useState(initialState);

  // Una función fácil de usar para invertir el valor
  const toggle = () => setState((prev) => !prev);

  // Devolvemos el valor y la función
  return [state, toggle];
};
```

Vamos por partes:

* `initialState = false` es un **valor por defecto**: si quien usa el hook no pasa nada, arranca en `false`.
* Adentro usamos `useState` como en cualquier componente.
* `toggle` invierte el valor. Usa la forma con función (`(prev) => !prev`) porque el valor nuevo depende del anterior, como vimos en la lección de `useState`.
* Devuelve un arreglo `[state, toggle]`, igual que `useState` devuelve `[valor, setter]`.

### Paso 2: usarlo en un componente

```jsx
import { useToggle } from './useToggle';

function DarkMode() {
  // Usamos true como valor inicial
  const [isDark, toggleDark] = useToggle(true);

  return (
    <button onClick={toggleDark}>
      {isDark ? 'Modo oscuro: On' : 'Modo oscuro: Off'}
    </button>
  );
}
```

Como el hook devuelve un arreglo y se desestructura **por posición**, los nombres los eliges tú: aquí usamos `isDark` y `toggleDark`, no `state` y `toggle`.

Ahora el menú, el panel y el resto de los componentes pueden hacer `useToggle()` y listo. La lógica vive en un solo lugar.

### Cada componente tiene su propio estado

Este punto confunde mucho al principio, así que fíjate bien:

**Usar el mismo hook en dos componentes NO comparte datos entre ellos.** Cada componente que llama a `useToggle()` recibe su **propio estado, aislado**.

```jsx
function Page() {
  return (
    <>
      <DarkMode />   {/* tiene su propio isDark */}
      <MenuButton /> {/* tiene su propio estado, independiente del de arriba */}
    </>
  );
}
```

Si tocas el botón de `DarkMode`, `MenuButton` no se entera. Un custom hook reutiliza la **lógica**, no los **datos**. Cada llamada crea su copia de la memoria.

```
DarkMode                          MenuButton
   |                                  |
   +-- useToggle()                    +-- useToggle()
         estado propio: isOn                estado propio: isOn
         (ahora vale true)                  (ahora vale false)

Mismo código (la lógica), pero cada componente tiene su propia copia de los datos.
```

Esto es una ventaja: los datos de un componente no se "filtran" a otro. Si lo que quieres es justo lo contrario, que varios componentes vean el mismo dato, necesitas [Context](04-React%20Context.md), que viene en la próxima lección.

### Cuándo crear uno

Crea un custom hook cuando tienes lógica con estado que:

* **Se repite en más de un componente.** Extraerla evita copiar y pegar.
* **Ensucia el componente**, aunque lo uses en un solo lugar. Si un componente tiene mucho `useState` y `useEffect` mezclados con el JSX, sacarlos a un hook lo deja más fácil de leer.

Para saber si algo es un custom hook, fíjate en estas dos cosas:

* Su nombre empieza con `use`.
* Llama a otros Hooks adentro.

Si una función **no llama a ningún Hook**, es una función común. No lleva `use` en el nombre (ni corresponde ponérselo).

```jsx
// Es una función común: no llama a ningún Hook, así que NO lleva "use"
function formatPrice(price) {
  return `$${price.toFixed(2)}`;
}

// Es un custom hook: llama a useState
function useCounter() {
  const [count, setCount] = useState(0);
  return { count, increment: () => setCount((prev) => prev + 1) };
}
```

### Qué devolver: arreglo u objeto

No hay una única forma correcta. Depende de cuántos valores devuelves:

* **Arreglo** (`return [state, toggle]`): cuando son **dos valores relacionados**, como un valor y su función para cambiarlo. Sigue la misma forma que `useState`, y quien lo usa puede ponerles el nombre que quiera.
* **Objeto** (`return { location, error }`): cuando son **tres o más valores**, o cuando el nombre dice más que la posición. Quien lo usa saca solo lo que necesita, por nombre, sin depender del orden.

```jsx
// Arreglo: renombras libremente
const [modoOscuro, alternarModoOscuro] = useToggle();

// Objeto: eliges qué sacar, por nombre
const { location, error } = useGeolocation();
```

Regla práctica: dos valores estrechamente relacionados, arreglo. Tres o más, o valores donde el nombre aporta más claridad que la posición, objeto.

-----

## Ejemplo completo: useLocalStorage

Vamos con un hook más útil: uno que funciona **como `useState`, pero recordando el valor** aunque cierres y vuelvas a abrir la página. Para eso guarda el dato en `localStorage`, un pequeño almacenamiento que tiene el navegador y que no se borra al recargar.

Este hook combina `useState` y `useEffect`, que es justo el caso típico de un custom hook.

```jsx
// useLocalStorage.js
import { useState, useEffect } from 'react';

export function useLocalStorage(key, initialValue) {
  // 1. Estado inicial: primero miramos si ya hay algo guardado
  const [value, setValue] = useState(() => {
    try {
      const saved = localStorage.getItem(key);
      return saved !== null ? JSON.parse(saved) : initialValue;
    } catch {
      return initialValue;
    }
  });

  // 2. Cada vez que el valor cambia, lo guardamos
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

Y se usa igual que `useState`:

```jsx
function Settings() {
  const [name, setName] = useLocalStorage('name', '');

  return (
    <input value={name} onChange={(e) => setName(e.target.value)} />
  );
}
```

Qué pasa, paso a paso:

1. Le pasas dos cosas: la **clave** con la que se guarda (`'name'`) y el **valor inicial** (`''`).
2. `useState(() => { ... })` recibe una **función** en lugar de un valor. React la ejecuta **solo la primera vez**. Ahí leemos `localStorage`: si hay un dato guardado lo usamos; si no, usamos `initialValue`. (Se llama *inicialización perezosa*; está explicada en "Para profundizar" de [useState](01-The%20State%20Hook.md).)
3. `localStorage` solo guarda texto, por eso usamos `JSON.stringify` para guardar y `JSON.parse` para leer.
4. El `useEffect` se ejecuta después de cada cambio de `value` (o de `key`) y guarda el valor nuevo.
5. Devolvemos `[value, setValue]`: el setter de `useState`, tal cual. Por eso `setName((prev) => ...)` también funciona.

> Este ejemplo asume que la `key` **no cambia** mientras el componente está en pantalla: si cambiara, el efecto guardaría el valor viejo bajo la clave nueva, sin volver a leer. Además, `setItem` también puede fallar (por ejemplo, si el almacenamiento está lleno), así que en una app real conviene envolverlo en `try/catch`, igual que la lectura.

-----

## Errores comunes

### 1. Nombrarlo sin `use`

```jsx
function toggle(initial) {     // Mal: no empieza con "use"
  const [state, setState] = useState(initial);
  return [state, () => setState((prev) => !prev)];
}
```

**Por qué pasa:** React en sí no mira el nombre de tu función. El prefijo `use` le avisa a las herramientas que revisan tu código (los *linters*, como el plugin de ESLint para Hooks) y a otros programadores que esa función llama a Hooks y debe seguir sus reglas. Sin él, esas herramientas no pueden vigilarla y nadie sabrá que es un Hook.
**Cómo se arregla:** renombrala a `useToggle`.

### 2. Llamarlo dentro de un `if`

```jsx
function Panel({ canToggle }) {
  if (canToggle) {
    const [isOn, toggle] = useToggle();   // Mal
  }
}
```

**Por qué pasa:** un custom hook llama a `useState` adentro, así que le aplican las mismas reglas que a los Hooks normales: tiene que llamarse siempre, en el mismo orden. Con un `if`, a veces se ejecuta y a veces no, y React se confunde ("Rendered fewer hooks than expected").
**Cómo se arregla:** llamalo siempre al principio del componente y pon la condición **después**, al decidir qué mostrar.

```jsx
function Panel({ canToggle }) {
  const [isOn, toggle] = useToggle();   // Bien: siempre se ejecuta
  if (!canToggle) return null;
  return <button onClick={toggle}>{isOn ? 'On' : 'Off'}</button>;
}
```

### 3. Esperar que dos componentes compartan el estado

```jsx
// En Header.jsx
const [isOpen, toggleOpen] = useToggle();
// En Sidebar.jsx
const [isOpen, toggleOpen] = useToggle();   // Mal: esperas que sea "el mismo" isOpen
```

**Por qué pasa:** cada llamada al hook crea su propio estado. Son dos memorias separadas, aunque el código sea idéntico.
**Cómo se arregla:** si dos componentes necesitan ver el mismo dato, usa [Context](04-React%20Context.md), o sube el estado a un componente padre y pasalo por props.

### 4. Tipar mal el setter

Lo vemos en la sección siguiente.

-----

## En TypeScript

Hay dos detalles importantes al tipar un custom hook.

### 1. Si devuelves un arreglo, tipalo como tupla

```tsx
export const useToggle = (initialState = false): [boolean, () => void] => {
  const [state, setState] = useState(initialState);
  const toggle = () => setState((prev) => !prev);
  return [state, toggle];
};
```

Sin la anotación `: [boolean, () => void]`, TypeScript deduce que el hook devuelve un arreglo común de "booleano **o** función". Pierde la información de que **la posición 0 es siempre el booleano y la posición 1 es siempre la función**. Con la tupla, al desestructurar `const [isOn, toggle] = useToggle()`, cada variable recibe su tipo correcto.

### 2. Si devuelves el setter de `useState` tal cual, usa `Dispatch<SetStateAction<T>>`

Esto pasa en hooks como `useLocalStorage`, que devuelven el setter sin envolverlo. Un tipo que parece razonable, pero está mal:

```tsx
// Mal: el tipo del setter es más angosto que el real
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [value, setValue] = useState<T>(initialValue);
  // ...sincroniza con localStorage...
  return [value, setValue];
}
```

Compila sin errores, pero le quita algo a quien use tu hook. El setter real de `useState` acepta **un valor o una función** `(prev) => ...`. Si tu tipo dice `(value: T) => void`, TypeScript le prohíbe a quien use tu hook escribir `setValue((prev) => prev + 1)`.

La forma correcta:

```tsx
import { useState, type Dispatch, type SetStateAction } from 'react';

// Bien: el tipo promete lo mismo que el setter real
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, Dispatch<SetStateAction<T>>] {
  const [value, setValue] = useState<T>(initialValue);
  // ...sincroniza con localStorage...
  return [value, setValue];
}
```

`Dispatch<SetStateAction<T>>` es justamente el tipo del setter de `useState`. Regla práctica:

* Si devuelves **tu propia función** con una firma acotada (como `toggle: () => void`, que no recibe nada), tipala tal cual.
* Si devuelves **el setter de `useState` sin envolver**, usa `Dispatch<SetStateAction<T>>`.

La idea de fondo: si tu hook se comporta como `useState`, su tipo tiene que prometer lo mismo que `useState`, ni más ni menos.

-----

## Cuándo sí y cuándo no

**Crea un custom hook cuando:**

* La misma combinación de `useState` / `useEffect` se repite en más de un componente.
* Una lógica con estado ensucia el componente y sacarla deja el JSX más fácil de leer, aunque se use en un solo lugar.

**No lo hagas cuando:**

* **La función no llama a ningún Hook.** Es una función común: dejala sin `use`.
* **Solo quieres compartir datos entre componentes.** Un hook no comparte estado, comparte lógica. Para compartir datos, mira [Context](04-React%20Context.md).
* **La lógica se usa una sola vez y es cortita.** Crear un hook para tres líneas suma un archivo más sin ganar claridad.

-----

## Resumen en 5 líneas

1. Un custom hook es una función común cuyo nombre empieza con `use` y que llama a otros Hooks adentro. Es una convención, no una función nueva de React.
2. Tiene que cumplir las reglas de los Hooks, igual que `useState` y `useEffect`.
3. Cada componente que lo usa tiene su **propio estado aislado**: comparte la lógica, no los datos.
4. Devuelve un arreglo si son dos valores relacionados y un objeto si son tres o más (o si el nombre aporta más que la posición).
5. En TypeScript, tipa el arreglo como **tupla**, y si devuelves el setter de `useState` sin envolver, usa `Dispatch<SetStateAction<T>>`.

-----

## Para profundizar

<details>
<summary>Un custom hook con useEffect adentro</summary>

Un custom hook también puede ejecutar efectos. Por ejemplo, `useToggle` podría hacer una animación cada vez que el valor cambia:

```jsx
export const useToggle = (initialState = false) => {
  const [state, setState] = useState(initialState);

  // performToggleAnimation es una función imaginaria, solo para el ejemplo
  useEffect(() => {
    performToggleAnimation(state);
  }, [state]);

  const toggle = () => setState((prev) => !prev);

  return [state, toggle];
};
```

Todo el que use `useToggle()` recibe también la animación, sin escribirla otra vez. Esa es la ventaja de esconder lógica compleja dentro del hook: el componente solo ve `[state, toggle]`.

</details>

<details>
<summary>Ejemplo: un hook con la geolocalización del navegador</summary>

Los navegadores traen una API para pedir la ubicación del dispositivo: `navigator.geolocation`. Tiene dos funciones principales: `getCurrentPosition()` (pide la posición una vez) y `watchPosition()` (la vigila de forma continua). Las dos reciben un callback de éxito, que recibe un objeto con la propiedad `coords` (las coordenadas), y opcionalmente un callback de error como segundo argumento.

Podemos envolver eso en un hook que combina `useState` (guardar la ubicación y el error) con `useEffect` (pedirla al montar el componente):

```tsx
import { useState, useEffect } from 'react';

function useGeolocation() {
  const [location, setLocation] = useState<GeolocationCoordinates | null>(null);
  const [error, setError] = useState<GeolocationPositionError | null>(null);

  useEffect(() => {
    navigator.geolocation.getCurrentPosition(
      (pos) => setLocation(pos.coords),   // éxito
      (err) => setError(err)              // error
    );
  }, []);

  return { location, error };
}
```

Devuelve un **objeto** porque son varios valores y el nombre aclara más que la posición. Cualquier componente que necesite la ubicación hace `const { location, error } = useGeolocation();`.

Para probarlo, el navegador tiene que tener los permisos de ubicación habilitados.

**En TypeScript:** los tipos `GeolocationPosition` y `GeolocationPositionError` (y `GeolocationCoordinates`) ya vienen incluidos con TypeScript, dentro de su librería de tipos del navegador (la librería `dom`, que los proyectos web incluyen por defecto). No hace falta instalar ningún paquete extra. Si quieres tipar el callback a mano, alcanza con `(pos: GeolocationPosition) => { ... }` y tienes autocompletado de `pos.coords.latitude`, `pos.coords.longitude`, etc.

</details>

-----

## Siguiente lección

Viste que un custom hook comparte **lógica**, pero no **datos**: cada componente tiene su propia copia. Para el caso contrario, cuando varios componentes necesitan ver el mismo dato, sigue [Context](04-React%20Context.md), que te deja compartir datos entre componentes sin pasarlos por props.
