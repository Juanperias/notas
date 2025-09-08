# Riscv encode

## Prefacio

Hace un tiempo me puse a trabajar en mi propio assembler, y aprendí algunas cosas de las que me gustaría compartir, este texto habla sobre como se codifican las instrucciones en riscv64 probablemente expanda este texto o lo remodele, para este texto se asume que tienes un mínimo de conocimiento sobre que es RISC-V, este texto no es una introducción a RISC-V, sino que es una especie de guía de como se hace esto.

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
Ahora si hace más sentido, pero como se supone que esto tiene que ser una explicación más profunda voy a dar parte del texto a explicar que es opcode, rd, rs1, imm y funct3 (una nota a aclarar es que no todos los formatos tienen estos parámetros, por ejemplo las instrucciones tipo R no reciben un inmediato, pero si reciben rs1 y rs2, lo que si está en todos los tipos es el opcode, que como especie de dato curioso siempre está en los bits 6:0 de la instrucción en todo los formatos).

Empecemos por lo más sencillo: el opcode, opcode es simplemente la keyword  de la instrucción, pero en binario, es útil la abstracción que proporcionan los assemblers usando palabras y no código binario. Imagínate recordar cómo se escribía addi en binario.

Cuando pones addi esto se traduce al opcode 0010011, algo es que algunas instrucciones tienen mismos opcodes (ejemplo addi y xori) pero tienen diferentes funct3 o en formatos que permitan funct7 aveces tambien tienen diferente funct7 y funct3 (en casos como addi y ecall tienen el mismo funct3 pero no el mismo opcode).

Después le sigue rd que es el registro de destino cuando haces un `addi a0, a1, 2` es como hacer a0 = a1 + 2, lo cual explica por qué cuando se quiere asignar un valor a un registro se usa el registro zero que siempre es cero.

Por el otro lado tenemos a rs1 que es el registro fuente número 1 (también esta rs2 que es el registro fuente número 2, pero no es usado en este tipo de instrucciones, rs2 solo está presente en los tipos S, B, R) este es el registro donde se saca la información por así decirlo, volviendo al ejemplo del `addi a0, a1, 2` en este caso a1 sería el registro rs1.

Y después, por último, pero no menos importante tenemos al inmediato, el valor inmediato es un valor constante que está dentro de la instrucción no es un registro ni nada por el estilo en este caso de la instrucción addi es bastante sencillo ver cuál es el valor inmediato, que si te preguntabas es 2.

Ahora suena sencillo codificar el inmediato y addi porque sabemos cuál es su representación binaria, pero no sabemos muy bien como son los registros, a la final en RISC-V los registros tienen un número asignado, en riscv64 tenemos 31 registros de propósito general al final del texto en la parte de recursos dejaré una página que lista los registros y su número que los representa, pero por ejemplo a0 es 10 a1 es 11 y así (también se suele decir x10 que sería a0 y así) estos números varían con registros.

### Finalmente, codificando nuestra primera instrucción
Bien, ahora, usando todos nuestros conocimientos adquiridos, codificaremos la instrucción `addi a0, zero, 10` y estaría bien que lo intentas por ti mismo en tu lenguaje de programación de preferencia, pero si no lo lograste, aquí está la respuesta y una explicación:
```
0000 0000 1010 0000 0000 0101 0001 0011
```

Los primeros 12 bits corresponden al inmediato tal y como hablamos  después sigue el rs1 que al ser, pero adivinaste su número es 0 (x0) y después sigue rd que al ser a0 (x10) se codifica como a10, y después de último sigue opcode que es el opcode de addi que hablamos más arriba (si tienes alguna duda de donde conseguir estos opcodes en la parte final de referencias daré un link que conduce a un PDF que contiene los opcodes en binario y sus funct3)... ¿Espera funct3? Pues si, si ves después del rs1 hay 3 bits que son 0 y eso corresponde al funct3 tal como lo dijimos arriba, en la sección de recursos dejaré un encoder/decoder para que puedas experimentar poniendo diferentes instrucciones y viendo como se ven en binario, también puedes poner instrucciones en binario y ver como son de forma legible para los humanos.

## Números negativos!
Ahora si habrás usado el Assembly de RISC-V sabrás qué se puede hacer `addi a0, zero, -1` entonces sabemos como se codifica addi, como se codifica a0, pero como se codifica -1, es bastante simple y aplica para todos los números negativos si vemos -1 es así en hexadecimal `ffffffff`, pero como se supone que llegamos a este punto (algo importante es que en algunos lenguajes ejemplo Rust si tengo un i32 y lo convierto a un u32 aplica este proceso por nosotros lo cual es bastante útil) partamos del 1 lo primero es aplicar un bitwise not que nos da como resultado `fffffffe`, pero si le sumamos 1 nos da ``ffffffff``  y esto se puede aplicar con cualquier negativo.

Si quieres pasar de negativo a positivo, puedes usar el mismo método, pero con el número negativo.

Ahora con esto sería capaz de codificar la instrucción addi con inmediato negativo usando el método de complemento a dos (que como dato curioso es usado en la mayoría de arquitecturas modernas como x86_64, arm, etc.).

## Pseudo instrucciones
Según la rae pseudo significa falso, así que serían instrucciones falsas, y tiene sentido. En esta sección explicaré cómo son las pseudo instrucciones y cómo se "codifican" (aunque realmente no se codifican como las demás).

Tomemos por ejemplo la instrucción `nop`, aunque busques por todos los lados sus opcodes no te saldrá nada te saldrá `addi x0, x0, 0` y si codificas eso en objdump por ejemplo te saldrá nop, para no meter más relleno una pseudo instrucción es una especie de ayuda para no escribir instrucciones más largas o complejas, ret también es una pseudo instrucción que se convierte en `jalr x0, x1, 0`.

Ahora no todo es tan bonito con las pseudo instrucciones hay unas más complejas como li (load immediate), la cual se traduce de varias formas si intentas cargar un valor, aunque parece inútil esta instrucción toma sentido, debido a que es por ejemplo para 32 es una combinación addi (12 bits) + lui (20 bits) para formar un valor de 32 bits.

Por ejemplo, tome la instrucción `li a0,73728` esto se convierte en `lui a0,0x12 addi a0,a0,837`.

## Funciones para codificar instrucciones
Bueno llego la parte donde apagas el cerebro y copias y pegas, aunque te recomendaría analizar las funciones detenidamente estas funciones están sacadas desde mi Assembly y retornar un `Vec<u8>` que contiene la instrucción de RISC-V codificada en Little endian actualmente solo llevo los tipos I e R

## Tipo S

```rust
pub struct StoreArgs {
    pub imm: u64,
    pub rs2: u32,
    pub rs1: u32,
    pub opcode: u32,
    pub funct3: u32,
}

pub fn store(arg: StoreArgs) -> Vec<u8> {
    let ins = ((arg.imm & 0b00000000_00000000_00000000_00000000_00000000_00000000_00001111_11100000) as u32) << 25
        | arg.rs2 << 20
        | arg.rs1 << 15
        | arg.funct3 << 12
        | ((arg.imm & 0b00000000_00000000_00000000_00000000_00000000_00000000_00000000_00011111) as u32) << 7
        | arg.opcode;

    ins.to_le_bytes().to_vec()
}
```

## Tipo I
```rust
pub struct ImmArgs {
    pub imm: u64,
    pub rs1: u32,
    pub rd: u32,
    pub funct3: u32,
    pub opcode: u32,
}

pub fn immediate(arg: ImmArgs) -> Vec<u8> {
    let ins = ((arg.imm as u32) << 20)
        | arg.rs1 << 15
        | arg.funct3 << 12
        | (arg.rd as u32) << 7
        | arg.opcode;

    ins.to_le_bytes().to_vec()
}
```

## Tipo R
```rust
pub struct RegArgs {
    pub rs1: u32,
    pub rs2: u32,
    pub rd: u32,
    pub funct7: u32,
    pub funct3: u32,
    pub opcode: u32,
}

pub fn register(arg: RegArgs) -> Vec<u8> {
    let ins = arg.funct7 << 25
        | arg.rs2 << 20
        | arg.rs1 << 15
        | arg.funct3 << 12
        | arg.rd << 7
        | arg.opcode;

    ins.to_le_bytes().to_vec()
}
```


# Recursos
- https://www.cs.sfu.ca/~ashriram/Courses/CS295/assets/notebooks/RISCV/RISCV_CARD.pdf
- https://luplab.gitlab.io/rvcodecjs/
- https://en.wikipedia.org/wiki/Two%27s_complement
