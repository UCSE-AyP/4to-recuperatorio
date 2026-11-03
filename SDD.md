# Software Design Document (SDD) — 4to Recuperatorio 2026
**Módulo 4: Arreglos de Estructuras (AoS) y Algoritmos de Ordenamiento**  
**Universidad Católica de Santiago del Estero — DASS**  
**Cátedra:** Algoritmos y Programación  
**Ciclo Lectivo:** 2026  

---

## 1. Visión General y Objetivos de Evaluación

El presente documento formaliza la arquitectura técnica, estructuras de datos, contratos algorítmicos e invariantes de memoria para el **4to Examen Recuperatorio 2026**.

### Modalidad Bimodal Obligatoria
1. **75% Laboratorio de Programación (C++):**
   - Resolución en computadora sobre un arreglo estático de registros (`struct` + array AoS).
   - Estructuras modeladas **100% con tipos primitivos** (`int`, `char`, `double`, `bool`), omitiendo el tipo `string` para permitir lectura limpia y directa con `cin >>` sin problemas de sincronización de buffers.
   - Implementación de las 6 operaciones fundamentales de la lógica pura (Búsqueda por clave, Inserción con unicidad y corrimiento a derecha, Modificación in-place con clave primaria inmutable, Baja física con compactación del arreglo mediante corrimiento a la izquierda, Ordenamiento Burbuja Mejorada con swap integral y Cálculo Estadístico Seguro con guarda defensiva).
   - Capa de presentación por consola (menús, tablas formateadas con `<iomanip>`, fichas y buffers) **100% pre-implementada** por la cátedra para optimizar el tiempo de examen.
2. **25% Instancia Escrita Individual (Lápiz y Papel):**
   - Trazabilidad manual paso a paso en hoja A4 de un algoritmo clásico de ordenamiento sobre un vector de 5 elementos enteros.
   - Tema 1: **Selección Directa (Selection Sort)** en sentido **Ascendente**.
   - Tema 2: **Inserción Directa (Insertion Sort)** en sentido **Descendente**.

---

## 2. Definición de Temas y Modelos de Datos

### Tema 1: Sistema de Inventario de Tienda Comercial
- **Entidad:** `ProductoTienda`
- **Contexto:** Gestión en RAM del catálogo comercial de una tienda departamental, cálculo de recargo según nivel de stock y valorización del inventario.
- **Estructura C++ (100% Primitivos):**
```cpp
struct ProductoTienda {
    int codigo;            // Clave Primaria unívoca (estrictamente inmutable)
    int stock;             // Existencias en depósito (>= 0)
    char categoria;        // 'B' (Bajo < 20), 'M' (Medio 20..50), 'A' (Alto > 50)
    double precioBase;     // Precio base del producto (> 0)
    double valorUnitario;  // Calculado: precioBase + % segun categoria (B:+20%, M:+15%, A:+10%)
    double valorStock;     // Calculado: valorUnitario * stock
};
```
- **Algoritmo en C++:** Burbuja Mejorada **Ascendente** por `precioBase`.
- **Instancia Escrita:** Selección Directa Ascendente sobre $V = [45, 11, 72, 28, 6]$.

### Tema 2: Sistema de Reservas Hoteleras
- **Entidad:** `ReservaHotel`
- **Contexto:** Administración del registro de reservas de un complejo hotelero, con recargos según tipo de habitación e importes totales calculados.
- **Estructura C++ (100% Primitivos):**
```cpp
struct ReservaHotel {
    int nroHabitacion;     // Clave Primaria unívoca (estrictamente inmutable)
    int noches;            // Cantidad de noches reservadas (> 0)
    char tipo;             // 'S' (Simple 0%), 'D' (Doble +10%), 'T' (Triple +20%)
    double precioNoche;    // Tarifa base diaria (> 0)
    double importeTotal;   // Calculado: noches * precioNoche * (1 + recargoTipo)
    bool confirmada;       // Estado de reserva: true (Confirmada), false (Pendiente)
};
```
- **Algoritmo en C++:** Burbuja Mejorada **Descendente** por `importeTotal`.
- **Instancia Escrita:** Inserción Directa Descendente sobre $V = [33, 19, 58, 12, 41]$.

---

## 3. Contratos de Funciones Algorítmicas (Lógica Pura C++)

### 3.1. Búsqueda por Clave Primaria (10%)
- **Firma Tema 1:** `int buscarPorCodigo(const ProductoTienda arr[], int tam, int codigoBuscado);`
- **Firma Tema 2:** `int buscarPorHabitacion(const ReservaHotel arr[], int tam, int nroBuscado);`
- **Contrato:**
  - Precondiciones: `arr` válido, `tam >= 0`.
  - Comportamiento: Recorrido lineal con salida temprana inmediata al coincidir la clave.
  - Postcondiciones: Retorna índice `[0 .. tam-1]` o `-1` si no existe.

### 3.2. Alta / Inserción con Unicidad de Clave (15%)
- **Firma Tema 1:** `bool insertarProducto(ProductoTienda arr[], int &tam, int capacidad, int pos, const ProductoTienda &nuevo);`
- **Firma Tema 2:** `bool insertarReserva(ReservaHotel arr[], int &tam, int capacidad, int pos, const ReservaHotel &nueva);`
- **Contrato:**
  - Control de capacidad: rechaza si `tam >= capacidad` o `pos < 0` o `pos > tam`.
  - Reutilización modular recomendada de búsqueda (80% funcionamiento algorítmico correcto, 20% modularidad): rechaza si la clave primaria ya existe (`buscar != -1`).
  - Right-Shift: desplaza desde `tam` hasta `pos` una celda a la derecha.
  - Inserción e incremento: `arr[pos] = nuevo`, `tam++`. Retorna `true`.

### 3.3. Modificación con Clave Inmutable (10%)
- **Firma Tema 1:** `bool modificarProducto(ProductoTienda arr[], int tam, const ProductoTienda &actualizado);`
- **Firma Tema 2:** `bool modificarReserva(ReservaHotel arr[], int tam, const ReservaHotel &actualizada);`
- **Contrato:**
  - Reutilización modular recomendada de búsqueda (80% funcionamiento, 20% modularidad) para hallar la posición de memoria.
  - Muta *in-place* los campos de datos y atributos derivados:
    - *Tema 1:* Recibe código, stock y precioBase; recalcula categoria, valorUnitario y valorStock; preserva codigo inalterado.
    - *Tema 2:* Recibe nroHabitacion, noches, tipo, precioNoche y confirmada (validada en 0 o 1); recalcula importeTotal; preserva nroHabitacion inalterado.
  - **Invariante:** La clave primaria no se modifica bajo ninguna circunstancia.
  - Retorna `true` si se modificó o `false` si no se encontró.

### 3.4. Baja Física con Compactación del Arreglo mediante Corrimiento a la Izquierda (15%)
- **Firma Tema 1:** `bool darDeBajaProducto(ProductoTienda arr[], int &tam, int codigoABorrar);`
- **Firma Tema 2:** `bool darDeBajaReserva(ReservaHotel arr[], int &tam, int nroABorrar);`
- **Contrato:**
  - Reutilización modular recomendada de búsqueda (80% funcionamiento, 20% modularidad) para hallar el índice.
  - Left-Shift: desplaza elementos desde `pos + 1` hasta `tam - 1` una celda a la izquierda.
  - Decremento atómico: `tam--`. Retorna `true` o `false`.

### 3.5. Ordenamiento Burbuja Mejorada con Intercambio de Estructuras (15%)
- **Firma Tema 1:** `void ordenarPorPrecioBaseBurbujaAsc(ProductoTienda arr[], int tam);`
- **Firma Tema 2:** `void ordenarPorImporteBurbujaDesc(ReservaHotel arr[], int tam);`
- **Contrato:**
  - Control de banderas: `huboIntercambio` para terminación anticipada en arreglos ya ordenados.
  - Criterio de permutación estricto:
    - *Tema 1:* Solo intercambia si `arr[j].precioBase > arr[j + 1].precioBase`.
    - *Tema 2:* Solo intercambia si `arr[j].importeTotal < arr[j + 1].importeTotal`.
  - Permutación: Invocación estricta a `intercambiar(T &a, T &b)` para intercambio completo de dos estructuras pasando ambas por referencia.

---

## 4. Instancia Escrita Individual (25% Papel)

### Tema 1: Selección Directa Ascendente ($V = [45, 11, 72, 28, 6]$)
1. **Pasada 0 ($i=0$):** Menor en subarreglo $[45, 11, 72, 28, 6]$ es $6$ en índice $4$. Swap $V[0] \leftrightarrow V[4]$. Result: $[6, 11, 72, 28, 45]$.
2. **Pasada 1 ($i=1$):** Menor en subarreglo $[11, 72, 28, 45]$ es $11$ en índice $1$. Sin swap ($\text{minIdx} == i$). Result: $[6, 11, 72, 28, 45]$.
3. **Pasada 2 ($i=2$):** Menor en subarreglo $[72, 28, 45]$ es $28$ en índice $3$. Swap $V[2] \leftrightarrow V[3]$. Result: $[6, 11, 28, 72, 45]$.
4. **Pasada 3 ($i=3$):** Menor en subarreglo $[72, 45]$ es $45$ en índice $4$. Swap $V[3] \leftrightarrow V[4]$. Result: $[6, 11, 28, 45, 72]$.

### Tema 2: Inserción Directa Descendente ($V = [33, 19, 58, 12, 41]$)
1. **Pasada 1 ($i=1$):** $\text{aux} = 19$. Comp: ¿$33 < 19$? Falso. 0 desplazamientos. Insertar en [1]. Result: $[33, 19, 58, 12, 41]$.
2. **Pasada 2 ($i=2$):** $\text{aux} = 58$. Comp: ¿$19 < 58$? Sí. ¿$33 < 58$? Sí. 2 desplazamientos. Insertar en [0]. Result: $[58, 33, 19, 12, 41]$.
3. **Pasada 3 ($i=3$):** $\text{aux} = 12$. Comp: ¿$19 < 12$? Falso. 0 desplazamientos. Insertar en [3]. Result: $[58, 33, 19, 12, 41]$.
4. **Pasada 4 ($i=4$):** $\text{aux} = 41$. Comp: ¿$12 < 41$? Sí. ¿$19 < 41$? Sí. ¿$33 < 41$? Sí. ¿$58 < 41$? Falso. 3 desplazamientos. Insertar en [1]. Result: $[58, 41, 33, 19, 12]$.

---

## 5. Matriz de Ponderación Oficial

| Bloque | Contenido | Soporte | Pts |
| :---: | :--- | :---: | :---: |
| 1 | Búsqueda por Clave Primaria (Índice, Centinela -1, Const) | C++ | 10% |
| 2 | Alta / Inserción con Unicidad de Clave (Capacidad, Unicidad, Right-Shift, tam++) | C++ | 15% |
| 3 | Modificación con Clave Inmutable (Reutilización búsqueda, Mutación in-place) | C++ | 15% |
| 4 | Baja Física con Compactación del Arreglo mediante Corrimiento a la Izquierda (Reutilización búsqueda, Left-Shift, tam--) | C++ | 15% |
| 5 | Ordenamiento Burbuja Mejorada con Intercambio de Estructuras (Bandera, Comparación, Intercambio) | C++ | 20% |
| 6 | Instancia Escrita: Traza de Ordenamiento (Pasadas, Mín/Aux, Desplazamientos) | Papel | 25% |
| **Total** | **Acreditación Bimodal de Competencias Módulo 4** | **100%** |
