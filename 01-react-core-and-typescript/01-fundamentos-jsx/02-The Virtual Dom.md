# React: The Virtual DOM

## Aprende cómo el **DOM virtual de React** evita manipulaciones innecesarias del DOM.

### **El Problema**

La manipulación del DOM es el corazón de la web moderna e interactiva. Desafortunadamente, también es mucho más lenta que la mayoría de las operaciones de JavaScript.

Esta lentitud se agrava por el hecho de que la mayoría de los frameworks de JavaScript **actualizan el DOM mucho más de lo necesario**.

Por ejemplo, supongamos que tienes una lista con diez elementos. Marcas el primer elemento como completado. La mayoría de los frameworks de JavaScript **reconstruirían toda la lista**. ¡Eso es diez veces más trabajo del necesario! Solo un elemento cambió, pero los nueve restantes se vuelven a crear exactamente como estaban antes.

Reconstruir una lista no es gran cosa para un navegador web, pero los sitios modernos pueden usar **una enorme cantidad de manipulaciones del DOM**. Las actualizaciones ineficientes se han vuelto un problema serio.

Para resolver este problema, el equipo de React popularizó algo llamado **DOM virtual**.

---

### **El DOM Virtual**

Aquí hay un repaso de lo que sucede detrás de escena en el **DOM virtual**.

En React, por cada objeto del DOM, hay un **objeto “virtual DOM”** correspondiente. Un objeto virtual DOM es una **representación de un objeto del DOM**, como una copia ligera.

Un objeto virtual DOM tiene las mismas propiedades que un objeto del DOM real, pero **no tiene el poder de cambiar directamente lo que se ve en pantalla**.

Manipular el DOM es lento. Manipular el virtual DOM es mucho más rápido porque **nada se dibuja en pantalla**. Piensa en manipular el virtual DOM como **editar un plano**, en lugar de mover habitaciones en una casa real.

---

### **Cómo ayuda**

Cuando renderizas un elemento JSX, **cada objeto del virtual DOM se actualiza**.

Esto suena increíblemente ineficiente, pero el costo es insignificante porque el virtual DOM se puede actualizar muy rápido.

Una vez que el virtual DOM se ha actualizado, React lo compara con **una instantánea del virtual DOM tomada justo antes de la actualización**.

Al comparar el nuevo virtual DOM con la versión previa a la actualización, React determina **exactamente qué objetos del virtual DOM han cambiado**. Este proceso se llama **“diffing”**.

Una vez que React sabe qué objetos del virtual DOM han cambiado, actualiza **solo esos objetos en el DOM real**. En nuestro ejemplo anterior, React sería lo suficientemente inteligente como para reconstruir **solo el elemento de la lista que marcaste** y dejar el resto de la lista intacto.

¡Esto hace una gran diferencia! React puede actualizar **solo las partes necesarias del DOM**. La reputación de React por su rendimiento proviene en gran parte de esta innovación.

---

**En resumen, esto es lo que sucede cuando intentas actualizar el DOM en React:**

1. **Todo el virtual DOM se actualiza.**
2. El virtual DOM se compara con cómo se veía antes de la actualización. React determina qué objetos han cambiado.
3. **Solo los objetos que cambiaron** se actualizan en el DOM real.
4. Los cambios en el DOM real **provocan que la pantalla se actualice**.

---
