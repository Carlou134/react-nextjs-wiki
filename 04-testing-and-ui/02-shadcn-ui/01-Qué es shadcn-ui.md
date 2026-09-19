# Qué es shadcn/ui

## No es una librería de componentes

La frase con la que shadcn/ui se presenta en su propia documentación es esta: *"Esto no es una librería de componentes. Es cómo construís tu librería de componentes."* Y es literal: no vas a encontrar un paquete `shadcn-ui` para instalar y usar con `import { Button } from "shadcn-ui"`.

Para entender qué significa, conviene comparar cómo funciona una librería tradicional y cómo funciona shadcn/ui.

**Una librería tradicional** (Material UI, Chakra y similares) se instala como dependencia. Los componentes viven dentro de `node_modules`: los usás, pero no los ves ni los controlás. Cuando el diseño te pide algo distinto de lo que la librería ofrece, tenés que pelear contra ella: sobrescribir sus estilos con CSS más específico, envolver sus componentes en los tuyos, o mezclar componentes de varias librerías. La documentación de shadcn/ui describe exactamente ese problema: terminás "envolviendo componentes de la librería, escribiendo parches para sobrescribir estilos, o mezclando componentes de distintas librerías".

**shadcn/ui** hace lo contrario: un comando (el CLI) **copia el código fuente del componente dentro de tu proyecto**, en una carpeta como `components/ui/button.tsx`. A partir de ese momento el archivo es tuyo: está en tu repositorio, lo ves, lo editás, lo versionás con git. Si querés que el botón tenga otra variante, no sobrescribís nada: abrís `button.tsx` y la agregás.

-----

## Los cinco principios

La documentación resume la filosofía en cinco ideas:

* **Open Code (código abierto en tu proyecto):** "la capa superior del código de tus componentes está abierta a modificación". Tenés el control total.
* **Composición:** todos los componentes comparten interfaces predecibles, así que aprender uno te enseña cómo funcionan los demás.
* **Distribución:** un esquema y un CLI que permiten compartir componentes entre proyectos y frameworks.
* **Valores por defecto cuidados:** un diseño bien pensado desde el primer momento, sin necesidad de configurar nada para que se vea bien.
* **Listo para IA:** el código es abierto y está en tu proyecto, así que un asistente de IA puede leerlo, entenderlo y modificarlo, algo que no puede hacer con código enterrado en `node_modules`.

-----

## De qué está hecho un componente

Un componente de shadcn/ui no es magia: combina cuatro piezas que ya conocés o vas a conocer en las próximas lecciones.

1. **Primitivas accesibles.** El comportamiento difícil de un componente interactivo (un diálogo que atrapa el foco, se cierra con `Escape` y se anuncia bien en un lector de pantalla; un menú desplegable navegable con el teclado) se apoya en una librería de **primitivas sin estilos** (*headless*). Históricamente esa base fue **Radix UI**; según el changelog oficial, desde **julio de 2026 Base UI pasó a ser la opción por defecto**, manteniendo el soporte de Radix. Esa capa se ocupa de la accesibilidad y el comportamiento, y no tiene ningún estilo propio.
2. **Tailwind CSS** para el aspecto visual, con clases escritas directamente en el componente (lo vimos en el módulo de Tailwind).
3. **class-variance-authority (`cva`)** para definir las **variantes** de un componente (`variant="outline"`, `size="lg"`) de forma ordenada.
4. **lucide-react** como librería de íconos.

Vale la pena internalizar la división de trabajo: las primitivas dan **comportamiento y accesibilidad**, Tailwind da **estilo**, y `cva` da una **API de variantes**. shadcn/ui es el pegamento que las junta bien, con buenos valores por defecto.

-----

## Qué ganás y qué cuesta

Ganás:

* **Control total.** Cualquier cambio de diseño o comportamiento es editar un archivo tuyo.
* **Sin sobrescribir estilos.** No hay especificidad CSS que pelear.
* **Aprendés cómo se construyen.** Leer `dialog.tsx` te enseña cómo se arma un componente accesible; con una librería cerrada, eso queda oculto.

Cuesta:

* **El mantenimiento es tuyo.** Si el proyecto mejora un componente, no lo recibís actualizando un paquete: vos decidís si traer ese cambio (lo vemos en la lección de buenas prácticas).
* **Más archivos en tu repositorio.** Cada componente agregado es código tuyo que revisar y mantener.
* **Exige Tailwind.** Todo el estilo está escrito con clases de Tailwind, así que sin el módulo anterior no vas a poder leer ni modificar los componentes con soltura.

### Cuándo conviene y cuándo no

Conviene cuando querés una interfaz de calidad **que sea realmente tuya**, con un diseño propio o al menos personalizable, en un proyecto que va a crecer. Es especialmente cómodo en proyectos React con Next.js o Vite.

Conviene pensarlo dos veces si necesitás muy pocos componentes (agregar Tailwind, `cva`, primitivas y configuración para un solo botón es demasiado), o si preferís que un paquete se actualice solo y no querés hacerte cargo del código de los componentes.

-----

## Qué tenés que saber antes

Para aprovechar este módulo, conviene haber pasado por cuatro temas del wiki, que shadcn/ui usa en cada componente:

* **Tailwind CSS**, en [01-tailwind-css](../01-tailwind-css/01-Fundamentos%20de%20Tailwind.md): clases, responsive, y el tema con `@theme`.
* **Composición y Compound Components**, en [Patrones de React](../../01-react-core-and-typescript/05-patrones-estilos-lifecycle/01-React%20Programming%20Patterns.md): la forma en que shadcn/ui arma componentes como `Dialog`.
* **Context**, en [React Context](../../01-react-core-and-typescript/04-hooks-y-context/04-React%20Context.md): el modo oscuro se implementa con un Provider.
* **React Hook Form y Zod**, en [Forms](../../01-react-core-and-typescript/06-forms/03-React%20Hook%20Form%20y%20Zod.md): los formularios de shadcn/ui se construyen sobre esas librerías.

-----

## Un proyecto que se mueve rápido

shadcn/ui cambia con frecuencia, y conviene saberlo antes de copiar un tutorial. Algunos hitos del changelog oficial:

| Fecha | Cambio |
| --- | --- |
| Febrero de 2025 | Soporte para Tailwind v4 y React 19 |
| Febrero de 2026 | Paquete unificado `radix-ui` |
| Julio de 2026 | Base UI como base por defecto, junto al soporte de Radix |
| Septiembre de 2026 | El helper `cn` pasa a ser un paquete propio |

Una consecuencia práctica: un tutorial de hace un año puede mostrar imports o comandos que ya cambiaron. La regla es la misma que con Tailwind: ante la duda, **la documentación oficial** (`ui.shadcn.com`) manda, y el código que el CLI te genera hoy es la mejor referencia de cómo se escribe hoy.

-----

## Resumen

* shadcn/ui **no es una librería instalable**: un CLI copia el código de los componentes a tu proyecto, y ese código pasa a ser tuyo.
* Sus cinco principios: código abierto en tu proyecto, composición, distribución, buenos valores por defecto, y listo para IA.
* Un componente combina **primitivas accesibles** (Radix UI o Base UI), **Tailwind** para el estilo, **`cva`** para las variantes y **lucide-react** para los íconos.
* Ganás control total y aprendizaje; a cambio, el mantenimiento y las actualizaciones son responsabilidad tuya.
* Requiere saber Tailwind. Es un proyecto que evoluciona rápido: verificá siempre contra la documentación actual.
