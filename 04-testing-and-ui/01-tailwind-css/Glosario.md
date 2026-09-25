# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**`@apply`.** Directiva de Tailwind que copia las declaraciones de varias utilidades dentro de una regla CSS propia. Se reserva para HTML plano, contenido que no controlas o estilos de librerías externas; en componentes suele ser mejor extraer un componente.

**`aspect-ratio`.** Propiedad CSS que mantiene la proporción de un elemento cuando cambia su ancho. En Tailwind se usa con clases como `aspect-video` (`16 / 9`) o `aspect-square` (`1 / 1`).

**Breakpoint.** Ancho de pantalla a partir del cual un prefijo empieza a aplicar. Por ejemplo, `md:` actúa desde `48rem` (768px con el tamaño de fuente por defecto).

**Candidato.** Fragmento de texto de tus archivos que podría ser una clase. Tailwind lo compara con las utilidades que conoce y lo descarta si no coincide.

**Clase dinámica.** Nombre de clase armado en tiempo de ejecución, por concatenación o interpolación, como `` `bg-${color}-600` ``. Tailwind no la detecta porque no ejecuta tu código.

**CLI de Tailwind.** Programa de línea de comandos (`@tailwindcss/cli` en v4) que lee un CSS de entrada, escanea tus archivos y escribe el CSS de salida. Es una opción para proyectos HTML sin framework.

**`clsx`.** Utilidad pequeña que arma un string de clases a partir de valores condicionales (strings, objetos, arrays).

**`cn`.** Función auxiliar, habitual en proyectos con Tailwind, que combina `clsx` y `tailwind-merge` para armar una lista de clases condicionales y resolver sus conflictos.

**Conflicto de clases.** Situación en que dos clases definen la misma propiedad CSS sobre el mismo elemento, como `p-2` y `p-4`. El orden dentro de `className` no decide cuál gana.

**`container`.** Clase de Tailwind que fija el ancho máximo del elemento al breakpoint activo y aplica `width: 100%`. En v4 no se centra ni añade padding por sí sola.

**Contenedor flex / grid.** Elemento que tiene `flex` o `grid`. Sus hijos directos son los ítems y se organizan según las reglas de ese layout.

**CSS de entrada.** Archivo CSS que escribes tú y que contiene `@import "tailwindcss"`. Es el punto de partida de la compilación.

**CSS de salida.** Archivo CSS que genera Tailwind con solo las reglas de las clases que usaste. Es el que carga el navegador.

**`@custom-variant`.** Directiva que define o redefine una variante. Se usa, por ejemplo, para que `dark:` dependa de una clase en lugar de la preferencia del sistema.

**`dark:`.** Variante que aplica un estilo solo en modo oscuro. Por defecto se basa en `prefers-color-scheme: dark`.

**Eje principal y eje transversal.** En Flexbox, el eje principal es la dirección en que se colocan los ítems (horizontal con `flex-row`, vertical con `flex-col`) y el transversal es su perpendicular. `justify-*` alinea sobre el principal e `items-*` sobre el transversal.

**Escala.** Conjunto ordenado de valores predefinidos (tamaños, espaciados, colores) entre los que eliges en lugar de inventar números. Los valores salen del tema.

**Escaneo de código fuente.** Lectura de tus archivos como texto plano para descubrir qué clases se usan. No se ejecuta ningún código.

**Flexbox.** Modelo de layout de una dimensión que organiza los ítems en una fila o una columna. En Tailwind se activa con `flex`.

**FOUC (flash of unstyled content).** Parpadeo que ocurre cuando la página se pinta con un estilo y después cambia a otro, por ejemplo del tema claro al oscuro.

**Grid.** Modelo de layout de dos dimensiones que organiza los ítems en filas y columnas. En Tailwind se activa con `grid`, y `grid-cols-N` define `N` columnas de igual ancho.

**Herramienta de build.** Programa que transforma tus archivos fuente en archivos listos para el navegador antes de que la app se ejecute. Tailwind funciona como una de ellas.

**Ítem.** Hijo directo de un contenedor flex o grid.

**Mapa de clases.** Objeto o tabla donde cada opción guarda su clase escrita completa, por ejemplo `primary: "bg-blue-600 text-white"`. Es la forma segura de elegir clases según una prop.

**Mobile-first.** Enfoque en el que el estilo base es el del móvil y los prefijos lo sobrescriben en pantallas más anchas. Por eso `sm:` significa "desde `sm` en adelante", no "solo en móvil".

**Modelo de caja.** En CSS, cada elemento es una caja formada por contenido, padding, borde y margin.

**Modo oscuro.** Versión de la interfaz con colores oscuros. En Tailwind se escribe con la variante `dark:` y puede depender de la preferencia del sistema o de una clase que tú controlas.

**Namespace (espacio de nombres).** Prefijo de una variable de tema, como `--color-*` o `--breakpoint-*`, que determina qué utilidades genera.

**Opacidad.** Cuánto deja ver lo que hay detrás de un color. `100%` es sólido y `0%`, invisible.

**Play CDN.** Forma de usar Tailwind con una etiqueta `<script>` que compila las clases en el navegador. Es solo para probar, no para producción.

**Plugin.** Paquete que agrega utilidades, componentes o estilos base a Tailwind, como `@tailwindcss/typography` y `@tailwindcss/forms`. También se llama así a la pieza que conecta Tailwind con el build de un framework.

**`@plugin`.** Directiva de v4 que activa un plugin desde el CSS. En v3 se activaba en el arreglo `plugins` del archivo de configuración.

**PostCSS.** Herramienta que procesa CSS mediante plugins. Next.js la usa para transformar el CSS, y ahí se conecta Tailwind.

**`prefers-color-scheme`.** Media query que refleja si el sistema operativo del usuario prefiere el tema claro o el oscuro. Es la base del `dark:` por defecto.

**Preflight.** Hoja de estilos base que Tailwind incluye al importarlo. Resetea los estilos por defecto del navegador: quita márgenes, hace que los títulos hereden tamaño y grosor, y elimina las viñetas de las listas.

**`prose`.** Clase del plugin Typography que se aplica una sola vez en el contenedor y da formato legible a todo el HTML sin clases que tenga adentro, como el que viene de Markdown o de un CMS.

**`@reference`.** Directiva que importa el tema y las utilidades en un archivo CSS separado (por ejemplo, CSS Modules) para poder usar `@apply` allí.

**`rem`.** Unidad relativa al tamaño de fuente base del navegador, normalmente 16px. `1rem` equivale a 16px salvo que el usuario cambie esa base.

**Safelist.** Lista de clases que se deben generar aunque no aparezcan en ningún archivo. En v3 se declaraba con `safelist`; en v4 se reemplaza por `@source inline()`.

**Servidor de desarrollo (dev server).** Servidor local que sirve la app mientras la programas y recarga el navegador cuando guardas.

**`@source`.** Directiva que agrega rutas al escaneo de clases, por ejemplo una librería dentro de `node_modules`. Las rutas son relativas al archivo CSS donde se declara.

**`@source inline()`.** Forma de `@source` que le indica a Tailwind que genere ciertas clases sin buscarlas en el código. Es útil cuando los nombres llegan desde una base de datos o un CMS.

**Tailwind CSS.** Framework de CSS utility-first. Es una herramienta de build que lee tus archivos, detecta las clases que usaste y genera un CSS con solo esas reglas.

**`tailwind-merge`.** Librería (función `twMerge`) que resuelve conflictos entre clases de Tailwind: cuando dos clases afectan la misma propiedad, conserva la última.

**Tema (theme).** Conjunto de valores predefinidos (colores, fuentes, espaciados, breakpoints) de los que Tailwind genera sus utilidades. En v4 se define en el CSS con `@theme`.

**`@theme`.** Directiva de v4 que declara las variables del tema. Cada variable crea una variable CSS y las utilidades que corresponden a su prefijo, algo que `:root` no hace.

**Token de diseño (design token).** Valor de diseño con nombre (un color, un espaciado, una tipografía) definido en un solo lugar. En Tailwind v4 se declaran como variables de tema con `@theme`.

**Utilidad.** Clase que aplica una sola declaración CSS o un grupo muy pequeño de ellas, por ejemplo `text-center`.

**Utility-first.** Enfoque en el que la interfaz se construye con clases de propósito único (utilidades) escritas en el marcado, sin inventar una clase con nombre propio para cada componente.

**Valor arbitrario.** Valor puntual entre corchetes, como `w-[137px]`, que no pertenece a la escala.

**Variable de tema.** Variable CSS declarada dentro de `@theme`. Su prefijo (`--color-`, `--font-`, etc.) indica qué utilidades genera.

**Variante.** Prefijo que hace condicional a una utilidad, por ejemplo `hover:`, `md:` o `dark:`. La clase con variante solo aporta el estilo de esa condición.

**Variante de componente.** Versión de un componente que cambia su aspecto según una prop, por ejemplo un botón `primary` o `secondary`. No es lo mismo que una variante de Tailwind.

**Viewport.** Área visible de la ventana del navegador.

**Watch mode (modo watch).** Modo en que un programa queda ejecutándose y repite su trabajo cada vez que guardas un archivo. Por ejemplo, la CLI con `--watch`.
