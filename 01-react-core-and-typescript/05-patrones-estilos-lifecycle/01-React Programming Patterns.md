# Patrones de programación en React: organizar y reutilizar lógica

## En una frase

Un **patrón** es una forma probada de organizar componentes para que la lógica se reutilice y cada pieza tenga una sola responsabilidad. Esta lección recorre los patrones clásicos de React y aclara cuáles se siguen usando hoy y cuáles los Hooks dejaron en segundo plano.

-----

## Antes de empezar

Conviene que ya sepas:

* Cómo un componente recibe datos con props: [Props](../02-componentes-y-props/03-Props.md).
* Cómo guardar un dato que cambia con `useState`: [useState](../04-hooks-y-context/01-The%20State%20Hook.md).
* Qué es un Hook propio (`useAlgo`): [Custom Hooks](../04-hooks-y-context/03-Custom%20Hooks.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Componente contenedor:** componente que maneja estado o lógica y no define la interfaz por sí mismo.
* **Componente presentacional:** componente que solo recibe props y devuelve JSX.
* **HOC (Higher-Order Component):** función que recibe un componente y devuelve otro componente con funcionalidad añadida.
* **Render prop:** prop cuyo valor es una función que devuelve JSX.
* **Composición:** construir componentes complejos combinando componentes simples.
* **Slot:** espacio dentro de un componente donde quien lo usa decide qué contenido va.
* **Prop drilling:** pasar una prop por componentes intermedios que no la usan.

-----

## El problema

Dos problemas aparecen cuando una aplicación crece:

1. **Un componente hace demasiado.** Guarda estado, hace cálculos, pide datos y además define el JSX. Es difícil de leer, de probar y de cambiar.
2. **La misma lógica se repite.** Varios componentes necesitan la misma protección de acceso, la misma suscripción o la misma estructura visual, y copiar código genera inconsistencias.

Los patrones de esta lección responden a esas dos preguntas: cómo separar responsabilidades y cómo reutilizar lógica o estructura.

-----

## Cómo funciona

### Container y Presentational

Este patrón divide un componente en dos:

* El **contenedor** guarda el estado, calcula y decide qué mostrar.
* El **presentacional** recibe datos y funciones por props y devuelve JSX. No sabe de dónde vienen los datos.

```tsx
// Presentational.tsx
export function Presentational({ active, toggle }: { active: boolean; toggle: () => void }) {
  return (
    <>
      <h1>Motores: {active ? 'encendidos' : 'apagados'}</h1>
      <button onClick={toggle}>Cambiar</button>
    </>
  );
}
```

```tsx
// Container.tsx
import { useState } from 'react';
import { Presentational } from './Presentational';

export function Container() {
  const [isActive, setIsActive] = useState(false);
  return <Presentational active={isActive} toggle={() => setIsActive(!isActive)} />;
}
```

Un componente presentacional no tiene estado propio, pero sigue siendo reactivo: si sus props cambian, su JSX también.

Hoy este patrón se aplica de forma más flexible. Como la lógica con estado se puede extraer a un custom hook, el componente suele quedar como "presentacional" llamando a un hook, sin necesidad de un contenedor aparte. La separación de responsabilidades sigue siendo válida; la estructura de dos archivos ya no es obligatoria.

### Comunicación padre-hijo y entre hermanos

Los datos bajan del padre al hijo por props. Un hijo no puede modificar sus props: son de solo lectura y, según la documentación de React, cuando un componente necesita cambiarlas debe "pedirle" a su padre que le pase otras.

```tsx
// Incorrecto: mutar una prop
function Presentational(props: { isActive: boolean }) {
  const onClick = () => {
    props.isActive = !props.isActive; // no se hace
  };
}
```

Para que el hijo avise un cambio, el padre le pasa una **función** por props. El hijo la llama y el padre actualiza su estado:

```tsx
function Container() {
  const [isActive, setIsActive] = useState(false);

  return (
    <>
      <Presentational active={isActive} toggle={setIsActive} />
      <OtherPresentational active={isActive} />
    </>
  );
}

function Presentational({ active, toggle }: { active: boolean; toggle: (v: boolean) => void }) {
  return <button onClick={() => toggle(!active)}>Motor: {active ? 'on' : 'off'}</button>;
}
```

Esto también resuelve la comunicación entre hermanos. Dos componentes con el mismo padre no se hablan directamente: uno avisa al padre, el padre actualiza el estado y le entrega el valor nuevo a ambos. A esto se le llama "elevar el estado" al ancestro común más cercano.

### Higher-Order Components (HOC)

Un **HOC** es una función que recibe un componente y devuelve otro. No es una característica de React: surge de que un componente es una función y una función puede recibir y devolver funciones.

Durante años fue la forma estándar de compartir lógica entre componentes sin relación directa. Ejemplo típico, proteger una pantalla según la autenticación:

```tsx
const withAuth = (Component) => {
  return (props) => {
    const isAuth = true; // en la práctica vendría de un contexto o de un hook
    return isAuth ? <Component {...props} /> : <Login />;
  };
};

const DashboardConAuth = withAuth(Dashboard);
```

`withAuth` no modifica `Dashboard`: lo envuelve. El componente devuelto decide, antes de renderizar, si muestra `Dashboard` o `Login`.

**Por qué los Hooks los desplazaron.** Envolver tiene un costo. Al combinar varios HOC (`withAuth(withTheme(withData(Componente)))`) el árbol se llena de capas que no aportan JSX, solo lógica. Esto se conoce como **wrapper hell**: más nodos en las React DevTools, más difícil saber de dónde viene cada prop y riesgo de colisión de nombres cuando dos HOC inyectan la misma prop.

Con Hooks, casi todo lo que hacía un HOC (compartir estado, efectos, suscripciones) se resuelve con un **custom hook** que se llama dentro del componente, sin nodos extra ni indirección de props. La documentación actual de React enseña la reutilización de lógica con custom hooks, y la documentación anterior ya señalaba que los Hooks evitan el anidamiento que producían HOC y render props. Por eso, para lógica nueva, la recomendación es empezar por un hook. Los HOC siguen siendo válidos y aparecen en código existente y en algunas librerías de terceros.

### HOC tipados y componibles

En un HOC real hay que tipar dos cosas: qué prop inyecta y que esa prop deje de ser obligatoria para quien use el componente resultante. Se logra con un generic y `Omit`:

```tsx
interface WithThemeProps {
  theme: 'light' | 'dark';
}

const withTheme = (theme: 'light' | 'dark') =>
  <P extends WithThemeProps>(
    Component: React.ComponentType<P>
  ): React.ComponentType<Omit<P, keyof WithThemeProps>> => {
    const Wrapped = (props: Omit<P, keyof WithThemeProps>) => (
      <Component {...(props as P)} theme={theme} />
    );
    Wrapped.displayName = `withTheme(${Component.displayName || Component.name})`;
    return Wrapped;
  };
```

Cómo leer la firma:

* `withTheme` recibe el tema y devuelve una función que toma un `Component`.
* `P extends WithThemeProps` exige que las props del componente incluyan `theme`.
* El resultado es un componente cuyas props son las de `P` **sin** `theme` (`Omit<P, keyof WithThemeProps>`), porque ahora las provee el HOC.
* `displayName` no es obligatorio, pero sin él todos los componentes envueltos aparecen como `Wrapped` en las DevTools.

Varios HOC tipados así se combinan sin perder seguridad de tipos:

```tsx
// withLogging y withUser siguen el mismo esquema que withTheme
const EnhancedDashboard = withLogging(withUser(withTheme('dark')(BaseDashboard)));

// Solo hay que pasar las props que ningún HOC de la cadena provee
<EnhancedDashboard title="Panel principal" />
```

Cada HOC inyecta su prop y la resta del tipo final, así que TypeScript sabe qué props faltan tras aplicar los tres envoltorios.

### Render Props

Una **render prop** es una prop cuyo valor es una función que devuelve JSX. El componente conserva la lógica y le entrega los datos a esa función; quien lo usa decide cómo se dibujan.

```tsx
function DataFetcher({ url, render }: { url: string; render: (data: unknown) => React.ReactNode }) {
  const [data, setData] = useState<unknown>(null);

  useEffect(() => {
    fetch(url).then((res) => res.json()).then(setData);
  }, [url]);

  return render(data);
}

<DataFetcher url="/api/users" render={(data) => <UserList users={data} />} />
```

Es común pasar la función como `children`:

```tsx
<DataFetcher url="/api/users">
  {(data) => <UserList users={data} />}
</DataFetcher>
```

Igual que con los HOC, los custom hooks reemplazan a las render props en la mayoría de los casos: `useFetch(url)` devuelve los datos sin envolver el JSX en una función ni añadir un nivel de anidamiento. (El ejemplo omite manejo de errores y cancelación para centrarse en el patrón; para datos reales, revisa [Fetch de datos con useEffect](../10-fetching-de-datos/01-Fetch%20de%20Datos%20con%20useEffect.md).) Las render props siguen siendo útiles cuando la lógica la ofrece una librería con esa API, o cuando el componente debe entregar valores a un fragmento de JSX que cambia según el uso.

### Composición vs. herencia

En la programación orientada a objetos clásica se reutiliza comportamiento con **herencia**: una clase extiende a otra (`Animal → Perro → Bulldog`). En React no se recomienda modelar componentes así. La documentación anterior de React lo decía explícitamente: tras usar React en miles de componentes, no encontraron casos donde recomendaran jerarquías de herencia de componentes.

La herencia en UI crea acoplamiento rígido: un cambio en la clase base puede romper todas las derivadas. Si `BotonPrimario`, `BotonConIcono` y `BotonDeAlerta` heredan de `BotonBase`, cada ajuste en la base se propaga a los tres.

React resuelve lo mismo con **composición**: componentes pequeños, con una sola responsabilidad, que se combinan.

```tsx
// Con herencia (no recomendado)
class BotonBase extends React.Component { /* ... */ }
class BotonConIcono extends BotonBase { /* ... */ }

// Con composición (idiomático)
function Boton({ children }: { children: React.ReactNode }) {
  return <button className="boton">{children}</button>;
}

function BotonConIcono({ icono, children }: { icono: React.ReactNode; children: React.ReactNode }) {
  return (
    <Boton>
      {icono}
      {children}
    </Boton>
  );
}
```

`BotonConIcono` no hereda de `Boton`: lo usa. Cada pieza se prueba y evoluciona por separado, y un nuevo tipo de botón es un componente nuevo, no un cambio en una jerarquía compartida.

### Slots y children

La prop `children` es la forma de React de crear **slots**: huecos que quien usa el componente rellena con JSX cualquiera. Según la documentación, un componente con `children` tiene un "hueco" que quien lo usa puede llenar; es habitual en envoltorios visuales como paneles o rejillas.

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}

<Card>
  <h1>Título</h1>
  <p>Contenido</p>
</Card>
```

`Card` solo aporta el contenedor; no conoce su contenido.

Cuando se necesita más de un hueco (cabecera, contenido, pie), `children` no alcanza porque es uno solo. Se añaden props que reciben JSX, que funcionan como slots con nombre:

```tsx
function Layout({ header, children, footer }: {
  header: React.ReactNode;
  children: React.ReactNode;
  footer: React.ReactNode;
}) {
  return (
    <div>
      <header>{header}</header>
      <main>{children}</main>
      <footer>{footer}</footer>
    </div>
  );
}

<Layout header={<Navbar />} footer={<Footer />}>
  <Contenido />
</Layout>
```

### Prop drilling y cómo evitarlo

El **prop drilling** ocurre cuando una prop atraviesa componentes que no la usan solo para llegar a uno más abajo:

```tsx
function App() {
  const user = useUser();
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />; // solo reenvía
}

function Sidebar({ user }) {
  return <UserMenu user={user} />; // solo reenvía
}

function UserMenu({ user }) {
  return <span>{user.name}</span>; // recién aquí se usa
}
```

Cada intermediario queda acoplado a un dato que no le pertenece, y renombrar la prop obliga a tocar toda la cadena.

La documentación de React sugiere estas opciones, en este orden:

1. **Empezar con props.** Pasar datos explícitamente hace claro qué componente usa qué. Con uno o dos niveles es lo más simple.
2. **Extraer componentes y pasar JSX como `children`.** Si un dato cruza muchas capas que no lo usan, a menudo falta extraer algún componente. En lugar de `<Layout user={user} />`, se escribe `<Layout><UserMenu user={user} /></Layout>` y `Layout` deja de saber del usuario.
3. **Context**, solo si lo anterior no funciona bien.

Con Context, un ancestro provee el valor y cualquier descendiente lo lee con `useContext`:

```tsx
const UserContext = createContext<User | null>(null);

function App() {
  const user = useUser();
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const user = useContext(UserContext);
  return <span>{user?.name}</span>;
}
```

Detalle completo, con la guardia de `undefined` y `useMemo`, en [React Context](../04-hooks-y-context/04-React%20Context.md). Para estado global con muchos cambios suele considerarse una librería dedicada (ver [Zustand, Redux y Context](../../03-state-management-and-data/01-Zustand,%20Redux%20y%20Context%20-%20Cuando%20usar%20cada%20uno.md)).

### Compound Components

Algunos componentes están formados por **varias partes que deben coordinarse**: pestañas (lista y paneles), acordeón, menú desplegable. La primera opción suele ser un único componente configurado por props:

```tsx
<Tabs
  tabs={[
    { id: 'perfil', label: 'Perfil', content: <Perfil /> },
    { id: 'config', label: 'Configuración', content: <Config /> },
  ]}
/>
```

Funciona hasta que llegan los pedidos de personalización (un ícono, una pestaña deshabilitada, un panel con otro layout). Cada uno agrega una prop nueva y la API se vuelve grande y rígida.

**Compound Components** divide el componente en partes que se componen con JSX y se comunican por un estado compartido a través de Context, sin que quien las usa las conecte a mano:

```tsx
<Tabs defaultTab="perfil">
  <Tabs.List>
    <Tabs.Tab id="perfil">Perfil</Tabs.Tab>
    <Tabs.Tab id="config">Configuración</Tabs.Tab>
  </Tabs.List>

  <Tabs.Panel id="perfil"><Perfil /></Tabs.Panel>
  <Tabs.Panel id="config"><Config /></Tabs.Panel>
</Tabs>
```

Quien lo usa controla la estructura (orden, contenido intermedio, estilos de cada parte) y las partes siguen sincronizadas. Implementación:

```tsx
import { createContext, useContext, useState, type ReactNode } from 'react';

type TabsContextType = {
  activeTab: string;
  setActiveTab: (id: string) => void;
};

const TabsContext = createContext<TabsContextType | undefined>(undefined);

function useTabs() {
  const context = useContext(TabsContext);
  if (context === undefined) {
    throw new Error('Las partes de Tabs deben usarse dentro de <Tabs>');
  }
  return context;
}

function Tabs({ defaultTab, children }: { defaultTab: string; children: ReactNode }) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div>{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: ReactNode }) {
  return <div role="tablist">{children}</div>;
}

function Tab({ id, children }: { id: string; children: ReactNode }) {
  const { activeTab, setActiveTab } = useTabs();

  return (
    <button role="tab" aria-selected={activeTab === id} onClick={() => setActiveTab(id)}>
      {children}
    </button>
  );
}

function TabPanel({ id, children }: { id: string; children: ReactNode }) {
  const { activeTab } = useTabs();
  return activeTab === id ? <div role="tabpanel">{children}</div> : null;
}

Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;
```

Es el mismo esqueleto de Context de la lección anterior: un contexto tipado, un hook con guardia de `undefined` y un padre (`Tabs`) que es Provider y dueño del estado. Lo propio del patrón son las tres últimas líneas, que cuelgan las partes del componente padre para importarlas y usarlas como una unidad. Por brevedad, el ejemplo no incluye la navegación con teclado ni los atributos `aria-controls` que un componente de pestañas accesible necesitaría.

* **Cuándo conviene:** varias partes comparten estado y el consumidor necesita libertad para acomodarlas (pestañas, acordeones, menús, selects, modales con encabezado, cuerpo y pie). Es el estilo que usan librerías como Radix UI, sobre la que se construye shadcn/ui.
* **Cuándo no:** si las partes no comparten estado, basta con composición común (`children` y slots). Si el componente es simple, una API por props es más corta.
* **Costo:** las partes solo funcionan dentro del padre (de ahí la guardia) y el contrato es implícito: la firma de `Tab` no indica que debe estar dentro de `Tabs`; solo lo avisa el error en ejecución. Existe una variante antigua con `React.Children` y `cloneElement` para inyectar props a los hijos, pero es frágil (se rompe si envuelves un hijo en otro componente) y hoy se prefiere Context.

-----

## Ejemplo completo

Un panel de usuarios que combina varios patrones: un componente con estado (contenedor), componentes presentacionales, un layout con slots y `children`, y la función de cambio pasada por props.

```tsx
import { useState, type ReactNode } from 'react';

type User = { id: number; name: string; active: boolean };

// Presentacional: solo recibe props y devuelve JSX
function UserFilter({ query, onQueryChange }: { query: string; onQueryChange: (q: string) => void }) {
  return (
    <input value={query} onChange={(e) => onQueryChange(e.target.value)} placeholder="Buscar" />
  );
}

function UserList({ users }: { users: User[] }) {
  if (users.length === 0) return <p>Sin resultados</p>;
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}{u.active ? '' : ' (inactivo)'}</li>
      ))}
    </ul>
  );
}

// Presentacional con slots: no conoce su contenido
function Panel({ title, toolbar, children }: { title: string; toolbar: ReactNode; children: ReactNode }) {
  return (
    <section>
      <header>
        <h2>{title}</h2>
        {toolbar}
      </header>
      {children}
    </section>
  );
}

// Contenedor: estado y lógica, sin definir estructura visual propia
export function UsersPage({ users }: { users: User[] }) {
  const [query, setQuery] = useState('');

  const visible = users.filter((u) => u.name.toLowerCase().includes(query.toLowerCase()));

  return (
    <Panel title="Usuarios" toolbar={<UserFilter query={query} onQueryChange={setQuery} />}>
      <UserList users={visible} />
    </Panel>
  );
}
```

Qué pasa:

1. `UsersPage` guarda `query` y calcula `visible`. Es el único que tiene estado.
2. `UserFilter` avisa cada cambio con `onQueryChange`; no modifica nada por su cuenta.
3. `Panel` ofrece dos huecos (`toolbar` y `children`) y no sabe qué se pondrá en ellos.
4. `UserList` recibe la lista ya filtrada y solo la dibuja.

Nada aquí necesita un HOC ni una render prop. Si mañana el filtrado se comparte con otra pantalla, esa lógica se extrae a un custom hook `useUserFilter`.

-----

## Errores comunes

### 1. Mutar una prop en el hijo

```tsx
function Toggle(props: { isActive: boolean }) {
  return <button onClick={() => { props.isActive = !props.isActive; }}>Cambiar</button>;
}
```

**Por qué pasa:** parece natural "cambiar el valor" donde se muestra. Las props son de solo lectura y el componente debe ser una función pura respecto a ellas, así que el cambio no se refleja ni provoca un nuevo render.
**Cómo se arregla:** el padre guarda el estado y pasa una función (`onToggle`) que el hijo llama.

### 2. Crear un HOC dentro del render

```tsx
function App() {
  const Enhanced = withAuth(Dashboard); // se crea un componente nuevo en cada render
  return <Enhanced />;
}
```

**Por qué pasa:** cada llamada a `withAuth` devuelve un componente distinto. React lo trata como un tipo nuevo, descarta el árbol anterior y pierde su estado en cada render.
**Cómo se arregla:** aplica el HOC una sola vez, fuera del componente, y usa el resultado (`const DashboardConAuth = withAuth(Dashboard);`).

### 3. Reenviar la misma prop por muchos niveles

Ver el ejemplo de prop drilling: los componentes intermedios quedan acoplados a un dato que no usan.
**Cómo se arregla:** primero extraer componentes y pasar JSX como `children`; si no alcanza, usar Context.

### 4. Usar `children` cuando hacen falta varios huecos

**Por qué pasa:** `children` es un único hueco, y forzar varias piezas dentro obliga a inspeccionarlas o a reordenarlas dentro del componente.
**Cómo se arregla:** props con nombre (`header`, `footer`) que reciben JSX.

### 5. Usar una parte de un Compound Component fuera de su padre

```tsx
<Tabs.Tab id="perfil">Perfil</Tabs.Tab> // sin <Tabs> alrededor
```

**Por qué pasa:** `Tab` lee el contexto que pone `Tabs`. Sin Provider arriba, `useContext` devuelve el valor por defecto (`undefined`).
**Cómo se arregla:** colócala dentro de `<Tabs>`. La guardia en `useTabs()` convierte el fallo en un mensaje claro.

### 6. Aplicar un patrón donde no hace falta

**Por qué pasa:** se envuelve, se separa o se pone Context "por si acaso".
**Cómo se arregla:** empieza con lo más simple (un componente con props) y añade un patrón cuando el problema aparece: lógica repetida, un componente demasiado grande o prop drilling real.

-----

## En TypeScript

* **HOC:** usa un generic `P extends TuProp` y `Omit<P, keyof TuProp>` en el tipo del componente devuelto, como en `withTheme` (sección "HOC tipados y componibles"). Así el consumidor no está obligado a pasar la prop que inyecta el HOC.
* **Render props:** tipa la función explícitamente (`render: (data: T) => React.ReactNode`). Con un generic en el componente (`DataFetcher<T>`), el tipo de `data` llega al callback sin conversiones.
* **Slots y children:** `children: React.ReactNode` cubre todo lo que React puede dibujar (texto, número, elemento, lista o nada). Los slots con nombre se tipan igual.
* **Compound Components:** el contexto se crea con `createContext<T | undefined>(undefined)` y un hook que lanza un error si el valor es `undefined`; así, dentro de las partes el tipo ya no incluye `undefined`. Al asignar `Tabs.List = TabList`, TypeScript reconoce las propiedades estáticas de una función declarada con `function`.

Más sobre tipado en [Tipado de Props y Funciones](../11-typescript-y-react/01-Tipado%20de%20Props%20y%20Funciones.md) y [Tipado de useReducer y Context API](../11-typescript-y-react/03-Tipado%20de%20useReducer%20y%20Context%20API.md).

-----

## Cuándo sí y cuándo no

Para lógica nueva, el orden habitual es: props y composición, luego custom hook, luego Context. HOC y render props quedan para casos específicos.

| Patrón | Qué problema resuelve | Cuándo elegirlo hoy | Dónde verlo |
| --- | --- | --- | --- |
| Container / Presentational | Separar lógica de la interfaz | Como principio de diseño; a menudo se cumple con un custom hook en vez de dos componentes | Esta lección |
| Composición, slots y `children` | Estructuras flexibles sin herencia | Primera opción para reutilizar estructura visual y evitar prop drilling | Esta lección; [Props](../02-componentes-y-props/03-Props.md) |
| Custom Hooks | Reutilizar lógica con estado | Primera opción para compartir lógica entre componentes | [Custom Hooks](../04-hooks-y-context/03-Custom%20Hooks.md) |
| HOC | Reutilizar lógica envolviendo un componente | Código existente o librerías que lo exigen; no para lógica nueva en la mayoría de casos | Esta lección |
| Render Props | Compartir lógica dejando que el consumidor decida el JSX | Cuando una librería lo ofrece o cuando hay que entregar datos a JSX variable; en otros casos, un hook | Esta lección |
| Compound Components | Partes coordinadas que se componen con JSX | Componentes de UI con varias partes que comparten estado | Esta lección |
| Provider (Context) | Compartir datos sin prop drilling | Datos transversales de cambio poco frecuente (tema, usuario, idioma) | [React Context](../04-hooks-y-context/04-React%20Context.md) |
| Reducer | Centralizar transiciones de estado complejas | Estado con muchas acciones relacionadas | [useReducer](../04-hooks-y-context/05-useReducer.md) |
| Controlled / Uncontrolled inputs | Quién es dueño del valor de un campo | Formularios | [React Forms](../06-forms/01-React%20Forms.md) |
| Error Boundary | Aislar fallos de renderizado | Proteger secciones de la interfaz | [Error Boundaries](../08-manejo-de-errores/01-React%20Error%20Boundaries.md) |
| Lazy loading y memoización | Cargar y renderizar solo lo necesario | Tras medir un problema real | [React Optimization](../09-performance/02-React%20Optimization.md) |
| Server Components | Traer datos en el servidor | Aplicaciones con Next.js App Router | [Next.js Server Components](../../02-nextjs-app-router/04-Next.js%20Server%20Components.md) |
| Organización por features / Atomic Design | Estructurar proyecto y UI | Al organizar carpetas y componentes | [Organización de carpetas](../13-arquitectura-frontend/01-Organizacion%20de%20Carpetas%20-%20Vertical%20Slice%20vs%20Horizontal.md) |

-----

## Resumen en 5 líneas

1. Separa lógica (estado, cálculos) de presentación (JSX); hoy suele bastar un custom hook en el mismo componente.
2. Los datos bajan por props y los cambios suben con funciones; las props no se mutan.
3. Los HOC y las render props reutilizan lógica, pero añaden anidamiento; para lógica nueva se prefiere un custom hook.
4. En React se compone en lugar de heredar: `children` y props con JSX crean slots, y Compound Components coordina partes con Context.
5. Ante prop drilling, primero extrae componentes y pasa `children`; usa Context si no alcanza.

-----

## Para profundizar

<details>
<summary>HOC: reglas prácticas</summary>

* Aplícalo fuera del render (ver error común 2).
* Pasa al componente envuelto las props que el HOC no consume (`{...props}`).
* Asigna `displayName` para que las DevTools muestren un nombre útil.
* Un HOC no debe mutar el componente original: lo envuelve.

</details>

<details>
<summary>Variante antigua de Compound Components</summary>

Antes de usar Context se inyectaban props a los hijos con `React.Children.map` y `cloneElement`. Solo funciona con hijos directos: si envuelves un hijo en otro componente, deja de recibir lo inyectado. La versión con Context no tiene esa limitación.

</details>

<details>
<summary>Herencia en componentes de clase</summary>

React sigue soportando componentes de clase (se crean extendiendo `React.Component`), pero la documentación actual recomienda definir componentes como funciones en código nuevo. Por eso, la herencia de clases casi no aparece en React moderno.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Los patrones de React organizan el código: se separa la lógica del JSX (contenedor y presentacional), se pasan datos por props y funciones para comunicar cambios, y se compone en lugar de heredar. Para reutilizar lógica antes se usaban HOC y render props; hoy se usan custom hooks.

### Respuesta ampliada (semi-senior)

* **Separación de responsabilidades:** el patrón container/presentational sigue vigente como idea; se implementa con un custom hook para la lógica y un componente que solo renderiza.
* **Reutilización de lógica:** los custom hooks comparten lógica sin añadir nodos al árbol. Los HOC y las render props funcionan, pero producen anidamiento (wrapper hell) y son más difíciles de tipar y depurar.
* **Reutilización de estructura:** composición con `children` y slots. La herencia de componentes no se recomienda por el acoplamiento que genera.
* **Prop drilling:** primero props explícitas, luego extraer componentes y pasar JSX como `children`, y por último Context. Para estado global de cambio frecuente, una librería de estado.
* **Compound Components:** varias partes que comparten estado vía Context y se componen con JSX (`Tabs.List`, `Tabs.Tab`). Dan libertad al consumidor a costa de un contrato implícito.
* **TypeScript:** los HOC se tipan con `P extends Inyectadas` y `Omit`; los compound components, con un contexto `T | undefined` y una guardia.

### Preguntas frecuentes de seguimiento

**1. ¿Sigue vigente container/presentational?**
Como principio, sí: separar lógica y presentación. Como estructura de dos componentes, ya no es obligatoria, porque un custom hook permite extraer la lógica sin crear un contenedor aparte.

**2. ¿HOC o custom hook?**
Para lógica nueva, custom hook: se llama dentro del componente, sin nodos extra ni colisión de props. Un HOC se justifica en código existente o cuando una librería lo expone.

**3. ¿Render props o hooks?**
Hooks en la mayoría de los casos. Las render props se mantienen cuando el componente debe entregar valores a un JSX que decide quien lo usa, o cuando una librería las ofrece.

**4. ¿Composición o herencia?**
Composición. La herencia acopla el componente derivado a la base y un cambio en esta puede romper a todos; con composición cada pieza se usa, prueba y cambia por separado.

**5. ¿Qué son los Compound Components y cuándo se usan?**
Un componente dividido en partes (`Tabs.List`, `Tabs.Panel`) que comparten estado con Context y se componen con JSX. Se usan en UI con varias partes coordinadas donde el consumidor necesita controlar la estructura.

**6. ¿Cómo evitas el prop drilling?**
Extrayendo componentes y pasando JSX como `children`, de modo que los intermedios no reciban el dato. Si sigue habiendo muchos niveles, Context; y para estado global complejo, una librería de estado.

-----

## Siguiente lección

Ahora que sabes organizar componentes, la siguiente lección trata cómo darles estilo: [React Styles](02-React%20Styles.md).
