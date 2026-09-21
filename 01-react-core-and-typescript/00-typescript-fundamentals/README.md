# Fundamentos de TypeScript

En esta carpeta aprendes a **describir tus datos con tipos** para que el compilador detecte errores antes de ejecutar el programa. Verás desde los tipos básicos hasta los tipos avanzados de objetos, y cómo se configura todo con `tsconfig.json`. Es la base para escribir componentes de React sin sorpresas.

-----

## Antes de empezar

Conviene que ya tengas:

* JavaScript básico: variables, funciones, arrays, objetos y `import`/`export`.
* Node.js instalado, si quieres compilar y probar los ejemplos en tu equipo (`tsc` se instala con `npm`).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Types](01-Types.md) | Los tipos básicos, la inferencia, las anotaciones, `any`, `unknown`, `void` y `never` | JavaScript básico |
| [2. Archivo tsconfig](02-Archivo%20tsconfig.md) | Configurar el compilador: alcance, rigor (`strict`), salida y módulos | Tipos básicos |
| [3. Functions](03-Functions.md) | Tipar parámetros, retornos, callbacks y sobrecargas | Tipos básicos |
| [4. Arrays](04-Arrays.md) | Tipar listas (`T[]`), tuplas y `readonly` | Tipos básicos y funciones |
| [5. Custom Types](05-Custom%20Types.md) | Crear tus propios tipos con `type`, `interface`, intersecciones y firmas de índice | Tipos básicos y arrays |
| [6. Union Types](06-Union%20Types.md) | Valores que pueden ser de varios tipos, literales y uniones discriminadas | Tipos propios |
| [7. Type Narrowing](07-Type%20Narrowing.md) | Estrechar una unión con comprobaciones (`typeof`, `in`, type guards) | Uniones |
| [8. Advanced Object Types](08-Advanced%20Object%20Types.md) | Derivar tipos con utility types, `keyof`, genéricos, mapped types y `satisfies` | Tipos propios y uniones |

-----

## El mapa completo en una mirada

```
Tipos básicos    ->  "digo qué clase de valor es cada dato"
Funciones        ->  "tipo lo que entra y lo que sale"
Arrays y tuplas  ->  "tipo listas de valores"
Tipos propios    ->  "nombro la forma de mis objetos (type, interface)"
Uniones          ->  "un valor puede ser de varios tipos"
Narrowing        ->  "compruebo cuál es y el compilador me deja usarlo"
Tipos avanzados  ->  "derivo tipos nuevos a partir de los que ya tengo"

tsconfig.json    ->  "define con qué reglas se revisa todo lo anterior"
```

-----

## Cómo está armada cada lección

Todas las lecciones tienen la misma estructura, así sabes dónde buscar cada cosa:

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El problema:** una situación concreta que muestra por qué existe la herramienta.
4. **Cómo funciona:** el paso a paso, con código chico y explicado.
5. **Ejemplo completo:** todo junto en un caso real.
6. **Errores comunes:** lo que suele salir mal, por qué pasa y cómo se arregla.
7. **Cuándo sí y cuándo no:** para decidir si es la herramienta correcta.
8. **Resumen en 5 líneas:** para repasar.
9. **Para profundizar:** detalles avanzados, plegados. Puedes omitirlos en la primera lectura.
10. **En entrevista:** una respuesta corta (junior), una ampliada (semi-senior) y preguntas de seguimiento frecuentes.
11. **Siguiente lección:** hacia dónde continuar.

-----

## Después de esta carpeta

Con estas lecciones ya puedes describir y proteger tus datos con tipos. El siguiente paso es aprender a construir interfaces con la carpeta de [fundamentos de JSX](../01-fundamentos-jsx/01-Intro%20to%20JSX.md).
