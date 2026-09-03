# React Programming Patterns

## Separate Container Components From Presentational Components

A medida que continúes creando tus aplicaciones en React, pronto te darás cuenta de que un componente tiene demasiadas responsabilidades y es difícil de mantener. En esta lección, aprenderás un patrón de programación que te ayudará a organizar tu código en React.

Si un componente necesita tener **estado**, hacer cálculos basados en las **props** o manejar cualquier otra lógica compleja, entonces ese componente no debería también encargarse de renderizar **JSX**.

Para ayudar a reducir la complejidad del componente, podemos dividirlo en varios componentes más simples. ¿Cómo deberías separarlo?

El patrón que aprenderemos se enfoca en dividir componentes complejos en componentes **con estado (contenedores)** y **sin estado (presentacionales)**, donde los componentes con estado gestionan el estado o la lógica compleja y los componentes sin estado solo renderizan JSX.

A lo largo de esta lección, veremos cómo aplicar este patrón a nuestra aplicación de ejemplo en React para dividir un componente complejo en componentes contenedores y presentacionales.

-----

## Create Container Component

Separar los componentes contenedores de los componentes de presentación es un patrón de programación popular en React.

La parte funcional de un componente (mantener un estado, hacer cálculos basados en props, etc.) puede separarse en un componente contenedor, también llamado componente con estado.

Este componente contenedor se encargará de mantener el estado (crearlo y actualizarlo) y pasarlo (veremos esto más adelante) a cualquier componente que renderice, usando props.

-----

## Create Presentational Component

Ahora que hemos creado un componente contenedor y separado la lógica, podemos crear un componente de presentación (o sin estado) para mostrar nuestro slideshow de cobayas.

El único trabajo del componente de presentación es contener JSX. Debe ser un componente exportado y no debe renderizarse a sí mismo, porque siempre será renderizado por un componente contenedor.

Por ejemplo, si tenemos componentes llamados Presentational y Container, Presentational.js debe exportar la función (o clase) del componente:

```js
function Presentational(/*...props*/) {
  // cuerpo del componente
}

export default Presentational;
```

Container.js debe importar ese componente:

```js
import { Presentational } from 'Presentational.js';
function Container() {
  // renderiza el componente Presentational
}
```

Es importante entender que, aunque un componente de presentación no mantiene estado, eso no significa que no sea reactivo. Recuerda que, al igual que el estado, un cambio en las props también puede cambiar el JSX renderizado.

----

## Parent/Child and Sibling/Sibling Communication

Hemos visto cómo los componentes contenedores se comunican con los componentes de presentación pasando su estado a través de las props, pero ¿cómo comunican los componentes de presentación cambios al contenedor?

Una idea sería actualizar las props directamente así:

```js
function Presentational(props) {
  const buttonClickHandler = () => {
    props.isActive = !props.isActive
  }
  // resto del código
}
```

Pero esto no sería correcto porque los componentes nunca deben actualizar sus props directamente. Recuerda que los componentes funcionales de React deben ser funciones puras y actualizar los valores de las props directamente violaría ese principio.

Para que un componente de presentación (sin estado) comunique cambios a un contenedor (con estado), el componente contenedor debe definir y proporcionar una forma para que el componente de presentación se comunique con él usando una función manejadora de cambios pasada como prop.

Por ejemplo:

```js
function Container() {
  const [isActive, setIsActive] = useState(false);                              
                                
  return (
    <>
      <Presentational active={isActive} toggle={setIsActive}/>
      <OtherPresentational active={isActive}/>
    </>
  );                          
}
                        
function Presentational(props) {
  return (
    <h1>Engines are {props.active}</h1>
    <button onClick={() => props.toggle(!props.active)}>Engine Toggle</button>
  );
}
                            
function OtherPresentational(props) {
  // render...
}
```

En el ejemplo anterior, Container mantiene el estado isActive y pasa setIsActive a Presentational a través de la prop toggle. Cuando Presentational necesita comunicar un cambio a la prop active, usa la función pasada a través de la prop toggle.

Usar este patrón también resulta, de forma indirecta, en comunicación entre componentes hermanos (componentes con un mismo padre), como se muestra en el ejemplo. Cuando Presentational comunica un cambio usando toggle, esto actualiza el estado en Container, que luego proporciona el valor actualizado de isActive tanto a Presentational como a OtherPresentational a través de la prop active.

----

## Render Presentational Components in Container Component

Hemos aprendido cómo separar la lógica en un componente contenedor y renderizar **JSX** en un componente presentacional.

Ahora, el componente contenedor debería renderizar los componentes presentacionales en lugar de renderizar JSX directamente. El estado del componente contenedor se pasará hacia abajo como **props** a los componentes presentacionales para mantenerlos reactivos.

-----

## Review

¡Felicidades! Has aprendido tu primer patrón de programación para organizar tu código React. Dividiste un componente complejo de React en un componente contenedor y un par de componentes de presentación.

Estos son los pasos que seguimos:

- Identificamos que el componente original necesitaba ser refactorizado: manejaba cálculos/lógica y presentación/renderizado.
- Creamos un componente contenedor con toda la lógica con estado.
- Creamos una función que llama al método para actualizar el estado proporcionado por useState().
- Creamos y exportamos componentes de presentación que solo contienen JSX.
- Importamos los componentes de presentación en el componente contenedor.
- Usamos los componentes de presentación en el return del componente contenedor.
- Pasamos el estado y las funciones para cambiar el estado como props a los componentes de presentación.

En este patrón, el componente contenedor decide qué mostrar usando el estado. El componente de presentación muestra el estado usando props. Si un componente hace mucho trabajo en ambas áreas, es señal de que debes usar este patrón.

-----

## Higher-Order Components (HOC)

Un **Higher-Order Component** (componente de orden superior) es una función que recibe un componente como argumento y devuelve un nuevo componente con funcionalidad adicional. No es una característica del framework, sino un patrón que surge naturalmente de que los componentes de React son, en última instancia, funciones: si una función puede recibir y devolver otra función, un componente puede recibir y devolver otro componente.

Este patrón fue, durante mucho tiempo, la forma estándar de compartir lógica entre componentes que no tenían relación jerárquica directa. Un caso típico es proteger rutas según el estado de autenticación:

```tsx
const withAuth = (Component) => {
  return (props) => {
    const isAuth = true; // vendría de un contexto o de un hook real
    return isAuth ? <Component {...props} /> : <Login />;
  };
};

const DashboardConAuth = withAuth(Dashboard);
```

`withAuth` no modifica `Dashboard`; lo envuelve. El componente devuelto decide, antes de renderizar, si `Dashboard` debe mostrarse o si en su lugar debe mostrarse `Login`. La idea central del patrón es "añado lógica sin tocar el componente original", lo que permite reutilizar la misma protección de autenticación envolviendo cualquier otro componente con `withAuth`.

### Por qué los Hooks desplazaron a los HOC

Envolver componentes tiene un costo. Cuando varios HOC se combinan sobre el mismo componente (`withAuth(withTheme(withData(Componente)))`), el árbol de componentes se llena de capas intermedias que no aportan JSX propio, solo lógica. A este problema se lo conoce como **wrapper hell**: cada envoltorio agrega una capa en las React DevTools, dificulta rastrear de dónde viene una prop y puede generar colisiones de nombres cuando dos HOC intentan inyectar una prop con el mismo nombre.

Con la llegada de los Hooks, la mayoría de los casos de uso de los HOC —compartir lógica de estado, efectos o suscripciones— se resuelven con un **custom hook**, sin necesidad de envolver ningún componente. Un custom hook se importa y se llama dentro del componente que lo necesita, sin crear nodos adicionales en el árbol ni indirecciones sobre las props. Por eso, la recomendación actual es: si el problema se puede resolver con un hook, se resuelve con un hook. Los HOC siguen siendo válidos, pero quedan reservados para casos puntuales, como ciertas integraciones con librerías de terceros que ya exponen su API en forma de HOC.

### Yendo más lejos: HOCs tipados y componibles

El ejemplo de `withAuth` de arriba es deliberadamente simple. En un HOC real conviene tipar con cuidado dos cosas: qué prop inyecta el HOC, y que esa prop deje de ser exigida a quien use el componente resultante (ya que el HOC se encarga de proveerla). Esto se logra combinando un generic con `Omit`:

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

Leyendo la firma con calma: `withTheme` recibe el tema y devuelve una función que toma un `Component` cuyas props incluyen `WithThemeProps` (`P extends WithThemeProps`), y devuelve un nuevo componente cuyas props son las de `P` **sin** `theme` (`Omit<P, keyof WithThemeProps>`) — porque `theme` ahora lo provee el HOC, no quien lo usa. `displayName` no es obligatorio, pero vale la pena: sin él, todos los componentes envueltos aparecen en React DevTools con el nombre genérico `Wrapped`, lo que hace mucho más difícil identificar cuál es cuál cuando se combinan varios HOCs.

Y varios HOCs tipados así se combinan sin perder seguridad de tipos en ningún paso:

```tsx
const EnhancedDashboard = withLogging(withUser(withTheme('dark')(BaseDashboard)));

// Quien usa EnhancedDashboard solo necesita pasar las props que
// ningún HOC de la cadena provee — en este caso, solo `title`.
<EnhancedDashboard title="Panel principal" />
```

Cada HOC de la cadena "consume" su prop (inyectándola) y la resta del tipo final, así que TypeScript sabe con precisión qué props le siguen faltando a `EnhancedDashboard` después de aplicar los tres envoltorios.

-----

## Render Props

**Render Props** es un patrón en el que un componente recibe, como prop, una función que le indica qué debe renderizar. En lugar de que el componente decida directamente el JSX final, delega esa decisión al consumidor a través de esa función, mientras conserva para sí la lógica que produce los datos que la función necesita.

```tsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url).then((res) => res.json()).then(setData);
  }, [url]);

  return render(data);
}

<DataFetcher url="/api/users" render={(data) => <UserList users={data} />} />
```

`DataFetcher` se encarga de obtener los datos; quien lo usa decide cómo se muestran, pasando una función distinta según el caso: una tabla, una lista, tarjetas, etc. Es habitual encontrar una variante donde la función se pasa como `children` en lugar de como una prop llamada `render`:

```tsx
<DataFetcher url="/api/users">
  {(data) => <UserList users={data} />}
</DataFetcher>
```

Ambas variantes resuelven el mismo problema: compartir lógica entre componentes que necesitan mostrar interfaces distintas.

Al igual que con los HOC, los custom hooks han reemplazado a Render Props en la mayoría de los casos: un `useFetch(url)` devuelve los datos directamente, sin necesidad de envolver el JSX en una función ni de agregar un nivel de anidamiento adicional. Render Props sigue siendo útil cuando no se controla el componente que consume la lógica —por ejemplo, en librerías que exponen su comportamiento mediante esta API— o cuando es necesario inyectar comportamiento en un árbol de JSX que no se puede reescribir como hook.

-----

## Composición vs. Herencia

En la Programación Orientada a Objetos clásica es común reutilizar comportamiento mediante **herencia**: una clase extiende a otra y hereda sus propiedades y métodos, formando jerarquías como `Animal → Perro → Bulldog`. React, en cambio, no recomienda modelar componentes de esta forma. Su documentación es explícita al respecto: en Facebook usaron React en miles de componentes y no encontraron ningún caso de uso donde recomendarían crear jerarquías de herencia de componentes.

El problema de la herencia aplicada a UI es que crea acoplamiento rígido: un cambio en la clase base puede romper a todas sus clases hijas, y a medida que la jerarquía crece se vuelve difícil predecir el efecto de una modificación. Un componente `BotonBase` del que heredan `BotonPrimario`, `BotonConIcono` y `BotonDeAlerta` obliga a que cualquier ajuste en `BotonBase` se propague —a veces de forma inesperada— a los tres.

React resuelve el mismo problema con **composición**: en lugar de extender un componente base, se construyen componentes pequeños y con una única responsabilidad, y se combinan para formar interfaces más complejas.

```tsx
// Con herencia (no recomendado en React)
class BotonBase extends React.Component { /* ... */ }
class BotonConIcono extends BotonBase { /* ... */ }

// Con composición (idiomático en React)
function Boton({ children }) {
  return <button className="boton">{children}</button>;
}

function BotonConIcono({ icono, children }) {
  return (
    <Boton>
      {icono}
      {children}
    </Boton>
  );
}
```

En la versión compuesta, `BotonConIcono` no hereda de `Boton`: lo usa. Cada componente puede evolucionar, testearse y reutilizarse de forma independiente, y agregar un nuevo tipo de botón no implica tocar una jerarquía compartida, sino escribir un componente nuevo que combina las piezas que ya existen.

-----

## Slots y children

La prop `children` es la forma que tiene React de crear lo que en otros frameworks se conoce como **slots**: espacios dentro de un componente donde el elemento que lo usa decide qué contenido colocar, sin que el componente que define el espacio necesite conocer ese contenido de antemano.

```tsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card>
  <h1>Título</h1>
  <p>Contenido</p>
</Card>
```

`Card` no sabe —ni le interesa saber— qué hay dentro de él. Su única responsabilidad es aplicar el contenedor (la clase `card`) y renderizar lo que reciba como `children`. Quien usa `Card` decide qué va dentro, del mismo modo en que se decide qué colocar dentro de una caja genérica.

Cuando un componente necesita más de un espacio de inserción —por ejemplo, una cabecera y un pie separados del contenido principal— `children` deja de ser suficiente, porque solo representa un único hueco. En esos casos se recurre a varias props, cada una destinada a recibir JSX, funcionando como slots nombrados:

```tsx
function Layout({ header, children, footer }) {
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

Este enfoque conserva la misma idea que `children` —el padre decide el contenido— pero permite distribuirlo en distintas posiciones del componente.

-----

## Prop Drilling y cómo evitarlo

**Prop drilling** es el nombre que recibe el problema de pasar una prop a través de varios componentes intermedios que no la utilizan, únicamente para que llegue a un componente descendiente que sí la necesita.

```tsx
function App() {
  const user = useUser();
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />; // Layout no usa 'user', solo la reenvía
}

function Sidebar({ user }) {
  return <UserMenu user={user} />; // Sidebar tampoco la usa
}

function UserMenu({ user }) {
  return <span>{user.name}</span>; // recién aquí se usa 'user'
}
```

`Layout` y `Sidebar` reciben y reenvían `user` sin necesitarla para nada propio; su única función respecto a esa prop es actuar de intermediarios. Cuantos más niveles tenga el árbol y más props se compartan de esta manera, más frágil se vuelve el código: cada componente intermedio queda acoplado a datos que no le pertenecen, y renombrar o reestructurar una prop obliga a tocar cada eslabón de la cadena.

La solución que ofrece React para datos que numerosos componentes distantes necesitan leer es la **Context API**. Un contexto permite que un componente ancestro provea un valor, y que cualquier descendiente —sin importar cuán profundo esté— lo consuma directamente con el hook `useContext`, sin que los componentes intermedios sepan que ese valor existe:

```tsx
const UserContext = createContext(null);

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
  return <span>{user.name}</span>;
}
```

Con este cambio, `Layout` y `Sidebar` ya ni siquiera necesitan mencionar `user` en sus props. La regla práctica para decidir cuándo conviene usar Context en lugar de props es: si los componentes intermedios de la cadena no utilizan la prop, solo la reenvían, es una señal de que ese dato debería vivir en un contexto. Para estados globales más complejos, con muchas actualizaciones o lógica de por medio, suele preferirse una librería de manejo de estado dedicada, como Redux o Zustand, en lugar de forzar ese volumen de datos dentro de la Context API.

-----

