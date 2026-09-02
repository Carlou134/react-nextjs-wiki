# Custom Hooks

## Introduction to Advanced Hooks

En este punto de tu viaje con React, deberías estar familiarizado con los conceptos básicos de los **hooks** de React. Los hooks nos permiten realizar lógica esencial con nuestros componentes de función. Estas acciones incluyen la gestión del estado con **useState()** y la ejecución de código después de un renderizado con **useEffect()**.

En esta lección, vamos a construir sobre estos hooks básicos y aprenderemos cómo crear nuestros propios **hooks personalizados (custom hooks)**. Los hooks personalizados son funciones que encapsulan lógica utilizando otros hooks de React. Se comportan igual que los hooks integrados en React, pero nos permiten combinarlos y reutilizarlos para reducir la complejidad lógica y la repetición.

Los hooks personalizados no son una característica nueva de React, sino una **convención** ampliamente utilizada en el mundo de React. Al aprender esta convención, puedes crear hooks para tareas más específicas y complejas.

En el camino hacia la creación de nuestros propios hooks personalizados, practicaremos y repasaremos algunos conceptos básicos sobre los hooks. Específicamente, repasaremos cómo funciona el hook `useEffect()` internamente, así como las **reglas** que todos los hooks deben seguir. Con estas habilidades, podemos comenzar a crear nuestros hooks personalizados. ¡Comencemos!

----

## Reviewing the Effect Hook

Antes de entrar en la creación de hooks personalizados, repasemos lo que sabemos sobre el hook **useEffect()**.

El hook `useEffect()` permite a los desarrolladores realizar una acción después del renderizado. Estas acciones son típicamente **efectos secundarios (side effects)** del renderizado del componente y a menudo son reacciones a cambios de estado. Un ejemplo común de estos "efectos secundarios" es la obtención de datos después de que el componente se renderiza.

`useEffect()` acepta dos argumentos:

1.  Una **función de callback** que se ejecuta después de que el componente se renderiza.
2.  Un **array de dependencias** que dicta cuándo debe volver a ejecutarse el callback.

```jsx
useEffect(() => {
  fetchData('someapi.com/key/123');
}, []); // <-- Un array de dependencias vacío
```

Cuando pasamos un **array de dependencias vacío** como segundo argumento a nuestro hook de efecto, el callback solo se ejecutará después del **primer renderizado** del componente. En este ejemplo, solo queremos obtener datos una vez después del primer renderizado.

Cuando se proporcionan variables en el array de dependencias, React solo ejecutará el callback pasado a `useEffect()` cuando esas **variables se actualicen**.

```jsx
const [searchQuery, setSearchQuery] = useState('');

useEffect(() => {
  fetchData(`someapi.com/search?q=${searchQuery}`);
}, [searchQuery]); // ← Una sola dependencia
```

En este ejemplo, obtendremos nuevos datos solo cuando el valor de `searchQuery` cambie. De esta manera, el array de dependencias nos permite lograr el equilibrio adecuado entre hacer demasiadas llamadas a la API y servir los datos más actualizados a nuestros usuarios.

-----

## Reviewing the Rules of Hooks

Repasemos las **reglas de los hooks**. Estas reglas se aplican a los hooks incorporados de React, como **useState()** y **useEffect()**, así como a cualquier hook personalizado que creemos.

**Regla #1: Solo llama a los hooks desde componentes de función de React.** Los hooks no son compatibles con componentes de clase ni con funciones regulares de JavaScript. Esto asegura que el comportamiento del hook sea predecible y consistente. Siguiendo esta regla, podemos separar fácilmente nuestra lógica basada en hooks del resto de la lógica de nuestra aplicación.

**Regla #2: Solo llama a los hooks en el nivel superior de tus componentes de función.** No los llames dentro de otras funciones, condicionales o bloques de bucle. Esta regla tiene que ver con asegurarnos de que nuestros hooks se llamen cada vez, y en el mismo orden, cada vez que un componente se vuelve a renderizar.

A medida que los usuarios interactúan con la aplicación, provocando re-renderizados, React ejecuta sus funciones, incluyendo todas las llamadas a hooks. Entonces, ¿cómo puede React hacer un seguimiento de las llamadas a `useState()` o `useEffect()` que se realizan entre renderizados?

React rastrea los datos y callbacks de los hooks por su **secuencia** en el componente. Si ejecutamos nuestros hooks solo durante algunos re-renderizados y no en otros, este orden se desordenará, causando resultados inesperados.

Por ejemplo, si pusiéramos una llamada a `useEffect()` dentro de una sentencia `if`:

```jsx
const [searchQuery, setSearchQuery] = useState('');

if (!searchQuery) {
  useEffect(() => {
    fetchData(`someapi.com/search?q=${searchQuery}`);
  }, [searchQuery]);
}
```

El componente llamaría a `useState()` cada vez, pero solo llamaría a `useEffect()` a veces. Si usáramos este hook en nuestra aplicación, podríamos encontrarnos con el siguiente error:

```
Error sin capturar: Se renderizaron menos hooks de los esperados. Esto puede ser causado por una declaración de retorno anticipada accidental.
```

En su lugar, podemos lograr el mismo objetivo mientras llamamos consistentemente a nuestro hook cada vez:

```jsx
const [searchQuery, setSearchQuery] = useState('');
useEffect(() => {
  if (!searchQuery) {
    fetchData(`someapi.com/search?q=${searchQuery}`);
  }
}, [searchQuery]);
```

Siguiendo esta regla, podemos asegurar que nuestros hooks se llamen en el mismo orden y en cada renderizado.

**Nota:** Ten cuidado de no confundir ejecutar un hook en cada renderizado con ejecutar el **callback** que se le pasa en cada renderizado. Los callbacks de `useEffect()` pueden no ser llamados en cada renderizado dependiendo de los valores del array de dependencias. Sin embargo, el hook `useEffect()` en sí mismo **debe** ser llamado en cada renderizado.

-----

## Custom Hooks

Los **hooks personalizados (custom hooks)** son funciones de JavaScript que nos permiten encapsular lógica con estado (stateful logic) y reutilizarla. Por ejemplo, podemos crear hooks personalizados para efectos de uso común, como el manejo de formularios, animaciones, temporizadores, etc.

Hay dos cosas a tener en cuenta:

1.  Como convención, los hooks personalizados deben tener nombres que comiencen con **`use`**.
2.  También deben seguir las **reglas de los hooks**.

Aparte de eso, no necesitan tener un diseño específico: el desarrollador decide qué argumentos toma y si debe devolver algo.

Considera este ejemplo de hook personalizado, **useToggle()**:

```jsx
// useToggle.js
export const useToggle = (initialState = false) => {
  // Usar el argumento `initialState` para inicializar el estado
  const [state, setState] = useState(initialState);

  // Realizar una animación cada vez que el estado cambia
  useEffect(() => {
    performToggleAnimation(state);
  }, [state])

  // Crear una función toggle fácil de usar
  const toggle = () => { setState(state => !state) }

  // Devolver el valor del estado y la función toggle
  return [state, toggle]
}
```

> **En TypeScript:** este hook devuelve `[state, toggle]` esperando que quien lo use haga `const [state, toggle] = useToggle(true)`, exactamente como con `useState()`. Pero hay una trampa: si anotás el tipo de retorno con `Array<boolean | (() => void)>` (o dejás que TypeScript lo infiera solo), el compilador no sabe que la primera posición siempre es el booleano y la segunda siempre la función — trata ambas posiciones como si pudieran contener cualquiera de los dos tipos, y `toggle()` en la posición 0 no daría error. Para que se comporte como una **tupla** (orden y tipos fijos, igual que el retorno real de `useState()`), hay que anotarlo explícitamente:
>
> ```tsx
> export const useToggle = (initialState = false): [boolean, () => void] => {
>   const [state, setState] = useState(initialState);
>   const toggle = () => setState((state) => !state);
>   return [state, toggle];
> };
> ```

En este ejemplo, creamos un hook personalizado llamado `useToggle()` que:

*   Usa **useState()** para gestionar un valor de estado de tipo "toggle" (encendido/apagado).
*   Usa **useEffect()** para ejecutar un efecto de animación de cambio.
*   Crea una función `toggle()` para interactuar con la función `setState()`.

Podemos imaginar que esta funcionalidad de "toggle" se usa en muchos lugares de nuestra aplicación. En lugar de copiar y pegar toda esta lógica cada vez que queramos usarla, ¡podemos simplemente importar y usar `useToggle()`!

```jsx
import { useToggle } from './useToggle';

const DarkMode = () => {
  // Obtener el estado y la función toggle de useToggle()
  // Usaremos un valor inicial de true
  const [state, toggle] = useToggle(true);

  return (
    <button onClick={toggle}> 
      {state ? 'On' : 'Off'}
    </button>
  )
}
```

En este ejemplo, creamos un componente `DarkMode` usando el hook personalizado `useToggle()`. `useToggle()` devuelve el valor del **estado** del toggle y una función `toggle()` para cambiar el estado. ¡Ahora, `DarkMode` puede usar estos valores y la lógica subyacente que los soporta, sin tener que escribir todo el código de nuevo!

Los hooks personalizados presentan varias ventajas cuando se usan correctamente en una aplicación:

*   Nos permiten **abstraer nuestro código**, ocultar lógica compleja y compartir lógica con estado entre múltiples componentes.
*   Al usar un hook personalizado en varios componentes, cada instancia opera de forma **aislada**, manteniendo su propio estado y efectos secundarios independientes. Esto significa que cualquier dato de un componente no se "filtrará" a otro.
*   Al crear un archivo separado del cual exportamos el hook personalizado, podemos **importarlo en cualquier parte de nuestra aplicación**.
*   Con una implementación adecuada, los hooks personalizados hacen que nuestro código sea inherentemente más **reutilizable**, **legible** y **rápido de desarrollar**.

-----

## Create Your Own Custom Hook

¡Ahora es el momento de crear nuestro propio hook personalizado! Lo haremos creando un hook que utilice la **API de Geolocalización (Geolocation API)** del navegador web. La API de Geolocalización nos permite obtener las coordenadas de un usuario, lo que nos permite proporcionar una experiencia personalizada a los usuarios según su ubicación. Veamos cómo funciona esta API antes de usarla.

La API está disponible fácilmente usando el objeto `window.navigator.geolocation` (puedes omitir la parte de `window`). La API proporciona funciones específicas para obtener la posición actual de un dispositivo (**.getCurrentPosition()**) o para vigilar continuamente la posición de un dispositivo (**.watchPosition()**).

Al usar cualquiera de estas funciones, es obligatorio pasar una **función de callback de éxito**. Este callback se ejecutará en caso de ejecución exitosa de la API de Geolocalización y se le pasará un objeto `position` que contiene la propiedad `.coords`: las coordenadas del dispositivo del usuario.

```jsx
navigator.geolocation.getCurrentPosition((pos) => {
  console.log('Ubicación Actual', pos.coords); // ← registra la posición actual del dispositivo
});
```

> **En TypeScript:** no hace falta tipar a mano el objeto `position` ni el `error` de la API de Geolocalización — TypeScript los incluye de fábrica a través de la librería `lib.dom.d.ts` que trae el propio compilador, con los tipos `GeolocationPosition` y `GeolocationPositionError`. Alcanza con anotar el parámetro del callback: `(pos: GeolocationPosition) => { ... pos.coords ... }`, y vas a tener autocompletado real de `pos.coords.latitude`, `pos.coords.longitude`, etc., sin instalar ningún paquete de tipos adicional.

Opcionalmente, se puede pasar una **función de callback de error** como segundo argumento que se ejecutará si la llamada a la API falla.

```jsx
function success(pos) {
  console.log('Ubicación Actual', pos.coords); // ← registra la posición actual del dispositivo
};

function fail(error) {
  console.log('Vaya, algo salió mal', error); // ← se ejecuta si la API falla
}

navigator.geolocation.watchPosition(success, fail);
```

Ahora que hemos repasado los conceptos básicos de la API de Geolocalización, ¡usemos lo que hemos aprendido y creemos un hook personalizado reutilizable!

Dirígete al editor de código. En la carpeta `/components` hay dos archivos de componentes para ver: `HemisphereDisplay.js` y `LongitudeLatitudeDisplay.js`. En cada uno, encontrarás que hay una variable `currentLocation` que cada componente espera que sea la posición de coordenadas actual del usuario. Actualmente, el valor está codificado como `null`.

Para que nuestra aplicación funcione correctamente, crearemos un hook personalizado que:

*   Gestione el estado de la ubicación con **useState()**.
*   Use un efecto para obtener la ubicación actual del dispositivo con **useEffect()**.
*   Devuelva la ubicación al usuario del hook.

¡Comencemos!

**Nota:** Asegúrate de que tu navegador tenga los permisos de ubicación habilitados para probar tu código.

----

## Review

¡Felicitaciones por terminar la lección de Hooks Personalizados (Custom Hooks)! En esta lección, aprendiste:

*   Cómo funcionan los hooks **useState()** y **useEffect()**.
*   Las **reglas** que rigen los hooks. Las reglas de los hooks se aplican tanto a los hooks incorporados como a cualquier hook personalizado que creemos.
*   La **convención de nomenclatura** de los hooks personalizados: todos comienzan con `use`, pero por lo demás pueden diseñarse como el desarrollador considere conveniente.
*   El concepto de que los hooks personalizados ayudan a **compartir lógica con estado (stateful logic)**.
*   El concepto de que los hooks personalizados son útiles para **encapsular lógica de hooks compleja y repetitiva** y a menudo se crean en su propio archivo y se exportan para una máxima portabilidad.

-----

