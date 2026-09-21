# Glosario

Las palabras que aparecen en las lecciones de esta carpeta, explicadas de forma simple. Están en orden alfabético.

-----

**Actualización.** Fase del ciclo de vida en la que cambian las props o el estado (o se llama a `forceUpdate`) y React vuelve a renderizar el componente.

**Ámbito (scope).** Zona donde un nombre de clase es válido. En CSS normal, el ámbito es toda la página.

**Bundler.** Herramienta (Vite, Next.js, etc.) que procesa y empaqueta el código y los estilos antes de servirlos al navegador.

**`children`.** Prop especial que contiene el JSX colocado entre las etiquetas de apertura y cierre de un componente. Es la forma de React de crear un slot.

**Ciclo de vida.** Etapas por las que pasa una instancia de componente: montaje, actualización y desmontaje.

**Componente contenedor.** Componente que maneja estado o lógica y no define la interfaz por sí mismo. Se combina con un componente presentacional.

**Componente de clase.** Clase que extiende `React.Component` y define un método `render()`. Hoy se usa sobre todo en código legado y en error boundaries.

**Componente presentacional.** Componente que solo recibe props y devuelve JSX. No sabe de dónde vienen los datos.

**Composición.** Construir componentes complejos combinando componentes simples. En React se prefiere a la herencia.

**Compound Components.** Patrón que divide un componente en varias partes (por ejemplo `Tabs.List` y `Tabs.Panel`) que se componen con JSX y comparten estado mediante Context.

**CSS-in-JS.** Enfoque en el que el CSS se escribe dentro de archivos JavaScript. Ejemplo: styled-components.

**CSS Modules.** Archivos `.module.css` en los que el bundler reescribe cada nombre de clase para hacerlo único, de modo que los estilos queden aislados por archivo.

**Desmontaje (desmontar).** Fase del ciclo de vida en la que la instancia del componente se elimina del DOM. Una instancia desmontada no vuelve a montarse; si el componente reaparece, React crea una nueva.

**Efecto secundario.** Trabajo que no consiste en calcular el JSX, como pedir datos, iniciar temporizadores o crear suscripciones.

**Error boundary.** Componente de clase que define `getDerivedStateFromError` o `componentDidCatch` para capturar errores de renderizado de sus hijos y mostrar una interfaz alternativa.

**Estilo en línea (inline style).** Estilo que se aplica con el atributo `style` de un elemento. En React recibe un objeto con propiedades en camelCase.

**Herencia.** Reutilizar comportamiento haciendo que una clase extienda a otra. En React no se recomienda para componentes porque acopla la clase derivada a la base.

**HOC (Higher-Order Component).** Función que recibe un componente y devuelve otro componente con funcionalidad añadida. Hoy, para lógica nueva, suele preferirse un custom hook.

**Montaje (montar).** Fase del ciclo de vida en la que el componente se crea y se coloca en el DOM por primera vez.

**Prop drilling.** Pasar una prop por componentes intermedios que no la usan, solo para que llegue a uno más abajo.

**Render prop.** Prop cuyo valor es una función que devuelve JSX. El componente conserva la lógica y quien lo usa decide cómo se dibuja.

**Sass.** Preprocesador que extiende CSS con variables, anidamiento, mixins y módulos, y se compila a CSS plano antes de llegar al navegador.

**Selector de clase.** Regla CSS que se aplica a los elementos que tienen cierta clase, por ejemplo `.card`.

**Slot.** Espacio dentro de un componente donde quien lo usa decide qué contenido va. Se crea con `children` o con props que reciben JSX.

**styled-components.** Librería de CSS-in-JS que define componentes con su CSS incorporado y genera el CSS en tiempo de ejecución.

**Utility-first.** Enfoque en el que la interfaz se compone con clases pequeñas que aplican una sola propiedad cada una. Ejemplo: Tailwind CSS.

**Wrapper hell.** Anidamiento excesivo de capas que solo aportan lógica, típico al combinar varios HOC. Complica la lectura del árbol en las DevTools.
