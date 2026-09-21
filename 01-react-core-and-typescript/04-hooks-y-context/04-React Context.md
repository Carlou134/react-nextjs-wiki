# Context: compartir datos sin pasarlos por props

## En una frase

**Context** te deja poner un dato en un lugar compartido, para que cualquier componente de esa zona lo lea directamente, sin que se lo pasen por props uno a uno.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo un componente recibe datos con props: [Props](../02-componentes-y-props/03-Props.md).
* Cómo guardar un dato que cambia con `useState`: [useState](01-The%20State%20Hook.md).
* Qué es un Hook propio (`useAlgo`): [Custom Hooks](03-Custom%20Hooks.md).

Palabras nuevas (todas están explicadas también en el [glosario](Glosario.md)):

* **Prop drilling (perforación de props):** pasar una prop por muchos componentes intermedios que no la usan, solo para que llegue a uno que está más abajo.
* **Context (contexto):** un lugar compartido donde guardas un dato para que los componentes de una zona del árbol lo lean sin usar props.
* **Provider (proveedor):** el componente que "pone" el dato en el contexto y lo deja disponible para todo lo que tiene adentro.
* **Consumidor:** un componente que lee el dato del contexto.
* **Árbol de componentes:** la jerarquía de componentes, donde unos están dentro de otros (padres, hijos, nietos...).
* **Memoizar:** guardar un resultado para reutilizarlo y no volver a calcularlo si nada cambió.
* **Referencia:** la "dirección" de un objeto en la memoria. Dos objetos con el mismo contenido son referencias distintas; React compara así para saber si un valor cambió.
* **Wrapper (envoltorio):** un componente cuyo trabajo es envolver a otros para darles algo (en este caso, el Provider).

-----

## El problema

Imagina que tienes que mandar un mensaje desde el piso 15 de un edificio hasta la planta baja, pero solo puedes dárselo a la persona de cada piso, una por una. Todos los del medio lo cargan sin usarlo. Es lento y molesto.

En React pasa lo mismo cuando un dato vive arriba de todo (el tema de la app, el usuario) y lo necesita un componente muy abajo:

```jsx
function App() {
  const theme = 'dark';
  return <Page theme={theme} />;
}

// Page no usa theme, solo lo pasa
function Page({ theme }) {
  return <Card theme={theme} />;
}

// Card tampoco lo usa, solo lo pasa
function Card({ theme }) {
  return <Title theme={theme} />;
}

// Recién aquí se usa
function Title({ theme }) {
  return <h1>Tema: {theme}</h1>;
}
```

Esto se llama **prop drilling**. Con 1 o 2 niveles es manejable, y en una app chica está bien. Con más niveles trae problemas:

* Los componentes intermedios (`Page`, `Card`) se llenan de props que no usan, y cuesta entenderlos o reutilizarlos.
* Si la prop cambia, los intermedios se vuelven a renderizar aunque ellos no la usen.

Necesitas una forma de que `Title` lea el dato directamente. Esa forma es Context.

-----

## Cómo funciona

La idea tiene dos roles:

* El **Provider** (proveedor) es un componente que pone un dato a disposición de todos los componentes que tiene adentro.
* Cada componente de adentro que lee ese dato es un **consumidor**.

Sin props en el medio. Mira la diferencia en el árbol de componentes:

```
SIN Context: theme viaja por todos los componentes

App                 tiene theme
 └── Page           recibe theme y lo reenvía (no lo usa)
      └── Card      recibe theme y lo reenvía (no lo usa)
           └── Title    recibe theme y por fin lo usa


CON Context: Title lo lee directamente

ThemeProvider       pone el dato (theme)
 └── Page           no recibe ni reenvía nada
      └── Card      tampoco
           └── Title    llama a useTheme() y lee el dato directo
```

Se arma con una receta de pasos.

### Paso 1: definir el tipo y crear el contexto

Antes de escribir lógica, pregúntate: ¿qué necesitan leer (y cambiar) los componentes que van a usar esto? Eso es tu tipo. Después creas el contexto con `createContext`:

```tsx
import { createContext } from 'react';

type Theme = 'light' | 'dark';

type ThemeContextType = {
  theme: Theme;
  setTheme: (theme: Theme) => void;
};

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);
```

`createContext` necesita un valor por defecto. Casi nunca tienes uno "real" con sentido, así que se pasa `undefined` y se avisa en el tipo (`ThemeContextType | undefined`). Más abajo, en "En TypeScript", vemos por qué esto es una ventaja.

### Paso 2: crear el componente Provider (el "wrapper")

Un **wrapper** (envoltorio) es un componente que envuelve a otros. Este es el único lugar de tu código que va a usar `ThemeContext.Provider` directamente. Aquí vive el estado real y el `value` que se comparte:

```tsx
import { useMemo, useState } from 'react';

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Por partes:

* `children` es todo lo que pones **entre** las etiquetas `<ThemeProvider>...</ThemeProvider>`. Es lo que va a tener acceso al dato.
* `useState` guarda el tema, como ya sabes.
* `value` es lo que el Provider comparte: un objeto con el tema **y** la función para cambiarlo. Los consumidores pueden usar las dos cosas, o solo una.
* `useMemo` evita re-renders de más. Lo explicamos en la sección "Cómo se actualiza".

### Paso 3: crear el Hook propio (`useTheme`)

```tsx
import { useContext } from 'react';

function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme debe usarse dentro de un ThemeProvider');
  }
  return context;
}
```

`useContext(ThemeContext)` es el Hook de React que lee el valor del contexto. Si no hay ningún Provider arriba, devuelve el **valor por defecto** que le diste a `createContext` (en nuestro caso, `undefined`). Por eso el Hook propio revisa eso y lanza un error claro.

Este Hook se crea **siempre**, aunque por ahora lo use un solo componente. No es cierto que "solo vale la pena cuando hay muchos consumidores". Te da tres cosas:

* No repites el chequeo de `undefined` en cada componente que consume el contexto.
* Escondes `ThemeContext` como detalle interno: nadie fuera de este archivo necesita saber que existe.
* Si mañana cambias cómo funciona el contexto por dentro, ningún consumidor tiene que cambiar.

Cuesta cuatro líneas escribirlo. No lo postergues.

### Paso 4: envolver con el Provider

Pones el `<ThemeProvider>` alrededor de la parte del árbol que necesita el dato, **lo más cerca posible** de esos componentes:

* Si todo tu sitio necesita el tema, envuelve en la raíz (`<App>`).
* Si solo lo necesita una sección (por ejemplo, un panel de configuración), envuelve solo esa sección.

No hace falta que el Provider viva en la raíz "por las dudas".

```tsx
<ThemeProvider>
  <Page />
</ThemeProvider>
```

### Paso 5: consumir con el Hook

Cualquier componente que esté adentro del Provider, sin importar cuán profundo, llama a `useTheme()`:

```tsx
function Title() {
  const { theme } = useTheme();
  return <h1>Tema: {theme}</h1>;
}
```

`Page` y `Card` ya no reciben ni pasan nada. `Title` lee el dato directo.

### Paso 6: todo junto en su propio archivo

El tipo, `createContext`, el Provider y el Hook (los pasos 1, 2 y 3) van juntos en **un archivo propio** desde el primer día, por ejemplo `ThemeContext.tsx`, aunque tengas un solo consumidor. Es una unidad completa: contexto + Provider + Hook. Así cualquiera sabe de dónde importarla, y no queda desparramada entre los componentes que la usan.

Desde ese archivo exportas solo `ThemeProvider` y `useTheme`.

### Paso 7: el error clásico, consumir en el mismo componente que crea el Provider

Es tentador escribir esto, sobre todo en una demo:

```tsx
// Mal: tira "useTheme debe usarse dentro de un ThemeProvider"
function DemoDelTema() {
  const { theme } = useTheme(); // se ejecuta AQUÍ...

  return (
    <ThemeProvider>            {/* ...pero el Provider recién existe AQUÍ */}
      <h1>Tema: {theme}</h1>
    </ThemeProvider>
  );
}
```

No es un tema del orden de las líneas: mover `useTheme()` más abajo no lo arregla. Es un tema de **jerarquía del árbol**. `DemoDelTema` es quien *crea* el Provider, así que nunca puede ser hijo de ese mismo Provider. Un componente no puede estar arriba y abajo de sí mismo.

En el árbol se ve así. `useTheme()` busca un Provider **por encima** del componente que lo llama:

```
Mal: el mismo componente crea el Provider y usa el Hook

DemoDelTema                <- llama a useTheme(): busca un Provider ARRIBA suyo
 └── ThemeProvider         <- el Provider recién existe aquí, DEBAJO
      └── h1

Arriba de DemoDelTema no hay ningún Provider, así que useTheme() falla.


Bien: el que usa el Hook queda DEBAJO del Provider

DemoDelTema
 └── ThemeProvider                 <- el Provider
      └── ContenidoDelTema         <- llama a useTheme(): encuentra el Provider arriba
           └── h1
```

La solución es separar en dos componentes: uno de afuera que solo pone el Provider, y uno de adentro que usa el Hook:

```tsx
// Bien: el de adentro sí es descendiente del Provider
function DemoDelTema() {
  return (
    <ThemeProvider>
      <ContenidoDelTema />
    </ThemeProvider>
  );
}

function ContenidoDelTema() {
  const { theme } = useTheme(); // ahora sí, adentro del Provider
  return <h1>Tema: {theme}</h1>;
}
```

Esto no significa que siempre tengas que dividir todo. La mayoría de las veces el `<ThemeProvider>` envuelve a un `<App>` que ya existía. La regla solo aplica cuando un mismo componente quiere hacer las dos cosas: proveer y consumir.

-----

## Cómo se actualiza

El setter viaja **dentro del `value`**. Cuando un consumidor lo llama, pasa esto, en orden:

```
1. Un consumidor llama a setTheme('dark')
2. Cambia el estado del Provider
3. El Provider se vuelve a renderizar y crea un value nuevo
4. React vuelve a renderizar a TODOS los consumidores de ese contexto
5. Cada uno muestra el tema nuevo
```

### Para qué sirve useMemo

React decide si el `value` cambió comparándolo **por referencia**: pregunta "¿es el mismo objeto de antes?", no "¿tiene el mismo contenido?".

Si escribes el objeto directamente en el Provider:

```tsx
// Mal: objeto nuevo en cada render
<ThemeContext.Provider value={{ theme, setTheme }}>
```

cada vez que el Provider se renderiza se crea un objeto distinto. Para React es "un valor nuevo", y vuelve a renderizar a todos los consumidores.

Ojo con cuándo importa esto. En nuestro ejemplo, el Provider se renderiza cuando cambia `theme`, y ahí el `value` cambia de todas formas, así que `useMemo` no ahorra nada. Donde sí ayuda es cuando el Provider se vuelve a renderizar **por otro motivo** (porque tiene otro estado propio, o porque se renderizó su componente padre) y `theme` sigue siendo el mismo: sin `useMemo`, todos los consumidores se re-renderizarían igual, aunque nada del tema haya cambiado.

`useMemo` guarda el objeto y solo crea uno nuevo cuando cambian las dependencias que le indicas (en el ejemplo, `[theme]`):

```tsx
// Bien: mismo objeto mientras theme no cambie
const value = useMemo(() => ({ theme, setTheme }), [theme]);
```

`setTheme` no hace falta ponerlo en las dependencias: la función que devuelve `useState` es siempre la misma.

-----

## Ejemplo completo: tema claro y oscuro

Un solo Provider, tres consumidores (`Card`, `CardTitle` y `ThemeButton`) y un componente intermedio (`Page`) que no sabe nada del tema. Fíjate que `CardTitle` está **adentro** de `Card`: la profundidad no importa.

```tsx
// ThemeContext.tsx
import { createContext, useContext, useMemo, useState } from 'react';

type Theme = 'light' | 'dark';

type ThemeContextType = {
  theme: Theme;
  setTheme: (theme: Theme) => void;
};

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme debe usarse dentro de un ThemeProvider');
  }
  return context;
}
```

```tsx
// App.tsx
import { ThemeProvider, useTheme } from './ThemeContext';

// Consumidor 1: lee el tema para elegir colores
function Card({ children }: { children: React.ReactNode }) {
  const { theme } = useTheme();
  const style = theme === 'dark'
    ? { background: '#222', color: '#fff' }
    : { background: '#fff', color: '#222' };

  return <div style={style}>{children}</div>;
}

// Consumidor 2: está adentro de Card, y también lee el tema
function CardTitle() {
  const { theme } = useTheme();
  return <h2>Tema actual: {theme}</h2>;
}

// Consumidor 3: lee el tema y también lo cambia
function ThemeButton() {
  const { theme, setTheme } = useTheme();

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Cambiar tema
    </button>
  );
}

// No usa el tema: solo organiza la pantalla
function Page() {
  return (
    <>
      <Card>
        <CardTitle />
      </Card>
      <ThemeButton />
    </>
  );
}

export default function App() {
  return (
    <ThemeProvider>
      <Page />
    </ThemeProvider>
  );
}
```

Qué pasa, paso a paso:

1. `App` envuelve todo con `<ThemeProvider>`. Ahí vive el estado `theme`, que arranca en `'light'`.
2. `Page` no recibe ni pasa props de tema. Es un intermedio que no se entera de nada.
3. `Card` y `CardTitle` llaman a `useTheme()` y leen el tema. `CardTitle` está más profundo, y le da igual.
4. `ThemeButton` llama a `setTheme`. Cambia el estado del Provider, se crea un `value` nuevo y React vuelve a renderizar a los tres consumidores.
5. El botón **también** es un componente adentro del Provider. Si el botón estuviera en `App`, junto al `<ThemeProvider>`, sería el error del paso 7.

> Cuando tu estado tiene muchas acciones distintas (agregar, quitar, editar), se combina Context con `useReducer`. Eso se explica en la lección [useReducer](05-useReducer.md).

-----

## Errores comunes

### 1. Usar el Hook fuera del Provider

```tsx
function App() {
  return <Title />;   // no hay ningún ThemeProvider arriba
}
```

**Por qué pasa:** `Title` llama a `useTheme()`, pero no tiene un `ThemeProvider` por encima. `useContext` devuelve el valor por defecto (`undefined`) y la guardia lanza el error `useTheme debe usarse dentro de un ThemeProvider`.
**Cómo se arregla:** envuelve el componente (o algún ancestro suyo) con `<ThemeProvider>`. El error de la guardia es tu amigo: te dice exactamente qué falta, en lugar de fallar más tarde con algo confuso.

### 2. Consumir en el mismo componente que crea el Provider

```tsx
function App() {
  const { theme } = useTheme();   // error: aquí todavía no hay Provider
  return <ThemeProvider>...</ThemeProvider>;
}
```

**Por qué pasa:** el componente que crea el Provider no es hijo de ese Provider. No es un problema del orden de las líneas, es de la jerarquía del árbol.
**Cómo se arregla:** separalo en un componente de afuera que pone el Provider y uno de adentro que llama al Hook (paso 7 de la receta).

### 3. Poner varios Providers pensando que "cada componente necesita el suyo"

```tsx
// Mal: tres temas independientes, no uno compartido
<>
  <ThemeProvider><Header /></ThemeProvider>
  <ThemeProvider><Content /></ThemeProvider>
  <ThemeProvider><Footer /></ThemeProvider>
</>
```

**Por qué pasa:** cada `<ThemeProvider>` crea **su propio** `useState`. Son tres estados aislados. Si el botón de `Header` cambia el tema, `Content` y `Footer` no se enteran.
**Cómo se arregla:** un solo Provider que envuelva a todos los que tienen que compartir el dato.

```tsx
// Bien: un solo estado compartido
<ThemeProvider>
  <Header />
  <Content />
  <Footer />
</ThemeProvider>
```

### 4. Olvidar useMemo en el value (una optimización, no un bug)

```tsx
// Menos eficiente: objeto nuevo en cada render del Provider
<ThemeContext.Provider value={{ theme, setTheme }}>
```

**Por qué pasa:** React compara el `value` por referencia. Si el Provider se re-renderiza por otro motivo (otro estado suyo, o su componente padre), se crea un objeto nuevo y todos los consumidores se vuelven a renderizar, aunque el tema sea el mismo. La app sigue funcionando bien: solo hace trabajo de más.
**Cómo se arregla:** `const value = useMemo(() => ({ theme, setTheme }), [theme]);`

-----

## En TypeScript

```tsx
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

function ThemeProvider({ children }: { children: React.ReactNode }) { /* ... */ }
```

* **`createContext<T | undefined>(undefined)`:** el valor por defecto determina el tipo del contexto. Como casi nunca tienes un valor por defecto real, se pasa `undefined` y se agrega `| undefined` al tipo.
* **Por qué ayuda la guardia:** como el tipo incluye `| undefined`, TypeScript te **obliga** a manejar ese caso (con el `if (context === undefined) throw ...`) antes de dejarte usar `context.theme`. La guardia en sí se ejecuta en tiempo real: en cuanto un componente usa `useTheme()` sin Provider, lanza un error claro en ese mismo momento, en lugar de fallar más tarde con algo confuso. Y dentro de `useTheme`, después de la guardia, TypeScript ya sabe que el valor existe, así que los consumidores reciben el tipo limpio, sin `undefined`.
* **`children: React.ReactNode`:** todo componente wrapper recibe `children`, y se tipa así. `ReactNode` cubre todo lo que React puede dibujar: texto, número, elemento JSX, lista de elementos o nada. Aplica a **todos** los wrappers de esta lección.

Para ver más sobre tipado de Context: [Tipado de useReducer y Context API](../11-typescript-y-react/03-Tipado%20de%20useReducer%20y%20Context%20API.md).

-----

## Cuándo sí y cuándo no

**Usa Context cuando** varios componentes, en distintos niveles del árbol, necesitan el mismo dato y pasarlo por props te obligaría a atravesar componentes que ni lo usan. Ejemplos típicos: el tema visual, el usuario autenticado, el idioma de la interfaz.

**No lo uses para:**

* **Todo el estado de tu app.** Cuando el `value` cambia, se vuelven a renderizar **todos** los consumidores. Un valor que cambia muy seguido (lo que escribe el usuario en un campo, la posición del mouse) vuelve lenta la app.
* **Un dato que solo necesitan uno o dos niveles más abajo.** Ahí el prop drilling es más simple y más claro.
* **Estado complejo y muy dinámico.** Para eso mira [Zustand, Redux y Context: cuándo usar cada uno](../../03-state-management-and-data/01-Zustand,%20Redux%20y%20Context%20-%20Cuando%20usar%20cada%20uno.md).

-----

## Resumen en 5 líneas

1. Context comparte un dato con todos los componentes de una zona del árbol, sin pasar props por los intermedios.
2. La receta: `createContext<T | undefined>(undefined)`, un componente Provider con `useState` y `useMemo`, y un Hook propio (`useTheme`) con la guardia de `undefined`.
3. Envuelve con el Provider solo la parte del árbol que lo necesita, y **nunca** consumas en el mismo componente que lo crea.
4. Cuando alguien cambia el estado, React vuelve a renderizar a **todos** los consumidores; `useMemo` en el `value` evita re-renders de más.
5. Un solo Provider comparte un estado; cada `<Provider>` extra crea un estado nuevo y aislado.

-----

## Para profundizar

<details>
<summary>Varios Providers del mismo contexto con valores distintos</summary>

Un mismo contexto se puede usar con varios Providers, cada uno con un valor distinto. Sirve cuando quieres el mismo tipo de dato pero diferente por sección de la app (por ejemplo, temas distintos en distintas áreas de un dashboard). Es lo opuesto al error común 3: aquí lo haces a propósito.

```jsx
const GreetingContext = createContext();

function Greeting() {
  const greeting = useContext(GreetingContext);
  return <h2>{greeting}</h2>;
}

function MyComponent() {
  return (
    <>
      <GreetingContext.Provider value="bonjour le monde!">
        <Greeting />
      </GreetingContext.Provider>
      <GreetingContext.Provider value="hallo welt!">
        <Greeting />
      </GreetingContext.Provider>
    </>
  );
}
```

El mismo `Greeting` muestra un texto distinto según el Provider en el que esté. Esto es útil cuando un componente que renderiza un Provider se usa varias veces en la app, y cada copia quiere darle su propio valor.

Con un wrapper puedes pasarle el valor por una prop. Aquí usamos un contexto más simple, `MessageContext`, que guarda solo un texto (no es el `ThemeContext` de la receta, que guarda un objeto con `theme` y `setTheme`):

```tsx
const MessageContext = createContext<string>('light');

function ThemedMessage({ children, theme }: { children: React.ReactNode; theme: string }) {
  return (
    <MessageContext.Provider value={theme}>
      Este contenido está en modo {theme}.
      {children}
    </MessageContext.Provider>
  );
}

// Uso:
<ThemedMessage theme="dark">Hola</ThemedMessage>
```

</details>

<details>
<summary>Providers anidados</summary>

Un Provider puede estar adentro de otro Provider del mismo contexto. Los componentes reciben el valor del Provider **más cercano** hacia arriba en el árbol. Esto se llama *anidar* Providers.

```jsx
<GreeterContext.Provider value="Salut!">
  <HighLevelComponent>
    {/* El valor de GreeterContext es "Salut!" aquí */}

    <GreeterContext.Provider value="Hallo!">
      <LowLevelComponent />
      {/* El valor de GreeterContext es "Hallo!" aquí */}
    </GreeterContext.Provider>

  </HighLevelComponent>
</GreeterContext.Provider>
```

`HighLevelComponent` recibe `"Salut!"`. `LowLevelComponent` recibe `"Hallo!"`, porque su Provider más cercano es el de adentro. Es útil, por ejemplo, para tener un tema general de toda la app y sobrescribirlo en una sección concreta.

</details>

<details>
<summary>Buenas prácticas y antipatrones</summary>

**No armes un contexto gigante que centralice todo.**

```tsx
// Mal: datos de dominios distintos mezclados
<AppContext.Provider value={{ user, theme, cart, settings }}>
```

Cualquier cambio en cualquiera de esas propiedades vuelve a renderizar a **todos** los que consumen `AppContext`, incluso a los que solo necesitan una. Mejor dividir por dominio: un `UserContext`, un `ThemeContext` y un `CartContext` independientes. Así, un cambio en el carrito solo afecta a quienes consumen `CartContext`.

**Memoiza el `value` y las funciones.** Ya viste `useMemo` para el `value`. Si el Provider además expone funciones, envolvelas con `useCallback`: una función definida en el cuerpo del componente se crea de nuevo en cada render, y eso rompe la memoización del objeto `value` que la contiene.

```tsx
const login = useCallback((u: User) => setUser(u), []);
const logout = useCallback(() => setUser(null), []);
const value = useMemo(() => ({ user, login, logout }), [user, login, logout]);
```

**Envuelve `useContext` en un Hook propio.** Es el paso 3 de la receta. Oculta el objeto de contexto, lanza un error claro si se usa fuera del Provider (en vez de fallar en silencio con `undefined`), y `useTheme()` comunica mejor la intención que `useContext(ThemeContext)`.

**Mide antes de optimizar.** Es tentador poner `useMemo` y `useCallback` en todos lados "por las dudas". Pero memoizar tiene su propio costo: React compara las dependencias en cada render, y el código se complica. Antes de optimizar, usa el **Profiler de React** para confirmar que hay un problema real de renders de más, que se note en la práctica. Si después de optimizar la mejora que mides es mínima, probablemente no vale la complejidad que agrega. La secuencia es: medir, optimizar, y volver a medir.

**Nota histórica.** En apps React antiguas puedes ver `MyContext.Consumer` en lugar de `useContext`. Ese estilo se considera una mala práctica hoy: es más verboso y difícil de leer.

</details>

<details>
<summary>Persistencia con localStorage</summary>

A veces el dato del contexto tiene que sobrevivir a una recarga de página: el tema elegido, el idioma, un token de sesión. Para eso sincronizas el estado con `localStorage`:

```tsx
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>(
    () => (localStorage.getItem('theme') as Theme | null) ?? 'light'
  );

  useEffect(() => {
    localStorage.setItem('theme', theme);
  }, [theme]);

  const value = useMemo(() => ({ theme, setTheme }), [theme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

Dos detalles:

* El valor inicial de `useState` es una **función** (`() => ... ?? 'light'`). Se llama *inicialización perezosa*: React ejecuta esa función solo en el primer render, en lugar de leer `localStorage` en cada render. El `as Theme | null` le dice a TypeScript que confíe en que lo guardado es un tema válido; en una app real, conviene validarlo antes de usarlo.
* Se guarda con `useEffect` y `theme` en las dependencias, así se escribe en `localStorage` cada vez que el tema cambia. Escribir en `localStorage` es un efecto secundario (algo que toca el mundo de afuera de React), y por eso no va directo en el cuerpo del componente. Repasa [useEffect](02-The%20Effect%20Hook.md).

No tiene sentido persistir datos que deberían reiniciarse en cada carga.

</details>

<details>
<summary>Operaciones asíncronas</summary>

Si el valor del contexto viene de una petición (por ejemplo, un usuario que se carga desde una API), los consumidores se renderizan **antes** de que llegue la respuesta. Mientras tanto, el contexto tiene un valor inicial (normalmente `null` o `undefined`). Los consumidores tienen que manejar ese **estado de carga** explícitamente y no asumir que el dato ya está:

```tsx
function Profile() {
  const { user } = useUser();
  if (user === null) return <p>Cargando...</p>;
  return <p>Hola, {user.name}</p>;
}
```

Por otro lado, la lógica asíncrona **no va dentro de un reducer**. Un reducer tiene que ser una función *pura*: con el mismo estado y la misma acción siempre devuelve el mismo resultado, sin efectos como llamar a una API. Lo correcto es hacer la petición desde un efecto o desde un manejador de eventos, y hacer `dispatch` de una acción con el resultado cuando la promesa se resuelve:

```jsx
useEffect(() => {
  let cancelled = false;

  fetchUser().then((user) => {
    if (!cancelled) {
      dispatch({ type: 'userLoaded', payload: user });
    }
  });

  return () => {
    cancelled = true;   // si el componente se desmonta, ignoramos la respuesta
  };
}, []);
```

Más sobre reducers y `dispatch` en [useReducer](05-useReducer.md). Más sobre pedir datos en [Fetch de datos con useEffect](../10-fetching-de-datos/01-Fetch%20de%20Datos%20con%20useEffect.md).

Si la lógica asíncrona empieza a dominar el contexto (reintentos, caché, invalidación de datos), es señal de que conviene una librería pensada para eso, como React Query para datos remotos, o un gestor de estado global como Redux Toolkit.

</details>

<details>
<summary>Context con useReducer</summary>

Cuando el estado compartido necesita muchas operaciones relacionadas (agregar, quitar, editar), exponer un setter distinto por cada una se vuelve pesado. En esos casos es común combinar Context con `useReducer`: el contexto expone el estado y una sola función `dispatch`. Todo eso se explica en la lección [useReducer](05-useReducer.md).

</details>

-----

## Siguiente lección

Cuando el estado se complica (muchas acciones distintas sobre el mismo dato), conviene ordenarlo con un reducer, y combinarlo con Context: [useReducer](05-useReducer.md).
