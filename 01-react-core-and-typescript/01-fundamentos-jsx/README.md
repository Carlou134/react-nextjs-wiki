# Fundamentos de JSX

En esta carpeta aprendes a **describir una interfaz con JSX**, a entender **cómo React la convierte en cambios reales en la pantalla** (el DOM virtual) y a **mezclar JavaScript con el marcado** para mostrar datos, reaccionar a clics, decidir qué se ve y repetir elementos. Es la base para escribir componentes.

-----

## Antes de empezar

Conviene que ya tengas:

* HTML básico: etiquetas, atributos y anidación.
* JavaScript básico: variables, funciones, arrays, objetos y `import`/`export`.

Opcional: si vas a usar TypeScript, las lecciones traen una sección "En TypeScript". Puedes leer antes [Fundamentos de TypeScript](../00-typescript-fundamentals/README.md), aunque no es obligatorio para seguir esta carpeta.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Intro to JSX](01-Intro%20to%20JSX.md) | Qué es JSX, en qué se transforma, sus reglas básicas y cómo mostrarlo en pantalla con `createRoot` | HTML y JavaScript básico |
| [2. The Virtual DOM](02-The%20Virtual%20Dom.md) | Cómo React calcula qué cambiar: render, reconciliación y commit | Intro to JSX |
| [3. Advanced JSX](03-Advanced%20JSX.md) | Llaves con expresiones, eventos, condicionales, listas y `key` | Intro to JSX y DOM virtual |
| [4. Use Multiline JSX in a Component](04-Use%20Multiline%20JSX%20in%20a%20Component.md) | JSX de varias líneas, lógica antes del `return` y manejadores dentro de un componente | Advanced JSX |

-----

## El mapa completo en una mirada

```
JSX            ->  "escribo la interfaz con forma de HTML dentro de JavaScript"
Elementos      ->  "el compilador lo convierte en objetos que describen la UI"
Render         ->  "React ejecuta mis componentes y obtiene un árbol nuevo"
DOM virtual    ->  "compara con el anterior y cambia solo lo que difiere"

JSX avanzado:
Expresiones    ->  {dato}          "inyecto valores de JavaScript"
Eventos        ->  onClick={fn}    "reacciono a lo que hace el usuario"
Condicionales  ->  if / ? : / &&   "decido qué se muestra"
Listas         ->  .map + key      "repito elementos a partir de un array"
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
7. **En TypeScript:** lo que cambia si tu proyecto usa TypeScript. Algunas lecciones omiten esta sección cuando no aplica.
8. **Cuándo sí y cuándo no:** para decidir si es la herramienta correcta.
9. **Resumen en 5 líneas:** para repasar.
10. **Para profundizar:** detalles avanzados, plegados. Puedes omitirlos en la primera lectura.
11. **En entrevista:** una respuesta corta (junior), una ampliada (semi-senior) y preguntas de seguimiento frecuentes.
12. **Siguiente lección:** hacia dónde continuar.

-----

## Después de esta carpeta

Con estas lecciones ya sabes escribir JSX, incluir lógica y eventos, y entiendes cómo React actualiza la pantalla. El siguiente paso es crear tus propios componentes y pasarles datos, en la carpeta de [componentes y props](../02-componentes-y-props/01-Your%20First%20React%20Component.md).
