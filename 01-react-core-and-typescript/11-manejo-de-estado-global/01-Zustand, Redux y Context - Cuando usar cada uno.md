# Zustand, Redux y Context: Cuándo Usar Cada Uno

## Why Global State?

En lecciones anteriores vimos cómo evitar el *prop drilling* (pasar una prop a través de varios componentes intermedios que no la usan) usando la Context API. Eso funciona muy bien cuando el dato que queremos compartir es relativamente simple y no cambia con demasiada frecuencia: el tema visual de la aplicación (claro u oscuro), el idioma seleccionado, o el usuario autenticado.

Pero a medida que una aplicación crece, aparecen necesidades más exigentes: un carrito de compras con múltiples operaciones, datos que deben persistir entre recargas de página, lógica asíncrona compleja que involucra múltiples llamadas a APIs, o un estado que cambia con tanta frecuencia que la Context API empieza a mostrar sus limitaciones de rendimiento. Para estos casos existen librerías dedicadas a la gestión de **estado global**, y las dos más relevantes hoy en el ecosistema de React son **Zustand** y **Redux**.

Entender cuándo usar cada una (Context, Zustand o Redux) no es una cuestión de moda, sino de elegir la herramienta cuyo costo de complejidad sea proporcional al problema que estamos resolviendo.

-----

## The Problem with Context API at Scale

La Context API resuelve el problema de compartir datos, pero tiene una particularidad importante: **vive dentro del árbol de componentes de React**. Cuando el valor de un contexto cambia, **todos** los componentes que consumen ese contexto (con `useContext()`) se vuelven a renderizar, sin importar si realmente necesitan la parte del valor que cambió. Si el valor del contexto es un objeto con muchas propiedades y solo una de ellas cambió, igual se re-renderizan todos los consumidores.

Para un contexto de tema visual que cambia rara vez, esto no representa ningún problema. Pero para un estado que cambia constantemente, como el contenido de un carrito de compras en una tienda con muchas interacciones, ese patrón de re-renderizados puede empezar a afectar el rendimiento de la aplicación.

-----

## Zustand: Simplicidad Fuera del Árbol de React

**Zustand** es una librería de gestión de estado diseñada explícitamente para resolver este problema de una forma minimalista. A diferencia de la Context API, el estado de Zustand vive **fuera** del árbol de componentes de React, en un *store* independiente. Esto significa que los componentes se suscriben únicamente a las porciones del estado que efectivamente usan, y solo se re-renderizan cuando esas porciones específicas cambian.

Crear un store en Zustand no requiere providers, reducers ni acciones tipadas como constantes: es, literalmente, una función:

```tsx
import { create } from 'zustand';

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));
```

Y usarlo desde cualquier componente, sin necesidad de envolver la aplicación en un provider, es igual de directo:

```tsx
function Counter() {
  const count = useCounterStore((state) => state.count);
  const increment = useCounterStore((state) => state.increment);

  return <button onClick={increment}>Contador: {count}</button>;
}
```

Nota algo importante en la línea `useCounterStore((state) => state.count)`: le estamos pasando al hook una función **selectora** que extrae únicamente la porción del estado que este componente necesita (`count`). Si el store tuviera otras propiedades que cambiaran, este componente no se volvería a renderizar por esos cambios, porque no está suscrito a ellos. Esta es la razón principal por la que Zustand suele tener mejor rendimiento que la Context API en estados que cambian con frecuencia: el re-renderizado es **selectivo**, no global.

Además de su rendimiento, Zustand es atractiva por otras razones prácticas: pesa apenas un par de kilobytes, no exige boilerplate (no hay que definir tipos de acción, reducers ni providers) y se integra de forma natural con TypeScript, infiriendo los tipos del estado automáticamente a partir de cómo se define el store.

-----

## Redux: Estructura y Control para Aplicaciones Grandes

**Redux** sigue siendo una opción sólida, especialmente en aplicaciones de gran escala con lógica de negocio compleja. Su modelo se basa en tres piezas: un **store** único e inmutable, **acciones** que describen qué ocurrió, y **reducers**, funciones puras que calculan el nuevo estado a partir del estado anterior y la acción recibida.

Esta estructura tiene un costo de verbosidad más alto que Zustand: hay que definir tipos de acción, escribir reducers, y despachar (`dispatch`) acciones en lugar de llamar funciones directamente. A cambio, Redux ofrece un modelo muy predecible y explícito, con herramientas de depuración maduras (como Redux DevTools, que permite viajar en el tiempo entre estados anteriores) y un ecosistema pensado para coordinar lógica asíncrona compleja, con muchas llamadas a APIs interdependientes y múltiples módulos de estado que interactúan entre sí.

-----

## Choosing the Right Tool

Con estas tres herramientas sobre la mesa, la pregunta práctica es: ¿cuál uso en cada situación? Una guía razonable es la siguiente:

* **Context API**: cuando el estado es simple y cambia con poca frecuencia. Ejemplos típicos son el tema visual, el idioma o los datos del usuario autenticado.
* **Zustand**: cuando necesitas un estado global que cambia con más frecuencia, en una aplicación de tamaño pequeño a mediano, y quieres evitar el boilerplate de Redux sin sacrificar rendimiento.
* **Redux**: cuando la aplicación es grande, con múltiples módulos de estado conectados entre sí, mucha lógica asíncrona, y necesitas la estructura y las herramientas de depuración que ofrece su ecosistema.

Ninguna de las tres opciones es "la correcta" de forma universal: la decisión depende del tamaño del problema. Usar Redux para un simple interruptor de tema visual sería sobre-ingeniería; usar la Context API para el estado de una aplicación de comercio electrónico con carritos, filtros e inventario en tiempo real probablemente termine generando problemas de rendimiento.

-----

## Persisting State: Keeping Data After a Reload

Un problema común con el estado global, sin importar la librería que uses, es que por defecto vive únicamente en memoria: si el usuario recarga la página, se pierde. Para un carrito de compras, un tema visual seleccionado o una sesión de usuario, esto suele ser inaceptable.

Zustand resuelve esto con un **middleware** llamado `persist`, que envuelve la definición del store y se encarga de guardar (y recuperar) el estado automáticamente usando `localStorage`:

```tsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

const useUserStore = create(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
    }),
    {
      name: 'app-storage',
    }
  )
);
```

Con esta configuración, cada vez que `user` cambia, Zustand serializa el store completo y lo guarda bajo la clave `'app-storage'` en `localStorage`. Cuando la aplicación vuelve a cargar, Zustand lee ese valor guardado y **rehidrata** el store con él antes de que el usuario note la diferencia. Este proceso de leer los datos guardados y reconstruir el estado a partir de ellos se conoce como **hidratación** (*hydration*).

No siempre conviene persistir el store completo. Si el store mezcla datos que deben sobrevivir a una recarga (como el usuario autenticado) con datos que no tiene sentido persistir (como un estado de `isLoading` temporal), podemos usar la opción `partialize` para elegir explícitamente qué guardar:

```tsx
persist(
  (set) => ({ /* ... */ }),
  {
    name: 'app-storage',
    partialize: (state) => ({ user: state.user }),
  }
)
```

Esto reduce el tamaño de lo que se guarda y evita persistir datos que de todas formas deberían reiniciarse en cada carga de la aplicación.

-----

## Versioning and Migrating Persisted State

Persistir el estado trae un riesgo a mediano plazo: la estructura de tus datos va a cambiar con el tiempo, pero los datos que ya están guardados en el `localStorage` de tus usuarios seguirán teniendo la estructura vieja. Imaginemos que en una primera versión de la aplicación guardábamos el usuario así:

```javascript
user: "Carlos"
```

Y en una versión posterior decidimos que necesitamos más información, así que cambiamos la forma del dato:

```javascript
user: { name: "Carlos" }
```

Si un usuario que ya tenía datos guardados con la estructura vieja abre la nueva versión de la aplicación, el código que espera `user.name` va a fallar, porque en su `localStorage` todavía existe la cadena `"Carlos"`, no un objeto.

El middleware `persist` de Zustand resuelve este problema con dos opciones: `version` y `migrate`.

```tsx
persist(
  (set) => ({ user: null }),
  {
    name: 'app-storage',
    version: 2,
    migrate: (persistedState, version) => {
      if (version === 1) {
        return {
          ...persistedState,
          user: { name: persistedState.user },
        };
      }
      return persistedState;
    },
  }
)
```

Cada vez que cambiamos la estructura del estado que persistimos, incrementamos el número de `version`. Cuando Zustand detecta que el número de versión guardado en `localStorage` es menor al número de versión actual del código, ejecuta la función `migrate`, pasándole el estado viejo y el número de versión con el que fue guardado. Dentro de esa función podemos transformar los datos viejos a la nueva estructura antes de que el resto de la aplicación los use.

Adoptar esta disciplina (definir siempre una `version` y una función `migrate` desde el principio) evita que un cambio en la forma de tus datos rompa la aplicación para los usuarios que ya tenían estado guardado, y es una práctica que conviene incorporar en cualquier store que persista información entre sesiones.
