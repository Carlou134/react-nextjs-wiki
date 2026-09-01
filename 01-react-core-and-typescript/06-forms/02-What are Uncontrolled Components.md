# What are Uncontrolled Components?

## Aprende sobre los componentes no controlados: qué son y cuándo usarlos.

Estás creando una aplicación en React y necesitas obtener información de tus usuarios. Usas un elemento `<input>` y sigues el [enfoque recomendado de crear un componente controlado](https://react.dev/reference/react-dom/components#form-components). ¿Pero sabías que también puedes crear componentes no controlados?

Este artículo explicará qué son los componentes no controlados, en qué se parecen y se diferencian de los componentes controlados, y cuándo usarlos en tus aplicaciones React.

## Componentes controlados  

Comencemos con un repaso rápido sobre los componentes controlados. Recuerda, aunque los elementos de formulario (`<input>`, `<textarea>`, etc.) pueden manejar su propio estado interno, en React normalmente preferimos mantener cualquier valor mutable en la propiedad de estado de nuestros componentes.

Para controlar el estado interno de un elemento de formulario, se puede usar el atributo `value` en el elemento `<input>` y asignarle una variable de estado del componente.

```jsx
import ReactDOM from "react-dom";
import React, { useState } from "react";

function PhoneNumberForm() {
  const [number, setNumber] = useState(0);

  const handleChange = (e) => {
    const newNumber = e.target.value;

    if (!Number.isNaN(Number(newNumber)) && newNumber.length <= 10) {
      setNumber(e.target.value);
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Sending confirmation code to ${number}.`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Phone Number:
        <input type="tel" value={number} onChange={handleChange} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}

export default PhoneNumberForm;
```

En este ejemplo, el componente PhoneNumberForm controla el elemento `<input>` asignando su propio valor de estado number al atributo value. Al hacer esto, se desactiva el comportamiento predeterminado del elemento de formulario de controlar su propio estado. Para mantener actualizado el valor de number, se registra un manejador onChange, que puede establecer el valor del estado cada vez que se detecta un cambio.

Con este enfoque, se puede realizar una validación inmediata en cada cambio del formulario. En este caso, se puede asegurar que solo se usen números en el formulario y que no supere los 10 caracteres de longitud (ver handleChange()).

Aunque la validación cambio por cambio es común, en algunos casos solo importa el valor del formulario después de enviarlo. En estos escenarios, mantener el valor actualizado en cada cambio puede ser innecesario. Aquí es donde entran los componentes no controlados.

## Componentes no controlados  

Un componente no controlado es un elemento de formulario que mantiene su propio estado en el DOM. En lugar de usar el valor de estado del componente para mantener el valor del input y actualizarlo en cada cambio, se puede usar una ref para obtener el valor directamente del DOM solo cuando se necesita.

Según React:

    Las refs proporcionan una forma de acceder a nodos del DOM o elementos de React creados en el método render.

Para crear un componente no controlado, primero se crea una ref usando el método [useRef()](https://react.dev/reference/react/useRef). Este método devuelve un objeto con una propiedad .current que se refiere al nodo del DOM al que está vinculado. Este objeto ref se vincula a un elemento de formulario usando el atributo ref, y cuando se necesita obtener el valor de ese elemento, simplemente se consulta la propiedad .current del objeto ref.

```jsx
import ReactDOM from "react-dom";
import React from "react";

function PhoneNumberForm() {
  const numberRef = React.useRef();

  const handleSubmit = (e) => {
    e.preventDefault(); 
    
    const number = numberRef.current.value;

    if (Number.isNaN(Number(number))) {
      alert('Error: Only numbers allowed.')
    }
    else if (number.length >= 10) {
      alert('Error: Number length exceeded 10 digits.')
    }
    else {
      alert(`Sending confirmation code to ${number}.`);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Phone Number:
        <input type="tel" ref={numberRef} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}

export default PhoneNumberForm;
```

En este ejemplo, se crea el objeto numberRef y luego se vincula al elemento de formulario `<input>`.

```js
const numberRef = React.useRef();

// ...

<input type="text" ref={numberRef} />
```

En handleSubmit, el valor de ese elemento de formulario se puede obtener del nodo del DOM almacenado en numberRef.current.

```js
const number = numberRef.current.value;
```

## ¿Cuándo deberías usar un componente no controlado?  

En algunos casos, crear componentes no controlados es más rápido y sencillo que crear componentes controlados. Sin embargo, como se apartan del patrón de React de guardar datos mutables en el estado del componente, los componentes controlados siguen siendo recomendados en la mayoría de los casos.

Hay una situación en la que siempre se deben usar componentes no controlados: los elementos de formulario `<input>` con el atributo `type="file"`. El valor de este tipo de `<input>` solo puede ser establecido por el usuario, no de forma programada, así que la única forma de obtener este valor es usando una ref.

```jsx
import ReactDOM from "react-dom";
import React, { useState } from "react";

function PhoneNumberForm() {
  const fileRef = React.useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    
    const size = fileRef.current.files[0].size;
    alert(
      `This file is ${size} bytes`
    );
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        <input type="file" ref={fileRef} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}

export default PhoneNumberForm;
```

En este ejemplo, nuevamente se crea una ref usando el método `React.createRef()` y luego se vincula al elemento de formulario `<input>`. El archivo subido se almacena en el objeto tipo arreglo [FileList](https://developer.mozilla.org/en-US/docs/Web/API/FileList) que devuelve `fileRef.current.files` y se accede a la propiedad `.size` de este archivo cuando el usuario envía el formulario.

## Resumen  

En la documentación de React sobre [componentes no controlados](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components), se recomienda usar componentes controlados siempre que sea posible. Los componentes controlados permiten rastrear los valores del formulario en cada cambio y se ajustan mejor al patrón de React de guardar datos mutables en el estado del componente.

Aun así, hay momentos en los que los componentes no controlados tienen ventajas. Si solo necesitas acceder al valor del formulario al enviarlo o usas un elemento `<input type='file'>`, los componentes no controlados pueden ser una herramienta útil en React.

------

