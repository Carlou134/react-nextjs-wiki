# Métodos de ciclo de vida de componentes de clase

## En una frase

Los métodos de ciclo de vida son funciones que React llama en momentos concretos de la vida de un componente de **clase** (al montarse, al actualizarse, al desmontarse). Hoy los componentes nuevos se escriben con funciones y Hooks, pero debes conocerlos para leer código legado y para entrevistas.

-----

## Antes de empezar

Conviene conocer:

* Cómo funciona un efecto: [useEffect](../04-hooks-y-context/02-The%20Effect%20Hook.md).
* Qué es una clase de JavaScript (`class`, `extends`, `this`).

Términos (también en el [Glosario](Glosario.md)):

* **Ciclo de vida:** etapas por las que pasa una instancia de componente: montaje, actualización y desmontaje.
* **Montar:** crear el componente y colocarlo en el DOM por primera vez. **Desmontar:** eliminarlo del DOM.
* **Efecto secundario:** trabajo que no consiste en calcular el JSX (pedir datos, temporizadores, suscripciones).
* **Componente de clase:** clase que extiende `React.Component` y define un método `render()`.

-----

## El problema

Un componente no solo calcula JSX. A veces debe hacer algo **en un momento preciso**: iniciar un temporizador cuando aparece, reaccionar cuando cambia una prop, liberar recursos cuando desaparece.

`render()` no sirve para eso: debe ser puro y React puede llamarlo más de una vez. Tampoco el constructor, que no debe iniciar efectos secundarios. Los métodos de ciclo de vida existen para dar un lugar a cada momento.

Cada instancia tiene su propio ciclo de vida: tres botones son tres instancias independientes. Una vez desmontada, una instancia no vuelve a montarse; si el componente reaparece, React crea una nueva.

-----

## Cómo funciona

### Las tres fases

* **Montaje:** la instancia se crea y aparece por primera vez.
* **Actualización:** cambian las props o el estado (o se llama a `forceUpdate`) y React vuelve a renderizar.
* **Desmontaje:** la instancia se elimina del DOM.

### Los métodos principales

| Método | Fase | Qué se usa para |
| --- | --- | --- |
| `constructor(props)` | Montaje | Declarar el estado inicial y enlazar métodos. Sin efectos secundarios. Debe llamar a `super(props)` primero. |
| `render()` | Montaje y actualización | Devolver el JSX. Debe ser puro. |
| `componentDidMount()` | Montaje | Trabajo que necesita el componente ya en pantalla: pedir datos, suscribirse, iniciar temporizadores. |
| `componentDidUpdate(prevProps, prevState, snapshot)` | Actualización | Reaccionar a un cambio ya aplicado. No se llama en el primer render. |
| `componentWillUnmount()` | Desmontaje | Limpiar: cancelar pedidos, quitar suscripciones, detener temporizadores. |

Los métodos `render`, `constructor` y los de tipo "Did" no corresponden uno a uno con las fases: `constructor` solo corre al montar, pero `render` corre al montar y al actualizar.

### Diagrama del ciclo de vida

```
MONTAJE
  constructor
      |
      v
    render
      |
      v
  React actualiza el DOM
      |
      v
  componentDidMount
      |
      |  cambian props o estado
      v
ACTUALIZACIÓN  <----------------+
    render                      |
      |                         |
      v                         |
  React actualiza el DOM        |
      |                         |
      v                         |
  componentDidUpdate -----------+
      |
      |  el componente se elimina
      v
DESMONTAJE
  componentWillUnmount
```

En la fase de actualización también pueden intervenir `getDerivedStateFromProps` (antes de `render`), `shouldComponentUpdate` (antes de `render`) y `getSnapshotBeforeUpdate` (justo antes de escribir en el DOM). Se usan poco.

### Métodos de error

Dos métodos permiten que una clase capture errores de renderizado de sus **hijos**:

* `static getDerivedStateFromError(error)`: se llama tras el error y devuelve el nuevo estado para mostrar una interfaz alternativa. Debe ser puro.
* `componentDidCatch(error, info)`: se usa para registrar el error (por ejemplo, en un servicio de monitoreo).

Un componente de clase que define alguno de los dos es un **error boundary**. Se explica en [Error Boundaries](../08-manejo-de-errores/01-React%20Error%20Boundaries.md).

-----

## Ejemplo completo

Un reloj que se actualiza cada segundo. Primero en clase:

```jsx
import { Component } from 'react';

class Clock extends Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  componentDidMount() {
    this.intervalId = setInterval(() => {
      this.setState({ date: new Date() });
    }, 1000);
  }

  componentWillUnmount() {
    clearInterval(this.intervalId);
  }

  render() {
    return <p>{this.state.date.toLocaleTimeString()}</p>;
  }
}
```

El mismo componente con Hooks:

```jsx
import { useState, useEffect } from 'react';

function Clock() {
  const [date, setDate] = useState(new Date());

  useEffect(() => {
    const intervalId = setInterval(() => {
      setDate(new Date());
    }, 1000);

    return () => clearInterval(intervalId);
  }, []);

  return <p>{date.toLocaleTimeString()}</p>;
}
```

### Equivalencias aproximadas

| Clase | Hooks |
| --- | --- |
| `constructor` + `this.state` | `useState` |
| `componentDidMount` | `useEffect(() => { ... }, [])` |
| `componentDidUpdate` | `useEffect(() => { ... }, [dep])` |
| `componentWillUnmount` | función de limpieza que devuelve el efecto |
| `render` | el cuerpo de la función |

### Por qué no es 1:1

Los métodos de clase se organizan por **momento** ("al montar", "al actualizar", "al desmontar"). `useEffect` se organiza por **sincronización**: describe qué debe mantenerse en sincronía con qué valores.

* Un efecto con dependencias corre tras el montaje **y** tras cada cambio de esas dependencias. Un solo efecto cubre lo que en clase requería `componentDidMount` y `componentDidUpdate` juntos.
* La limpieza no corre solo al desmontar: también corre antes de cada re-ejecución del efecto. `componentWillUnmount` corre una sola vez.
* `componentDidUpdate` recibe `prevProps` y `prevState` y decides tú qué comparar. En un efecto, React compara las dependencias por ti.
* Los efectos capturan los valores del render en que se crearon. `this.props` y `this.state` en una clase siempre apuntan a los valores actuales.
* Algunos métodos no tienen equivalente en funciones, como `getSnapshotBeforeUpdate` y los métodos de error boundary.

Por eso la guía oficial recomienda pensar en "qué sincronizar" y no en "en qué fase estoy". Traducir mecánicamente cada método a un `useEffect` suele producir código confuso.

-----

## Errores comunes

### 1. `setState` en `componentDidUpdate` sin condición

```jsx
componentDidUpdate() {
  this.setState({ count: this.state.count + 1 });   // Mal
}
```

**Qué pasa:** bucle infinito de renderizados.
**Por qué:** `setState` provoca una actualización y toda actualización vuelve a llamar a `componentDidUpdate`.
**Cómo se arregla:** compara con los valores previos y actualiza solo si cambió lo relevante.

```jsx
componentDidUpdate(prevProps) {
  if (this.props.userId !== prevProps.userId) {
    this.loadUser(this.props.userId);   // Bien
  }
}
```

### 2. Olvidar limpiar suscripciones y temporizadores

```jsx
componentDidMount() {
  setInterval(() => this.setState({ date: new Date() }), 1000);
  // Mal: nadie lo detiene
}
```

**Qué pasa:** el intervalo sigue activo tras desmontar el componente y sigue intentando actualizar un estado que ya no se usa. Además consume recursos.
**Por qué:** React no sabe qué recursos creaste; solo tú puedes liberarlos.
**Cómo se arregla:** guarda el identificador y cancela en `componentWillUnmount` (como en el ejemplo completo). La lógica de `componentWillUnmount` debe reflejar la de `componentDidMount`.

### 3. `this` sin enlazar

```jsx
class Counter extends Component {
  state = { count: 0 };

  handleClick() {
    this.setState({ count: this.state.count + 1 });   // this es undefined
  }

  render() {
    return <button onClick={this.handleClick}>Sumar</button>;
  }
}
```

**Qué pasa:** al hacer clic, `this` es `undefined` y lanza un error.
**Por qué:** un método de clase pasado como callback pierde su `this`; React lo invoca sin objeto asociado.
**Cómo se arregla:** enlázalo en el constructor con `this.handleClick = this.handleClick.bind(this)`, o define el método como propiedad con función flecha:

```jsx
handleClick = () => {
  this.setState({ count: this.state.count + 1 });
};
```

Con funciones y Hooks este problema no existe: no hay `this`.

-----

## En TypeScript

Se tipan las props y el estado como parámetros genéricos de `Component<Props, State>`:

```tsx
import { Component } from 'react';

type ClockProps = { label: string };
type ClockState = { date: Date };

class Clock extends Component<ClockProps, ClockState> {
  state: ClockState = { date: new Date() };
  private intervalId?: ReturnType<typeof setInterval>;

  componentDidMount() {
    this.intervalId = setInterval(() => {
      this.setState({ date: new Date() });
    }, 1000);
  }

  componentDidUpdate(prevProps: ClockProps) {
    if (prevProps.label !== this.props.label) {
      // reaccionar al cambio de la prop
    }
  }

  componentWillUnmount() {
    clearInterval(this.intervalId);
  }

  render() {
    return <p>{this.props.label}: {this.state.date.toLocaleTimeString()}</p>;
  }
}
```

Si el componente no tiene estado, se omite el segundo parámetro: `Component<Props>`.

-----

## Cuándo sí y cuándo no

**Para código nuevo:** usa componentes de función con Hooks. La documentación oficial desaconseja escribir clases y las considera el camino antiguo.

**Aún verás o usarás clases en dos casos:**

* **Código legado:** proyectos anteriores a los Hooks (React 16.8, 2019). React sigue soportando las clases, y no hace falta migrarlas por obligación.
* **Error boundaries:** siguen requiriendo una clase con `getDerivedStateFromError` o `componentDidCatch`; no existe un equivalente en funciones. En la práctica muchos proyectos usan la librería `react-error-boundary` para no escribir la clase a mano.

**Al migrar** una clase a Hooks, no traduzcas método por método. Identifica qué se sincroniza con qué valores y escribe un efecto por cada proceso.

-----

## Resumen en 5 líneas

1. El ciclo de vida tiene tres fases: montaje, actualización y desmontaje.
2. `constructor` inicializa, `render` calcula el JSX, `componentDidMount` inicia efectos, `componentDidUpdate` reacciona a cambios y `componentWillUnmount` limpia.
3. En `componentDidUpdate`, compara siempre con los valores previos antes de llamar a `setState`.
4. `useEffect` cubre `componentDidMount`, `componentDidUpdate` y `componentWillUnmount`, pero se organiza por sincronización, no por momento: no es equivalencia 1:1.
5. Las clases persisten en código legado y en error boundaries; el código nuevo usa funciones.

-----

## Para profundizar

<details>
<summary>Métodos legacy con prefijo UNSAFE_</summary>

Los métodos `componentWillMount`, `componentWillReceiveProps` y `componentWillUpdate` se consideran legacy. Hoy solo funcionan con el prefijo: `UNSAFE_componentWillMount`, `UNSAFE_componentWillReceiveProps` y `UNSAFE_componentWillUpdate`.

Se llaman **antes** del renderizado. Con el renderizado concurrente y Suspense, un render puede iniciarse y descartarse, así que estos métodos pueden ejecutarse sin que el componente llegue a montarse o actualizarse. Por eso no son un lugar seguro para efectos secundarios.

La documentación recomienda migrar así:

* `UNSAFE_componentWillMount`: mover la lógica al constructor o a `componentDidMount`.
* `UNSAFE_componentWillReceiveProps` y `UNSAFE_componentWillUpdate`: mover la lógica a `componentDidUpdate`.

</details>

<details>
<summary>StrictMode y los métodos de ciclo de vida</summary>

En desarrollo, `<StrictMode>` ejecuta dos veces algunas funciones que deben ser puras (como `constructor`, `render` y `shouldComponentUpdate`) para revelar código impuro. También ejecuta un ciclo extra al montar: `componentDidMount`, luego `componentWillUnmount` y otra vez `componentDidMount`. Así comprueba que la limpieza refleja la configuración.

Es el mismo mecanismo que ocurre con los efectos en componentes de función. No ocurre en producción.

</details>

-----

## En entrevista

### Respuesta corta (junior)

Los métodos de ciclo de vida son funciones que React llama en las fases de un componente de clase: montaje, actualización y desmontaje. Los principales son `constructor`, `render`, `componentDidMount`, `componentDidUpdate` y `componentWillUnmount`. En componentes de función se reemplazan con `useEffect` y su función de limpieza.

### Respuesta ampliada (semi-senior)

* **Fases:** montaje (`constructor`, `render`, `componentDidMount`), actualización (`render`, `componentDidUpdate`) y desmontaje (`componentWillUnmount`). `render` corre en montaje y actualización.
* **Efectos secundarios:** van en `componentDidMount` y `componentDidUpdate`, no en `render` ni en el constructor. Cada suscripción creada se cancela en `componentWillUnmount`.
* **Hooks:** `useEffect` reemplaza a los tres métodos, pero no es una traducción 1:1: se organiza por sincronización con dependencias, y su limpieza corre también antes de cada re-ejecución.
* **Diferencias de modelo:** en clases `this.props` y `this.state` son siempre los valores actuales; un efecto captura los del render donde se creó.
* **Sin equivalente en funciones:** `getSnapshotBeforeUpdate` y los métodos de error boundary.
* **Vigencia:** las clases siguen soportadas, pero las guías oficiales recomiendan funciones para código nuevo.

### Preguntas frecuentes de seguimiento

**1. ¿Cuáles son las fases del ciclo de vida?**
Montaje, actualización y desmontaje. En montaje se llaman `constructor`, `render` y `componentDidMount`; en actualización `render` y `componentDidUpdate`; en desmontaje `componentWillUnmount`.

**2. ¿Qué equivalentes tienen con Hooks?**
`componentDidMount` se aproxima a `useEffect(fn, [])`, `componentDidUpdate` a `useEffect(fn, [deps])` y `componentWillUnmount` a la función de limpieza. El estado de la clase corresponde a `useState`.

**3. ¿Por qué los efectos no son 1:1 con los métodos?**
Porque `useEffect` sincroniza con valores en lugar de reaccionar a fases. Corre tras el montaje y tras cada cambio de dependencias, y su limpieza corre antes de cada re-ejecución, no solo al desmontar.

**4. ¿Por qué migrar a Hooks?**
Se reutiliza lógica con Hooks propios sin envolturas, la lógica relacionada queda junta en vez de repartida entre métodos, y desaparecen `this` y el enlazado. La documentación oficial además recomienda funciones para código nuevo.

**5. ¿Dónde se hace el fetch en una clase?**
En `componentDidMount` para la carga inicial y en `componentDidUpdate`, comparando con los valores previos, para repetirlo cuando cambia una prop. Se cancela o ignora la respuesta en `componentWillUnmount`. Nunca en `render` ni en el constructor.

**6. ¿Qué son los error boundaries y por qué siguen siendo de clase?**
Son componentes que capturan errores de renderizado de sus hijos y muestran una interfaz alternativa. Se crean con `getDerivedStateFromError` y `componentDidCatch`, que no tienen equivalente en Hooks, así que requieren una clase.

-----

## Siguiente lección

Pasamos a cómo capturar datos del usuario con formularios controlados: [React Forms](../06-forms/01-React%20Forms.md).
