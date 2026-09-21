# Patrones, estilos y ciclo de vida

En esta carpeta aprendes a **organizar** tus componentes con patrones probados, a **darles estilo** sin que unos pisen a otros y a entender el **ciclo de vida** clásico de los componentes de clase, que hoy se reemplaza con Hooks. Es el paso que convierte componentes sueltos en una interfaz ordenada y mantenible.

-----

## Antes de empezar

Conviene que ya hayas leído la carpeta anterior: [Hooks y Context](../04-hooks-y-context/README.md). Los patrones de reutilización y el ciclo de vida se apoyan en `useState`, `useEffect`, Custom Hooks y Context.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Patrones de programación](01-React%20Programming%20Patterns.md) | Separar lógica de interfaz, reutilizar lógica y estructura (HOC, render props, composición, slots, Compound Components) y evitar el prop drilling | Props, useState y Custom Hooks |
| [2. Estilos](02-React%20Styles.md) | Aplicar CSS a los componentes sin choques de nombres: `style`, `className`, CSS Modules, Sass, CSS-in-JS y Tailwind | JSX y props |
| [3. Ciclo de vida](03-Component%20Lifecycle%20Methods.md) | Los métodos de ciclo de vida de los componentes de clase y su equivalente con `useEffect` | useEffect y clases de JavaScript |

-----

## El mapa completo en una mirada

```
Patrones de composición  ->  "organizo y reutilizo componentes"
   contenedor / presentacional, composición, slots,
   HOC y render props (hoy, casi siempre un custom hook),
   Compound Components

Estilos                  ->  "decido cómo se ve cada componente"
   style, className, CSS Modules, Sass, CSS-in-JS, Tailwind

Ciclo de vida clásico    ->  "en qué momento actúa un componente de clase"
   componentDidMount     ->  useEffect(() => { ... }, [])
   componentDidUpdate    ->  useEffect(() => { ... }, [dep])
   componentWillUnmount  ->  función de limpieza del efecto
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

-----

## Después de esta carpeta

Con estas lecciones ya sabes organizar, estilar y entender el ciclo de vida de tus componentes. El siguiente paso es aprender a capturar datos del usuario, en la carpeta de formularios: [React Forms](../06-forms/01-React%20Forms.md).
