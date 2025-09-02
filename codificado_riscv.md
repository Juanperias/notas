# Riscv encode

## Prefacio

Hace un tiempo me puse a trabajar en mi propio Assembly, y aprendí algunas cosas de las que me gustaría compartir, este texto habla sobre como se codifican las instrucciones en riscv64 probablemente expanda este texto o lo remodele, para este texto se asume que tienes un mínimo de conocimiento sobre que es RISC-V, este texto no es una introducción a RISC-V, sino que es una especie de guía de como se hace esto.

## Instrucciones.. pero vistas desde el binario

Esta primera parte es una especie de adopción de como se ven el código máquina, a la final el Assembler es una abstracción que toma un código Assembly y lo convierte a código máquina de la arquitectura dada, la mayoría ya debe saber incluso sin estar muy metido en el mundo del low level es que las computadoras usan el código binario, y cuando vemos una instruccion como por ejemplo:
```
addi a0, zero, 10
```
Nos podría venir la pregunta a la mente, como esa instrucción es codificada por el assembler, para después ser leída y ejecutada por el procesador, la respuesta es muy sencilla es por el iSA, que se puede ver de forma muy sencilla como un contrato que describe como se va a comportar el procesador, lo que nos interesa es que también describe como se codifican las instrucciones, en este caso RISC-V nos da varios formatos de instrucciones que describen como en un bloque de 4 bytes (en caso de instrucciones comprimidas son 2 bytes) se pondrá las instrucciones, si echamos un vistazo veremos que addi es una instrucción de tipo I (inmediato) así que se codifica como tal, la codificación de una instrucción tipo i es de la siguiente forma:
```
imm[11:0] rs1 funct3 rd opcode
```
Ahora sí, retomamos la sintaxis de addi que es así:
```
addi rd, rs1, imm
```
Ahora si hace mas sentido, pero como se supone que esto tiene que ser una explicacion mas profunda voy a dar parte del texto a explicar que es opcode, rd, rs1, imm y funct3 (una nota a aclarar es que no todos los formatos tienen estos parámetros, por ejemplo las instrucciones tipo R no reciben un inmediato pero si reciben rs1 y rs2, lo que si esta en todos los tipos es el opcode, que como especie de dato curioso siempre esta en los bits 6:0 de la instruccion en todo los formatos).

Empecemos por el mas sencillo el opcode, opcode es simplemente la keyword  de la instruccion pero en binario, es útil la abstracción que proporcionan los assemblers usando palabras y no codigo binario, imagínate recordar como se escribia addi en binario.

Cuando pones addi esto se traduce al opcode 0010011, una cosa importante es que todos los opcodes son diferentes no hay algo como dos opcodes iguales , por si alguno se preguntaba.

Después le sigue rd que es el registro de destino cuando haces un `addi a0, a1, 2` es como hacer a0 = a1 + 2, lo cual explica porque cuando se quiere asignar un valor a un registro se usa el registro zero que siempre es cero.

Por el otro lado tenemos a rs1 que es el registro fuente numero 1 (tambien esta rs2 que es el registro fuente numero 2 pero no es usado en este tipo de instrucciones, rs2 solo esta presente en los tipos S, B, R) este es el registro donde se saca la informacion por asi decirlo, volviendo al ejemplo del `addi a0, a1, 2` en este caso a1 seria el registro rs1.

Y después por último pero no menos importante tenemos al inmediato, el valor inmediato es un valor constante que esta dentro de la instruccion no es un registro ni nada por el estilo en este caso de la instruccion addi es bastante sencillo ver cual es el valor inmediato, que si te preguntabas es 2.

Ahora suena sencillo codificar el inmediato y addi porque sabemos cual es su representacion binaria pero no sabemos muy bien como son los registros, a la final en riscv los registros tienen un numero asignado, en riscv64 tenemos 31 registros de propósito general al final del texto en la parte de recursos dejare una pagina que lista los registros y su numero que los representa pero por ejemplo a0 es 10 a1 es 11 y asi (tambien se suele decir x10 que seria a0 y asi) estos números varían con registros.

### Finalmente, condificando nuestra primera instruccion
Bien ahora usando todos nuestros conocimientos adquiridos codificaremos la instrucción `addi a0, zero, 10` y estaria bien que lo intetaras por ti mismo en tu lenguaje de programación de preferencia pero si no lo lograste aqui esta la respuesta y una explicación:
```
0000 0000 1010 0000 0000 0101 0001 0011
```

Los primeros 12 bits corresponden al inmediato tal y como hablamos  después sigue el rs1 que al ser zero adivinaste su numero es 0 (x0) y después sigue rd que al ser a0 (x10) se codifica como a10, y después de último sigue opcode que es el opcode de addi que hablamos mas arriba (si tienes alguna duda de donde conseguir estos opcodes en la parte final de referencias dare un link que conduce a un PDF que contiene los opcodes en binario y sus funct3)... Espera funct3? Pues si, si ves despues del rs1 hay 3 bits que son 0 y eso corresponde al funct3 tal como lo dijimos arriba, en la seccion de recursos dejare un encoder/decoder para que puedas experimentar poniendo diferentes instrucciones y viendo como se ven en binario, tambien puedes poner instrucciones en binario y ver como son de forma legible para los humanos.

## Números negativos!
Ahora si habras usado el assembly de riscv sabrás que se puede hacer `addi a0, zero, -1` entonces sabemos como se codifica addi, como se codifica a0, pero como se codifica -1, es bastante simple y aplica para todos los números negativos si vemos -1 es asi en hexadecimal `ffffffff` pero como se supone que llegamos a este punto (algo importante es que en algunos lenguajes ejemplo rust si tengo un i32 y lo convierto a un u32 aplica este proceso por nosotros lo cual es bastante util) partamos del 1 lo primero es aplicar un bitwise not que nos da como resultado `fffffffe` pero si le sumamos 1 nos da ``ffffffff``  y esto se puede aplicar con cualquier negativo.

Si quieres pasar de negativo a positivo puedes usar el mismo método pero con el numero negativo.

Ahora con esto seria capaz de codificar la instrucción addi con inmediato negativo usando el método de complemento a dos (que como dato curioso es usado en la mayoria de arquitecturas modernas como x86_64, arm etc).

# todo: ortografía pseudo instrucciones y recursos
