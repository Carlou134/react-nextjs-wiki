# Custom Hooks: reutilizar lógica con estado

## En una frase

Un **custom hook** es una función propia, con nombre que empieza con `use`, que llama a otros Hooks (como `useState` o `useEffect`) para encapsular lógica y reutilizarla en varios componentes.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo guardar datos que cambian con [useState](01-The%20State%20Hook.md).
* Cómo ejecutar código cuando pasa algo con [useEffect](02-The%20Effect%20Hook.md).

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Custom hook (hook personalizado):** una función de JavaScript cuyo nombre empieza con `use` y que llama a otros Hooks.
* **Lógica con estado:** código que usa `useState` (y a veces `useEffect`) para conservar datos entre renders y reaccionar a cambios.
* **Convención:** acuerdo de nomenclatura que React no impone en tiempo de ejecución, pero que el linter y el equipo usan para reconocer los Hooks.
* **Tupla:** un arreglo con una cantidad fija de posiciones, donde cada posición tiene su propio tipo (por ejemplo, "primero un booleano, segundo una función").

-----

## El problema

Varios componentes suelen necesitar un valor de "encendido / apagado": modo oscuro, un menú, un panel. Sin abstracción, cada uno repite la misma lógica:

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

Con dos componentes es tolerable. Si la lógica crece (por ejemplo, con un `useEffect`), la duplicación multiplica el costo de mantenimiento: cada corrección debe replicarse en todas las copias.

La solución es definir la lógica **una sola vez** y reutilizarla.

-----

## Cómo funciona

### Qué es un custom hook

Es una función de JavaScript con dos rasgos:

1. Su nombre **empieza con `use`** (`useToggle`, `useLocalStorage`...).
2. **Llama a otros Hooks** (`useState`, `useEffect`, `useContext`, u otros custom hooks).

No es una API de React: no se importa ni se registra nada. El prefijo `use` es una convención que el linter aprovecha para aplicar las reglas de los Hooks.

Como cualquier Hook, **debe cumplir las reglas de los Hooks**: llamarse solo desde componentes de función u otros Hooks, y siempre en el nivel superior, nunca dentro de un `if`, un bucle o una función anidada. Las explicamos en "Las reglas de los Hooks", dentro de "Para profundizar" de [useState](01-The%20State%20Hook.md).

Fuera de eso, la firma es libre: tú decides qué argumentos recibe y qué devuelve.

### Definir el hook

La lógica repetida se extrae a una función, normalmente en su propio archivo:

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

* `initialState = false` es un **valor por defecto**: sin argumento, el estado arranca en `false`.
* `useState` funciona igual que dentro de un componente.
* `toggle` usa la forma funcional (`(prev) => !prev`) porque el valor nuevo depende del anterior (ver [useState](01-The%20State%20Hook.md)).
* Devuelve un arreglo `[state, toggle]`, con la misma forma que `[valor, setter]` de `useState`.

### Usarlo en un componente

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

Al desestructurar un arreglo **por posición**, los nombres los eliges tú: aquí `isDark` y `toggleDark`, no `state` y `toggle`.

Cualquier otro componente puede llamar a `useToggle()`. La lógica vive en un solo lugar.

### Cada componente tiene su propio estado

**Usar el mismo hook en dos componentes NO comparte datos entre ellos.** Cada llamada a `useToggle()` crea su **propio estado, aislado**, asociado a la instancia del componente que la ejecuta.

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

Cambiar el estado en `DarkMode` no afecta a `MenuButton`. Un custom hook reutiliza la **lógica**, no los **datos**.

```
DarkMode                          MenuButton
   |                                  |
   +-- useToggle()                    +-- useToggle()
         estado propio: isOn                estado propio: isOn
         (ahora vale true)                  (ahora vale false)

Mismo código (la lógica), pero cada componente tiene su propia copia de los datos.
```

Este aislamiento es deseable: evita acoplar componentes. Para que varios componentes lean el mismo dato se usa [Context](04-React%20Context.md) (próxima lección) o se sube el estado a un ancestro común.

### Cuándo crear uno

Crea un custom hook cuando tienes lógica con Hooks que:

* **Se repite en más de un componente.** Extraerla evita copiar y pegar.
* **Ensucia el componente**, aunque lo uses en un solo lugar. Si un componente tiene mucho `useState` y `useEffect` mezclados con el JSX, sacarlos a un hook lo deja más fácil de leer.

Una función es un custom hook si su nombre empieza con `use` y llama a otros Hooks. Si **no llama a ningún Hook**, es una función común y no debe llevar `use`.

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

* **Arreglo** (`return [state, toggle]`): para **dos valores relacionados**, como un valor y su función para cambiarlo. Imita a `useState` y permite renombrar libremente.
* **Objeto** (`return { location, error }`): para **tres o más valores**, o cuando el nombre aporta más que la posición. El consumidor elige por nombre, sin depender del orden, y añadir campos no rompe a quienes ya lo usan.

```jsx
// Arreglo: renombras libremente
const [modoOscuro, alternarModoOscuro] = useToggle();

// Objeto: eliges qué sacar, por nombre
const { location, error } = useGeolocation();
```

Regla práctica: dos valores estrechamente relacionados, arreglo; en los demás casos, objeto.

-----

## Ejemplo completo: useLocalStorage

Un hook que funciona **como `useState`, pero persistiendo el valor** entre visitas. Usa `localStorage`, un almacenamiento síncrono del navegador (clave-valor, solo texto) que sobrevive a las recargas.

Combina `useState` y `useEffect`, el caso típico de un custom hook.

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

Funcionamiento:

1. Recibe la **clave** de almacenamiento (`'name'`) y el **valor inicial** (`''`).
2. `useState(() => { ... })` recibe una **función** en lugar de un valor: la *inicialización perezosa*. React la ejecuta solo en el primer render (ver "Para profundizar" de [useState](01-The%20State%20Hook.md)). Ahí se lee `localStorage`; si no hay dato, se usa `initialValue`.
3. `localStorage` solo guarda texto: `JSON.stringify` para escribir y `JSON.parse` para leer.
4. El `useEffect` se ejecuta tras el render en que cambió `value` o `key`, y guarda el valor.
5. Devuelve `[value, setValue]`: el setter de `useState` sin envolver, por eso `setName((prev) => ...)` también funciona.

> Este ejemplo asume que la `key` **no cambia** mientras el componente está en pantalla: si cambiara, el efecto guardaría el valor viejo bajo la clave nueva, sin volver a leer. Además, `setItem` también puede fallar (por ejemplo, si el almacenamiento está lleno), así que en una app real conviene envolverlo en `try/catch`, igual que la lectura. Con renderizado en servidor (por ejemplo Next.js), `localStorage` no existe en el servidor: leerlo en el inicializador de `useState` falla o provoca diferencias de hidratación, y requiere un enfoque distinto.

-----

## Errores comunes

### 1. Nombrarlo sin `use`

```jsx
function toggle(initial) {     // Mal: no empieza con "use"
  const [state, setState] = useState(initial);
  return [state, () => setState((prev) => !prev)];
}
```

**Por qué pasa:** React no inspecciona el nombre en tiempo de ejecución. El prefijo `use` señala al linter (`eslint-plugin-react-hooks`) y a otros desarrolladores que la función llama a Hooks y debe cumplir sus reglas. Sin él, el linter no la vigila.
**Cómo se arregla:** renómbrala a `useToggle`.

### 2. Llamarlo dentro de un `if`

```jsx
function Panel({ canToggle }) {
  if (canToggle) {
    const [isOn, toggle] = useToggle();   // Mal
  }
}
```

**Por qué pasa:** React asocia cada estado con el **orden** de las llamadas a Hooks en cada render. Un custom hook llama a `useState` adentro, así que aplican las mismas reglas: si un render ejecuta la llamada y otro no, el orden se desalinea y React lanza errores como "Rendered fewer hooks than expected".
**Cómo se arregla:** llámalo siempre en el nivel superior del componente y aplica la condición **después**.

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
**Cómo se arregla:** si dos componentes necesitan ver el mismo dato, usa [Context](04-React%20Context.md), o sube el estado a un componente padre y pásalo por props.

### 4. Tipar mal el setter

Lo vemos en la sección siguiente.

-----

## En TypeScript

Hay dos detalles importantes al tipar un custom hook.

### 1. Si devuelves un arreglo, típalo como tupla

```tsx
export const useToggle = (initialState = false): [boolean, () => void] => {
  const [state, setState] = useState(initialState);
  const toggle = () => setState((prev) => !prev);
  return [state, toggle];
};
```

Sin la anotación `: [boolean, () => void]`, TypeScript infiere `(boolean | (() => void))[]`, un arreglo cuyos elementos son "booleano **o** función". Se pierde que **la posición 0 es siempre el booleano y la posición 1 la función**. Con la tupla, al desestructurar `const [isOn, toggle] = useToggle()`, cada variable recibe su tipo correcto.

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

* Si devuelves **tu propia función** con una firma acotada (como `toggle: () => void`, que no recibe nada), típala tal cual.
* Si devuelves **el setter de `useState` sin envolver**, usa `Dispatch<SetStateAction<T>>`.

La idea de fondo: si tu hook se comporta como `useState`, su tipo tiene que prometer lo mismo que `useState`, ni más ni menos.

-----

## Cuándo sí y cuándo no

**Crea un custom hook cuando:**

* La misma combinación de `useState` / `useEffect` se repite en más de un componente.
* Una lógica con estado ensucia el componente y sacarla deja el JSX más fácil de leer, aunque se use en un solo lugar.

**No lo hagas cuando:**

* **La función no llama a ningún Hook.** Es una función común: déjala sin `use`.
* **Solo quieres compartir datos entre componentes.** Un hook no comparte estado, comparte lógica. Para compartir datos, mira [Context](04-React%20Context.md).
* **La lógica se usa una sola vez y es corta.** Es una abstracción prematura: añade indirección sin ganar claridad. Extrae cuando la duplicación o la complejidad lo justifiquen.

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

Cada componente que use `useToggle()` obtiene también la animación (con su propio estado). El hook encapsula la complejidad: el componente solo ve `[state, toggle]`.

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

Devuelve un **objeto** porque el nombre aclara más que la posición. Cualquier componente que necesite la ubicación hace `const { location, error } = useGeolocation();`.

Para probarlo, el navegador debe tener habilitado el permiso de ubicación (requiere contexto seguro: HTTPS o `localhost`). Este ejemplo no vigila la posición; con `watchPosition` haría falta `clearWatch` en la limpieza del efecto.

**En TypeScript:** los tipos `GeolocationPosition` y `GeolocationPositionError` (y `GeolocationCoordinates`) ya vienen incluidos con TypeScript, dentro de su librería de tipos del navegador (la librería `dom`, que los proyectos web incluyen por defecto). No hace falta instalar ningún paquete extra. Si quieres tipar el callback a mano, alcanza con `(pos: GeolocationPosition) => { ... }` y tienes autocompletado de `pos.coords.latitude`, `pos.coords.longitude`, etc.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Un custom hook es una función cuyo nombre empieza con `use` y que llama a otros Hooks, como `useState` o `useEffect`. Se crea para extraer lógica repetida o compleja de los componentes y reutilizarla sin copiar y pegar. Comparte la lógica, no el estado: cada componente que lo llama obtiene su propio estado.

### Respuesta ampliada (semi-senior)

* **Qué se comparte:** la lógica, no el estado. Cada llamada al hook crea estado independiente en el componente que lo ejecuta.
* **Frente a un componente:** un componente devuelve JSX y se renderiza; un hook devuelve cualquier valor y no produce UI. **Frente a una función utilitaria:** el hook puede usar Hooks de React (estado, efectos, contexto); una función común no puede y no debe llevar `use`.
* **Prefijo `use` y reglas:** React no lo verifica en ejecución, pero `eslint-plugin-react-hooks` lo usa para exigir las reglas: llamar solo en el nivel superior y solo desde componentes u otros Hooks.
* **Diseño de la API:** tupla para dos valores relacionados (imita `useState`, renombrable); objeto para tres o más, porque es extensible sin romper a los consumidores.
* **TypeScript:** anotar la tupla explícitamente (`[boolean, () => void]`); usar genéricos (`<T>`) para valores arbitrarios; devolver `Dispatch<SetStateAction<T>>` si se expone el setter de `useState` tal cual.
* **Cuándo no extraer:** si la lógica se usa una vez y es corta, o si no llama a ningún Hook. Extraer pronto es abstracción prematura: añade indirección y una API que luego hay que mantener.
* **Testing:** se prueba con `renderHook` de React Testing Library (envolviendo los cambios de estado en `act`) o a través de un componente de prueba. La lógica pura sin Hooks se prueba como función normal.

### Preguntas frecuentes de seguimiento

**¿Dos componentes que usan el mismo hook comparten estado?**
No. Cada llamada crea su propio estado. Para compartirlo se usa Context o se sube el estado a un ancestro común.

**¿Custom hook, componente o función normal?**
Si devuelve UI, es un componente. Si encapsula lógica que usa Hooks, es un custom hook. Si no usa Hooks, es una función normal.

**¿Por qué debe empezar con `use`?**
Es una convención que React no valida en ejecución. El linter la usa para identificar Hooks y verificar sus reglas; sin el prefijo, esas comprobaciones no se aplican.

**¿Cómo tipas el retorno?**
Si devuelves un arreglo, anótalo como tupla; si devuelves un objeto, TypeScript infiere sus propiedades. Si expones el setter de `useState`, usa `Dispatch<SetStateAction<T>>`.

```tsx
function useToggle(initial = false): [boolean, () => void] { /* ... */ }
```

**¿Cómo lo pruebas?**
Con `renderHook` de React Testing Library: renderizas el hook, lees `result.current` y ejecutas las acciones dentro de `act`. Alternativamente, a través de un componente que lo use.

**¿Cuándo no crearías uno?**
Cuando la lógica no usa Hooks, se usa en un solo lugar y es corta, o cuando el objetivo es compartir datos (eso es Context). Extraer sin necesidad es abstracción prematura.

-----

## Siguiente lección

Un custom hook comparte **lógica**, no **datos**: cada llamada tiene su propio estado. Para que varios componentes vean el mismo dato, sigue [Context](04-React%20Context.md), que permite compartir datos sin pasarlos por props.
