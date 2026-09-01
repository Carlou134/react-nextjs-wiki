# The tsconfig.json File

## 📌 ¿Qué es `tsconfig.json`?

Es un **archivo de configuración** que le dice a TypeScript **cómo compilar tu proyecto**.

### 📌 ¿Para qué sirve?

1. **Personalizar reglas** → puedes decidir qué tan estricto será el compilador.
2. **Decidir qué archivos compilar** → por ejemplo, todos los `.ts`.
3. **Evitar escribir parámetros largos** → con este archivo, solo pones `tsc` en la consola y se aplica todo.

### 📌 Ejemplo básico:

```json
{
  "compilerOptions": {
    "target": "es2017",        // Qué versión de JS generar
    "module": "commonjs",      // Cómo manejar imports/exports
    "strictNullChecks": true   // Control estricto de null/undefined
  },
  "include": ["**/*.ts"]       // Qué archivos revisar (todos los .ts)
}
```

👉 En una frase:
**El `tsconfig.json` es el manual que usa TypeScript para saber qué reglas aplicar y qué archivos compilar en tu proyecto.**

Opciones para el compilador de ts:

https://www.typescriptlang.org/docs/handbook/compiler-options.html

---


