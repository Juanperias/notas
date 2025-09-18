# Procesos y hilos

## Prefacio

Bueno esta es una versión mejorada de lo que viene siendo mis notas del libro Sistemas operativos modernos de Andrew Tanenbaum, el cual es referencia en este recurso, pero no se queda ahí, trataré de aportar lo más que pueda de lectura de libros previos o de experiencia con viejos sistemas operativos que he desarrollado

## ¿Qué es un programa?

Un programa no es algo muy especial es simplemente un archivo el cual contiene información una analogía excelente es que un programa es como un libro de recetas, los ingredientes son los datos que tiene este programa (lo que suele venir siendo la sección de datos) y las receta viene siendo las instrucciones de este programa.

Ahora es difícil pensar en que es un programa simplemente con eso y cuando haces una lectura directa a un ejecutable (ejemplo haciendo algo como cat mi-programa) vas a obtener un montón de texto aleatorio y raro, pero ese texto aleatorio y raro es el ejecutable, solo que obviamente no se encodea en algún tipo de formato legible para el humano (aunque suele contenerlo por ejemplo cuando usas .assci en gnu-as o jusm), ahora si simplemente pusieras instrucciones y datos en el programa metido sería un desastre por eso se suele usar un formato como el ELF (usado en la mayoría de unix-like), aunque por ejemplo un bootloader para real-mode no suele estar en elf si no en binario crudo pero mejor ignoremos eso.

## Entonces, Que es un proceso?

Bueno en términos vagos es un programa en ejecución, es decir es llevar esas instrucciones y datos a memoria y después empezar a ejecutar estas instrucciones (cosa que ya es trabajo del CPU, pero la parte de leer el programa del disco y cargarlo en memoria es trabajo del sistema operativo), pero un proceso no solo se limita a ser eso, un proceso es algo mas complejo algunos autores lo suelen referenciar como PCB (process control block) por ejemplo en un sistema basado en unix suele tener los file-descriptors abiertos, alertas pendientes, procesos hijos etc (algunos sistemas operativos suelen tener diferentes propiedades ejemplo en Windows no existe una jerarquía de procesos como tal por ende en el PCB no hay una entrada de procesos hijos).

Aunque algo que te debe de haber generado una duda es sobre los procesos hijos y la jerarquía de procesos, esto es algo común en sistemas unix-like donde hay un proceso padre y uno hijo las jerarquías de procesos son como la de archivos tienen un inicio (en un sistema de archivos suele ser / en procesos suele ser el primer proceso que comúnmente es el init system).

Otra cosa de la que me es importante hablar es que en cada PCB se encuentra el contexto (y esto es algo que se incluye en todos los sistemas operativos) el contexto no es mas que el conjunto de registros en los que operaba el proceso, pero además contiene cosas como el apuntador de pila (rsp en x86_64, sp en riscv) el programa counter entre otras cosas comúnmente suele estar todos los registros de proposito general y otros registros dependiendo de la arquitectura.

Aunque parece tonto guardar los registros vamos a dar un ejemplo, suponga usted que estamos en una maquina x86_64 linux por ende estamos usando la convención de llamadas System V 64 (detalle importante) y tenemos dos procedimientos en C

(Nota: puede que se encuentre alguno que otro error de sintaxis si es así hágamelo saber)
```
int funcion_importante(int a, int b) {
	return a + b;
}

int main() {
	printf(“%n”, funcion_importante(2, 2));
}
```

En este caso damos por hecho que este código va a imprimir 4, ahora si lo ves desde un punto de vista en assembly se interpretaría como mov rdi, 2 mov rsi, 2 (si no entiende de que estoy hablando heche un vistazo a las convenciones de llamada y a system v) y damos por hecho que en rax se va a encontrar 4 pero suponga que no guardamos el contexto cuando cambiemos a otro proceso b, ese proceso va a interactuar con los registros de la cpu y vamos a tener el problema de que cómo no guardamos el contexto no sabemos cual es el program counter en que nos quedamos con nuestro proceso a, pero bueno omitamos este detalle suponga entonces que tiene un método para guardar el program counter cuando volvamos a ese código va a tener un mal-funcionamiento esto se debe a que nos quedamos con los valores de los registros del otro proceso.

Ahora puede ver usted que hasta el código más simple requiere de su contexto

## Contexto de un proceso

En la otra sección hable sobre el contexto de un proceso, pero en esta sección se va a tratar de forma mucho más detallada.

Ahora suponga que tenemos dos procesos (proceso a y proceso b), cada proceso tiene su propio contexto el cual es la serie de registros entre otras cosas que necesita ese proceso para funcionar, cuando se va a ejecutar el proceso a no se salta a lo bruto a ejecutar el codigo de ese proceso primero se agarra el contexto de ese proceso a ejecutar y se carga (una cosa importante es que el program counter no se carga como los demás registros de propósito general comúnmente suele ser de forma diferente cosa que explicare en otra seccion), y ya con el contexto del proceso cargado se puede ejecutar correctamente, este procedimiento tiene nombre y es llamado cambio de contexto.

Además cuando se cambia de un proceso a otro se guarda el estado de ese proceso para después volver a ser cargado (a menos que el proceso finalice en ese caso no es necesario guardar el contexto del proceso), aunque esto no es solo con el contexto el sistema operativo le debe de asegurar al proceso que toda la información esté en su lugar (por ejemplo si el proceso a abre un file descriptors y se cambia al proceso b el sistema operativo le debe de asegurar al proceso a que el file descriptor sigue ahí).

## Creando un proceso

Los procesos no se crean de la nada, comunmente suele ser porque un proceso crea a otro (ejemplo cuando usted escribe un comando en terminal ese comando es un proceso que fue creado por la shell), la creación de un proceso me gusta verla de dos formas la primera es del espacio de usuario y la segunda es las operaciones que hace el kernel para crear el proceso.

Empecemos desde la punta del iceberg, la parte de espacio de usuario es la que más suele cambiar por ejemplo en windows se suele usar CreateProcess, pero no voy a usar como referencia a windows por obvias razones, en mi opinion la mejor api para creacion de procesos la tiene unix que solo tiene una función que crea directamente un proceso la cual es fork, fork crea una copia exacta del proceso creando un proceso hijo, tiene 3 posibles casos de retorno el primero es 0 significando que el proceso actual es el hijo, el segundo es un número mayor a cero que significa que se está ejecutando el proceso padre ese número es el PID (process id) del proceso hijo, y el tercero es un número negativo significando un error.

Después se suele ejecutar una syscall tipo exec para cambiar la imagen de memoria, ejemplo de uso de fork:

```c
int main() {
    int pid = fork();

    if (pid < 0) {
         return -1;
     }

     if (pid == 0) {
	// Proceso hijo, Ejecutar alguna syscall como exec
      } else {
	// Proceso padre, aquí se puede ejecutar alguna syscall como waitpid para esperar a que el proceso hijo finalice
       }

      return 0;
}
```

una duda que puede surgir es porque no tener una syscall forkexec que hiciera todo por nosotros incluso mejor que fuera una syscall forkexecwait que cree el proceso y empiece a correr un programa y espere a que ese programa termine, esto puede tener sentido pero pierdes la flexibilidad de tenerlos como syscalls separadas, algo importante de decir es que en la jerarquía de procesos los cambios del hijo no afectan al padre, por ende comúnmente quieres modificar los file descriptors abiertos del hijo o hacer algo antes de hacer exec esto con nuestra syscall forkexec/forkexecwait seria no imposible pero muy incomodo cosa que teniéndolo en syscall diferentes seria mejor.

Un ejemplo claro de esto son las shells que necesitan hacer por ejemplo cambiar el stdin por un archivo en específico o usar Pipes esto requiere cambiar los file descriptors solamente del hijo, esto con syscalls separadas sería muy sencillo pero con syscalls unidas sería posible pero sería más código de forma innecesario y quedaría bastante feo.

## Estados de un proceso

Un proceso pasa por varias etapas a lo largo se su vida, que son en estado listo, bloqueado, en ejecución y zombie.

Cuando un proceso esta en su estado listo significa que el proceso puede ejecutarse pero por algún motivo no lo esta haciendo (uno de ellos es que el scheduler decidió que otro proceso debe ejecutarse).

Bloqueado significa que el proceso esta en espera de un recurso, por ejemplo cuando performas una lectura a un disco tu proceso entra en estado bloqueado en espera a que el disco termine de performar la lectura, cuando el proceso esta en este estado no puede ser ejecutado.

En ejecucion significa que el proceso esta ejecutándose ahora mismo, una vez que el sistema operativo cree que ya el proceso se ha ejecutado por suficiente tiempo lo manda al estado de listo, o tambien puede ser que el proceso en plena ejecución se ponga a la espera de un recurso en ese caso entra al estado de bloqueo.

Por último esta el estado zombie en este el proceso ya termino aunque el sistema operativo lo conserva debido que puede tener información importante que el padre de este proceso quiere obtener (como por ejemplo el codigo de retorno).

## Tabla de Procesos

Suponga que tenemos varios procesos activos, como ya sabra tenemos su contexto, sus file descriptors abiertos etc, ahora ¿Dónde se guarda toda esta information?, en la tabla de procesos, es bastante sencillo el concepto de una tabla de procesos no es más que una lista que puede crecer dinámicamente que contiene el PCB de cada proceso.

La tabla de procesos es bastante sencilla, aunque el comportamiento de cuando se agrega/elimina un proceso es importante.

Cuando se agrega un proceso se agrega a la tabla y en estado de listo, cuando se elimina un proceso a veces se suele poner en estado de zombie.


## ¿Como se produce el cambio de un proceso?

Muchos sistemas reparten al procesador entre varios procesos y aqui es donde entra la multiplexacion por tiempo (para más información vea la nota de multiplexacion), en la cual se le da a cada proceso un tiempo en el que puede ejecutarse, despues de que ese tiempo pase el sistema cambia de proceso no si antes guardar el contexto de ese proceso y poner su estado en listo.

Tambien puede pasar que el proceso se ponga a la espera de un recurso eso tambien es motivo para que el sistema operativo haga el cambio de proceso.

### Punto de vista técnico usando RISC-V

Ahora si, viene la parte mayormente practica, vamos a hacer la abstracción de un proceso usando como base la arquitectura riscv64, lo primero que todo voy a considerar que el sistema tiene algun tipo de timer ya que se encuentra en el modo supervisor, además de que tiene paginado (cosa que no suele estar en sistemas embedded).

En lo principal tomemos la abstracción del proceso com punto de partida vamos a tener en una struct los 32 registros de propósito general de riscv, después tendremos la tabla de paginas, eso seria lo básico le siguen cosas como los file descriptors abiertos, los procesos hijos etc.

Algo importante es tambien el ELF del archivo debemos de tener bien localizado la seccion de texto, datos etc, asi que podríamos crear un layout de procesos tal que asi (comunmente esto se suele poner de forma aleatoria)
Pagina trampolín
Pagina de trapframe
Stack del kernel
Pagina de guardia
Texto
Datos
Stack del usuario
Pagina de guardia
Heap

Los sistemas mas complejos usaran la página de guardia para manejar un overflow del stack y quizás agrandarlo, los demás lo usaran para que cuando ocurra un stack overflow matar al proceso, el kernel stack cobra mucha utilidad después al momento de manejar la syscall.

La pagina de trampolin es una pagina global que contiene codigo que realiza lo siguiente: 
Carga la tabla de paginas del kernel en satp
Guardar todos los registros del proceso de usuario
Carga el kernel stack en sp
Llamar a una funcion de algun lenguaje de alto nivel
La razón por la que se usa esta pagina es porque siempre esta mapeada en la tabla de paginas de todos los procesos y tambien en la tabla de páginas del kernel, algo importante es que pagina de trampolin tambien es usada en el kernel.

Ahora para los mas conocedores de riscv sabrán que para escribir en memoria (que es lo que buscamos guardando los registros en el trapframe del proceso) se necesita poner la direccion en un registro, el problema es que todos los registros de propósito general estan siendo ocupados por el proceso, asi que por eso riscv nos da un CSR que es sscratch (accesible con csrw y csrr) el cual guardamos cualquier registro (ejemplo a0) y ya que el contenido original de a0 esta en sscratch se puede usar a0 para poner la dirección del trapframe y escribir todo el contenido de los registros ahí.

Algo importante es que cuando un proceso se ejecuta la dirección de la pagina de trampolin suele estar en stvec, de esa forma cuando se produce una trap/interrupción se llama al trampolín de stvec, en la función del lenguaje de alto nivel se suele cargar el stvec del kernel y despues se procesa la interrupción.

Aqui hay una bifurcación si es una interrupción del timer el sistema va a cambiar de proceso (eligiendo algún proceso con el scheduler cargando la informacion de su PCB etc), si no entonces simplemente llama a alguna función de la pagina de trampolin que cargue los registros del trapframe (incluyendo el apuntador de pila, sp) y después aqui hay otra bifurcación si era una interrupción que no es syscall simplemente se retorna el pc sin modificación si es una syscall se agarra el PC y se le suma 4 (esto se debe a que el PC que nos da el cpu en sepc apunta a ecall, si ejecutaramos directamente tendríamos un loop infinito de syscalls, cuando le sumas 4 al PC pasas a la siguiente interrupcion, para más información vea la nota de decodeando instrucciones de riscv) y en cualquiera de los casos termina con una instrucción sret (configurada correctamente) para ejecutar lo que sea que el sistema operativo crea conveniente.


## Finalizando un proceso

Bien todo tiene que terminar, incluido los procesos, por eso todos los procesos terminan con una syscall exit (en sistemas unix) que le dice al sistema operativo que ya termino de ejecutarse y le pasa el codigo de retorno (0 si todo fue bien).

Por debajo el sistema operativo lo convierte en un proceso zombie y en algún punto liberaría la memoria del proceso.

### Le recomiendo que antes de pasar a la siguiente parte investigue un poco mas sobre lo mencionado aqui.


# TODO: hilos
