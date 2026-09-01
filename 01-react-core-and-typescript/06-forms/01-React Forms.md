# React Forms

## React Forms

Piensa en cómo funcionan los formularios en un entorno típico que no usa React. Un usuario escribe datos en los campos de un formulario, y el servidor no lo sabe. El servidor no se entera hasta que el usuario hace clic en el botón de “enviar”, que manda todos los datos del formulario al servidor al mismo tiempo.

En React, como en otros entornos de JavaScript, esta no es la mejor manera de hacerlo.

El problema es el tiempo en el que el formulario cree que el usuario ha escrito una cosa, pero el servidor cree que ha escrito otra. ¿Qué pasa si, en ese momento, otra parte del sitio web necesita saber lo que el usuario ha escrito? Podría preguntar al formulario o al servidor y obtener dos respuestas diferentes. En una aplicación compleja de JavaScript con muchas partes interdependientes, este tipo de conflicto puede causar problemas.

En un formulario de React, se quiere que el servidor sepa sobre cada nuevo carácter o eliminación tan pronto como ocurra. Así, la pantalla siempre estará sincronizada con el resto de la aplicación.

-----

## Input onChange

Hablemos de cómo se manejan los formularios en React.

En un formulario HTML normal, el estado del formulario normalmente lo maneja el navegador. No se actualiza en el servidor hasta que el usuario hace clic en enviar.

En React, las cosas funcionan un poco diferente. En un formulario de React, el estado del formulario puede ser manejado por el componente, y las actualizaciones se activan con el evento onChange. Cuando el usuario interactúa con un input, como al escribir o borrar caracteres, el evento onChange se activa y actualiza el estado del componente.

Esto permite que el componente refleje de inmediato cualquier cambio hecho por el usuario y actualice la vista en consecuencia.

Veamos cómo funciona.

-----

## Write an Input Event Handler

Definamos una función manejadora de eventos que se ejecuta cada vez que un usuario escribe o borra cualquier carácter dentro del elemento `<input>`.

La función manejadora de eventos escuchará los eventos de cambio. Puedes ver un ejemplo de una función manejadora que escucha eventos de cambio en Example.js.

----

## Set the Input's Initial State

¡Bien! Cada vez que alguien escribe o borra en el campo `<input>`, la función `handleUserInput()` actualizará `userInput` con el texto del `<input>`.

Como estamos usando `setUserInput`, eso significa que `userInput` necesita un estado inicial. Necesitaremos usar el hook de estado. ¿Cuál debería ser el valor inicial del estado?

Bueno, `userInput` se mostrará en el elemento `<input>`. ¿Cuál debería ser el texto inicial en el `<input>` cuando un usuario visita la página por primera vez?

El texto inicial debe ser una cadena vacía. De lo contrario, parecería que alguien ya escribió algo.

-----

## Update an Input's Value

Cuando un usuario escribe en el campo `<input>`, eso dispara un evento de cambio, que llama a `handleUserInput()`. ¡Eso está bien!

`handleUserInput()` hará que `userInput` sea igual al texto actual en el campo `<input>`. ¡Eso también está bien!

Solo hay un problema: puedes poner cualquier valor en `userInput`, pero la propiedad `value` de `<input>` no se actualizará.

En React, la propiedad `value` de un elemento input se usa para controlar el valor del input y mantenerlo sincronizado con el estado del componente. Si no se establece la propiedad `value`, los cambios hechos en el input no se reflejarán en el estado del componente, lo que puede causar inconsistencias y errores.

Para asegurarse de que el valor del input coincida con el estado del componente, se puede establecer la propiedad `value` y usar el evento `onChange` para actualizar el estado con el nuevo valor. Cuando el estado se actualiza, el componente se vuelve a renderizar y la propiedad `value` se establece con el nuevo valor del estado.

Esto hace que el estado del componente sea la “fuente de la verdad” para el valor del input, asegurando que los datos del formulario sean consistentes y fáciles de manejar y enviar.

Por ejemplo, si tuviéramos un formulario de inicio de sesión con un input de email y otro de contraseña, pondríamos la propiedad `value` en ambos inputs y actualizaríamos el estado del componente cada vez que el usuario escriba un nuevo email o contraseña. Así, los datos del formulario siempre estarán actualizados con lo que escribe el usuario.

-----

## Controlled vs Uncontrolled

Hay dos términos que probablemente aparecerán cuando hables de formularios en React: componente controlado y componente no controlado.

Un componente no controlado es un componente que mantiene su propio estado interno. Un componente controlado es un componente que no mantiene ningún estado interno. Como un componente controlado no tiene estado, debe ser controlado por otro.

Piensa en un elemento típico `<input type='text' />`. Aparece en pantalla como una caja de texto. Si necesitas saber qué texto hay en la caja, puedes preguntarle al elemento `<input>`, por ejemplo, con este código:

```js
let input = document.querySelector('input[type="text"]');
let typedText = input.value; // input.value será igual al texto actual en la caja de texto.
```

Lo importante aquí es que el elemento `<input>` lleva el control de su propio texto. Puedes acceder a ese texto en cualquier momento.

El hecho de que `<input>` lleve el control de la información lo convierte en un componente no controlado. Mantiene su propio estado interno recordando datos sobre sí mismo.

Un componente controlado, en cambio, no tiene memoria. Si le pides información sobre sí mismo, tendrá que obtenerla a través de las props. La mayoría de los componentes de React son controlados.

En React, cuando le das a un elemento `<input>` un atributo `value`, ocurre algo curioso: el elemento `<input>` se vuelve controlado. Deja de usar su almacenamiento interno. Esta es una forma más “React” de hacer las cosas.

Recuerda que en nuestros ejercicios, la página se actualizaba cada vez que escribíamos en el input. React controlaba el valor del input con el estado. ¡Hemos estado demostrando la idea de un componente controlado todo el tiempo!

Puedes encontrar más información sobre [componentes controlados y no controlados en la documentación de React](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components).

----

## Review

¡Buen trabajo! Acabas de crear tu primer formulario en React.

Aquí tienes un repaso:

- El estado de un formulario en React es manejado por el componente, y las actualizaciones se activan con el evento onChange.  
- El evento onChange usa una función manejadora para capturar los cambios y decidir qué acciones tomar.  
- Un formulario en React usa el hook de estado para guardar el valor del campo de entrada en el estado del componente. El estado se puede actualizar con el setter.  
- Los componentes de React pueden ser controlados o no controlados. La mayoría de los formularios en React son controlados, ya que controlan el valor del input con el estado.

¡Eso marca el final de esta lección! Las habilidades que aprendiste con formularios en React serán muy útiles al crear más aplicaciones con React.

----

## 🧠 ¿Qué es un *controlled form* en React?

👉 Es un formulario donde **React controla el valor del input**, no el navegador.

## 📌 La frase explicada en simple

> *“To build a controlled form component in React…”*

Significa:

### 1️⃣ El valor del input viene del **estado**

```js
value={state}
```

### 2️⃣ El input **no se cambia solo**

Cada cambio dispara un evento (`onChange`)

### 3️⃣ Ese evento llama a una función

```js
onChange={handleChange}
```

### 4️⃣ La función actualiza el estado del padre

```js
setState(e.target.value)
```

## 🧩 Ejemplo mínimo (controlado)

```js
function Form() {
  const [name, setName] = useState('');

  const handleChange = (e) => {
    setName(e.target.value);
  };

  return (
    <input
      value={name}
      onChange={handleChange}
    />
  );
}
```

✔️ El input **muestra** lo que hay en `name`
✔️ Cada tecla → `onChange` → `setName`
✔️ React manda


## ❌ No controlado (lo contrario)

```js
<input />
```

🚫 El navegador maneja el valor
🚫 React no sabe qué hay dentro

## 🧠 Regla mental final

> **Input controlado = value + onChange + state**

## 🎯 Frase de entrevista

> Un formulario controlado es aquel donde el valor de los campos depende del estado de React y se actualiza mediante manejadores de eventos.

---

