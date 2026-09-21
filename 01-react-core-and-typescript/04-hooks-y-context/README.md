# Hooks y Context

En esta carpeta aprendes a darle **memoria** a tus componentes, a hacer que **reaccionen** a lo que pasa, a **reutilizar** lógica y a **compartir** datos entre componentes lejanos. Es el paso que convierte una página estática en una aplicación que responde al usuario.

-----

## Antes de empezar

Conviene que ya hayas visto:

* Componentes de función y JSX: [01-fundamentos-jsx](../01-fundamentos-jsx/01-Intro%20to%20JSX.md).
* Props (cómo un componente recibe datos): [Props](../02-componentes-y-props/03-Props.md).

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. useState](01-The%20State%20Hook.md) | Guardar datos que cambian y hacer que la pantalla los muestre | Componentes y props |
| [2. useEffect](02-The%20Effect%20Hook.md) | Ejecutar código cuando algo pasa (pedir datos, temporizadores, título de la página) | useState |
| [3. Custom Hooks](03-Custom%20Hooks.md) | Guardar tu lógica en una función para reutilizarla | useState y useEffect |
| [4. Context](04-React%20Context.md) | Compartir datos entre componentes sin pasarlos por props | useState |
| [5. useReducer](05-useReducer.md) | Ordenar cambios de estado complejos, y combinarlo con Context | useState y Context |

-----

## El mapa completo en una mirada

```
useState      ->  "mi componente recuerda un dato"
useEffect     ->  "mi componente hace algo cuando algo cambia"
Custom Hook   ->  "guardo esa lógica en una función para no repetirla"
Context       ->  "comparto un dato con muchos componentes a la vez"
useReducer    ->  "ordeno los cambios cuando el estado se complica"
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
10. **Para profundizar:** detalles avanzados, plegados. Puedes saltearlos la primera vez.

-----

## Después de esta carpeta

Con estas lecciones ya puedes construir componentes que recuerdan, reaccionan y comparten datos. El siguiente paso es aprender a organizar todo eso con orden, en la carpeta de [patrones, estilos y ciclo de vida](../05-patrones-estilos-lifecycle/01-React%20Programming%20Patterns.md).
