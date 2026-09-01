# React Context

## Introduction

Supongamos que tienes la tarea de enviar un mensaje desde el piso 15 de un edificio hasta el primer piso. En lugar de enviarlo directamente, tienes que pasarlo a través de una persona en cada uno de los pisos intermedios. Parece ineficiente, ¿verdad? Esto es similar a lo que sucede en algunas aplicaciones de React.

Es posible que estés construyendo una aplicación React donde los datos se almacenan en un componente "Abuelo" (GrandParent) de alto nivel. Luego, necesitas pasar esos datos a través de un componente "Padre" (Parent) y, finalmente, a un componente "Hijo" (Child) para mostrarlos.

Este es un patrón común y bastante poderoso en las aplicaciones de React llamado **"prop drilling" (perforación de props)**. En el prop drilling, un fragmento de datos se transmite en cascada a través de múltiples componentes en la jerarquía, como en nuestra analogía del edificio. La prop se "perfora" desde un componente de alto nivel a través de componentes de nivel medio hasta llegar a un componente de bajo nivel.

Cuando las **props** solo tienen que perforarse a través de 1 o 2 componentes, el prop drilling puede ser manejable y preferible para aplicaciones pequeñas. Sin embargo, con más capas, introduce desafíos como:

*   Infla tu código y dificulta la comprensión o reutilización de los componentes intermedios.
*   Provoca que los componentes intermedios se vuelvan a renderizar debido a cambios en las props perforadas, incluso si ellos mismos no usan esas props.

En esta lección, veremos el **Context API (API de Contexto)** de React, un enfoque común para evitar el prop drilling innecesario. El **Contexto (Context)** es una característica de React que nos permite crear una pieza de **estado centralizado** a la que cualquier componente dentro de un área de tu aplicación puede suscribirse.

Al final de la lección, aprenderás:

*   Cómo un contexto de React proporciona estado compartido a los componentes de React.
*   A crear un contexto para proporcionar un valor a componentes descendientes de React.
*   A consumir ese valor en componentes descendientes de React.
*   A actualizar el valor del contexto en componentes descendientes de React.
*   A estructurar proveedores anidados para el mismo tipo de contexto.

¡Comencemos!

----

## Creating and Consuming Context

Es hora de explorar cómo podemos compartir datos de manera eficiente en nuestra aplicación y eliminar el "prop drilling" utilizando la **API de Contexto (Context API)**.

A través de un patrón de **Proveedor (Provider)** y **Consumidor (Consumer)** , la API de Contexto proporciona un mecanismo para compartir datos entre componentes sin complicaciones. El **proveedor** es un componente de React que pone los datos a disposición de sus componentes descendientes. Cuando uno de esos descendientes accede a los datos compartidos, se convierte en un **consumidor**.

Para usar la API de Contexto de React, comenzamos creando un **objeto de contexto de React**, un objeto con nombre creado por la función `React.createContext()`.

```jsx
const MyContext = React.createContext();
```

Los objetos de contexto incluyen una propiedad **.Provider** que es un componente de React. Este componente recibe una prop `value` que se almacena en el contexto.

```jsx
<MyContext.Provider value="Hello world!">
   <ChildComponent />
</MyContext.Provider>
```

Ese valor —en este caso, el string `"Hello world!"`— está disponible para todos los componentes descendientes. Los componentes descendientes —en este caso, `ChildComponent`— pueden entonces recuperar el valor del contexto con el hook **useContext()** de React.

```jsx
import { useContext } from 'react';
import { MyContext } from './MyContext.js';

const ChildComponent = () => {
  const value = useContext(MyContext);
  return <p>{value}</p>;
};
// Renderiza <p>Hello, world!</p>
```

El hook `useContext()` acepta el objeto de contexto como argumento y devuelve el valor actual del contexto. ¡Regocíjate, ya no es necesario perforar (prop drilling) ese valor!

**Nota:** Si un componente intenta usar un contexto que no es proporcionado por uno de sus ancestros, `useContext()` devolverá `undefined`.

**Nota:** En algunas aplicaciones React antiguas, es posible que en su lugar veas `SomeContext.Consumer` utilizado para suscribirse a un Contexto. Esa alternativa generalmente se considera una mala práctica y se evita por ser demasiado verbosa y difícil de trabajar.

----

## Multiple Providers

En el ejercicio anterior, aprendiste cómo crear un objeto de **contexto** de React. También aprendiste cómo asignar al componente **.Provider** del contexto una prop `value` para que los descendientes puedan acceder a ese valor. Ahora, avancemos en nuestra comprensión. ¿Qué pasa si nuestra aplicación requiere el mismo contexto en múltiples áreas, cada una con un valor distinto? Piensa en temas (themes) variables en diferentes secciones de la aplicación.

Un componente **.Provider** puede ser reutilizado con el mismo contexto múltiples veces en una aplicación con diferentes valores. Esto es útil si el contexto es renderizado por un componente que se utiliza varias veces en toda la aplicación. Cada instancia del componente podría querer dar un valor diferente al contexto. Por ejemplo, un componente podría renderizar dos componentes **.Provider** que reciban cada uno un valor diferente:

```jsx
const GreetingContext = React.createContext();

const ChildComponent = () => {
  const greeting = useContext(GreetingContext);
  return <h2>{greeting}</h2>;
};

const MyComponent = () => {
  return (
    <>
      <GreetingContext.Provider value="bonjour le monde!">
        <ChildComponent />
      </GreetingContext.Provider>
      <GreetingContext.Provider value="hallo welt!">
        <ChildComponent />
      </GreetingContext.Provider>
    </>
  );
};
```

Para practicar este concepto, refactoricemos nuestra aplicación. Configuraremos `ThemeContext` para que cada vez que se use su componente **.Provider** para diferentes secciones de contacto, se le dé una prop `value` distinta.

----

## Provider Wrappers

En los últimos tres ejercicios viste cómo los Contextos pueden compartir datos en un área de una aplicación React. Ahora vamos a mostrar los patrones de codificación comunes que muchos desarrolladores de React utilizan con los Contextos y sus componentes **.Provider**.

Para empezar, es común que las aplicaciones React creen un **componente "envoltorio" (wrapper)** alrededor de un componente **.Provider**. El componente envoltorio puede proporcionar un valor de su elección, a menudo una de sus **props** o un literal de cadena, al componente **.Provider**.

Por ejemplo, aquí hay un componente `ThemedMessage` que devuelve el componente `ThemeContext.Provider` envuelto alrededor de cualquier componente `children` que reciba. `ThemedMessage` también asigna un valor usando una prop `theme`:

```jsx
const ThemeContext = React.createContext();

const ThemedMessage = ({ children, theme }) => {
  return (
    <ThemeContext.Provider value={theme}>
      This content is in {theme} mode!
      {children}
    </ThemeContext.Provider>
  );
};
```

Los componentes envoltorio como `ThemedMessage` pueden entonces usarse en lugar de un componente **.Provider** para envolver componentes hijos:

```jsx
// Renderiza:
// This content is in dark mode! Hooray!
const MyComponent = () => {
  return (
    <ThemedMessage theme="dark">
      Hooray!
    </ThemedMessage>
  )
};
```

En este ejercicio, prepararás la aplicación para futuros cambios configurando un componente envoltorio dedicado para el **Contexto** `ThemeContext`. Refactorizaremos `ThemeContext` para que más adelante pueda proporcionar más que solo el string del tema a sus consumidores.

-----

## Updating Context

El ejercicio anterior te hizo configurar un componente envoltorio alrededor de un componente **.Provider** de un **Contexto**. Ahora vamos a hacer uso de ese componente envoltorio trabajando con el estado de React.

Muchas aplicaciones React utilizan "prop drilling" para pasar dos valores: un **fragmento de estado (piece of state)** y la **función actualizadora de estado (state updater function)** para actualizar ese estado. Los componentes hijos pueden entonces usar la función actualizadora de estado para cambiar el estado de sus ancestros. Por ejemplo:

```jsx
const CounterApp = () => {
  const [count, setCount] = useState(0);

  return (
    <Counter count={count} setCount={setCount} />
  );
};
```

En este ejemplo, `Counter` puede usar la función `setCount()` para actualizar el valor `count` de `CounterApp`.

Los Contextos de React también se pueden usar para proporcionar estado y funciones actualizadoras de estado. Un patrón común es que el contexto proporcione un **objeto** que contenga ambos valores. Los componentes hijos que consumen el contexto pueden entonces usar ambos (o cualquiera de ellos): el estado y la función actualizadora.

En este ejemplo, `CounterArea` proporciona el valor `count` y la función `setCount()` a sus descendientes usando contexto. El componente `Counter` extrae ambos valores del contexto proporcionado.

```jsx
const CounterContext = React.createContext();

const CounterArea = ({ children }) => {
  const [count, setCount] = useState(0);

  return (
    <CounterContext.Provider value={{ count, setCount }}>
      {children}
    </CounterContext.Provider>
  );
};

const Counter = () => {
  const { count, setCount } = useContext(CounterContext);

  return (
    <button onClick={() => setCount(count => count + 1)}>
      {count}
    </button>
  );
};

const CounterApp = () => {
  return (
    <CounterArea>
      <Counter />
    </CounterArea>
  );
};
```

En este ejercicio, añadirás lógica al componente `ThemeArea` que configure un fragmento de estado y una función actualizadora de estado para su contexto. Al hacerlo, permitirás que `ThemeContext` tenga su tema actualizado por componentes hijos.

----

## Nested Providers

Has visto cómo los **Proveedores (Providers)** de **Contexto** de React pueden usarse en múltiples lugares en una aplicación React. Aquí hay un breve ejemplo de un `GreeterContext` utilizado para configurar diferentes saludos para componentes hijos:

```jsx
<GreeterContext.Provider value="Salut!">
  <ChildComponent />
</GreeterContext.Provider>
<GreeterContext.Provider value="Hallo!">
  <ChildComponent />
</GreeterContext.Provider>
```

Un Proveedor de Contexto puede ser un hijo en el árbol de la aplicación, ubicado debajo de un Proveedor de Contexto anterior. Los componentes que se suscriben al Proveedor de un Contexto recibirán el valor del componente **.Provider** que esté **más cerca de ellos** en el árbol de la aplicación. Este patrón a veces se conoce como **anidamiento (nesting)** .

```jsx
<GreeterContext.Provider value="Salut!">
  <HighLevelComponent> {/* El valor de GreeterContext es "Salut!" aquí */}

    <GreeterContext.Provider value="Hallo!">  
      <LowLevelComponent /> {/* El valor de GreeterContext es "Hallo!" aquí */}
    </GreeterContext.Provider>

  </RootComponent>
</GreeterContext.Provider>
```

En este ejemplo, tenemos dos componentes que se utilizan dentro de `GreeterContext.Provider`: `HighLevelComponent` y `LowLevelComponent`. Cada componente usará el valor del `GreeterContext.Provider` más cercano. En este caso, `HighLevelComponent` tendrá el valor `"Salut!"` al usar `GreeterContext`, mientras que `LowLevelComponent` tendrá el valor `"Hallo!"`.

Vamos a usar el anidamiento de Contexto para configurar temas anidados en nuestra aplicación de gestión de contactos:

*   Un contexto raíz alrededor de toda la aplicación configurará un **tema raíz (root theme)** para toda la aplicación.
*   Un contexto anidado en cada sección de contactos **anulará (override)** el tema raíz para los elementos de contacto de su sección.

----------

## Review

¡Felicitaciones, has terminado la lección de **Context** de React! Recapitulemos lo que aprendiste:

*   **Prop drilling** es el término para el patrón común de React en el que los datos se pasan como prop a través de una gran cantidad de componentes en una aplicación.
*   Los **Contexts** nos permiten evitar el prop drilling de piezas de estado de la aplicación compartidas por muchos componentes. Los Contexts vienen con un componente **.Provider** que también puede tomar un `value` para ponerlo a disposición de los componentes hijos, sin tener que pasar el valor mediante prop drilling.
*   Los componentes hijos, que actúan como **Consumidores (Consumers)** , pueden suscribirse al valor de un Context desde su Provider padre más cercano con el hook **useContext()** de React. Los componentes que se suscriben a un Context recibirán el valor del Provider más cercano a ellos en el árbol de la aplicación.
*   A los Providers se les puede dar un **objeto** que contenga el estado de React y su correspondiente **función actualizadora de estado**. Los componentes hijos suscriptores pueden entonces usar la función actualizadora de estado para actualizar el estado del Context.

**Context** es una de las muchas formas en que las aplicaciones React pueden compartir estado entre muchos componentes sin prop drilling manual. Ten en cuenta, sin embargo, que es mejor usarlo con moderación y solo para valores que se pasan por prop drilling y que **no cambian con mucha frecuencia**. Más adelante, en la lección de Rendimiento (Performance) de este curso, verás cómo el uso excesivo de Context puede causar problemas de rendimiento.

Ten en cuenta también que **Context no siempre es la mejor solución** para la arquitectura de tu aplicación. Hay otras formas de manejar el estado, como Redux, `useReducer`, o simplemente `useState` y prop drilling (que no siempre es un problema).

-----

## Buenas prácticas y antipatrones

A medida que una aplicación crece, es fácil caer en algunos usos de **Context** que funcionan, pero que terminan generando problemas de rendimiento o de mantenibilidad. A continuación repasamos los antipatrones más comunes y su solución.

### Evitar un contexto gigante que centraliza todo

Un error frecuente es crear un único contexto que agrupa datos de dominios completamente distintos:

```tsx
<AppContext.Provider value={{ user, theme, cart, settings }}>
```

El problema es que **cualquier** cambio en cualquiera de esas propiedades —el usuario, el tema, el carrito o las preferencias— provoca que **todos** los componentes que consumen `AppContext` se vuelvan a renderizar, incluso aquellos que solo necesitan una de esas piezas de información. La solución es dividir el contexto por dominio, creando un `UserContext`, un `ThemeContext` y un `CartContext` independientes entre sí. De esta forma, un cambio en el carrito solo afecta a los componentes que realmente consumen `CartContext`.

### Memoizar el valor que recibe el Provider

Otro problema sutil aparece incluso con contextos bien separados. Cada vez que el componente que renderiza un `.Provider` se vuelve a ejecutar, una expresión como esta:

```tsx
<UserContext.Provider value={{ user }}>
```

crea un **objeto literal nuevo** en cada renderizado. Como React compara el valor del contexto por referencia y no por contenido, este nuevo objeto se considera "distinto" al anterior aunque `user` no haya cambiado, y todos los componentes suscritos se vuelven a renderizar innecesariamente.

La solución es envolver ese valor con **useMemo()**, para que solo se cree un nuevo objeto cuando sus dependencias realmente cambien:

```tsx
const value = useMemo(() => ({ user, login, logout }), [user]);

return (
  <UserContext.Provider value={value}>
    {children}
  </UserContext.Provider>
);
```

Si el Provider también expone funciones, conviene envolverlas con **useCallback()** por la misma razón: una función definida directamente en el cuerpo del componente se recrea en cada renderizado, invalidando la memoización del objeto `value` que la contiene.

### Envolver useContext() en un Hook personalizado

En lugar de que cada componente consumidor llame directamente a `useContext(UserContext)`, es una buena práctica exponer un **Hook personalizado** dedicado, como `useUser()` o `useTheme()`, que encapsule esa llamada:

```tsx
function useTheme() {
  const context = useContext(ThemeContext);

  if (context === undefined) {
    throw new Error('useTheme debe usarse dentro de un ThemeProvider');
  }

  return context;
}
```

Este patrón tiene varias ventajas: oculta el objeto de contexto como detalle de implementación, permite lanzar un error claro si el Hook se usa fuera de su Provider correspondiente (en lugar de fallar silenciosamente con `undefined`), y hace que el código consumidor sea más legible, ya que `useTheme()` comunica mejor la intención que `useContext(ThemeContext)`.

### Medir antes de optimizar

Es tentador aplicar `useMemo()` y `useCallback()` de forma reflexiva en todo el código de Context "por si acaso". Sin embargo, la memoización tiene un costo propio —React debe comparar dependencias en cada renderizado— y añade una capa extra de complejidad al código. Antes de optimizar, conviene usar el **Profiler de React** para confirmar que existe un problema real de renderizados innecesarios. Como regla general, si una optimización mejora el tiempo de renderizado en menos de un 5%, probablemente no vale la pena la complejidad adicional que introduce. La secuencia correcta es siempre: **medir, luego optimizar, y volver a medir** para confirmar la mejora.

-----

## Persistencia de estado con localStorage

En muchos casos, el estado que vive en un contexto debería sobrevivir a una recarga de página: el tema elegido por el usuario, el idioma de la interfaz, o un token de sesión son buenos ejemplos. Para lograrlo, sincronizamos el estado del contexto con `localStorage`.

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(
    () => localStorage.getItem('theme') || 'light'
  );

  useEffect(() => {
    localStorage.setItem('theme', theme);
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Hay dos detalles importantes en este ejemplo. Primero, el valor inicial de `useState()` se pasa como una **función inicializadora** (`() => localStorage.getItem('theme') || 'light'`) en lugar de como un valor directo. Esto es una **inicialización perezosa (lazy initialization)**: React solo ejecuta esa función una vez, durante el primer renderizado, en lugar de leer `localStorage` en cada renderizado del componente. Segundo, usamos un **useEffect()** con `theme` en su arreglo de dependencias para escribir en `localStorage` cada vez que el tema cambia, en lugar de escribir directamente durante el renderizado, ya que acceder a APIs del navegador es un efecto secundario y debe manejarse como tal.

-----

## Context con useReducer

Cuando el valor compartido por un contexto necesita soportar varias operaciones relacionadas entre sí —agregar, quitar o editar elementos, por ejemplo—, exponer un actualizador de estado distinto para cada operación a través del Provider se vuelve difícil de mantener. En estos casos, es común combinar **Context** con **useReducer()**: el contexto expone el estado actual junto con una única función `dispatch()`, y toda la lógica de las transiciones de estado queda centralizada en el reducer.

```jsx
const CartContext = createContext();

const initialState = { items: [] };

function cartReducer(state, action) {
  switch (action.type) {
    case 'add':
      return { items: [...state.items, action.payload] };
    case 'remove':
      return { items: state.items.filter((item) => item.id !== action.payload) };
    default:
      return state;
  }
}

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  const value = useMemo(() => ({ state, dispatch }), [state]);

  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}
```

Los componentes consumidores ya no necesitan recibir múltiples funciones actualizadoras a través del contexto; les basta con `dispatch()`:

```jsx
const { state, dispatch } = useContext(CartContext);

dispatch({ type: 'add', payload: newItem });
```

Este patrón —a veces descrito informalmente como "Redux sin librería externa"— funciona muy bien para estado global de complejidad media. Para aplicaciones con una cantidad de acciones y de estado mucho mayor, suele ser preferible migrar a una librería dedicada de manejo de estado, como Redux Toolkit o Zustand, que veremos en profundidad en la lección de `useReducer`.

-----

## Advertencias con operaciones asíncronas

Cuando el valor de un contexto depende de datos obtenidos de forma asíncrona —por ejemplo, un usuario que se carga desde una API—, hay que tener en cuenta que los componentes consumidores se renderizan **antes** de que esa petición se resuelva. Durante ese intervalo, el contexto expone un valor inicial (normalmente `null` o `undefined`), por lo que los componentes consumidores deben manejar explícitamente ese estado de carga en lugar de asumir que el dato ya está disponible.

Por otro lado, la lógica asíncrona **no debe vivir dentro del reducer**. Un reducer debe ser una función **pura**: dado el mismo estado y la misma acción, siempre debe devolver el mismo resultado, sin efectos secundarios como llamadas a una API. El patrón correcto es disparar la petición asíncrona desde un efecto o desde un manejador de eventos, y despachar una acción con el resultado una vez que la promesa se resuelve:

```jsx
useEffect(() => {
  let cancelled = false;

  fetchUser().then((user) => {
    if (!cancelled) {
      dispatch({ type: 'userLoaded', payload: user });
    }
  });

  return () => {
    cancelled = true;
  };
}, []);
```

Combinar Context con lógica asíncrona compleja —reintentos, caché, invalidación de datos— tiende a volverse difícil de escalar con las herramientas básicas de React. Cuando el manejo de datos asíncronos empieza a dominar la lógica de un contexto, suele ser una señal de que conviene migrar a una librería especializada en manejo de datos remotos, como React Query, o a un gestor de estado global como Redux Toolkit.

-----