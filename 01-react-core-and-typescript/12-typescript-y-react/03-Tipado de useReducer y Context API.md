# Tipado de useReducer y Context API

## Why These Two Need Extra Care

`useReducer` y la Context API son, de las herramientas que vimos hasta ahora, las que manejan la lógica de estado más compleja: `useReducer` centraliza múltiples formas en las que el estado puede cambiar, y la Context API distribuye datos (y funciones para modificarlos) a través de todo un árbol de componentes. Precisamente por eso, son también los lugares donde un tipado flojo genera los errores más difíciles de rastrear: una acción mal escrita en un `dispatch`, o un valor de contexto usado antes de que exista, pueden fallar en un componente completamente distinto de donde está el error real.

Tipar correctamente estas dos herramientas consiste en aplicar la misma idea que ya venimos usando (declarar la forma exacta de los datos), pero llevada a dos estructuras nuevas: la forma del **estado y las acciones** de un reducer, y la forma del **valor** que expone un contexto.

-----

## Typing useReducer

Un reducer recibe el estado actual y una acción, y devuelve el nuevo estado. Para tiparlo correctamente, empezamos definiendo dos tipos: uno para la forma del estado, y otro para las acciones válidas.

```ts
type State = {
  count: number;
};

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset'; payload: number };
```

El tipo `State` es directo: describe que el estado de este reducer es un objeto con una propiedad `count` numérica. El tipo `Action` es más interesante: es una **unión de tipos** (usando el operador `|`), donde cada miembro de la unión representa una acción distinta que el reducer sabe manejar. Las dos primeras acciones, `increment` y `decrement`, no necesitan datos adicionales más allá de su `type`. La tercera, `reset`, sí necesita un dato adicional (`payload`, el valor al que queremos reiniciar el contador), y ese dato también queda tipado como parte de la acción.

Con estos dos tipos definidos, tipamos la función reducer y la llamada a `useReducer()`:

```ts
const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: action.payload };
    default:
      return state;
  }
};

const [state, dispatch] = useReducer(reducer, { count: 0 });
```

Fijate en algo particularmente útil dentro del `case 'reset'`: como `Action` es una unión discriminada por la propiedad `type`, en el momento en que TypeScript ve que `action.type === 'reset'`, automáticamente sabe que `action` tiene también una propiedad `payload` de tipo `number`, y te va a dar autocompletado para ella. Esto es exactamente lo que ganamos al tipar las acciones como una unión, en lugar de dejarlas como un `string` genérico.

El beneficio se nota apenas alguien intenta despachar una acción incorrecta:

```ts
dispatch({ type: 'incremment' }); // Error: 'incremment' no es un tipo de acción válido
```

Ese error tipográfico (`incremment` en lugar de `increment`) se detecta en el editor, antes de ejecutar la aplicación. Sin tipar `Action`, este mismo error pasaría desapercibido hasta que, en tiempo de ejecución, el `switch` cayera silenciosamente en el `default` y el estado nunca cambiara, dejándote con un bug mucho más difícil de rastrear.

-----

## Typing Context API

Cuando creamos un contexto con `createContext()`, también necesitamos declarar la forma del valor que ese contexto va a exponer a los componentes que lo consuman. Consideremos un contexto de tema visual:

```ts
type ThemeContextType = {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);
```

Dos decisiones de tipado merecen atención aquí:

* `theme: 'light' | 'dark'` no lo tipamos como un `string` genérico, sino como una unión de dos **literales de texto**. Esto es más restrictivo (y por eso más seguro): TypeScript solo va a aceptar exactamente los valores `'light'` o `'dark'` para esta propiedad, y va a rechazar cualquier otro texto, como `'Light'` o `'darkmode'`.
* El tipo del contexto en sí es `ThemeContextType | undefined`, y no simplemente `ThemeContextType`. Esto refleja una realidad del funcionamiento de la Context API: si un componente intenta consumir este contexto sin estar envuelto por su `Provider`, el valor que recibe es `undefined`. Tipar el contexto como `ThemeContextType | undefined` obliga a manejar ese caso explícitamente en lugar de asumir que el valor siempre va a estar disponible.

Para evitar tener que verificar `undefined` cada vez que consumimos el contexto, es una práctica común encapsular ese chequeo dentro de un **custom hook**:

```ts
const useTheme = () => {
  const context = useContext(ThemeContext);

  if (context === undefined) {
    throw new Error('useTheme debe usarse dentro de un ThemeProvider');
  }

  return context;
};
```

Con este hook, cualquier componente que llame a `useTheme()` recibe directamente un valor tipado como `ThemeContextType` (ya sin el `| undefined`), porque el `throw` dentro del hook garantiza que, si la ejecución llega hasta el `return`, el contexto necesariamente tiene un valor. Este patrón centraliza la verificación en un único lugar, en vez de repetirla en cada componente que consume el contexto.

-----

## The Common Thread

Tanto en `useReducer` como en la Context API, estamos aplicando la misma idea de fondo: **definir la estructura exacta del dato y limitar explícitamente qué operaciones son válidas sobre él**. En el caso del reducer, eso significa declarar qué acciones existen y qué forma tiene cada una. En el caso del contexto, significa declarar qué valor expone y contemplar el caso en el que ese valor todavía no existe. Este mismo principio (nombrar y restringir la forma de tus datos) es el hilo conductor de todo lo que venimos viendo sobre TypeScript en React, y se vuelve especialmente valioso a medida que la lógica de estado de una aplicación crece en complejidad.
