# Apuntes del libro

Este repositorio contiene el código fuente y apuntes derivados del libro: [*Hacking: The Art of Exploitation*](https://github.com/intere/hacking) de Jon Erickson.

*Proyecto en desarrollo

# Indice

1. [Programa básico en C](#programa-basico-en-c)
2. [Descompilación](#descompilacion)
3. [Depurador](#depurador)
4. [Lenguaje Ensamblador](#lenguaje-ensamblador)

# Programa basico en C
Es un programa que explica la logica de este lenguaje:
- El programa empieza en la funcion `main()` 
- La primera linea `#incluse <stdio.h>` incluye una libreria basica en C para el estandar input/ouput (I/O), esto se añade al programa al compilarlo y esta situado en `/usr/incluse/stdio.h` Esta libreria define funciones basicas de C como `main()` o `printf()`
- Compilar:
```
 $ gcc 0x250.c -o 0x250.out
``` 
- Ejecutar:
```
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
```
# Descompilacion 
Al compilar el programa, se combierte en lenguaje maquina, depende de que arquitectura de procesador tengamos, se traducira en un tipo u otro de lenguaje maquina.

En este caso, podemos el contenido de la funcion `main()` en lenguaje maquina ejecutando el programa `objdump`:

```
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
```
El programa `objdump` muestra  muchas lineas, por eso se filtra con `grep` las 20 primeras lineas tras `main`, filtrando hasi el contenido que nos interesa.

Los numeros hexadecimales como `0x2349085` en la izquierda son direcciones de memoria, esto es una coleccion de bits que se guarda temporalmente con esta direccion.

## Direcciones encontradas en el binario

| Dirección | Descripción |
|-----------|------------|
| `0x1139`  | Inicio de `main` |
| `0x115d`  | Salto dentro de `main` |
| `0x2004`  | Dirección relativa cargada con `lea` |
| `0x1030`  | Dirección de `puts@plt` |
| `0x1169`  | `ret` de `main` |
| `0x116c`  | Inicio de `_fini` |

En este caso, como el procesador es nuevo, utiliza 64-bits. Se puede saber hasi:
```
└─$ file 0x250.out

0x250.out: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7d23cd86c9945fc3561a88f151c85cf4ed6ff47f, for GNU/Linux 3.2.0, not stripped
```
El lenguaje maquina o ensamblador, es simplemetne un lenguaje que pueden enterder los procesadores. Hay dos grandes tipo: `AT&T` e `Intel syntax`. 

La salida que vemos es sintaxis `AT&T`, se puede diferenciar por los simbolos `%` y `$` que inician casi todo. 

Pero el mismo codigo se puede visualizar en sintaxis intel hasi:
```
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
```
Esta sintaxis en mas facil de entender.

Aqui podemos ver registros, que son partes de memoria de CPU mucha mas rapidos que RAM como `rbp`, `rsp`... son la base de esto.

# Depurador

Tenemos un preocesador `x86`. El procesador `x86` tiene muchos registros que son como variables internas del procesador.
 
Con el debugeador `GDB` podemos ejecutar paso a paso programas compilados, examinar la mamoria del programa y ver los registradores del procesador.
```
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
```
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

El puntero mas importante es la instruccion `RIP` (instruction pointer), almacena la direccion de la proxima instruccion a ejecutar, en este caso `main`.

# Lenguaje Ensamblador

Vamos a configurar `GDB` para que utilice la sintaxix `Intel`:
```
 echo "set disassembly intel" > ~/.gdbinit 
```
Normalmente las intstrucciones aparecen hasi:
```
Operation   destino     origen
```
El destino y origen pueden ser un registrador, direccion de memoria o un valor. Por ejemplo:
```
mov ebp,esp
sub esp,0x8
```
`mov` moveria el valor de su origen a su destino y `sub` sustraeria 8 de `esp`, viendo el resultado en `esp`.

A estos se le llama `mnemonics` y los`mnemonics` mas comunes en `x86-64` son:

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

Para poder conseguir mas informacion al debugear, podemos compilar nuestro programa con la flag `-g`.

```$ gcc -g 0x250.c```


Una vez compilado podemos debugear nuestro programa. Primeramente listaremos el codigo de esta manera:
```
 $ gdb -q ./a.out
Reading symbols from ./a.out...
(gdb) list
1       #include <stdio.h>
2
3       int main(){
4               //Bucle hello word 10 veces
5               int i;
6               for(i=0; i < 10; i++){
7                       printf("Hello, world!\n");
8               }
9               return 0;
10      }
```
Tambien podemos desensablar la funcion main para ver el contenido en lenguaje ensamblador. A esto se le podria llamar reversing estatico.
```
(gdb) disassemble main
Dump of assembler code for function main:
   0x0000000000001139 <+0>:     push   %rbp
   0x000000000000113a <+1>:     mov    %rsp,%rbp
   0x000000000000113d <+4>:     sub    $0x10,%rsp
   0x0000000000001141 <+8>:     movl   $0x0,-0x4(%rbp)
   0x0000000000001148 <+15>:    jmp    0x115d <main+36>
   0x000000000000114a <+17>:    lea    0xeb3(%rip),%rax        # 0x2004
   0x0000000000001151 <+24>:    mov    %rax,%rdi
   0x0000000000001154 <+27>:    call   0x1030 <puts@plt>
   0x0000000000001159 <+32>:    addl   $0x1,-0x4(%rbp)
   0x000000000000115d <+36>:    cmpl   $0x9,-0x4(%rbp)
   0x0000000000001161 <+40>:    jle    0x114a <main+17>
   0x0000000000001163 <+42>:    mov    $0x0,%eax
   0x0000000000001168 <+47>:    leave
   0x0000000000001169 <+48>:    ret
End of assembler dump.
```
Una vez hecho esto, haremos reversing de forma dinamica, es decir mientras ejecutamos el programa. Para esto primero ponemos como breakpoint la funciion main, esto detendra la ejecuccion justo al entrar en main. 

Con el comando `run` el programa empieza a ejecutarse desde el principio y se para en el punto indicado, aqui, podrermos ver el contenido del punto en el que se ha detenido.
```
(gdb) break main
Breakpoint 1 at 0x1141: file 0x250.c, line 6.
(gdb) run
Breakpoint 1, main () at 0x250.c:6
6               for(i=0; i < 10; i++){
```
En arquitectura 64 bits (como los PC actuales), `rip` (Punto de Instruccion) indica la procima instruccion que sera ejectuada. En caso de 32 bits (como indica el libro) seria `eip`.

```
(gdb) info register rip
rip            0x555555555141      0x555555555141 <main+8>
```
Este resultado significa que el punto de memoria que se va a ejecutar es el siguiente:

```
0x0000000000001141 <+8>:     movl   $0x0,-0x4(%rbp)
```
Las instruccions anteriores a este punto de memoria, son conocidas colectivamenet como prologo de la funcion y son generadas por el compilador para configurar la memoria para el resto de las variables locales de la función main(). Parte de la razón por la que es necesario declarar variables en C es para facilitar la construcción de esta sección de código. El depurador sabe que esta parte del código se genera automáticamente y es lo suficientemente inteligente como para omitirla.

Examinar la memoria es una habilidad fundamental para cualquier hacker. La mayoría de las vulnerabilidades de los hackers son como trucos de magia: parecen asombrosas y mágicas, a menos que se tenga experiencia con juegos de manos y distracciones. 

Tanto en la magia como en el hacking, si se mira en el lugar correcto, el truco es obvio. Esa es una de las razones por las que un buen mago nunca repite el mismo truco. Pero con un depurador como GDB, cada aspecto de la ejecución de un programa puede examinarse de forma determinista, pausarse, recorrerse paso a paso y repetirse tantas veces como sea necesario. Dado que un programa en ejecución es básicamente un procesador y segmentos de memoria, examinar la memoria es la primera manera de ver qué está sucediendo realmente.


El comando examinar en GDB puede usarse para examinar una dirección de memoria específica de diversas maneras. Este comando espera dos argumentos al usarse: la ubicación en la memoria que se examinará y cómo se mostrará esa memoria.

El formato de visualización también utiliza una abreviatura de una sola letra, que opcionalmente puede ir precedida de un recuento de elementos a examinar. Algunas letras de formato comunes son las siguientes:

| Letra | Formato de visualización               |
|-------|----------------------------------------|
| o     | Se muestra en octal.                   |
| x     | Se muestra en hexadecimal.             |
| u     | Se muestra en decimal sin signo (base 10). |
| t     | Se muestra en binario.                 |

Estos se pueden usar con el comando examinar para examinar una dirección de memoria específica. En el siguiente ejemplo, se utiliza la dirección actual del registro RIP. Los comandos abreviados se usan a menudo con GDB, e incluso el registro de información "rip" puede abreviarse simplemente como "i r rip".

```
(gdb) i r rip
rip            0x555555555141      0x555555555141 <main+8>
(gdb) x/o 0x555555555141
0x555555555141 <main+8>:        077042707
(gdb) x/x $rip
0x555555555141 <main+8>:        0x00fc45c7
(gdb) x/u $rip
0x555555555141 <main+8>:        16532935
(gdb) x/t $rip
0x555555555141 <main+8>:        00000000111111000100010111000111
(gdb)
```
También se puede anteponer un número al formato del comando examine para examinar varias unidades en la dirección de destino.

```
(gdb) x/2x $rip
0x555555555141 <main+8>:        0x00fc45c7      0xeb000000
(gdb) x/12x $rip
0x555555555141 <main+8>:        0x00fc45c7      0xeb000000      0x058d4813      0x00000eb3
0x555555555151 <main+24>:       0xe8c78948      0xfffffed7      0x01fc4583      0x09fc7d83
0x555555555161 <main+40>:       0x00b8e77e      0xc9000000      0xf30000c3      0x48fa1e0f
```
El tamaño predeterminado de una unidad simple es una unidad de cuatro bytes llamada `word`. El tamaño de las unidades de visualización del comando examinar se puede cambiar añadiendo una letra de tamaño al final de la letra de formato.

| Letra | Tamaño        | Descripción                                 |
|-------|---------------|---------------------------------------------|
| b     | 1 byte        | Un byte simple                              |
| h     | 2 bytes       | Media palabra (halfword)                    |
| w     | 4 bytes       | Palabra (word)                              |
| g     | 8 bytes       | Gigante (giant)                             |

Esto es un poco confuso, ya que a veces el término `word` también se refiere a valores de 2 bytes. En este caso, una palabra doble o `DWORD` se refiere a un valor de 4 bytes. En este libro, tanto los `word` como los `DWORD` se refieren a valores de 4 bytes. Si me refiero a un valor de 2 bytes, lo llamaré palabra corta o media palabra. La siguiente salida de GDB muestra la memoria en varios tamaños.

```
(gdb) x/8xb $rip
0x555555555141 <main+8>:        0xc7    0x45    0xfc    0x00    0x00    0x00    0x00    0xeb
(gdb) x/8xh $rip
0x555555555141 <main+8>:        0x45c7  0x00fc  0x0000  0xeb00  0x4813  0x058d  0x0eb3  0x0000
(gdb) x/8xw $rip
0x555555555141 <main+8>:        0x00fc45c7      0xeb000000      0x058d4813      0x00000eb3
0x555555555151 <main+24>:       0xe8c78948      0xfffffed7      0x01fc4583      0x09fc7d83
```


