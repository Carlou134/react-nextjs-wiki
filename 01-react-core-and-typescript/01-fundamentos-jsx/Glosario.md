# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Atributo.** Un valor que se escribe dentro de la etiqueta de un elemento, como `src` en `<img src="a.png" />`. En JSX se escribe en camelCase (`className`, `onClick`).

**Batching (agrupación de actualizaciones).** React agrupa varias actualizaciones de estado hechas en un mismo evento y las procesa en un solo render.

**Commit.** La fase en la que React aplica al DOM real los cambios que calculó. Solo se modifican los nodos que cambiaron.

**Compilador.** Una herramienta (Babel, SWC, esbuild o TypeScript) que convierte tu código en otro que el navegador sí entiende. Convierte JSX en JavaScript estándar.

**Componente.** Una función que devuelve lo que se ve en pantalla (JSX). Es la pieza básica de una app de React.

**Condicional.** Una forma de decidir qué se muestra. En JSX se usa `if` fuera del JSX, el operador ternario (`? :`) o `&&`.

**DOM.** La representación de la página que tiene el navegador: el árbol de elementos HTML que JavaScript puede leer y modificar.

**DOM virtual.** Un patrón: React describe la UI como un árbol de objetos en memoria, lo compara con el anterior y cambia en el DOM real solo las diferencias. No es una copia del DOM ni algo que uses directamente.

**Elemento de React.** Un objeto simple que describe qué debe mostrarse (tipo, props y hijos). Cada expresión JSX produce uno. No es un nodo del DOM.

**Evento.** Algo que ocurre en la página por una acción del usuario, como un clic o escribir en un campo. En JSX se enlazan con atributos como `onClick`.

**Expresión.** Código que produce un valor, como `2 + 3`, `name` o `a ? b : c`. Es lo único que cabe entre llaves en JSX.

**Fiber.** Un objeto interno de React que guarda información de un componente y su lugar en el árbol. Es un detalle de implementación: no lo usas directamente.

**Fragmento (Fragment).** Un elemento especial, `<>...</>`, que agrupa varios elementos bajo un solo elemento raíz sin añadir un nodo extra al DOM.

**Handler (manejador de evento).** Una función que React ejecuta cuando ocurre un evento. Por convención se nombra `handleAlgo` y se pasa como `onClick={handleAlgo}`.

**JSX.** Una extensión de la sintaxis de JavaScript que permite escribir etiquetas con forma de HTML. No es HTML y el navegador no la entiende sin compilar.

**Key.** Un atributo especial que identifica cada elemento de una lista entre renders. Debe ser único entre hermanos y estable; lo ideal es un `id` de tus datos.

**Llaves `{}`.** En JSX marcan el inicio y el fin de una expresión de JavaScript, ya sea como contenido entre etiquetas o como valor de un atributo.

**Nodo.** Cada pieza del árbol del DOM: un elemento, un texto, etc.

**Props.** Los datos que se le pasan a un componente mediante atributos JSX. El componente los recibe y los usa para decidir qué mostrar.

**Raíz (root).** El punto del DOM donde React toma el control y dibuja la interfaz. Se crea con `createRoot`.

**Reconciliación.** El proceso por el que React compara el árbol de elementos nuevo con el anterior para decidir qué cambiar.

**Render (renderizar).** Que React ejecute los componentes para calcular qué UI debe mostrar. Es un cálculo en memoria: no toca el DOM por sí mismo.

**Sentencia.** Una instrucción que ejecuta una acción pero no produce un valor, como `if`, `for` o `const x = 1`. No cabe entre llaves en JSX.

**Ternario.** El operador `condición ? A : B`. Es una expresión, así que se puede usar entre llaves en JSX cuando hay dos resultados posibles.

**Transformar (transpilar).** Convertir un código en otro equivalente. Con JSX, convertirlo en llamadas a funciones de JavaScript antes de que llegue al navegador.
