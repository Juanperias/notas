# Respuestas a las problemas del libro “Sistemas operativos modernos”

## Prefacio

Algo que quiero aclarar es quwe no todos los problemas se responderán ya sea por dudas en su realización entre otros.

## Problemas del capítulo 1.

### ¿Qué es la multiprogramación?
La multiprogramación es un método que permite que varios programas se ejecute de forma concurrente, incluso aunque solo esté presente una sola CPU, Si recuerdo correctamente la multiprogramación fue introducida (o popularizada) en el sistema operativo OSX/360

### ¿Qué es el spooling?
El spooling es una técnica que consiste en que los programas puedan acceder a los dispositivos de forma rápido incluso aunque estén en una tarea esto se hace por medio buferes por ejemplo una impresora puede llegar a ser lenta pero si un proceso quiere imprimir y otro está imprimiendo se puede guardar la información que iba a imprimir en un buffer hasta que la impresora deje de imprimir

### ¿Cree usted que las computadoras personales avanzadas tendrían spooling como característica estándar en el futuro?

Lo más probable es que si, hay veces en dónde el spooling es de mucha utilidad para manejo de dispositivos.

Ahora los dispositivos están mejorando y siendo más potentes por lo que quizás en un futuro se deje de utilizar.

Tome como ejemplo a los discos duros, antiguamente solo se podía hacer una lectura a la vez, ahora con unidades extremadamente avanzadas (como los Nvme) ya no es necesario tener spooling en este tipo de dispositivos.

### En las primeras computadoras, cada byte de datos leídos o escritos se manejaba mediante la CPU (es decir, no había DMA). ¿Qué implicaciones tiene esto para la multiprogramación?

Esto tiene varias implicaciones en sí sería que se tiene la cpu tiene más trabajo, lo que por ejemplo hace menos eficiente el cambio de proceso al momento de que se realice una operación I/O, además de que sin un DMA no habría multiprogramación real.

Es decir la cpu queda bloqueada además de que las operaciones io son lentas por lo que la multiprogramación sería bastante ineficiente.

###  ¿Cuánta RAM de video se necesita para dar soporte a una pantalla de texto monocromático de 25 líneas x 80 caracteres?

Aquí voy a hacer algunas intuiciones, la primera es que como es monocromatico entonces no hay un byte de color, la segunda es que usa algo tipo ASCII lo que nos da como resultado que cada elemento consume un byte.

Con esto sería multiplicar 80 * 1, lo cual obviamente es 80 y después multiplicar 80 * 25, lo cual nos da 2000, así que se necesitan 2000 bytes de ram de video para una pantalla de ese tipo.

### ¿Cuánta memoria de vídeo se necesita para un mapa de bits de 1024x768 y colores de 24 bits?

Lo primero sería multiplicar 1024 * 768 lo cual es 786,432, después convertimos 24 bits a bytes lo cual da 3 bytes. Por último efectuamos la multiplicación 786,432 * 3, que nos da 2,359,296 bytes.

## Hay varias metas de diseño a la hora de crear un sistema operativo, por ejemplo: la utilización de recursos, puntualidad, que sea robusto, etcétera. De un ejemplo de dos metas de diseño que puedan contradecirse entre sí

Portabilidad y optimización, la portabilidad habla que el sistema operativo tiene que correr en diferentes entornos sin muchas modificaciones, mientras que con optimización me refiero a que a veces los sistemas operativos para tener un mejor rendimiento suelen usar diferentes optimizaciones que dependen de la arquitectura (ejemplo cuando usas pusha en vez de hacer varios push) pero esto se contradice con la portabilidad porque las optimizaciones no son siempre portables.

### ¿Cuál de las siguientes instrucciones debe permitirse en modo kernel?

Deshabilitar todas las Interrupciones 
Leer el reloj de la hora del día
Establecer el reloj de la hora del día
Cambiar mapa de memoria
es la a y b, la a de hecho ya está en x86 debido que la instrucción cli con la que se deshabilitan las instrucciones es solo accesible en kernel mode.

Y la b es bastante obvia debido que el cambio de mapa de memoria es una operación que requiere cambiar cosas de la MMU que solo se puede ejecutar en kernel mode.

### Cuando un programa de usuario realiza una llamada al sistema para leer o escribir en un archivo en disco, proporciona una indicación de que archivo desea un apuntador al buffer de datos y la cuenta.

Después el control se transfiere al sistema operativo, el cual llama al driver apropiado. Suponga que el driver inicia el disco y termina hasta que ocurre una interrupción. En el caso de leer del disco, es obvio que el procedimiento que hizo la llamada tiene que ser bloqueado (debido a que no hay datos para leer). ¿Qué hay del caso de escribir en el disco? ¿Necesita ser bloqueado el procedimiento llamador, para esperar a que se complete la transferencia del disco?

Técnicamente no, porque no se escribe directamente al disco físico si no que se copia el buffer a un buffer interno del kernel.

Así que no tendrías que esperar a que la escritura se complete en una escritura.

### ¿Qué es una instrucción de trap?

Una instrucción de trap es una instrucción que permite de forma manual generar una trap, las más conocidas son instrucciones como int, syscall (también conocida en Riscv como ecall).

### ¿Cuál es la diferencia entre una trap y una interrupción?

Principalmente que las interrupciones vienen del hardware, mientras las traps del software, otra cosa es lo que notifican mientras las Interrupciones notifican de información del hardware (ejemplo: un disco duro puede notificar con una interrupción que ya terminó de ejecutar una cierta operación) mientras que una trap notifican cosas del software (ejemplo una trap puede notificar una llamada al sistema o un error como división por 0)

### ¿Cuál es el propósito de una llamada al sistema en un sistema operativo?

Para conectar el espacio del kernel con el espacio del usuario.

### Para cada una de las siguientes llamadas al sistema, proporcione una condición que haga que falle: fork, exec y unlink

Fork: si el sistema tiene algún límite de procesos fork puede dar error

Exec: si el archivo del ejecutable no existe o no es válido

Unlink: si no tienes permisos para acceder a ese archivo

### ¿Podría la llamada cuenta = write(fd, bufer, nbytes); devolver un valor de cuenta distinto de nbytes? Si es así, ¿Por qué?

En caso de error cuenta tendrá el código de error en vez de nbytes

### ¿Cuál es la diferencia esencial entre un archivo especial de bloque y un archivo especial de carácter

El de carácter su lectura/escritura es línear (ejemplo una terminal) mientras que el de bloque permite rw aleatorio (ejemplo un disco duro)

### En el ejemplo que se da en la figura 1-17, el procedimiento de biblioteca se llama read y la misma llamada al sistema se llama read, ¿Es esencial que ambos tengan el mismo nombre? Si no es así, ¿Cuál es más importante?

No es 100% necesario que tengan el mismo nombre, aunque sería más importante el nombre de la función de la biblioteca debido que es el que llama el programador (excluyendo a los devs de assembly que algunos usan directamente las llamadas al sistema).

### El modelo cliente-servidor es popular en los sistemas distribuidos. ¿Puede utilizarse también en un sistema de una sola computadora?

Si, de hecho muchos microkernels usan este modelo en dónde el cliente es el proceso y el servidor es algún tipo de driver 

### Para un programador, una llamada al sistema se ve igual que cualquier otra llamada a un procedimiento de biblioteca. ¿Es importante que un programador sepa cuáles procedimientos de biblioteca resultan en llamadas al sistema?

No, aunque es lo mejor, pero por ejemplo win api expone varias llamadas, que no necesariamente son llamadas al sistema
