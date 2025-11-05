# Lab 07: Primer paso con PicoSoC y PicoRV32 

Este laboratorio tiene como objetivo introducir el diseño de un sistema en chip (SoC) basado en la arquitectura RISC-V, empleando el núcleo PicoRV32 y un conjunto mínimo de periféricos. Se utilizará el entorno de desarrollo Intel Quartus Prime para sintetizar el diseño en una FPGA MAX 10, y se desarrollará un firmware embebido que primero generará una secuencia de LEDs, y posteriormente enviará un mensaje por UART.

## Introducción

### ¿Qué es un SoC?

Un *System on Chip* (SoC) es una arquitectura de hardware digital que integra todos los componentes esenciales de un sistema computacional en un solo circuito integrado o diseño programable. Generalmente incluye:

* Un procesador central (CPU).

* Memoria ROM/RAM.

* Periféricos (temporizadores, UART, GPIO, etc.).

* Interconexión interna (bus de datos/control/direcciones).

En este caso, el SoC será implementado en una FPGA, donde se integrarán los siguientes componentes:

* **PicoRV32**: un núcleo de procesador RISC-V.

* **Memoria ROM**: que contendrá el firmware embebido.

* **Memoria RAM**: para almacenamiento temporal de datos.

* **UART**: para comunicación serie con un computador externo.

* **GPIO para LEDs**: que permitirá una demostración visual sencilla del funcionamiento del firmware.

### ¿Qué es PicoRV32?

¿Qué es PicoRV32?

PicoRV32 es un núcleo de procesador implementado en Verilog HDL, que ejecuta la arquitectura de conjunto de instrucciones RISC-V de 32 bits. Fue diseñado por Clifford Wolf con el objetivo de ser minimalista, eficiente en área y de bajo consumo de recursos, cualidades que lo hacen especialmente adecuado para implementación en FPGAs pequeñas o aplicaciones embebidas que no requieren alto rendimiento.

#### Características principales

* Características principales

    * **Compatible con RISC-V (RV32)**: Soporta extensiones como ```I``` (instrucciones enteras), ```M``` (multiplicación y división), y ```C``` (instrucciones comprimidas), de manera opcional y configurable.

    * **Altamente configurable**: Se pueden habilitar o deshabilitar extensiones según las necesidades del diseño, como soporte para interrupciones, registros de control y estado (CSRs), alineación estricta de datos, etc.

    * **Diseño ligero y autónomo**: Está completamente escrito en Verilog sin dependencias externas ni macros, lo que facilita su integración en cualquier flujo de diseño compatible con Verilog (Intel Quartus, Vivado, yosys, etc.).

    * **Ideal para FPGAs pequeñas**: Su tamaño reducido lo hace ideal para FPGAs como la Intel MAX 10, donde el espacio lógico y los recursos RAM/ROM son limitados.

    * **Fácil de integrar gracias a su interfaz de memoria sencilla**:
    En lugar de usar buses complejos como AXI o Wishbone, PicoRV32 emplea una interfaz de tipo handshake, compuesta por dos señales principales:

        * ```mem_valid```: la CPU indica que quiere realizar una lectura o escritura.

        * ```mem_ready```: la memoria o periférico responde cuando la operación ha sido completada.

        Esta simplicidad permite conectar directamente el procesador a memorias y periféricos personalizados, sin necesidad de controladores intermedios ni lógica compleja. Es ideal para quienes diseñan sus propios bloques de RAM, UART, GPIO, etc., directamente en Verilog.

    * Ejecuta firmware en bare-metal (sin OS), lo que lo hace más rápido para aplicaciones simples.

#### Repositorio

[PicoRiscV](https://github.com/YosysHQ/picorv32)

#### Aplicaciones típicas

* Sistemas embebidos minimalistas.

* Controladores de dispositivos en FPGA.

* Proyectos de enseñanza de arquitectura de procesadores.

* Prototipado de sistemas tipo SoC (System-on-Chip).

### ¿Qué es firmware?

Firmware es un tipo especial de software que está diseñado para controlar hardware específico. En el caso de una FPGA donde implementas un procesador en Verilog, el firmware es:

**El código que corre dentro del procesador implementado en la FPGA.**

Es el programa que el procesador ejecuta, como si fuera el “sistema operativo” o la “aplicación embebida”.

Ejemplos típicos de firmware:

* Un archivo binario ```.hex``` o ```.bin``` que contiene instrucciones en lenguaje máquina (por ejemplo, instrucciones RV32I en el caso de PicoRV32).

* Código en ```C``` que se comila para para el procesador y se carga en su memoria (interna o externa).

### ¿Qué es software?

Software en general es cualquier programa que se ejecuta sobre hardware. En este contexto:

* Firmware es un tipo de software embebido.

* También hay software de desarrollo, como:

    * Compiladores (ej. GCC para RISC-V)

    * Herramientas de síntesis (ej. Yosys, Vivado, Quartis)

    * Simuladores (ej. Icarus Verilog)

    * Cargadores (ej. scripts que escriben en la RAM o Flash de la FPGA)


### ¿Qué es compilación cruzada (cross-compilation)?

Cuando se usa un compilador, normalmente se produce un binario para el mismo sistema local. Pero en FPGA, el procesador (como PicoRV32) no es la PC loca, así que no se puede compilar directamente con ```gcc``` del sistema local.

Entonces la compilación cruzada es usar un compilador en una arquitectura (por ejemplo la PC local, x86_64) para generar código para otra arquitectura (por ejemplo, RISC-V 32 bits).

Para lo anterior se necesita un **cross-compiler**: por ejemplo, ```riscv32-unknown-elf-gcc```.


Por ejemplo:

```
riscv32-unknown-elf-gcc -o firmware.elf main.c
riscv32-unknown-elf-objcopy -O verilog firmware.nex firmware.bin
```

Ese ```firmware.bin``` es lo que luego se carga en la memoria del procesador implementado en la FPGA.

### ¿Cómo se relaciona todo esto?

Veámoslo en un flujo completo:

1. **Implementación completa en FPGA**
    
    * **Implementación del procesador**:

        * Se escribe o instancia un procesador en Verilog (ej. PicoRV32)

        * Se sintetiza y se carga en la FPGA con herramientas como Yosys, NextPNR, Vivado o, en este caso Quartus.

    * **Diseño de memoria**

        El procesador necesita memoria para correr software.

        * Se puede usar RAM interna de la FPGA, BRAM (Block RAM), o externa (SRAM, SDRAM, SPI Flash)

        * Se puede cargar un ```.bin``` al iniciar la FPGA con ese firmware.

    *  **Escritura de firmware**

        * Se escribe el programa (en ```C``` o ensamblador RISC-V)

        * Se compila con un **cross-compiler** para RISC-V (```riscv32-unknown-elf-gcc```)

        * Se genera el ```.bin``` para cargar en la RAM de la FPGA

    * **Carga y ejecución**

        * El procesador empieza a ejecutar el firmware desde la memoria

        * El firmware controla periféricos, UART, LEDS, GPIOs, sensores, etc.


Entonces:

**En una implementación de un procesador en Verilog sobre FPGA, el firmware es el programa binario que se debe compilar con compilación cruzada, y que el procesador implementado en FPGA ejecutará para interactuar con el hardware.**

## Herramientas necesarias

### 1. Intel Quartus Prime (Lite Edition) – para sintetizar el diseño en una FPGA MAX 10.

### 2. MYSYS2

MSYS2 es un entorno de desarrollo basado en Windows que proporciona una terminal estilo Unix junto con una colección de herramientas GNU. Está diseñado para facilitar la compilación y el uso de software originalmente desarrollado para sistemas Linux o Unix, directamente sobre Windows, sin requerir una máquina virtual o un sistema dual.

#### ¿Por qué es necesario al trabajar con PicoSoC?

El proyecto PicoSoC incluye scripts, Makefiles, herramientas como make, y toolchains cruzados como riscv64-unknown-elf-gcc, que son comunes en entornos Linux. En Windows nativo, muchas de estas herramientas no existen o no funcionan de manera estándar. MSYS2 permite:

* Acceso a herramientas tipo Unix, como ```bash```, ```ls```, ```make```, ```grep```, etc.

* Uso de rutas Unix-like, como ```/c/Users/...``` en lugar de ```C:\Users\...```.

* Gestión de paquetes con pacman, para instalar dependencias de desarrollo fácilmente.

* Compatibilidad con toolchains cruzados, necesarios para compilar código para arquitecturas como RISC-V.

#### Pasos para usar MSYS2:

1. Instalar MSYS2: 

    1. Descargar el instalador en https://www.msys2.org/.
    2. Correr el instalador que es el ejecutable que se descarga en el item anterior.
    3. Seguir las instrucciones que se presentan en el link del ítem 1.

2. Abrir la terminal ```MSYS2 MinGW 64-bit```

3. En esta terminal intalar ```make``` y herramientas necesarias:

```
pacman -Syu         
pacman -S make gcc git mingw-w64-x86_64-toolchain
```

### 3. Toolchain de RISC-V (riscv32-unknown-elf-gcc)

El toolchain RISC-V ```riscv32-unknown-elf-gcc``` es un conjunto de herramientas diseñadas para compilar código C/C++ y generar binarios que puedan ejecutarse en procesadores con arquitectura RISC-V de 32 bits, como el PicoRV32. 

#### Pasos para obtenerlo:

1. Descargarlo siguiente [link](https://github.com/xpack-dev-tools/riscv-none-elf-gcc-xpack/releases). **Escoger el toolchain de acuerdo al sistema operativo de su PC**

2. Descomprimir el archivo que se descarga en el ítem anterior en la ubicación ```C:\tools\riscv```.

3. En la terminal de ```MSYS2 MinGW 64-bit``` ejecutar lo siguiente:

    ```
    export PATH="/c/tools/riscv/bin:$PATH"
    ```

4. Pruebe que funcione.  En la terminal de ```MSYS2 MinGW 64-bit``` ejecutar lo siguiente:

    ```
    riscv-none-elf-gcc --version
    ```

## Procedimiento para el pico SoC:

1. Clonar el siguiente repositorio: https://github.com/DianaNatali/picosoc_simple.git

    En este paquete de trabajo encontrarán un archivo ``Makefile```que es un archivo de texto que contiene un conjunto de reglas e instrucciones utilizadas por la herramienta make para automatizar tareas repetitivas, principalmente la compilación de programas.

    En el contexto de PicoSoC es un archivo de automatización que contiene una serie de instrucciones para compilar, enlazar, sintetizar, programar y/o simular todo el proyecto de hardware y software que constituye el SoC.

    Porque un proyecto como PicoSoC involucra varias tareas complejas:

    * Compilar el programa en C para la CPU RISC-V.

    * Generar los archivos de firmware (firmware.bin, firmware.v) que se integran en el hardware.

    * Sintetizar el diseño RTL en Verilog (como PicoRV32 + memoria + periféricos) a través de Quartus.

    * Generar el bitstream para el FPGA.

    * Cargar el bitstream a la FPGA.

    * (Opcional) Correr simulaciones.

    Todo lo anterior **Sin necesidad de abrir el IDE de Quartus**

2. En la terminal de ```MSYS2 MinGW 64-bit```:

    ```
    export PATH=/c/intelFPGA_lite/23.1std/quartus/bin:$PATH
    ```

    **Ojo: Tienen que modificar este comando para configurar correctamente el path de la carpeta bin de su instalación de Quartus**

3. Generar el ```.bin```

    En la terminal de ```MSYS2 MinGW 64-bit```:

    1. Ir hasta la ruta donde clonaron el repositorio que contiene el PicoSoC.

    2. Ejecutar:

        ```
        make
        ```

        Generara el ```main.bin``` que contiene el firmware y adicionalmente generará el archivo ```progmem.v``` que en el contexto de PicoSoC es un módulo de memoria ROM en Verilog que contiene el firmware embebido en forma de una tabla de inicialización. Es generado automáticamente a partir del binario ```main.bin.```

4.  Lanzar el flujo de síntesis completo de Quartus:

    En la terminal de ```MSYS2 MinGW 64-bit```:

    ```
    make build TARGET=max10
    ```

    Paso a paso del make build:

    1. ```map```:
        Ejecuta ```quartus_map```, que realiza el análisis y mapeo del diseño Verilog.

    2. ```fit```:
        Ejecuta ```quartus_fit```, que asigna el diseño a los recursos físicos del FPGA (pines, LUTs, etc.).

    3. ```asm```:
        Ejecuta ```quartus_asm```, que genera el archivo .sof (bitstream) para cargar en el FPGA.

    Requisitos:

    1. Debe existir un archivo ```.qsf``` (Quartus Settings File), en este caso top.qsf.

    2. Debe tener los archivos Verilog del PicoSoC y ```progmem.v``` generados correctamente.

    3. La variable TOP (nombre del top module) debe coincidir con el nombre del proyecto de Quartus.

5.  Cargar a la FPGA:


    ```
    make TARGET=max10 program
    ```




