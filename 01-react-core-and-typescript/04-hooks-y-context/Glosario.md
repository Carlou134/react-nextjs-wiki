# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Acción (action).** Un objeto que describe "qué pasó", por ejemplo `{ type: 'add', payload: producto }`. Se le envía al reducer con `dispatch`.

**Array (arreglo).** Una lista de valores en orden, como `['a', 'b', 'c']`.

**Árbol de componentes.** La jerarquía de componentes de tu app, donde unos están dentro de otros (padres, hijos, nietos...).

**Callback.** Una función que le pasas a otra para que la ejecute más tarde. Ejemplo: la función que le das a `onClick` se ejecuta cuando hacen clic.

**Cleanup (limpieza).** La función que un efecto puede devolver para deshacer lo que hizo (por ejemplo, quitar un listener) antes de volver a ejecutarse o cuando el componente desaparece.

**Componente.** Una función que devuelve lo que se ve en pantalla (JSX). Es la pieza básica de una app de React.

**Consumidor.** Un componente que lee un dato de un Context.

**Context.** Una forma de compartir un dato con muchos componentes a la vez, sin pasarlo por props uno a uno.

**Convención.** Un acuerdo entre programadores que no exige el lenguaje ni la herramienta, pero que todos siguen para que el código sea fácil de entender. Ejemplo: los Hooks propios empiezan con `use`.

**Custom Hook.** Una función tuya, cuyo nombre empieza con `use`, que usa otros Hooks adentro para reutilizar lógica.

**Dependencia.** Un valor del que depende un efecto. Si cambia, el efecto se vuelve a ejecutar. Se listan en el array de dependencias.

**Desestructuración.** Una forma corta de sacar valores de un array o de un objeto y ponerles nombre. En un array se asigna por **posición** (`const [a, b] = lista`); en un objeto, por **nombre** (`const { a, b } = objeto`).

**Dispatch.** La función que devuelve `useReducer` para enviarle una acción al reducer.

**DOM.** La representación de la página que tiene el navegador: el árbol de elementos HTML que se ve en pantalla.

**Efecto.** La función que le pasas a `useEffect`. Adentro va el efecto secundario que quieres ejecutar.

**Efecto secundario (side effect).** Algo que pasa fuera de "dibujar la pantalla": pedir datos a una API, cambiar el título de la pestaña, usar un temporizador.

**Estado (state).** Un dato que el componente recuerda y que, al cambiar, cambia lo que se ve.

**Hook.** Una función especial de React, con nombre que empieza con `use`, que te deja usar herramientas de React (memoria, efectos, contexto) dentro de un componente.

**Inmutable.** Que no se modifica: en lugar de cambiar un valor, se crea uno nuevo. Con el estado de React, arrays y objetos se tratan así.

**JSX.** La sintaxis que mezcla JavaScript con etiquetas parecidas a HTML para describir lo que se ve.

**Lógica con estado.** Código que usa `useState` (y a veces `useEffect`) para recordar datos y reaccionar a cambios. Es lo que se extrae a un Custom Hook para reutilizarlo.

**Máquina de estados.** Una forma de modelar algo que solo puede estar en una de varias situaciones posibles a la vez (por ejemplo, un pedido a una API: sin empezar, cargando, con éxito o con error).

**Memoizar.** Guardar un resultado para reutilizarlo y no volver a calcularlo si nada cambió.

**Montar / desmontar.** Montar es cuando un componente aparece por primera vez en pantalla; desmontar, cuando desaparece.

**Función pura.** Una función que, con los mismos datos de entrada, siempre devuelve lo mismo y no hace nada "por fuera" (no llama a APIs, no cambia otras variables). Un reducer debe serlo.

**Payload.** El dato extra que viaja dentro de una acción, por ejemplo el producto que quieres agregar. Sale de quien llama a `dispatch` y llega al reducer.

**Prop drilling.** Pasar una prop por muchos componentes intermedios que no la usan, solo para que llegue a uno que está abajo.

**Props.** Los datos que un componente recibe de su padre.

**Provider.** El componente que "pone" un dato a disposición de todos los componentes que tenga adentro (en un Context).

**Reducer.** Una función que recibe el estado actual y una acción, y devuelve el estado nuevo. No modifica el estado que recibe.

**Referencia.** La "dirección" de un objeto o array en la memoria. Dos arrays con el mismo contenido son referencias distintas; React compara por referencia para saber si algo cambió.

**Renderizar.** Que React ejecute tu componente y dibuje en pantalla lo que devuelve. "Volver a renderizar" es ejecutarlo de nuevo con datos nuevos.

**Setter.** La función que devuelve `useState` para cambiar el estado.

**Spread (`...`).** Tres puntos que significan "copia todo lo que había". Sirve para crear un array u objeto nuevo a partir de otro: `[...lista, nuevo]`.

**Wrapper (envoltorio).** Un componente cuyo trabajo es envolver a otros para darles algo. En Context, el componente Provider suele ser un wrapper.

**Tupla.** En TypeScript, un array con una cantidad fija de posiciones y un tipo definido para cada una, como `[boolean, () => void]`.

**TypeScript.** JavaScript con tipos: te avisa de errores (por ejemplo, pasar un número donde va un texto) antes de ejecutar el programa.

**Unión discriminada.** En TypeScript, un tipo formado por varias formas posibles de un objeto, que se distinguen por un campo en común (como `type`). Es como se tipan las acciones de un reducer.
