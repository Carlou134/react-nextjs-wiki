# Tipado de Props y Funciones

## Why TypeScript in React?

Hasta ahora, todos los componentes que escribimos han sido en JavaScript puro. Esto tiene una consecuencia que quizás ya experimentaste sin darte cuenta: si un componente espera recibir una prop `label` como texto, y por error le pasás un número o directamente te olvidás de pasarla, JavaScript no te avisa nada durante el desarrollo. El error, si aparece, lo vas a ver recién cuando la aplicación ya está corriendo, muchas veces en forma de un mensaje críptico en la consola del navegador.

**TypeScript** es un superset de JavaScript que agrega un sistema de **tipos estáticos**. En la práctica, esto significa que le podés decir a tu editor y al compilador exactamente qué forma tienen los datos que tu código espera: qué tipo de dato es cada prop, qué recibe y qué devuelve cada función. TypeScript revisa estas reglas **antes** de que el código se ejecute, y te avisa inmediatamente si algo no encaja, directamente en el editor, con un subrayado rojo y un mensaje de error explicativo.

Esta verificación anticipada es la diferencia central respecto de JavaScript: en lugar de descubrir un error de tipos cuando un usuario real se topa con él en producción, lo descubrís vos, mientras escribís el código.

-----

## Typing Component Props

La forma más común de tipar las props de un componente es definir un **tipo** (usando la palabra clave `type`) que describa la forma exacta del objeto de props que ese componente espera recibir.

```ts
type ButtonProps = {
  onClick: () => void;
  label: string;
};
```

Este tipo `ButtonProps` establece dos reglas: la prop `onClick` debe ser una función que no recibe argumentos y no devuelve ningún valor (`() => void`), y la prop `label` debe ser una cadena de texto (`string`).

Ahora usamos ese tipo en la definición del componente:

```tsx
const Button = ({ onClick, label }: ButtonProps) => {
  return <button onClick={onClick}>{label}</button>;
};
```

La anotación `: ButtonProps` después del parámetro desestructurado le dice a TypeScript exactamente qué forma tiene el objeto que este componente recibe. A partir de este momento, si en algún lugar de la aplicación alguien intenta usar `<Button label="Enviar" />` sin pasar `onClick`, o le pasa `onClick={5}` en lugar de una función, TypeScript va a marcar el error inmediatamente en el editor, antes de que ese código llegue a ejecutarse.

-----

## Typing Functions Passed as Props

Es muy común que un componente reciba, como prop, una función que debe ejecutar (por ejemplo, un manejador de eventos definido en el componente padre). Tipar correctamente la **forma** de esa función es tan importante como tipar cualquier otra prop, porque le indica a quien use el componente exactamente qué argumentos va a recibir esa función y qué se espera que devuelva.

Por ejemplo, si un componente `SearchInput` necesita avisarle a su padre cada vez que el texto cambia, y ese aviso debe incluir el nuevo valor como texto:

```ts
type SearchInputProps = {
  onSearch: (query: string) => void;
};

const SearchInput = ({ onSearch }: SearchInputProps) => {
  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    onSearch(event.target.value);
  };

  return <input onChange={handleChange} />;
};
```

Hay dos detalles que vale la pena separar aquí:

* El tipo `(query: string) => void` en `onSearch` describe la forma de la función que el componente **espera recibir**: una función que toma un `string` y no devuelve nada. Esto asegura que quien use `<SearchInput onSearch={...} />` le pase una función compatible, y que dentro de `SearchInput` no se intente llamar a `onSearch` con un argumento que no sea texto.
* El tipo `React.ChangeEvent<HTMLInputElement>` en el parámetro `event` de `handleChange` describe la forma del objeto de evento que React le pasa a los manejadores de `onChange` en un elemento `<input>`. Tipar el evento correctamente es lo que le permite a TypeScript saber que `event.target.value` existe y es un `string`, en lugar de marcarlo como un acceso a una propiedad desconocida.

-----

## What You Gain

El beneficio de tipar props y funciones no es solamente "evitar errores". Al declarar explícitamente qué espera cada componente, el tipo funciona como una forma de **documentación que nunca se desactualiza**: cualquier persona (incluido vos mismo, unos meses después) puede abrir el archivo, ver el tipo `ButtonProps` o `SearchInputProps`, y saber exactamente qué props acepta el componente sin tener que leer toda su implementación.

Además, los editores modernos usan esta información para ofrecer **autocompletado**: al escribir `<Button ` el editor te va a sugerir automáticamente `onClick` y `label` como las props disponibles, y va a marcar en rojo cualquier prop que no exista en el tipo. Esta combinación (menos errores, código autodocumentado, mejor autocompletado) es la razón principal por la que la gran mayoría de los proyectos React actuales adoptan TypeScript desde el inicio.
