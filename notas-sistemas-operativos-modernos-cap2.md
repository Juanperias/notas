# Notas variadas de la lectura del capítulo 2 del libro Sistemas operativos modernos.

## Nota 

Este recurso son más que todo ideas cortas y no está hecho para ser un resumen del capítulo

## los sistemas tipo unix cuando se hace un fork suelen compartir la sección de texto debido que siempre es de solo lectura 

## en algunos sistemas cuando se elimina un proceso todos los procesos que creo se eliminan, aunque ni unix ni Windows trabajan de esta forma

## en sistemas tipo unix los procesos son un árbol con el proceso de inicio (init) en la raíz

## existen 3 estados de proceso: en ejecución, en espera, bloqueado 

### en ejecución significa que tiene el control de la cpu

### en espera significa que se puede ejecutar pero el sistema operativo decidió darle la cpu a otro proceso.

### bloqueado significa que el sistema no puede avanzar por lógica debido a la espera de un recurso (ejemplo la lectura de un disco duro)

## El sistema operativo mantiene una tabla de procesos (un array) que cada entrada tiene el PCB (bloque de control de proceso)

## Cuando se ejecuta una interrupción el 
manejador de interrupciones suele cambiar la pila a una dirección específica (en el caso de xv6 por ejemplo cada proceso tiene una página de 4 kib la cual es el kernel stack usada por el dekernel al momento de cargar las Interrupciones), después guarda todos los registros, el mapa de memoria, etc y llama a un procedimiento escrito en un lenguaje de alto nivel (ejemplo: C, rust, zig etc), después de eso hay 2 casos, el primero es manejar la interrupción y devolverle el control al proceso actual, el segundo es manejar la interrupción y cambiar de proceso (esto en xv6 suele pasar cuando hay una interrupción del timer)

## si tenemos un tiempo gastado esperando en E/S es p, con n procesos en memoria la probabilidad de que todos los procesos esten esperando por E/S es de pn, lo cual se le conoce como grado de multiplicación (algo importante es que este método supone que hay 5 procesos 3 corriendo y 2 inactivos pero en una sola CPU con un solo núcleo no pueden haber 3 procesos al mismo tiempo)

## Los hilos son como tener varios mini-procesos dentro de un proceso la diferencia entre varios procesos y varios hilos es que los procesos son independientes y no comparten el espacio de memoria, los hilos comparten el espacio de memoria lo cual es útil para aplicaciones que desean compartir información 

## Otra cosa es que los hilos son más fáciles de crear y destruir que los procesos (debido que no son tan completos como un proceso, para crear un proceso se necesita hacer varias allocations mientras que para un hilo son unas pocas y así sucesivamente)

## una forma es que los hilos se pueden tratar como procesos la diferencia es que comparten el mismo espacio de direcciónes, archivos abierto etc

## Al igual que un proceso un hilo puede tener varios estados bloqueado, listo, 
