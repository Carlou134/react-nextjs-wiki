# React Developer Tools

## Use React Developer Tools to debug your React applications.

Depurar aplicaciones de manera efectiva es una piedra angular de la programación. Después de crear una aplicación en React, un siguiente paso importante es configurar tu entorno para poder depurarla. En este artículo cubriremos los conceptos básicos utilizando el esqueleto inicial creado con **create-react-app**.

Este artículo asume que estás familiarizado con **create-react-app** y con **Chrome DevTools**. También vamos a tocar brevemente el tema del **estado (state)** y las **props** dentro de un componente, por lo que es recomendable que tengas una comprensión básica de estos conceptos antes de continuar con el artículo.

También hemos incluido este artículo en formato de video. Puedes verlo aquí o desplazarte hacia abajo para seguir leyendo.

[Video](youtube.com/watch?v=fXRB6wgeKOo&embeds_referring_euri=https%3A%2F%2Fwww.codecademy.com%2F&embeds_referring_origin=https%3A%2F%2Fwww.codecademy.com&source_ve_path=MjM4NTE)

### **1. Instalar React Developer Tools**

Facebook creó una extensión para Chrome que ayuda a depurar aplicaciones de React. Se llama **React Developer Tools** y permite a los desarrolladores inspeccionar componentes de React, ver sus propiedades e interactuar con ellos mientras observan la aplicación en Google Chrome. Puedes añadir esta funcionalidad a Chrome yendo a la página de la extensión, seleccionando **“Add to Chrome”** y siguiendo las indicaciones de instalación.

![react-devtools](/Images/react-dev-tools-install.webp)

### **2. Inspeccionar componentes de React**

Con la extensión instalada, si inicias tu aplicación de React (`npm start`) y visitas el sitio en Chrome, el ícono de **React Developer Tools** en la barra de menú de Chrome debería cambiar de inactivo a activo.

![react-inactive](/Images/react_devtools_inactive.webp)

![react-active](/Images/react_devtools_active.webp)

Esto indica que el sitio que estás navegando es una aplicación de React en modo de desarrollo. Cuando una página utiliza la versión de producción de React, el ícono se verá de forma diferente.

![react-icon](/Images/react-dev-tools-production-icon.webp)

Para abrir **React Developer Tools**, primero abre **Chrome DevTools** (View > Developer > Developer Tools). En la misma barra donde aparecen **Elements**, **Sources**, **Console**, etc., habrá dos nuevas pestañas de React: **Components** y **Profiler**. Estas dos pestañas solo aparecerán en sitios que usen React. (Si no son visibles, deberás hacer clic en la flecha para expandir la selección de pestañas).

Haz clic en la pestaña **Components**.

![react-devtools](/Images/react-dev-tools-open.webp)

En este momento, todo lo que podemos ver es **App**. ¡Pero también queremos ver el contenido de **App**!

En la imagen de arriba, verás que el cursor está apuntando a un ícono de engranaje. Haz clic en el ícono de engranaje para abrir la configuración y luego haz clic en la pestaña **Components** en la ventana emergente.

![react-filter](/Images/react-dev-tools-components-filter.webp)

De forma predeterminada, hay un filtro que hace que los nodos del host (DOM) estén ocultos. Elimina este filtro por ahora y luego cierra la ventana de configuración. ¡Siempre puedes volver a la configuración y aplicar los filtros que prefieras!

Ahora verás un árbol con el contenido de **App**. Al pasar el cursor sobre los elementos del lado izquierdo, estos se resaltan en la vista renderizada, de manera similar a **Chrome DevTools**. Si haces clic en los elementos del lado izquierdo de la ventana, sus propiedades se mostrarán en el lado derecho. (Si tus Chrome DevTools aparecen en forma vertical en el lado izquierdo o derecho de la ventana, **App** y su contenido aparecerán en la parte superior, y sus propiedades se mostrarán debajo).

![react-devtools](/Images/react-dev-tools-content-tree.webp)

También puedes usar la barra de búsqueda para localizar elementos por nombre.

![react-devtools-search](/Images/react-dev-tools-search.webp)

Si ya has usado **React Developer Tools** antes, puede que notes que esto se ve un poco diferente a lo que recuerdas. Si ese es el caso, quizá quieras revisar la [documentación oficial de React](https://legacy.reactjs.org/blog/2019/08/15/new-react-devtools.html) para ver qué ha cambiado o cómo volver a la versión anterior si te resulta más cómoda.

### **3. Modificar componentes con JavaScript**

Con **React Developer Tools** y la consola, es posible modificar componentes de React ya renderizados. Esto te permite experimentar cambiando valores de los componentes, llamando métodos y probando la interacción entre componentes.

Puedes acceder y actualizar el **state** y las **props** de un componente dentro de la pestaña **Components**. Haz clic y edita las props y el state desde el lado derecho. Para que el state aparezca, primero necesitarás inicializar el componente con algún estado dentro de tus archivos.

![props-state](/Images/react-dev-tools-edit-props-state.webp)

¡También funciona con **React Hooks**!

![react-hooks](/Images/react-dev-tools-edit-hooks.webp)

También puedes hacer esto seleccionando el componente, cambiando a la vista de la consola y accediendo al componente usando **$r**. Al hacer `console.log($r)`, podrás ver que efectivamente se trata del componente seleccionado.

![react-console](/Images/react-dev-tools-console.webp)

Con estas herramientas, ¡ya estás listo para comenzar a depurar aplicaciones de React!

Para más detalles y práctica sobre cómo usar las herramientas actualizadas, revisa este [tutorial interactivo](https://react-devtools-tutorial.vercel.app/).

----