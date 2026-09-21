# Componentes y Props

En esta carpeta aprendes a **crear** componentes, a **combinarlos** unos dentro de otros y a **pasarles datos** con props. Es la base de todo lo que sigue: en React, una interfaz es un árbol de componentes que se comunican mediante props.

-----

## Antes de empezar

Conviene que ya hayas visto:

* JSX: [01-fundamentos-jsx](../01-fundamentos-jsx/README.md).
* Tipos básicos de TypeScript (solo para las secciones "En TypeScript"): [00-typescript-fundamentals](../00-typescript-fundamentals/README.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Tu primer componente](01-Your%20First%20React%20Component.md) | Definir un componente, exportarlo y mostrarlo en la página | JSX |
| [2. Componentes que renderizan otros componentes](02-Components%20Render%20Other%20Components.md) | Usar un componente dentro de otro y entender el árbol de componentes | Primer componente |
| [3. Props](03-Props.md) | Pasar datos y funciones de padre a hijo, y usar `children` | Componentes anidados |

-----

## El mapa completo en una mirada

```
Componente   ->  "una función que devuelve JSX"
Composición  ->  "unos componentes dentro de otros forman un árbol"
Props        ->  datos hacia abajo (padre -> hijo)
                 eventos hacia arriba (el hijo llama a una función del padre)
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
7. **En TypeScript:** lo que cambia si tu proyecto usa TypeScript.
8. **Cuándo sí y cuándo no:** para decidir si es la herramienta correcta.
9. **Resumen en 5 líneas:** para repasar.
10. **Para profundizar:** detalles avanzados, plegados. Puedes omitirlos en la primera lectura.
11. **En entrevista:** una respuesta corta (junior), una ampliada (semi-senior) y preguntas de seguimiento frecuentes.
12. **Siguiente lección:** hacia dónde continuar.

-----

## Después de esta carpeta

Con estas lecciones ya puedes construir y combinar componentes que reciben datos. El siguiente paso es preparar tu entorno de trabajo, en la carpeta de [tooling y devtools](../03-tooling-y-devtools/01-Creating%20a%20React%20App.md). Los Hooks, que le dan memoria a los componentes, vienen más adelante, en [Hooks y Context](../04-hooks-y-context/README.md).
