# Tailwind CSS

En esta carpeta aprendes a **estilar** una interfaz componiendo clases de utilidad de Tailwind CSS: cómo se instala, cómo detecta las clases que usas, cómo resuelve tipografía, espaciado, color y layout, cómo se adapta a cada pantalla y cómo se organiza todo en un proyecto real. Es el paso que te permite construir interfaces consistentes sin escribir una hoja de estilos para cada componente.

-----

## Antes de empezar

Necesitas:

* Conocer HTML y CSS básicos: qué es un selector, una clase y una propiedad.
* Haber visto la lección de estilos de React, como contexto de las distintas formas de dar estilo a un componente: [Estilos en React](../../01-react-core-and-typescript/05-patrones-estilos-lifecycle/02-React%20Styles.md).

Las lecciones usan **Tailwind v4** y señalan en cada caso las diferencias con **v3**, porque todavía es común encontrar tutoriales y proyectos de esa versión.

Si en algún momento aparece una palabra que no conoces, búscala en el [Glosario](Glosario.md).

-----

## Orden de lectura

Las lecciones se apoyan una en la otra, así que conviene leerlas en este orden:

| Lección | Qué aprendes | Qué necesitas saber antes |
| --- | --- | --- |
| [1. Fundamentos de Tailwind](01-Fundamentos%20de%20Tailwind.md) | Qué significa utility-first, cómo se compara con CSS tradicional, estilos en línea y Bootstrap, y qué son las variantes y Preflight | HTML, CSS básico y `className` en JSX |
| [2. Instalación e Integración](02-Instalaci%C3%B3n%20e%20Integraci%C3%B3n.md) | Las tres formas de ejecutar Tailwind (Play CDN, CLI y plugin del build), cómo integrarlo con Vite, Next.js y Astro, y cómo distinguir v3 de v4 | Fundamentos de Tailwind y Node.js |
| [3. Cómo Tailwind Detecta las Clases](03-C%C3%B3mo%20Tailwind%20Detecta%20las%20Clases.md) | Por qué las clases deben escribirse completas, qué archivos se escanean y cómo usar `@source` y `@source inline()` | Instalación e Integración |
| [4. Tipografía, Espaciado y Colores](04-Tipograf%C3%ADa%2C%20Espaciado%20y%20Colores.md) | La convención de nombres, las escalas de tipografía, espaciado, tamaños, bordes, sombras y colores | Detección de clases |
| [5. Layout y Responsive](05-Layout%20y%20Responsive.md) | Flexbox, Grid, contenedores, breakpoints y el enfoque mobile-first | Espaciado y detección de clases |
| [6. Tema, Modo Oscuro y Plugins](06-Tema%2C%20Modo%20Oscuro%20y%20Plugins.md) | Personalizar el tema con `@theme`, activar el modo oscuro con `dark:` y usar plugins como Typography y Forms | Colores, layout y detección de clases |
| [7. Buenas Prácticas y Flujo de Trabajo](07-Buenas%20Pr%C3%A1cticas%20y%20Flujo%20de%20Trabajo.md) | Reutilizar con componentes, combinar variantes sin conflictos, usar `@apply` con criterio, cuidar la accesibilidad y revisar código ajeno o generado por IA | Todas las lecciones anteriores |

-----

## El mapa completo en una mirada

```
1. Instalar                  ->  Play CDN, CLI o plugin del build (Vite, Next.js, Astro)
        |                        una línea en el CSS: @import "tailwindcss"
        v
2. Tailwind detecta clases   ->  escanea tus archivos como texto plano
        |                        solo genera las clases escritas completas
        v
3. Utilidades base           ->  tipografía (text-, font-)
        |                        espaciado y cajas (m-, p-, w-, border, rounded-)
        |                        colores (bg-, text-) con una escala de valores
        v
4. Layout y responsive       ->  Flexbox (una dimensión) y Grid (dos dimensiones)
        |                        prefijos mobile-first: sm:, md:, lg:
        v
5. Tema, modo oscuro, plugins -> @theme define tus valores (tokens de diseño)
        |                        dark: aplica estilos en modo oscuro
        |                        plugins como prose para texto largo
        v
6. Buenas prácticas          ->  componentes en lugar de copiar cadenas de clases
                                 cn (clsx + tailwind-merge), @apply con moderación,
                                 accesibilidad y revisión del código ajeno
```

-----

## Cómo está armada cada lección

Todas las lecciones tienen la misma estructura, así sabes dónde buscar cada cosa:

1. **En una frase:** qué es, sin jerga.
2. **Antes de empezar:** qué tienes que saber y las palabras nuevas.
3. **El problema:** una situación concreta que muestra por qué existe la herramienta.
4. **Cómo funciona:** el paso a paso, con código chico y explicado.
5. **Ejemplo completo:** todo junto en un caso real.
6. **En React:** cómo se aplica lo aprendido dentro de componentes.
7. **Errores comunes:** lo que suele salir mal, por qué pasa y cómo se arregla.
8. **Cuándo sí y cuándo no:** para decidir si es la herramienta correcta.
9. **Resumen en 5 líneas:** para repasar.
10. **Para profundizar:** detalles avanzados, plegados. Puedes omitirlos en la primera lectura.
11. **En entrevista:** una respuesta corta (junior), una ampliada (semi-senior) y preguntas de seguimiento frecuentes.
12. **Siguiente lección:** hacia dónde continuar.

-----

## Después de esta carpeta

Con estas lecciones ya sabes construir interfaces con utilidades, adaptarlas a cada pantalla y mantenerlas ordenadas. El siguiente paso es ver esa misma idea llevada a una biblioteca de componentes que viven en tu propio repositorio: [Qué es shadcn/ui](../02-shadcn-ui/01-Qu%C3%A9%20es%20shadcn-ui.md).
