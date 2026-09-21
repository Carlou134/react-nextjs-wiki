# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Árbol de componentes.** La jerarquía de componentes de tu app, donde unos están dentro de otros (raíz, padres, hijos, hojas).

**Árbol de renderizado.** La estructura que React construye al renderizar, con un nodo por cada componente y una relación padre-hijo entre ellos. Puede cambiar entre renders.

**Callback.** Una función que se pasa a otra para que esta la llame cuando corresponda. Ejemplo: la función que recibe un hijo para avisarle algo al padre.

**Children.** La prop especial que recibe todo lo escrito entre la etiqueta de apertura y la de cierre de un componente. Sirve para crear contenedores reutilizables.

**Componente.** Una función que devuelve lo que se ve en pantalla (JSX). Es la pieza básica de una app de React.

**Componente hijo.** El componente que se usa dentro del JSX de otro.

**Componente padre.** El componente que usa a otro dentro de su JSX.

**Componente raíz.** El componente de más arriba del árbol, del que cuelgan todos los demás. Suele llamarse `App`.

**Composición.** Construir componentes complejos combinando componentes más simples.

**Desestructuración.** Una forma corta de sacar valores de un objeto y ponerles nombre. En las props se hace en el parámetro: `function Button({ text })`.

**DOM.** La representación de la página que tiene el navegador: el árbol de elementos HTML que se ve en pantalla.

**Estado (state).** Un dato que el componente controla y que, al cambiar, cambia lo que se ve. A diferencia de una prop, pertenece al propio componente.

**Export / import.** Las palabras de JavaScript para compartir código entre archivos: `export` expone algo desde su archivo e `import` lo trae a otro. `export default` expone un valor principal; la exportación con nombre permite varios.

**Flujo unidireccional.** Los datos viajan en una sola dirección: de padre a hijo mediante props, nunca al revés.

**Handler (manejador de eventos).** Una función que se ejecuta como respuesta a un evento, como un clic. Por convención se nombra `handleX`.

**Hoja.** Un componente que no renderiza ningún otro componente, solo elementos HTML.

**Instancia.** Cada uso de un componente en el JSX. Un mismo componente puede tener varias instancias, y cada una es independiente.

**Prop drilling.** Pasar una prop por varios componentes intermedios que no la usan, solo para que llegue a uno más profundo.

**PascalCase.** Convención donde cada palabra del nombre empieza con mayúscula (`MyComponent`). Los componentes se nombran así para que React los distinga de las etiquetas HTML.

**Props.** El objeto con los datos que un componente padre entrega a un hijo mediante atributos JSX. Son de solo lectura.

**ReactNode.** Un tipo de TypeScript que cubre todo lo que React puede renderizar: texto, números, elementos, arreglos, `null` y `undefined`. Se usa para tipar `children`.

**Render (renderizar).** Producir la interfaz que describe un componente y mostrarla en pantalla. Ocurre cada vez que React ejecuta la función del componente.

**Raíz (root).** El punto de la página donde React toma el control de la interfaz. Se crea con `createRoot`.

**Valor por defecto.** El valor que toma una prop cuando no se pasa o vale `undefined`. Se define al desestructurar: `{ text = 'Hola' }`.
