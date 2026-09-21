# Componentes que renderizan otros componentes

## En una frase

Un componente puede usar a otros dentro de su JSX, como si fueran etiquetas (`<Boton />`). Así una interfaz grande se arma con piezas pequeñas que forman un **árbol de componentes**.

-----

## Antes de empezar

Conviene que ya sepas:

* Qué es un componente de función y cómo devuelve JSX: [Tu primer componente](01-Your%20First%20React%20Component.md).

Palabras nuevas (también están en el [Glosario](Glosario.md)):

* **Componente padre:** el que usa a otro dentro de su JSX.
* **Componente hijo:** el que es usado dentro del JSX de otro.
* **Composición:** construir componentes complejos combinando componentes más simples.
* **Árbol de renderizado:** estructura que React construye al renderizar, con un nodo por cada componente y una relación padre-hijo entre ellos.
* **Componente raíz:** el componente de más arriba del árbol. Todos los demás cuelgan de él.

-----

## El problema

Una pantalla real tiene cabecera, listas, tarjetas, botones y formularios. Si todo vive en una sola función, ocurre esto:

1. El archivo crece y cuesta encontrar qué parte hace qué.
2. Si un bloque de interfaz se repite (por ejemplo, una tarjeta de producto), hay que copiarlo y mantener cada copia.
3. Un cambio pequeño obliga a leer y modificar una función enorme.

Se necesita una forma de **dividir** la interfaz en piezas con nombre, reutilizables y fáciles de leer. React lo resuelve permitiendo que un componente use a otros.

-----

## Cómo funciona

### Un componente devuelve a otros

Hasta ahora los componentes devolvían solo elementos HTML. Pero también pueden devolver otros componentes, escribiéndolos como etiquetas:

```jsx
function PurchaseButton() {
  return <button onClick={() => alert('Compra realizada')}>Comprar</button>;
}

function ItemBox() {
  return (
    <div>
      <h1>50% de descuento</h1>
      <h2>Artículo: camisa pequeña</h2>
      <PurchaseButton />
    </div>
  );
}
```

`ItemBox` es el **padre** y `PurchaseButton` es su **hijo**. Cada vez que escribes `<PurchaseButton />` en un JSX, estás pidiendo a React una **instancia** de ese componente. Puedes usarlo tantas veces como quieras, y cada instancia es independiente.

Con las etiquetas HTML pasa lo mismo, solo que las etiquetas HTML ya vienen definidas por el navegador. Un componente propio es una etiqueta nueva que defines tú.

### El árbol de componentes

Cuando los componentes se usan unos dentro de otros, se forma un árbol. Este es el de una tienda sencilla:

```
App                      <- raíz
├── Header
│   ├── Logo
│   └── Menu
└── ProductList
    ├── ProductCard
    │   └── PurchaseButton
    ├── ProductCard
    │   └── PurchaseButton
    └── ProductCard
        └── PurchaseButton
```

Cómo leerlo:

* `App` es la **raíz**. Nadie más la renderiza.
* `Header` y `ProductList` son hijos de `App` y hermanos entre sí.
* `Logo`, `Menu` y `PurchaseButton` son **hojas**: no renderizan ningún otro componente.
* `ProductCard` aparece tres veces: es un solo componente con tres instancias.

React construye este árbol mientras renderiza. Puede cambiar entre renders: si un componente se muestra según una condición, solo aparece en el árbol cuando la condición se cumple.

### El componente raíz

En la lección anterior exportabas un componente y lo mostrabas desde `App`. Ese es el mismo mecanismo: `App` es solo el componente de más arriba, y usa a los demás como hijos.

```jsx
import Button from './Button';

function App() {
  return <Button />;
}

export default App;
```

No hay nada especial en `App` más que su posición. La raíz suele estar en un archivo de entrada (por ejemplo `App.js` o `App.tsx`). En frameworks como Next.js la raíz cambia según la ruta.

### Orden de ejecución

Para saber qué dibujar, React necesita primero lo que devuelve el padre. Por eso:

1. React ejecuta la función del **padre** (`ItemBox`).
2. Lee el JSX que devolvió y encuentra `<PurchaseButton />`.
3. Ejecuta la función del **hijo** con las props que el padre le pasó.
4. Repite hacia abajo hasta llegar a las hojas, que solo devuelven elementos HTML.

En términos prácticos: el padre se ejecuta antes que sus hijos, y los datos fluyen de arriba hacia abajo. No dependas del orden exacto entre hermanos: cada componente debe producir su resultado solo a partir de sus props y su estado, sin apoyarse en que otro se haya ejecutado antes.

### Un componente por archivo

A medida que el proyecto crece, es habitual mover cada componente a su propio archivo y **exportarlo**, para **importarlo** donde se use:

```jsx
// PurchaseButton.jsx
export default function PurchaseButton() {
  return <button>Comprar</button>;
}
```

```jsx
// ItemBox.jsx
import PurchaseButton from './PurchaseButton';

export default function ItemBox() {
  return (
    <div>
      <h1>50% de descuento</h1>
      <PurchaseButton />
    </div>
  );
}
```

Reglas de los módulos de JavaScript:

* Un archivo puede tener **un solo** `export default` y cualquier cantidad de exportaciones con nombre (`export function Boton() {}`).
* Con `export default` el nombre al importar es libre; con exportación con nombre debe coincidir (`import { Boton } from './Boton'`).
* Si un archivo tiene un solo componente, lo habitual es `export default`. Si agrupa varios componentes pequeños relacionados, se usan exportaciones con nombre.

Tener varios componentes pequeños en un mismo archivo es válido mientras estén estrechamente relacionados. Conviene separarlos cuando el archivo crece o cuando otros lugares necesitan reutilizarlos.

Nota: con la transformación moderna de JSX no necesitas `import React from 'react'` para usar JSX.

-----

## Ejemplo completo

Una lista de productos dividida en tres componentes:

```jsx
// ProductCard.jsx
export default function ProductCard() {
  return (
    <article>
      <h2>Camisa pequeña</h2>
      <button>Comprar</button>
    </article>
  );
}
```

```jsx
// ProductList.jsx
import ProductCard from './ProductCard';

export default function ProductList() {
  return (
    <section>
      <h1>Productos</h1>
      <ProductCard />
      <ProductCard />
      <ProductCard />
    </section>
  );
}
```

```jsx
// App.jsx
import ProductList from './ProductList';

export default function App() {
  return (
    <main>
      <ProductList />
    </main>
  );
}
```

Puntos clave:

1. `App` es la raíz, `ProductList` su hijo y `ProductCard` una hoja con tres instancias.
2. Cada archivo tiene una responsabilidad clara y un solo `export default`.
3. Las tres tarjetas muestran lo mismo porque todavía no reciben datos. Para que cada una muestre un producto distinto, el padre debe pasarle **props**: es el tema de la [siguiente lección](03-Props.md).

-----

## Errores comunes

### 1. Definir un componente dentro de otro

```jsx
function Gallery() {
  function Profile() {          // mal
    return <img src="..." />;
  }
  return <Profile />;
}
```

**Qué pasa:** funciona en apariencia, pero es lento y puede causar errores, como perder el estado de `Profile` en cada render.
**Por qué:** cada vez que `Gallery` se ejecuta, crea una función `Profile` nueva. Para React es un componente distinto del anterior, así que descarta el viejo y monta uno nuevo.
**Solución:** define los componentes siempre en el nivel superior del archivo, y pasa los datos por props.

```jsx
function Profile() {
  return <img src="..." />;
}

function Gallery() {
  return <Profile />;
}
```

### 2. Nombrar el componente con minúscula

```jsx
function profile() { return <p>Hola</p>; }

<profile />   // React lo trata como una etiqueta HTML
```

**Qué pasa:** React busca una etiqueta HTML llamada `profile`, no encuentra tu función y no la ejecuta.
**Por qué:** JSX distingue por la primera letra. Mayúscula es un componente; minúscula es una etiqueta HTML.
**Solución:** nombra siempre los componentes en PascalCase (`Profile`).

### 3. Llamar al componente como función

```jsx
return <div>{Profile()}</div>;   // evítalo
```

**Por qué es un problema:** al llamarlo así, el código de `Profile` se ejecuta como parte del padre. Si `Profile` usa Hooks, esos Hooks pasan a pertenecer al padre, y React ya no lo trata como un componente separado.
**Solución:** úsalo como etiqueta: `<Profile />`.

### 4. Olvidar importar o exportar

```jsx
// ProductCard.jsx
function ProductCard() { /* ... */ }      // falta export

// ProductList.jsx
<ProductCard />                           // falta import
```

**Qué pasa:** el error habitual es que el componente no está definido (`ProductCard is not defined`), o que la importación no encuentra lo que se exporta.
**Solución:** exporta en el archivo que lo define (`export default` o `export`) e importa en el que lo usa, con la sintaxis que corresponda.

### 5. Mezclar `export default` con importación con nombre

```jsx
export default function Boton() {}

import { Boton } from './Boton';   // mal: no hay exportación con nombre
```

**Solución:** con `export default` importa sin llaves (`import Boton from './Boton'`); con exportación con nombre, usa llaves.

-----

## En TypeScript

Que un componente renderice a otro no añade complejidad de tipos por sí mismo. TypeScript comprueba `<PurchaseButton />` igual que una etiqueta HTML: verifica que las props que pasas coincidan con las que el componente declara.

```tsx
type ProductCardProps = { name: string };

function ProductCard({ name }: ProductCardProps) {
  return <h2>{name}</h2>;
}

function ProductList() {
  return (
    <>
      <ProductCard name="Camisa" />
      <ProductCard />   {/* error: falta la prop name */}
    </>
  );
}
```

En archivos con JSX, la extensión es `.tsx`. El tipado de props se trata en [Props](03-Props.md) y en [Tipado de Props y Funciones](../11-typescript-y-react/01-Tipado%20de%20Props%20y%20Funciones.md).

-----

## Cuándo sí y cuándo no

**Divide en componentes cuando:**

* Un bloque de interfaz **se repite** (tarjetas, filas, botones).
* Un bloque tiene una **responsabilidad propia** que se puede nombrar (`Header`, `SearchBar`).
* El componente se vuelve **largo o difícil de leer**.
* Quieres **reutilizar** esa pieza en otro lugar.

**No dividas cuando:**

* El fragmento es de pocas líneas, se usa una sola vez y no gana claridad con un nombre.
* Separarlo te obliga a pasar muchas props solo para reconstruir lo que estaba junto.
* Lo haces por costumbre y no por una razón concreta: cada componente es un nivel más que seguir al leer el código.

Regla práctica: si puedes darle a la pieza un nombre claro que describa lo que hace, probablemente merece ser un componente.

-----

## Resumen en 5 líneas

1. Un componente puede usar a otros como etiquetas (`<Hijo />`) dentro de su JSX; cada uso crea una instancia.
2. Los componentes forman un **árbol**: la raíz arriba, hijos debajo y hojas al final.
3. React ejecuta primero al padre y luego a sus hijos; los datos bajan de padre a hijo.
4. Define los componentes en el nivel superior del archivo, nunca dentro de otro componente, y con nombre en mayúscula.
5. Es habitual poner un componente por archivo, con `export default`, e importarlo donde se use.

-----

## Para profundizar

<details>
<summary>Árbol de componentes y árbol de renderizado</summary>

La documentación de React llama **árbol de renderizado** al que se forma durante un render: cada nodo es un componente y cada relación indica quién renderizó a quién. No es lo mismo que el árbol de archivos ni el de imports: un componente puede estar importado en un archivo y no aparecer en el render.

Este árbol puede cambiar de un render a otro. Con renderizado condicional, distintos componentes aparecen según los datos.

Los componentes cercanos a la raíz influyen en todos los que están debajo. Los componentes hoja, sin hijos, suelen ser los que se re-renderizan con más frecuencia.

</details>

<details>
<summary>Composición con children</summary>

Además de usar hijos fijos en el JSX, un componente puede recibir contenido desde afuera con la prop especial `children`:

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

<Card>
  <h2>Título</h2>
  <p>Contenido</p>
</Card>
```

`Card` no sabe qué contiene: solo lo envuelve. Es la forma habitual de crear contenedores reutilizables. Se ve en detalle en la lección de props.

</details>

<details>
<summary>Un componente puede devolver otro directamente</summary>

Un componente puede devolver a otro sin envoltorio:

```jsx
function App() {
  return <ProductList />;
}
```

Aquí `App` no agrega ninguna etiqueta HTML propia. Es válido: solo asegúrate de devolver **un único elemento raíz** (o un Fragment), igual que con cualquier JSX.

</details>

-----

## En entrevista

### Respuesta corta (junior)

En React, un componente puede usar otros componentes en su JSX, escribiéndolos como etiquetas. Al componente que los usa se le llama padre y a los usados, hijos. Así se divide la interfaz en piezas pequeñas y reutilizables, y se forma un árbol de componentes. Cada componente suele vivir en su propio archivo, se exporta y se importa donde se necesita.

### Respuesta ampliada (semi-senior)

* **Composición:** la interfaz se construye combinando componentes. Cada `<Hijo />` crea una instancia independiente, con su propio estado.
* **Árbol de renderizado:** React lo construye durante el render. Puede cambiar entre renders con renderizado condicional.
* **Flujo de datos:** unidireccional, de padre a hijo mediante props.
* **Definición en el nivel superior:** definir un componente dentro de otro crea una función nueva en cada render. React la interpreta como otro tipo de componente, desmonta el anterior y pierde su estado.
* **Organización:** un componente por archivo con `export default` es una convención habitual, no una obligación. Varios componentes pequeños y relacionados pueden convivir en un archivo.
* **Cuándo dividir:** por repetición, responsabilidad o legibilidad; no dividir por dividir.
* **Nombres:** JSX distingue componentes de etiquetas HTML por la mayúscula inicial.

### Preguntas frecuentes de seguimiento

**1. ¿Por qué los componentes deben empezar con mayúscula?**
Porque JSX usa la primera letra para decidir: minúscula es una etiqueta HTML, mayúscula es un componente. Con minúscula, React buscaría una etiqueta HTML con ese nombre.

**2. ¿Por qué no se debe definir un componente dentro de otro?**
Cada render del padre crea una función nueva, y React la trata como un componente distinto. Esto vuelve la app lenta y hace perder el estado del hijo. Se define al nivel superior y se le pasan datos por props.

**3. ¿Cuál es la diferencia entre `export default` y una exportación con nombre?**
Un archivo solo puede tener un `export default`, que se importa sin llaves y con cualquier nombre. Las exportaciones con nombre pueden ser varias y se importan con llaves y el mismo nombre.

**4. ¿Qué es el componente raíz?**
Es el componente de más arriba del árbol, del que cuelgan todos los demás. Suele llamarse `App`, pero el nombre no es especial: lo es su posición.

**5. ¿Es obligatorio un componente por archivo?**
No. Es una convención que mejora la organización cuando el proyecto crece. Componentes pequeños y muy relacionados pueden compartir archivo.

**6. ¿Qué diferencia hay entre `<Profile />` y `Profile()`?**
`<Profile />` le pide a React una instancia del componente, y React gestiona sus Hooks y su estado. Llamarlo como función ejecuta su código dentro del padre, y sus Hooks pasan a ser del padre.

-----

## Siguiente lección

Los componentes ya se pueden anidar, pero todos muestran lo mismo. Para que un padre le pase datos distintos a cada hijo, necesitas **props**: [Props](03-Props.md).
