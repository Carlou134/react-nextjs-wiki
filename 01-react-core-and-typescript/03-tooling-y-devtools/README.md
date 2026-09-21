# Tooling y DevTools

En esta carpeta aprendes a **crear** un proyecto de React con Vite, a **entender su estructura** y a **depurarlo** con React Developer Tools. Es el paso en que dejas de leer ejemplos sueltos y trabajas en una app real que corre en tu computadora.

-----

## Antes de empezar

Necesitas:

* Haber leído la carpeta anterior: [02-componentes-y-props](../02-componentes-y-props/README.md).
* **Node.js** instalado. Puedes comprobarlo con `node -v` en la terminal.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Crear una app de React](01-Creating%20a%20React%20App.md) | Crear un proyecto con Vite, recorrer su estructura y usar el servidor de desarrollo | Componentes y Node.js |
| [2. React Developer Tools](02-React%20Developer%20Tools.md) | Inspeccionar el árbol de componentes, sus props, estado y Hooks desde el navegador | Un proyecto corriendo y nociones de props y estado |

-----

## El mapa completo en una mirada

```
1. Crear el proyecto   ->  npm create vite@latest (React + TypeScript)
2. Estructura y dev    ->  npm install, npm run dev, cambios al guardar (HMR)
3. Depurar             ->  React DevTools: Components (árbol) y Profiler (rendimiento)
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
7. **En TypeScript:** lo que cambia si tu proyecto usa TypeScript (solo en las lecciones donde aplica).
8. **Cuándo sí y cuándo no:** para decidir si es la herramienta correcta.
9. **Resumen en 5 líneas:** para repasar.
10. **Para profundizar:** detalles avanzados, plegados. Puedes omitirlos en la primera lectura.
11. **En entrevista:** una respuesta corta (junior), una ampliada (semi-senior) y preguntas de seguimiento frecuentes.
12. **Siguiente lección:** hacia dónde continuar.

-----

## Después de esta carpeta

Con un proyecto funcionando y una forma de inspeccionarlo, ya puedes darles memoria a tus componentes. Continúa con [Hooks y Context](../04-hooks-y-context/README.md).
