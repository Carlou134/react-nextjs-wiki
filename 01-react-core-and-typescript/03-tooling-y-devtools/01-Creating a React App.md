# Creating a React App

## Usa **Vite** para crear fácilmente una aplicación de React en tu propia computadora.

React es una poderosa biblioteca de JavaScript desarrollada por Meta (antes Facebook) para construir **interfaces de usuario dinámicas**. Su arquitectura basada en componentes y su renderizado eficiente han impulsado su adopción masiva entre los desarrolladores, ubicándola como el **segundo framework web más amado** en la encuesta a desarrolladores de StackOverflow 2024. Este artículo te guiará para configurar tu primera aplicación de React y asume que estás familiarizado con editores de texto y la navegación por la línea de comandos.

**Nota:** Anteriormente, recomendábamos usar **create-react-app (CRA)** para iniciar un proyecto de React. Desde entonces, el equipo de React ha descontinuado esta herramienta. Este artículo se centra en usar **Vite**, pronunciado “veet”, la herramienta de construcción moderna que se ha convertido en la solución ligera preferida para el desarrollo de React sin framework pesado.

### Preparándose

Usaremos el **Node Package Manager (npm)**, por lo que necesitarás tener **Node** instalado en tu computadora. Para verificar si tienes Node instalado, ejecuta este comando en tu terminal:

```bash
node -v
```

Si tienes Node instalado, este comando devolverá un número de versión, como `v23.1.0`.

Si aún no está instalado, sigue los pasos en **Configuración de Node localmente** antes de continuar.

Para usar **Vite**, necesitarás **Node v18+ o v20+**, así que asegúrate de instalar la versión correcta antes de seguir.

Cuando instalas Node, automáticamente también se instala **npm** en tu computadora. Sin embargo, npm es independiente de Node.js y tiende a actualizarse con más frecuencia. Como resultado, incluso si acabas de instalar Node (y por lo tanto npm), es buena idea actualizar npm. ¡Afortunadamente, npm sabe cómo actualizarse a sí mismo!

Para actualizar a la versión más reciente de npm en sistemas tipo *nix (OSX, Linux, etc.), puedes ejecutar este comando en tu terminal:

```bash
sudo npm install -g npm@latest
```

Para actualizar en Windows, sigue los pasos que se encuentran en la **documentación de npm**.

-----


### Configurando la Aplicación Base (Boilerplate)

Aunque es posible crear una aplicación de React manualmente, usar una herramienta de construcción como **Vite** nos permite generar rápidamente una aplicación base de React con toda la configuración necesaria.

Usar **Vite** ofrece varios beneficios:

* Una estructura de proyecto consistente que reconocerás en diferentes proyectos de React.
* Un proceso de construcción optimizado desde el inicio.
* Un servidor de desarrollo rápido con **hot module replacement** (recarga en caliente de módulos).
* Herramientas modernas de frontend con **configuración mínima**.

Para crear un nuevo proyecto con Vite, puedes usar el comando **create** de npm, que ejecuta el paquete **create-vite** sin necesidad de instalarlo globalmente primero. Al ejecutar el comando, npm pedirá tu permiso para descargar y ejecutar el paquete. Este enfoque mantiene tus dependencias globales limpias y asegura que siempre uses la última versión de la herramienta.

Abre tu terminal y ejecuta uno de los siguientes comandos según tu gestor de paquetes preferido:

**Con npm:**

```bash
npm create vite@latest
```

**Con yarn:**

```bash
yarn create vite
```

**Con pnpm:**

```bash
pnpm create vite
```

Después de ejecutar el comando, sigue las indicaciones interactivas:

1. Ingresa el **nombre de tu proyecto** (por ejemplo, `my-react-app`).
2. Selecciona **React** como tu framework.
3. Elige tu **variante preferida** (JavaScript o TypeScript).

Usaremos la variante **JavaScript**, que no tiene comprobación de tipos. Localmente, debes usar la que sea más adecuada para tu caso.

También puedes especificar estas opciones directamente en el comando:

**npm:**

```bash
npm create vite@latest my-react-app -- --template react
```

**yarn:**

```bash
yarn create vite my-react-app --template react
```

**pnpm:**

```bash
pnpm create vite my-react-app --template react
```

Al completar el proceso, recibirás un resumen de tus selecciones y la ubicación del proyecto:

```
> npx
> create-vite

│
◇  Project name:
│  my-react-app
│
◇  Select a framework:
│  React
│
◇  Select a variant:
│  JavaScript
│
◇  Scaffolding project in /Users/codecademy/Desktop/vite/my-react-app...
│
└  Done. Now run:

 cd my-react-app
 npm install
 npm run dev
```

Siguiendo las instrucciones, cambia al directorio de tu proyecto (`cd my-react-app`) y luego ejecuta el comando para instalar las dependencias (`npm install`, `yarn`, o `pnpm install`). Esto variará según el gestor de paquetes que hayas elegido.

Antes de ejecutar la aplicación, echemos un vistazo a la **estructura del proyecto** para ver qué ha creado Vite para nosotros.

-----

### Estructura de una Aplicación React

Abre la aplicación usando el editor de texto de tu preferencia. Deberías ver la siguiente estructura de archivos:

```
└── 📁my-react-app
    ├── 📁public
    │   └── vite.svg
    ├── 📁src
    │   ├── App.css
    │   ├── App.jsx
    │   ├── 📁assets
    │   │   └── react.svg
    │   ├── index.css
    │   └── main.jsx
    ├── .gitignore
    ├── eslint.config.js
    ├── index.html
    ├── package-lock.json
    ├── package.json
    ├── README.md
    └── vite.config.js
```

Vite se ha encargado de configurar la **estructura principal de la aplicación** y varios ajustes para desarrolladores. La mayoría de lo que ves **no será visible para los visitantes** de tu aplicación web. Vite utiliza un sistema de construcción moderno que transforma estos directorios y archivos en **activos estáticos optimizados**. Los visitantes de tu sitio recibirán esos activos estáticos.

No te preocupes si no sabes mucho sobre herramientas de construcción. Uno de los beneficios de usar Vite para configurar nuestra aplicación React es que podemos **evitar cualquier configuración manual del proceso de build**. Vite maneja esta complejidad por nosotros, ofreciendo un desarrollo extremadamente rápido y builds de producción optimizados.

Vamos a explorar qué hace cada archivo y directorio:

---

### Directorios Clave

**/node_modules**
Este directorio contiene todas las dependencias y sub-dependencias especificadas en tu archivo `package.json`. No necesitas modificar nada aquí, y se agrega automáticamente a `.gitignore` ya que **estos archivos no necesitan ser subidos al repositorio**.

**/public**
Este directorio contiene **activos estáticos** que se servirán directamente sin ser procesados por Vite. Los archivos en este directorio se copiarán al directorio de build durante la producción. Contiene:

* `vite.svg` – El logo de Vite, que puedes reemplazar con tu propio favicon.

**/src**
Aquí vive el **código de tu aplicación React** y es donde pasarás la mayor parte del tiempo. La estructura inicial incluye:

* **/assets** – un directorio para almacenar activos estáticos como imágenes que serán procesadas por Vite.
* **App.jsx** – el componente principal de React de tu aplicación.
* **App.css** – estilos para el componente App.
* **main.jsx** – el punto de entrada de tu aplicación React (este archivo cumple el mismo propósito que `index.js` o `index.jsx` en otras configuraciones de React).
* **index.css** – estilos globales para tu aplicación.

**Nota:** A lo largo de Learn React, nos referiremos a `index.jsx` como el archivo de punto de entrada, lo cual es común en muchos proyectos de React. En la plantilla predeterminada de Vite, este archivo se llama `main.jsx`, pero cumple **exactamente el mismo propósito**. Si prefieres seguir las lecciones al pie de la letra, puedes **renombrar `main.jsx` a `index.jsx`** y actualizar la referencia del script en `index.html` en consecuencia.

-----

### Archivos Clave

**index.html**
Ubicado en el directorio raíz, este archivo HTML sirve como **punto de entrada** para tu aplicación. Vite inyecta tu JavaScript en él durante el proceso de build. Puedes editar este archivo directamente para **agregar meta tags, cambiar el título de la página o incluir scripts o estilos adicionales**.

**package.json**
Este archivo describe todas las configuraciones de tu aplicación React, incluyendo:

* **Dependencias** necesarias para la aplicación.
* **Scripts** para desarrollo, build y previsualización de tu app.
* **Otros metadatos** sobre tu proyecto.

Aquí encontrarás scripts importantes como:

* **dev**: inicia el servidor de desarrollo.
* **build**: crea una versión lista para producción.
* **preview**: sirve la build de producción localmente para pruebas.

**vite.config.js**
Este es el archivo de configuración de Vite, donde puedes personalizar el comportamiento de la herramienta de construcción. Por defecto, solo incluye el plugin de React, pero puedes extenderlo para agregar características como:

* **Alias de rutas** para imports.
* **Opciones de build personalizadas.**
* **Plugins adicionales.**
* **Configuración de proxy** para solicitudes a APIs.

**eslint.config.js**
Este archivo configura **ESLint** para tu proyecto, lo que ayuda a detectar errores y mantener un estilo de código consistente. La plantilla de Vite incluye una configuración básica que extiende las reglas recomendadas para React.

**.gitignore**
Este archivo indica a Git **qué archivos y directorios ignorar** al hacer commits, como `node_modules`, directorios de build y archivos de entorno.

A medida que tu aplicación React crezca, es común agregar un directorio **/components** para organizar componentes y archivos relacionados, y un directorio **/views** para organizar vistas de React y archivos relacionados.

---

### Iniciando el Servidor de Desarrollo de React

Anteriormente, cambiaste al directorio de tu proyecto. Lo único que queda por hacer ahora es **iniciar tu proyecto** con:

```bash
npm run dev
```

o

```bash
yarn dev
```

o

```bash
pnpm dev
```

La aplicación debería estar ahora en **[http://localhost:5173/](http://localhost:5173/)**. Puedes escribir esta dirección manualmente en la barra de URL de tu navegador. Verás una página que se parece a la siguiente imagen:

![vite-startup](/Images/vite_startup.webp)

Cualquier cambio en el **código fuente** se actualizará en vivo aquí. Vamos a verlo en acción.

Deja la **pestaña del terminal abierta** (está ocupada sirviendo la aplicación React) y abre `src/App.jsx` en tu editor de texto favorito. Verás lo que parece una mezcla de **JavaScript y HTML**. Esto es **JSX**, la forma en que React agrega sintaxis tipo XML a JavaScript. Proporciona una manera intuitiva de construir componentes de React y se **compila a JavaScript en tiempo de ejecución**. Hablaremos más sobre esto en otros contenidos, pero por ahora, hagamos una edición simple y veamos la actualización en el navegador.

En `App.jsx`, reemplaza el contenido con lo siguiente:

```javascript
import './App.css'

function App() {
  return (
    <>
      Hello, Codecademy!
    </>
  )
}

export default App
```

Si dejaste el terminal corriendo, deberías poder **cambiar a tu navegador** y ver la actualización.

![update](/Images/hello_codecademy.webp)

¡Felicidades! Ya tienes tu aplicación **React funcionando** y puedes comenzar a agregar funcionalidad a tu proyecto.

### Próximos Pasos

Ahora que has configurado tu primera aplicación React con Vite, esto es lo que puedes hacer a continuación:

* **Completa tu aprendizaje de React** para dominar los fundamentos de React.
* **Configura tu entorno de depuración**: Consulta nuestro [artículo sobre **React Developer Tools**](https://www.codecademy.com/paths/web-development/tracks/front-end-applications-with-react/modules/react-development-setup-and-ravenous-part-1/informationals/ready-react-developer-tools) para aprender a inspeccionar componentes y seguir el rendimiento.
* **Considera explorar frameworks de React**: Una vez que entiendas los conceptos básicos, prueba **Learn Next.js** para aprender características de producción como renderizado del lado del servidor y rutas optimizadas.
* **Practica construyendo pequeños proyectos** para reforzar lo que has aprendido y sigue explorando el ecosistema de React a medida que desarrollas tus habilidades.

¡Feliz codificación!

-----