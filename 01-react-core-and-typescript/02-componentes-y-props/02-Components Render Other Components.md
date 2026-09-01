# Components Render Other Components

## Components Interact

Una aplicación en React puede contener docenas, o incluso cientos, de componentes.

Cada componente podría ser pequeño y relativamente poco destacado por sí solo. Sin embargo, cuando se combinan, pueden formar enormes y fantásticamente complejos ecosistemas de información.

En otras palabras, las aplicaciones React están hechas de componentes, pero lo que hace especial a React no son los componentes en sí mismos. Lo que hace especial a React son las formas en que los componentes interactúan.

Esta lección es una introducción a la interacción entre componentes. Después de esta lección, deberías estar familiarizado con:

* Cómo los componentes pueden hacer referencia a otros componentes.
* Cómo esto nos permite separar nuestros componentes en archivos distintos.

-----

## Returning Another Component

Como hemos visto antes, cada componente de React es responsable de una parte de la interfaz. A medida que la aplicación crece en complejidad, los componentes necesitan poder interactuar entre sí para soportar las características necesarias.

Hasta ahora, hemos explorado componentes que devuelven elementos **JSX**, como por ejemplo:

```jsx
function PurchaseButton() {
  return <button onClick={()=>{alert("Purchase Complete")}}>Comprar</button>
}
```

En este ejemplo, el componente de React no está interactuando con otros componentes. Sin embargo, puedes hacer que los componentes interactúen entre sí pasando información o incluso devolviendo otros componentes.

```jsx
function PurchaseButton() {
  return <button onClick={()=>{alert("Purchase Complete")}}>Comprar</button>
}

function ItemBox() {
  return (
    <div>
      <h1>50% de descuento</h1>
      <h2>Artículo: Camisa pequeña</h2>
      <PurchaseButton />
    </div>
  );
}
```

En el ejemplo anterior, `ItemBox` devuelve una instancia de `PurchaseButton`. ¡Este es un ejemplo de cómo un componente puede hacer referencia a otros componentes!

Nota: Fíjate que no necesitamos importar React para usar JSX. Gracias a la transformación moderna de JSX, los componentes pueden usar JSX sin importar explícitamente React.

-----

## Apply a Component in a Render Function

¡Puede que hayas notado que has visto este comportamiento antes!

En las lecciones anteriores, cuando definimos componentes y los exportamos, usualmente los exportábamos a nuestro archivo de nivel superior, **App.js**. Dentro de **App.js**, importábamos los componentes y los devolvíamos dentro de nuestro componente **App**, ¡que luego se exportaba para ser renderizado!

```jsx
import Button from './Button'

function App() {
  return <Button />;
}

export default App;
```

¿Te suena familiar? Esta capacidad nos permite separar los componentes en funciones más pequeñas y conectarlas entre sí para crear componentes más complejos.

Podemos tratar el componente **Button** como un hijo del componente principal **App**. Al descomponer un componente en partes más pequeñas, podemos reutilizar las partes individuales cuando sea necesario.

-----

## Review

¡Buen trabajo! Has llegado al final de esta breve pero importante lección.

Hagamos un resumen:

* Una aplicación en React puede contener varios componentes.
* Los componentes pueden interactuar entre sí devolviendo instancias unos de otros.
* La interacción entre componentes permite que se dividan en componentes más pequeños, se almacenen en archivos separados y se reutilicen cuando sea necesario.

------

