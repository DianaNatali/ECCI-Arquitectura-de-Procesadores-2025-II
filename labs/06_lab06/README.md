# Lab06: Diseño de Banco de Registros

### Contenido

1. [Objetivos de aprendizaje](#1-objetivos-de-aprendizaje)  
2. [Fundamento teórico](#2-fundamento-teórico)  
3. [Procedimiento](#3-procedimiento)  
4. [Entregables](#4-entregables)

---

## 1. Objetivos de aprendizaje

* Comprender la función y organización de un banco de registros dentro de un sistema digital o microarquitectura.
* Implementar en Verilog un banco de 16 registros de $4$ bits con soporte de lectura dual y escritura controlada.
* Desarrollar simulaciones que validen la lectura, escritura y comportamiento del reset.
* Integrar posteriormente la ALU del laboratorio anterior y almacenar sus resultados en el banco de registros.
* Visualizar las operaciones aritméticas y lógicas mediante displays de $7$ segmentos.

---

## 2. Fundamento teórico

### 2.1 Banco de registros

Un **banco de registros** es un conjunto organizado de registros que permite almacenar y acceder a datos de manera rápida dentro de un sistema digital. Constituye una unidad fundamental en la arquitectura de procesadores, ya que permite:

* Guardar valores temporales  
* Proporcionar operandos a unidades funcionales  
* Mantener estados internos durante la ejecución  

El banco de registros de este laboratorio incluye:

* $16$ registros de $4$ bits  
* Escritura habilitada mediante `regwrite`  
* Lectura simultánea de dos registros  
* Reset global (`rst`) para inicializar los registros

A continuación se muestra el diagrama de bloques del banco de registros que encontraran en Github Classroom:


 <p align="center">
 <img src="../figs/lab6/bancoregistro.png" alt="alt text" width=450 >
</p>

### 2.2 Señalización y control

El funcionamiento del banco depende de señales fundamentales:

* **`addrRa`, `addrRb`** → direcciones de los registros a leer  
* **`addrW`** → dirección del registro a escribir  
* **`datW`** → dato que será almacenado  
* **`RegWrite`** → habilita o bloquea la escritura  
* **`rst`** → reinicia todos los registros a un valor predefinido  


## 3. Procedimiento

### **Parte 1: Simulación del banco de registros**

1. Clone el repositorio de [Github Classroom](https://classroom.github.com/a/lmUsg_xX).
2. Cree un módulo top que implemente el banco de registros en `BancoRegistro.v` el cual cuenta con:
   * $16$ posiciones de $4$ bits  
   * Doble lectura  
   * Escritura controlada por `regwrite`  
   * Reset síncrono o asíncrono (según especificación)

   Adicionalmente se debe integrar el conversor binario a BCD y BCD  a 7 segmentos, tal como se ha hecho en laboratorios anteriores.

3. Precargue el banco con valores fijos (usar `$readmemh`).
4. Realice un *testbench* que pruebe:
   * Lectura de registros precargados  
   * Escritura en distintas posiciones  
   * Correcto funcionamiento del reset  
5. Genere diagramas de tiempo que evidencien el comportamiento.
6. Cree un nuevo proyecto en Quartus con esta implementación y muestra su funcionamiento en clase.


### **Parte 2: Integración con la ALU del laboratorio anterior**

En esta parte se deberá:

1. Integrar la **ALU del laboratorio anterior** dentro del diseño para ello realice el respectivo diagrama de bloques.
2. Modificar el ancho del banco de registros, es decir, la cantidad de bits de cada registro a $8$ bits y justificar por qué.
3. Agregar **dos operaciones adicionales**:
   * Corrimiento lógico 1 bit a la izquierda (SHL1)
   * Corrimiento lógico 1 bit a la derecha (SHR1)
4. Los resultados de **cada operación** deben almacenarse en el banco de registros en posiciones fijas:  
   * `address 0`: suma  
   * `address 1`: resta  
   * `address 2`: multiplicación  
   * `address 3`: operación lógica #1  
   * `address 4`: operación lógica #2  
   * `address 5`: SHL1  
   * `address 7`: SHR1  

5. **Visualización final**  
   * Cada resultado de cada operación debe visualizarse en **7 segmentos**, no en LEDs.  
   * Mostrar el valor en **decimal**, separando las decenas/unidades.
   * Debido a que se cuenta con dos direcciones de lectura, se deben visualizar dos resultados a la vez.
    * Visualizar el `op code` en LEDs.
---

## 4. Entregables

1. Realice las partes 1 y 2 descritas en el [procedimiento](#3-procedimiento) y presentelas en clase.
2. Realice la respectiva documentación incluyendo diagrama de bloques según corresponda, diagrama RTL generado por Quartus y diagramas de simulación, así como evidencias de implementación.




