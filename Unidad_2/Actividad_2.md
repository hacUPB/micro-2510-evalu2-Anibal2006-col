# 🔧 Actividad 2: Lenguaje C en Sistemas Embebidos

## 🧾 Definición

**C** es un lenguaje de programación de propósito general que ofrece un equilibrio entre abstracción (sintaxis relativamente legible) y control de bajo nivel (acceso directo a memoria y registros de hardware).  
Es ampliamente usado en **sistemas embebidos**, donde los recursos (CPU, RAM, almacenamiento) son limitados y se requiere **eficiencia**.

---

## ✅ Ventajas de programar en C

- **Código compacto y claro**: Sintaxis eficiente para expresar algoritmos complejos.
- **Mantenimiento sencillo**: Separación de interfaz (archivos `.h`) e implementación (archivos `.c`).
- **Trabajo en equipo**: Facilita la división de proyectos en módulos.
- **Portabilidad**: Código adaptable a diferentes microcontroladores con mínimos cambios de compilador.

---

## ⚠️ Desventajas de programar en C

- **Costo de compilador**: Algunas toolchains profesionales pueden ser costosas.
- **Eficiencia extrema**: En rutinas muy críticas, el lenguaje ensamblador puede ofrecer mayor optimización.

---

## 🏗️ Estructura básica de un programa en C

- **Directivas de preprocesador**: `#include`, `#define`.  
  Se procesan antes de compilar, permiten incluir librerías y definir macros.

- **Prototipos de funciones**: Declaraciones anticipadas para funciones usadas.

- **Función `main()`**: Punto de entrada del programa.

- **Funciones auxiliares**: Módulos reutilizables que cumplen tareas específicas.

---

## 🧩 Macros y preprocesador

- **Definición**: Fragmentos de código sustituidos literalmente por el preprocesador.

- **Macros con parámetros**:  
  Ejemplo: `#define SQUARE(x) ((x)*(x))`  
  Generan código inline sin sobrecarga por llamada a función.

- **Mapeo de hardware**:  
  Se usan punteros `volatile` para acceder a registros de memoria en direcciones fijas.

---

## 🧮 Tipos de datos en C

### 🔤 Tipos básicos:

- `char`: 1 byte (carácter).
- `int`: 2–4 bytes (entero).
- `float`: 4 bytes (real de simple precisión).
- `double`: 8 bytes (real de doble precisión).

### 📐 Tipos con tamaño fijo (`<stdint.h>`):

- `uint8_t`, `int16_t`, `uint32_t`, etc.  
  Garantizan un ancho de bit específico.

---

## ⚙️ Operadores

- **Aritméticos**: `+`, `-`, `*`, `/`, `%`
- **Lógicos**: `&&`, `||`, `!`
- **A nivel de bits**: `&`, `|`, `^`, `~`
- **Desplazamientos**: `<<` (izquierda), `>>` (derecha)

---

## 🔄 Control de flujo

- **Condicionales**: `if`, `else if`, `else`, `switch`
- **Bucles**: `for`, `while`, `do ... while`

---

## 🧠 Punteros y variables `volatile`

- **Punteros**:  
  Variables que almacenan direcciones de memoria.  
  Esenciales para interactuar con registros de hardware.

- **`volatile`**:  
  Indica al compilador que no optimice una variable cuyo valor puede cambiar por interrupciones o hardware externo.

---

## 🛠️ Funciones y recursión

- **Funciones**:  
  Bloques nombrados que reciben parámetros y devuelven valores.  
  Permiten modularización.

- **Recursión**:  
  Función que se llama a sí misma.  
  Requiere un **caso base** para evitar bucles infinitos y desbordamiento de pila.

---

## 📦 `struct` y `union`

- **`struct`**:  
  Agrupa diferentes tipos de datos bajo un mismo nombre.  
  Útil para representar estructuras complejas.

- **`union`**:  
  Todos los campos comparten la misma dirección de memoria.  
  Se usa para reinterpretar datos, por ejemplo, para ver los bytes individuales de un `float`.

---





