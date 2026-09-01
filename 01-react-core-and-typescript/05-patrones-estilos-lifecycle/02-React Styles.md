# React Styles

## Intro to Styling React Apps

El **estilizado** es un aspecto fundamental de cualquier aplicación en React, ya que puede impactar la experiencia del usuario y ayudar a crear una identidad distintiva para tu aplicación. A medida que tu aplicación crece en complejidad, la forma en que aplicas los estilos se vuelve cada vez más importante. Es esencial elegir el enfoque correcto para mantener tus estilos organizados y fáciles de manejar.

En esta lección, cubriremos los conceptos básicos del estilizado en React, incluyendo los diferentes enfoques y técnicas que puedes utilizar. Comenzaremos con una explicación del estilizado en línea y el uso de variables de estilo como objetos, y explicaremos las reglas de sintaxis únicas que son específicas de React. Luego, profundizaremos en los **módulos CSS** y te mostraremos cómo usarlos para hacer que tus estilos sean modulares y reutilizables.

Al final de esta lección, tendrás una base sólida sobre el estilizado en React y contarás con el conocimiento necesario para dar estilo a tus componentes de una manera que sea mantenible, escalable y organizada.

-----

## Inline Styles and Style Object Variables

Hay muchas formas diferentes de usar estilos en React. Este ejercicio se enfoca en dos de ellas: **estilos en línea** y **variables de objetos de estilo**.

Un estilo en línea es un estilo que se escribe como un atributo, por ejemplo:

```jsx
<h1 style={{ color: 'red' }}>Hello world</h1>
```

Observa que tiene **dobles llaves**. Las llaves externas indican que todo lo que está dentro debe interpretarse como **JavaScript**. Las llaves internas crean un **objeto literal de JavaScript**.

Sin embargo, usar estilos en línea puede volverse rápidamente desordenado si quieres aplicar más que solo unos pocos estilos. Una alternativa es guardar un objeto de estilos en una variable y luego inyectar esa variable como el valor del atributo `style`.

Para hacer esto, podemos inicializar un objeto con propiedades y valores de esta forma:

```js
const darkMode = {
  color: 'white',
  background: 'black'
};
```

Luego, el objeto puede inyectarse para dar estilo a un componente:

```jsx
<h1 style={darkMode}>Hello world</h1>
```

----

## Style Syntax

Hay algunas cosas que debes tener en cuenta al aplicar estilos a componentes con **JSX**.

Al igual que cuando referenciamos propiedades CSS en el objeto `style` del DOM en JavaScript, en React escribimos los nombres de las propiedades CSS usando **camelCase**:

```js
const styles = {
  marginTop: '20px',
  backgroundColor: 'green'
};
```

Esta sintaxis proviene de una regla simple. El guion (`-`) es un operador reservado en JavaScript. Si usamos `background-color`, el guion se interpreta como un signo de resta. Por lo tanto, para ser consistentes con los nombres de las propiedades en el objeto `style` del DOM en JavaScript, usamos **camelCase**.

En JavaScript normal, los valores de estilo casi siempre son **strings**. Incluso si un valor de estilo es numérico, normalmente debes escribirlo como un string para poder especificar una unidad. Por ejemplo, escribirías `'450px'` o `'20%'`.

Si escribes un valor de estilo como un número, entonces se asume automáticamente la unidad `'px'`. Por ejemplo, si quieres un tamaño de fuente de 30px, puedes escribir:

```js
{ fontSize: 30 }
```

Si quieres usar unidades distintas de `'px'`, puedes usar un string:

```js
{ fontSize: "2em" }
```

Especificar la unidad `'px'` dentro de un string seguirá funcionando, aunque es redundante.

Algunos estilos específicos no completan automáticamente la unidad `'px'`. Estos son estilos en los que normalmente no usarías `'px'` de todos modos, así que no tienes que preocuparte demasiado por ellos.

-----

## Multiple Stylesheets

Aunque los **estilos en línea** y las **variables de objetos de estilo** son métodos válidos para aplicar estilos en React, puede volverse difícil mantener una buena organización y seguimiento de los estilos a medida que tu aplicación crece.

Una forma de hacer que los estilos sean **modulares, organizados y reutilizables** es crear hojas de estilo separadas para cada componente.

Podemos importar una hoja de estilos usando la palabra clave `import`:

```js
import './App.css'
```

Sin embargo, si tenemos múltiples hojas de estilo con los mismos nombres de clases, estos nombres pueden **colisionar** y crear conflictos de estilos.

Una forma de evitar esto es usar **módulos CSS**. Al importarlos como un módulo, los estilos solo estarán disponibles para el componente que importó el archivo. Esto se hace automáticamente creando nombres de clase únicos para cada módulo. De esta manera, no tenemos que preocuparnos por llevar un registro de los nombres de clase usados en todas las hojas de estilo.

Para usar módulos CSS, comenzamos nombrando nuestra hoja de estilos con el siguiente formato, donde `fileName` debe reemplazarse por el nombre del componente que estás estilizando:

```text
fileName.module.css
```

Esto indica que el archivo debe procesarse como un módulo CSS.

Luego, debe importarse en el archivo que contiene nuestro componente:

```js
import styles from './fileName.module.css'
```

A partir de esta importación, podemos ver que el objeto `styles` ahora contiene los selectores de clase de `fileName.module.css`. Para acceder a los selectores, usamos la notación de punto, por ejemplo:

```jsx
<div className={styles.divStyle}></div>
```

Ten en cuenta que aplicamos los estilos usando el atributo `className` en lugar de `class`. `class` es una palabra reservada en JavaScript, por lo que React utiliza `className` para evitar conflictos.

Aunque React no impone una forma específica de definir estilos, este es el método preferido para estilizar en React, ya que mantiene la filosofía composicional de React.

-----

## Review

¡Bien hecho! Has llegado al final de esta lección sobre el estilizado en aplicaciones React.

Antes de terminar, aquí tienes un resumen:

* Los componentes de React pueden estilizarse de varias maneras: **estilos en línea**, **estilos con variables de objeto**, **hojas de estilo** y **módulos CSS**.
* Los estilos en línea pueden usarse para aplicar estilos a un solo elemento. Se hacen dando al elemento un atributo llamado `style`, cuyo valor es un objeto literal rodeado por llaves.

  ```jsx
  <h1 style={{ color: "red" }}> Hello, World! </h1>
  ```
* También se puede usar una **variable de objeto** para aplicar estilos a un solo elemento. La sintaxis es similar al estilizado en línea, pero en lugar de pasar un objeto literal, se pasa el nombre de la variable.

  ```js
  const myStyle = { color: "red" }
  <h1 style={myStyle}> Hello, World! </h1>
  ```
* Los nombres de las propiedades de estilo en React deben estar en **camelCase**. Por ejemplo, `background-color` se convierte en `backgroundColor`.
* En React, cuando un valor de estilo es un número, se interpreta automáticamente con la unidad **px**.
* Los estilos pueden separarse y almacenarse en archivos de **módulos CSS**. Estos estilos pueden importarse y usarse aplicando atributos `className` a los elementos correspondientes.

------

## Sass en React

**Sass** (Syntactically Awesome Style Sheets) es un preprocesador de CSS: se escribe en un superset del lenguaje CSS que agrega características que CSS por sí solo no tiene, y luego se compila a CSS plano antes de llegar al navegador. En un proyecto de React, esa compilación la resuelve automáticamente el bundler (Vite, Create React App, etc.) en cuanto el paquete `sass` está instalado como dependencia; no requiere configuración manual adicional.

Las características que Sass añade por sobre CSS son, principalmente:

- **Variables**, para centralizar valores reutilizados como colores o espaciados.
- **Anidamiento (nesting)**, para escribir selectores hijos dentro del selector de su padre, reflejando la jerarquía del HTML.
- **Mixins**, para reutilizar bloques de declaraciones CSS en distintos selectores.

```scss
$color-primario: #2563eb;

.card {
  border-radius: 8px;

  .card-title {
    color: $color-primario;
    font-weight: bold;
  }
}
```

En React, los archivos Sass se usan igual que los módulos CSS vistos anteriormente, pero con extensión `.scss`. Un archivo `Card.module.scss` se importa como módulo, obteniendo nombres de clase únicos por componente, con la ventaja adicional de poder usar variables, anidamiento y mixins dentro de esas reglas:

```jsx
import styles from './Card.module.scss';

<div className={styles.card}>
  <h2 className={styles.cardTitle}>Título</h2>
</div>
```

Sass no reemplaza a los módulos CSS: es un lenguaje que se compila a CSS y puede combinarse con ellos. La decisión de incorporarlo suele depender del tamaño del proyecto: en aplicaciones donde las hojas de estilo crecen mucho y se repiten valores o patrones de selectores, las variables y los mixins ayudan a mantener el CSS organizado y evitan la duplicación.

-----

## Styled Components

**styled-components** es una librería que permite escribir CSS directamente dentro de JavaScript, usando template literals, para crear componentes que ya llevan sus estilos incorporados. A este enfoque se lo conoce como **CSS-in-JS**.

```jsx
import styled from 'styled-components';

const Boton = styled.button`
  background: blue;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
`;

<Boton>Enviar</Boton>
```

`Boton` es, en sí mismo, un componente de React: al usarlo, React renderiza un `<button>` con las reglas CSS definidas en el template literal ya aplicadas. No hace falta un archivo `.css` aparte ni preocuparse por colisiones de nombres de clase, porque styled-components genera un nombre de clase único para cada componente estilizado.

La característica distintiva de esta librería es que las reglas CSS pueden depender de las **props** que recibe el componente, igual que cualquier otra lógica de React:

```jsx
const Boton = styled.button`
  background: ${(props) => (props.primary ? '#2563eb' : '#6b7280')};
`;

<Boton primary>Confirmar</Boton>
<Boton>Cancelar</Boton>
```

Con esto, `<Boton primary>` se renderiza en azul y `<Boton>` en gris, sin necesidad de alternar clases CSS manualmente ni de escribir condicionales en el `className`.

### Ventajas y desventajas frente a Tailwind

styled-components resulta cómodo cuando los estilos dependen fuertemente de props y se prefiere mantener CSS y lógica de componente en el mismo archivo. Sin embargo, tiene un costo: genera los estilos **en tiempo de ejecución (runtime)**, es decir, el navegador debe procesar JavaScript para producir el CSS final, lo que añade trabajo adicional en comparación con una hoja de estilos ya compilada.

Por esa razón, en proyectos nuevos es hoy más común optar por **Tailwind CSS**, que no incurre en ese costo de runtime porque genera el CSS de forma estática durante el build. styled-components sigue siendo una opción válida —sobre todo en proyectos que ya lo usan o que necesitan estilos muy dinámicos basados en props— pero ha dejado de ser la opción por defecto para proyectos que arrancan desde cero.

-----

## Tailwind CSS

**Tailwind CSS** es un framework de utilidades: en lugar de escribir reglas CSS propias con selectores y nombres de clase inventados, se construye la interfaz combinando clases ya definidas por el framework, directamente en el JSX.

```jsx
<h1 className="text-3xl font-bold underline">
  Hello world
</h1>
```

Cada clase (`text-3xl`, `font-bold`, `underline`) aplica una única propiedad CSS. El estilo final del elemento surge de combinar varias de estas clases utilitarias, sin necesidad de escribir un solo selector CSS propio.

### Instalación

La instalación de Tailwind ha cambiado entre versiones. En **Tailwind v3**, el proceso requiere instalar el paquete junto con PostCSS y Autoprefixer, generar los archivos de configuración, indicarle a Tailwind en qué archivos debe buscar clases, y activar el framework mediante directivas en el CSS principal:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

En **Tailwind v4**, integrado con Vite, la configuración se simplifica considerablemente: basta con una única línea de importación en el archivo CSS principal, sin necesidad de generar `tailwind.config.js` ni declarar las tres directivas por separado.

```css
@import "tailwindcss";
```

### Diferencia filosófica con CSS o Sass

Escribir CSS propio —con o sin Sass— separa la definición del estilo (en un archivo `.css` o `.scss`) de su aplicación (en el `className` del componente), y exige nombrar cada selector. Tailwind invierte ese enfoque: el estilo se declara directamente donde se usa, componiendo clases ya existentes en lugar de inventar nombres nuevos para cada elemento. Esto elimina el problema de nombrar clases y mantiene el estilo visible junto al marcado, a costa de que el JSX incluya cadenas de clases más largas. Como esas clases se resuelven durante el build y no en tiempo de ejecución, Tailwind no tiene el costo de runtime asociado a librerías de CSS-in-JS como styled-components.

------

