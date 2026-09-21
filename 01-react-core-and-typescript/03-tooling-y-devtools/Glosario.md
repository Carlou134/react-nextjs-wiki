# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Árbol de componentes.** La jerarquía de componentes de React que forman la aplicación, con `App` como raíz.

**Build.** El proceso que genera los archivos optimizados para producción. En Vite se ejecuta con `npm run build` y el resultado queda en `dist/`.

**Build de desarrollo / de producción.** El de desarrollo incluye advertencias e información de depuración. El de producción está optimizado y no las incluye.

**Bundler.** Herramienta que toma muchos archivos de código y sus dependencias, y los combina y optimiza para el navegador.

**Components.** Pestaña de React Developer Tools que muestra el árbol de componentes y permite inspeccionar sus props, estado y Hooks.

**Dependencia.** Paquete de código externo que tu proyecto necesita, como `react` o `react-dom`. Se declara en `package.json` y se instala en `node_modules/`.

**Dev server (servidor de desarrollo).** Servidor local que sirve la app mientras la programas. En Vite se inicia con `npm run dev`.

**DevTools (herramientas de desarrollo).** Panel del navegador para inspeccionar y depurar una página. React Developer Tools agrega sus pestañas a este panel.

**ESM (módulos ES).** El sistema de módulos nativo de JavaScript, basado en `import` y `export`. Vite lo usa en desarrollo para servir el código al navegador.

**Extensión del navegador.** Programa que se instala en el navegador para agregarle funciones. React Developer Tools es una extensión para Chrome, Firefox y Edge.

**HMR (Hot Module Replacement).** Actualización de solo los módulos modificados en el navegador, sin recargar la página completa.

**Node.js.** Entorno que ejecuta JavaScript fuera del navegador. Vite y npm funcionan sobre él.

**node_modules.** Carpeta donde npm instala las dependencias del proyecto. No se sube a Git.

**Nodo del host.** Elemento del DOM real (`div`, `button`...), a diferencia de un componente de React.

**npm.** El gestor de paquetes que se instala junto con Node.js. Descarga dependencias y ejecuta los scripts del proyecto.

**package.json.** Archivo de la raíz del proyecto que describe sus dependencias y scripts.

**Plantilla (boilerplate).** Proyecto inicial ya armado que sirve de punto de partida. Vite ofrece varias, como `react-ts`.

**Preview.** Comando (`npm run preview`) que sirve localmente el contenido de `dist/` para probarlo. No es un servidor de producción.

**Profiler.** Pestaña de React Developer Tools que registra renders para medir cuánto tardan y por qué ocurrieron.

**Scaffolding.** Generar la estructura inicial de un proyecto a partir de una plantilla.

**Script.** Comando con nombre definido en `package.json`, que se ejecuta con `npm run`. Ejemplos: `dev`, `build`, `preview`.

**SPA (Single Page Application).** App que carga una sola página HTML y actualiza la interfaz con JavaScript.

**Vite.** Herramienta de construcción que crea la estructura del proyecto, ofrece un servidor de desarrollo con HMR y genera los archivos para producción.
