# Buenas Prácticas y Mantenimiento

## Leé el código antes de usarlo

La ventaja más grande de shadcn/ui, y a la vez lo que más se desaprovecha, es que el código está frente a vos. Antes de usar un componente por primera vez, **abrí el archivo** que el CLI te generó y leelo: cuáles son sus variantes, qué primitiva usa, qué props acepta. Son archivos cortos, y con lo visto en este módulo ya tenés las herramientas para entenderlos.

Hay además dos comandos del CLI pensados para eso: `view` muestra un componente del registro **antes** de instalarlo, y `docs` trae su documentación. Y con `add <componente> --dry-run` podés ver qué haría el comando sin escribir ningún archivo.

-----

## El código es tuyo: tratalo como tal

Como los componentes viven en tu repositorio, valen las reglas de cualquier otro código propio:

* **Commitealos.** Los archivos de `components/ui/` van en git, igual que el resto. Es lo que permite ver, en un pull request, exactamente qué cambió en un componente.
* **Separá los cambios.** Cuando agregás un componente nuevo, conviene hacer un commit con solo ese componente tal como lo generó el CLI, **antes** de personalizarlo. Así, más adelante, el historial te muestra con precisión qué modificaste vos y qué venía de origen, lo que hace mucho más fácil actualizarlo (ver más abajo).
* **No lo trates como caja negra ni como intocable.** Si necesitás un cambio, hacelo. Para eso está.

-----

## Personalizar en el nivel correcto

Cuando el diseño no coincide con lo que querés, hay cuatro niveles de cambio posibles, **del más barato al más costoso**. Conviene empezar siempre por el más barato:

1. **Los tokens del tema** (variables CSS): cambiar `--primary`, `--radius`, el color base. Cambia toda la aplicación de una vez, sin tocar ningún componente. Resuelve la mayoría de los casos de "quiero que se vea como mi marca".
2. **La prop `className`** en un uso puntual: `<Button className="w-full">`. Cambia una sola instancia.
3. **Una variante nueva** en el componente base (`cva`): cuando un estilo se va a repetir en la aplicación (un `success`, un `xl`).
4. **Un componente propio que envuelve al de shadcn/ui**: cuando el caso agrega comportamiento (un botón con estado de carga, un diálogo de confirmación reutilizable).

El error común es empezar por el nivel 3 o 4 cuando el nivel 1 alcanzaba: editar componentes para cambiar un color que se resolvía con una variable.

-----

## Actualizar componentes

Como no es un paquete, los componentes **no se actualizan solos**. Si el proyecto mejora el componente `dialog` (una corrección de accesibilidad, un ajuste de estilo), tu copia no cambia hasta que vos decidas traer la novedad. Esto es una consecuencia directa de tener el control: la contracara es el mantenimiento.

El CLI ofrece las herramientas para hacerlo con criterio, según su referencia oficial:

* `add <componente> --diff` permite comparar tu versión con la del registro.
* `add <componente> --dry-run` muestra qué se escribiría, sin hacerlo.
* `add <componente> --overwrite` reemplaza tu archivo por la versión actual.

Y acá aparece el motivo del consejo de la sección anterior: `--overwrite` **pisa cualquier modificación que hayas hecho** al archivo. Por eso conviene tener el componente commiteado antes de actualizarlo: si el resultado no te gusta, `git diff` te muestra qué se perdió y podés recuperar tus cambios, y si quedó bien, ya sabés qué personalizaciones hay que volver a aplicar.

Un criterio práctico sobre **cuándo** actualizar: no hace falta hacerlo por rutina. Tiene sentido cuando hay una corrección que te afecta (un bug, un problema de accesibilidad), o cuando el proyecto cambia una convención que querés tener en todos tus componentes. Para eso último existe el comando `migrate`: por ejemplo, con el cambio de septiembre de 2026 que movió el helper `cn` a un paquete propio, `shadcn migrate cn` actualiza los proyectos existentes en lugar de tener que editar cada archivo a mano.

-----

## Cuándo conviene no usarlo

Ya lo mencionamos en la primera lección, pero vale como criterio de decisión concreto. Preguntate:

* **¿Voy a usar más de tres o cuatro componentes de este tipo?** Si la interfaz es muy simple, la configuración de Tailwind, `cva`, primitivas y alias puede ser más trabajo que escribir los pocos componentes a mano.
* **¿Alguien en el equipo se va a hacer cargo del mantenimiento?** Si nadie quiere ser dueño de esos archivos, una librería que se actualiza por `npm` puede ser más sensata.
* **¿Tengo un diseño completamente propio?** Si es así, los valores por defecto de shadcn/ui te van a estorbar más de lo que ayudan, y quizás convenga usar directamente las primitivas (Radix UI o Base UI) con tus propios estilos.

No hay una respuesta universal: shadcn/ui es una herramienta excelente para un caso específico (una aplicación React con Tailwind que va a crecer y donde importa el control del diseño), no una solución para todos los proyectos.

-----

## Problemas frecuentes

Una lista de revisión, de lo más común a lo menos común:

1. **`Cannot find module '@/components/ui/...'`.** El alias `@/` no está bien configurado. Revisá los tres lugares de la lección de instalación (`tsconfig.json`, `tsconfig.app.json` y `vite.config.ts`, en Vite), y reiniciá el servidor de desarrollo y el editor.
2. **Los componentes se ven sin estilo.** Casi siempre es un problema de Tailwind, no de shadcn/ui: revisá que el CSS con `@import "tailwindcss"` se esté importando en tu aplicación y que la ruta `tailwind.css` de `components.json` apunte al archivo correcto. Después, repasá la lista de la lección de Tailwind sobre "qué revisar cuando no se ven los estilos".
3. **Un componente nuevo no tiene el estilo que esperás.** Verificá que los tokens que usa (`bg-primary`, `text-muted-foreground`) estén definidos en tu CSS. Si agregaste un color propio, revisá que esté declarado tanto en `:root` como en `@theme inline`.
4. **El CLI dice que no encuentra `components.json`.** Falta ejecutar `init`, o estás corriendo el comando desde otra carpeta.
5. **El código de un tutorial no coincide con lo que generó el CLI.** Es esperable: el proyecto cambia rápido. Guiate por el código que **tu** CLI generó y por la documentación actual, no por el tutorial (recordá las diferencias entre Radix y Base UI, y la ubicación de `cn`).
6. **No sabés qué versión o configuración tiene tu proyecto.** `shadcn info` muestra la configuración detectada.

-----

## Un flujo de trabajo recomendado

Para un proyecto nuevo, en orden:

1. Dejá Tailwind funcionando y verificá que una clase (`text-3xl font-bold`) se aplica.
2. Ejecutá `init` y elegí el color base con criterio, porque **no se puede cambiar después**.
3. Ajustá los **tokens del tema** a tu marca antes de agregar componentes: así todo lo que agregues después ya nace con tu aspecto.
4. Agregá componentes **a medida que los necesites**, no todos de una: cada uno es código que mantener.
5. Después de cada `add`, hacé un commit con el componente sin modificar; recién después personalizalo.
6. Personalizá desde el nivel más barato (tokens) al más costoso (componente propio).

-----

## Resumen

* **Leé el código** de cada componente antes de usarlo: es corto y ahí está la documentación más precisa. `view`, `docs` y `--dry-run` ayudan a informarte antes de instalar.
* Los archivos son tuyos: **commitealos**, y hacé un commit del componente tal cual antes de personalizarlo.
* Personalizá **del nivel más barato al más costoso**: tokens del tema, `className`, una variante nueva, un componente propio.
* Los componentes **no se actualizan solos**: `--diff` compara, `--dry-run` previsualiza y `--overwrite` reemplaza pisando tus cambios; `migrate` cubre los cambios de convención.
* Es una gran herramienta para una aplicación React con Tailwind que va a crecer, no para todos los proyectos.
* Ante un problema, revisá primero Tailwind y el alias `@/`; ante un tutorial que no coincide, confiá en el código que generó tu CLI y en la documentación actual.
