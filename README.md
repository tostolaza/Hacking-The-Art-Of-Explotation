APUNTES DEL LIBRO
This repository contains the source code that comes from the book: Hacking The Art of Exploitation: https://github.com/intere/hacking
# Indice

1. [Programa básico en C](#programa-basico-en-c)
2. [Descompilación](#descompilacion)
3. [Depurador](#depurador)
4. [Lenguaje Ensamblador](#lenguaje-ensamblador)

# Programa basico en C
Es un programa en explicativo de la logica de este lenguaje:
- El programa empieza en la funcion main() 
- La primera linea #incluse <stdio.h> incluye una libreria basica en C para el estandar input/ouput (I/O), esto se añade al programa al compilarlo y esta situado en /usr/incluse/stdio.h Esta libreria define funciones basicas de C como main() o printf()
- Se compila hasi $ gcc 0x250.c -o 0x250.out 
- Se ejecuta hasi
└─$ ./0x250    
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world

# Descompilacion 
El programa al compilarlo, se combierte en lenguaje maquina, depende de que arquitectura de procesador tengamos se traducira en ese tipo de lenguaje maquina.
En este caso podemos ver con el promgrama objdump el contenido de la funcion main en lenguaje maquina:

└─$ objdump -D 0x250 | grep -A20 main.:
0000000000001139 <main>:
    1139:       55                      push   %rbp
    113a:       48 89 e5                mov    %rsp,%rbp
    113d:       48 83 ec 10             sub    $0x10,%rsp
    1141:       c7 45 fc 00 00 00 00    movl   $0x0,-0x4(%rbp)
    1148:       eb 13                   jmp    115d <main+0x24>
    114a:       48 8d 05 b3 0e 00 00    lea    0xeb3(%rip),%rax        # 2004 <_IO_stdin_used+0x4>
    1151:       48 89 c7                mov    %rax,%rdi
    1154:       e8 d7 fe ff ff          call   1030 <puts@plt>
    1159:       83 45 fc 01             addl   $0x1,-0x4(%rbp)
    115d:       83 7d fc 09             cmpl   $0x9,-0x4(%rbp)
    1161:       7e e7                   jle    114a <main+0x11>
    1163:       b8 00 00 00 00          mov    $0x0,%eax
    1168:       c9                      leave
    1169:       c3                      ret

Disassembly of section .fini:

000000000000116c <_fini>:
    116c:       48 83 ec 08             sub    $0x8,%rsp
    1170:       48 83 c4 08             add    $0x8,%rsp

El programa onjdump expulsa muchas lineas, por eso se hace grep en las 20 lineas que siguen a main para filtrar el contenido que nos interesa.

Este byta esta representado en hexadecimal notation, que utiliza el sistema de  numeracion base-16.

Los numeros hexadecimales como 0x2349085 en la izquierda on direcciones de memroia, esto es una coleccion de bits que se guarda temporalmente con esta direccion.

## Direcciones encontradas en el binario

| Dirección | Descripción |
|-----------|------------|
| `0x1139`  | Inicio de `main` |
| `0x115d`  | Salto dentro de `main` |
| `0x2004`  | Dirección relativa cargada con `lea` |
| `0x1030`  | Dirección de `puts@plt` |
| `0x1169`  | `ret` de `main` |
| `0x116c`  | Inicio de `_fini` |

En este caso como el procesador es nuevo, utiliza 64-bits. Se puede saber hasi:
└─$ file 0x250.out

0x250.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7d23cd86c9945fc3561a88f151c85cf4ed6ff47f, for GNU/Linux 3.2.0, not stripped

Ensamblador es simplemetne un lenguaje que pueden enterder los procesadores, hay dos grandes tipo AT&T e Intes syntax. La salida que vemos es sintaxis AT&T, se puede diferenciar por los simbolos % y $ que inician casi todo. 

Pero el mismo codigo se puede visualizar en sintaxis intel hasi:
└─$ objdump -M intel -D 0x250.out | grep -A20 main.:
0000000000001139 <main>:
    1139:       55                      push   rbp
    113a:       48 89 e5                mov    rbp,rsp
    113d:       48 83 ec 10             sub    rsp,0x10
    1141:       c7 45 fc 00 00 00 00    mov    DWORD PTR [rbp-0x4],0x0
    1148:       eb 13                   jmp    115d <main+0x24>
    114a:       48 8d 05 b3 0e 00 00    lea    rax,[rip+0xeb3]        # 2004 <_IO_stdin_used+0x4>
    1151:       48 89 c7                mov    rdi,rax
    1154:       e8 d7 fe ff ff          call   1030 <puts@plt>
    1159:       83 45 fc 01             add    DWORD PTR [rbp-0x4],0x1
    115d:       83 7d fc 09             cmp    DWORD PTR [rbp-0x4],0x9
    1161:       7e e7                   jle    114a <main+0x11>
    1163:       b8 00 00 00 00          mov    eax,0x0
    1168:       c9                      leave
    1169:       c3                      ret

Disassembly of section .fini:

000000000000116c <_fini>:
    116c:       48 83 ec 08             sub    rsp,0x8
    1170:       48 83 c4 08             add    rsp,0x8

Esta sintaxis en mas facil de entender.

Aqui podemos ver registros, que son partes de memoria de CPU mucha mas rapidos que RAM como rbp, rsp... son la base de esto.

# Depurador

Tenemos un preocesador x86. El procesador x86 tiene muchos registros que son como variables internas del procesador. 
Con el debugeador GDB podemos ejecutar paso a paso programas compilados, examinar la mamoria del programa y y ver los registradores del procesador.

└─$ gdb -q ./0x250
Reading symbols from ./0x250...
(No debugging symbols found in ./0x250)
(gdb) break main
Breakpoint 1 at 0x113d
(gdb) run
Starting program: /Documents/HackingTheArtOfExplotation/0x250 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x000055555555513d in main ()
(gdb) info register
rax            0x555555555139      93824992235833
rbx            0x7fffffffdde8      140737488346600
rcx            0x555555557dd8      93824992247256
rdx            0x7fffffffddf8      140737488346616
rsi            0x7fffffffdde8      140737488346600
rdi            0x1                 1
rbp            0x7fffffffdcd0      0x7fffffffdcd0
rsp            0x7fffffffdcd0      0x7fffffffdcd0
r8             0x0                 0
r9             0x7ffff7fcbf40      140737353924416
r10            0x7fffffffda10      140737488345616
r11            0x206               518
r12            0x0                 0
r13            0x7fffffffddf8      140737488346616
r14            0x7ffff7ffd000      140737354125312
r15            0x555555557dd8      93824992247256
rip            0x55555555513d      0x55555555513d <main+4>
eflags         0x246               [ PF ZF IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
fs_base        0x7ffff7dad740      140737351702336
gs_base        0x0                 0
(gdb) quit

Al poner el breackpoint en main(), la ejecucion se parara justo en el punto marcado. GDB muestra los registros hasta ese punto. 

Estos son los registros encontrados.

| Registro | Valor | Significado |
|----------|------------|--------------------------|
| **`rax`** | `0x555555555139` | Dirección del código en ejecución. |
| **`rbx`** | `0x7fffffffdde8` | Dirección de un dato en la pila. |
| **`rcx`** | `0x555555557dd8` | Dirección en la memoria del proceso. |
| **`rdx`** | `0x7fffffffddf8` | Dirección de otro dato en la pila. |
| **`rdi`** | `0x1` | Primer argumento en `main(argc)`. |
| **`rsi`** | `0x7fffffffdde8` | Segundo argumento (posible `argv`). |
| **`rbp`** | `0x7fffffffdcd0` | Base del marco de pila. |
| **`rsp`** | `0x7fffffffdcd0` | Tope de la pila. |
| **`rip`** | `0x55555555513d` | Dirección de la siguiente instrucción a ejecutar (`main+4`). |
| **`eflags`** | `0x246` | Contiene flags del procesador (`ZF` está activo). |

El puntero mas importante es la instruccion RIP (instruction pointer), almacena la direccion de la procima instruccion a ejecutar, en este caso main.

# Lenguaje Ensamblador

Vamos a configurar GDB para que utilice la sintaxix Intel:

 echo "set disassembly intel" > ~/.gdbinit 

Normalmente las intstrucciones aparecen hasi:

Operation   destino     origen

El destino y origen pueden ser un registrador, direccion de memoria o un valor. Los operation son mnemonics: Por ejemplo:

mov ebp,esp
sub esp,0x8

mov moveria el valor de su origen a su destino y sub sustraeria 8 de esp, viendo el resultado en esp.

Ejemplos de mnemonicos comunes en x86-64
| Mnemónico | Significado | Ejemplo |
|-----------|------------|---------|
| `mov`  | Mueve datos de un lugar a otro | `mov rax, rbx` |
| `add`  | Suma dos valores | `add rax, 5` |
| `sub`  | Resta dos valores | `sub rax, 10` |
| `cmp`  | Compara dos valores | `cmp rax, rbx` |
| `jmp`  | Salta a una dirección | `jmp 0x400123` |
| `call` | Llama a una función | `call printf` |
| `ret`  | Retorna de una función | `ret` |
| `push` | Guarda un valor en la pila | `push rax` |
| `pop`  | Extrae un valor de la pila | `pop rax` |



