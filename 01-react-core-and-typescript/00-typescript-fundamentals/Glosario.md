# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Alias de tipo (type alias).** Un nombre que le das a cualquier tipo con `type Nombre = ...`. No crea un tipo nuevo, solo un nombre para uno existente: `type Id = string | number`.

**Análisis de flujo de control.** El mecanismo del compilador que sigue `if`, `return`, `switch` y demás para saber qué tipo tiene una variable en cada punto del código.

**Anotación de tipo.** El tipo escrito de forma explícita después de `:`. Ejemplo: `let nombre: string`.

**Any.** Un tipo que desactiva el chequeo: acepta cualquier valor y permite cualquier operación. Es poco seguro porque "contagia" a lo que toca; conviene usar `unknown`.

**Argumento.** El valor concreto que se pasa al llamar una función. Ejemplo: en `saludar('Ana')`, `'Ana'` es el argumento.

**Array (arreglo).** Una lista de valores en orden. Tipado, todos los elementos son del mismo tipo: `string[]` o `Array<string>`.

**Bundler.** Una herramienta (Vite, webpack, esbuild) que junta los módulos de una app en archivos listos para el navegador.

**Callback.** Una función que le pasas a otra para que la ejecute más tarde. Ejemplo: la función que recibe `map`.

**Compilador (`tsc`).** El programa de TypeScript que revisa los tipos y, si se le pide, genera JavaScript.

**Declaration merging.** Cuando declaras dos `interface` con el mismo nombre, TypeScript las fusiona en una sola. Con `type`, repetir el nombre es un error.

**Discriminante.** Una propiedad con tipo literal, distinta en cada miembro de una unión, que identifica a qué miembro corresponde un valor. Ejemplo: `type: 'ok'` o `type: 'error'`.

**Enum.** Una forma de nombrar un conjunto de valores constantes. A diferencia de casi todo en TypeScript, genera código en ejecución; una unión de literales es una alternativa más ligera.

**Excess property check.** La revisión que rechaza propiedades sobrantes cuando asignas un literal de objeto directamente a un tipo. Si el valor viene de una variable, no se aplica.

**Firma (signature).** La parte de una función que describe sus parámetros y su tipo de retorno, sin el cuerpo. Ejemplo: `(a: number, b: number) => number`.

**Firma de índice (index signature).** La declaración de un objeto cuyas claves no se conocen de antemano, pero sí el tipo de clave y de valor: `{ [clave: string]: number }`.

**Genérico (generic).** Un tipo o función con un **parámetro de tipo** (`<T>`) que se fija al usarlo. Ejemplo: `Array<string>` fija `T` en `string`.

**Inferencia de tipos.** Cuando TypeScript deduce el tipo a partir del valor, sin que lo escribas. Ejemplo: `let cantidad = 0` se infiere como `number`.

**Interface.** Una declaración que nombra la forma de un objeto: `interface Usuario { nombre: string }`. Admite `extends` y declaration merging.

**Intersección.** Un tipo que combina varios y exige cumplir todos a la vez, con `&`. Es el "y" de los tipos: `A & B`.

**JSONC.** JSON que admite comentarios y comas finales. `tsconfig.json` funciona así.

**Keyof.** Un operador que devuelve la unión de las claves de un tipo. Ejemplo: `keyof { a: 1; b: 2 }` da `'a' | 'b'`.

**Literal (tipo literal).** Un tipo que representa un único valor concreto, como `'red'` o `404`.

**Mapped type.** Un tipo que recorre las claves de otro y construye uno nuevo a partir de ellas: `{ [K in keyof T]: ... }`. Los utility types se construyen así.

**Miembro.** Una propiedad o método de un tipo de objeto. En una unión, cada tipo que la forma también se llama miembro.

**Narrowing (estrechamiento).** La reducción de un tipo amplio a uno más específico según las comprobaciones del código. Ejemplo: tras `typeof x === 'string'`, `x` es `string` en esa rama.

**Never.** El tipo de algo que nunca produce un valor: una función que siempre lanza un error o entra en un bucle infinito. También aparece cuando TypeScript descarta todos los casos de una unión.

**Parámetro.** La variable que aparece en la declaración de una función. Ejemplo: en `function saludar(nombre: string)`, `nombre` es el parámetro.

**Readonly.** Un modificador que impide mutar un valor a través de ese tipo. Solo existe en tiempo de compilación.

**Resolución de módulos.** Las reglas con las que se decide qué archivo corresponde a un `import`.

**Satisfies.** Un operador que valida que un valor cumple un tipo sin ensanchar el tipo inferido, a diferencia de una anotación. Ejemplo: `const c = { a: 1 } satisfies Record<string, number>`.

**Sistema de tipos.** Las reglas con las que TypeScript decide si el código es coherente.

**Strict.** La opción de `tsconfig.json` que agrupa varias comprobaciones estrictas (`strictNullChecks`, `noImplicitAny`, entre otras). Se recomienda partir de `strict: true`.

**Superset (superconjunto).** TypeScript es un superset de JavaScript: todo JavaScript válido es TypeScript válido, y TypeScript solo añade cosas.

**Tipado estructural.** TypeScript compara tipos por su **forma** (las propiedades que tienen), no por el nombre con que fueron declarados.

**Tipo.** El conjunto de valores posibles y de operaciones válidas sobre ellos (`string`, `number`, `boolean`...).

**Tipo de objeto.** Un tipo que describe las propiedades de un objeto y el tipo de cada una: `{ nombre: string; edad: number }`.

**tsconfig.json.** El archivo donde se declara qué archivos forman el proyecto TypeScript y con qué reglas se revisan y se traducen.

**Transpilar.** Traducir código de un lenguaje a otro del mismo nivel (aquí, de TypeScript a JavaScript). Los tipos se eliminan en ese paso.

**Tupla.** Un array con una cantidad fija de posiciones y un tipo definido para cada una: `[string, number]`.

**Type guard (guarda de tipo).** Una expresión o función cuya comprobación en tiempo de ejecución permite a TypeScript hacer narrowing. Ejemplo: `typeof x === 'string'`.

**Type predicate.** El tipo de retorno `x is T` de una función que actúa como type guard. El compilador confía en él sin verificar el cuerpo de la función.

**Unión (union type).** Un tipo formado por varios miembros, donde un valor pertenece a al menos uno de ellos. Se escribe con `|`: `string | number`.

**Unión discriminada.** Una unión de objetos que se distinguen por un campo literal en común (el discriminante). Permite estrechar toda la forma del objeto y modelar estados y acciones.

**Unknown.** Un tipo que acepta cualquier valor, pero obliga a estrechar el tipo antes de usarlo. Es la alternativa segura a `any`.

**Utility type.** Un tipo genérico que ya viene en TypeScript y transforma otro tipo. Ejemplo: `Partial<T>`, `Pick<T, K>`, `Omit<T, K>`.

**Void.** El tipo de retorno de una función que termina, pero no devuelve un valor útil.
