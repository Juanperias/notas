# PEIQS

## Abstracto 

PEIQS (**P**rocess **E**xecution **I**s **Q**uantum **S**election) , es un algoritmo sencillo el cual se basa en una serie de comportamientos de un proceso para determinar su Quantum.

Dentro de este documento se verán los problemas que se han planteado y una posible solución.

## Introducción 

Este algoritmo es bastante sencillo y fue planeado de esta forma, imagine que tiene un proceso A y un proceso B, A ejecuta mayormente E/S por lo que queremos darle el suficiente tiempo para que entre a una operación E/S, y B realiza cálculos de X cosa.

Por lógica queremos que B tenga más Quantum que A, y aquí es donde entra está parte del algoritmo.

Algo importante de recalcar es que este algoritmo es apropiado.

Entonces tomemos empecemos por la raíz, A y B se crean en este caso se crean con un Quantum por defecto ejemplo DQ, primero se ejecuta A y el sistema le da a A su Quantum hasta que se presente algún tipo de E/S bloqueante, este primer caso es una penalización de Quantum del sistema hacia el proceso A.

Cuando A conmuta a B el sistema de recalcular su Quantum basado en el porqué de la conmutación por ejemplo si es E/S se puede reducir el Quantum en N.

Cuando se ejecuta B quizás haga unas pocas syscalls para reservar memoria por lo que el sistema podría darle una recompensa de N en su Quantum.

## Causas que provocan una modificación del quantum

El quantum puede ser incrementado/decrementado dependiendo de lo que pase por eso desarrolle esta tabla que menciona varios casos posibles y su consecuencia visible en el quantum.

| Acción | Consecuencia | Explicación |
| :---- | :---- | :---- |
| Terminación de un proceso /hilo | Ninguna | Este proceso no se va a volver a ejecutar |
| Lectura de E/S | Penalización de nivel medio (además de conmutación inmediata a otro proceso) | La lectura de E/S muchas veces suele cara, además de que el proceso queda en estado de bloqueo |
| Escritura de E/S | Penalización leve (no es necesario conmutación) | Los kernels modernos suelen intentar agrupar varias escrituras en el algunos dispositivos, además de que esta penalización leve intenta predecir si el programa va a hacer puramente escrituras |
| Syscalls relacionadas con el filesystem | Penalización leve (depende mucho de la syscall) | Algunas syscalls pueden ser extremadamente rápidas por lo que no es necesario penalización, en las mas lentas se suele hacer una penalización leve  |
| Syscalls relacionadas con la memoria | Aumento leve del quantum  | Esto se debe para intentar predecir si el proceso intenta reservar memoria para un algoritmo |
| Syscalls relacionadas con creación de procesos | Ninguna | Esto es porque un proceso que cree muchos procesos como una shell puede obtener más quantum del que necesita |
| Creación de hilos | Ninguno | Cada hilo tiene su propio quantum |

Pero el objetivo de la tabla se entiende perfectamente, favorecer a llamadas al sistema/acciones que simbolizan cálculo de alguna información y penalizar a acciones que son lentas.

## Definición de penalización

Una penalización es cuando el quantum es decrementado por un motivo X, el decremento del quantum depende completamente del motivo

## Problema 1: Quantum sin un límite

Ahora si ves que un proceso puede crecer o decrecer indefinidamente, este problema se soluciona de manera sencilla teniendo un límite L y un mínimo M, el sistema debe asegurar que el Quantum sea igual o menor que L y que sea igual o mayor que M.

## Problema 2: Busy loop

Tome en cuenta que un proceso X empieza un loop sin hacer nada, técnicamente el sistema creería que esta haciendo cálculos intensos por lo cual empezaría a aumentar su quantum de esta forma este proceso puede causar molestia al usuario debido que fácilmente este proceso X puede llegar al límite L, la solución se tendría que encargar de encontrar y castigar con una reducción de quantum a procesos que simplemente hagan un busy loop.

La solucion podria ser que el sí en toda el quantum del programa no hay ninguna modificación de los registros, del stack, o una llamada al sistema se puede penalizar con una reducción del quantum de esta forma cuando el proceso X haga un busy loop indefinido empezará a reducir su quantum.

Otra cosa es verificar si el program counter es igual a la última vez en este caso es una penalización.

Para esto tiene que cumplir al menos 2 de las condiciones anteriormente mencionadas si no será una reducción de quantum algo sutil pero que en unas iteraciones puede hacer que el proceso X llegue al M de quantum.

## Problema 3: Abuso de hilos para obtener más quantum

Si leíste la tabla pensarás que no tiene sentido que el sistema favorezca cuando se ejecuta algo relacionado con el cálculo pero la creación de hilos no.

Esto se debe a que imagínese que tiene un proceso X y este proceso crea una N cantidad de hilos (suponga que N es un número grande) ese proceso si recibiera una compensación por crear hilos llegaría al límite sin hacer prácticamente nada.

La solución a esto es bastante sencilla y es quitar la compensación por creación de hilos y que cada hilo tenga su propio quantum, cuando se crea un hilo recibe el quantum por defecto.

## 

## Problema 4: Abuso de quantum alto

Imagínese que un hilo/proceso llega hasta un quantum alto (por ejemplo por hacer cálculos complejos) el sistema después de los cálculos va a seguir recibiendo un quantum alto y tendría que pasar por varios procesos para llegar a un quantum normal (suponiendo que ya no va a hacer más cálculos).

La solución es bastante sencilla y es simplemente restar al quantum un valor constante (que no sea muy grande).

## Conclusión

Teniendo en cuenta todo los problemas planteados y su solución vamos a definir de nuevo este algoritmo.

Siguiendo con el ejemplo del proceso A y B, A tiene 1 solo hilo, mientras que B tiene 2 (B1, B2).

Es importante recalcar que B1 Y B2 tienen quantums diferentes, todos los hilos tienen quantums diferentes (algo importante es que si el sistema trabaja con procesos en vez de con hilos como unidades de ejecución entonces sería que todos los procesos tienen quantums diferentes).

El sistema debe declarar un límite de quantum y un mínimo de quantum los cuales dejarían al posible rango de quantums de L-M (en donde L es el límite y M es el mínimo).

También el sistema debe de hacer un rastro (algo no tan complejo) de la actividad de la unidad de ejecución (hilos/procesos) para después ajustar su quantum y además a este ajuste restarle el valor constante para “normalizar” el quantum.

Lo de las penalizaciones se sigue por la tabla y además la esencia que deja la tabla.

Al final el sistema debería tener un quantum justo en donde cada proceso tiene el quantum que necesita, cuando termina ese quantum sin importar lo que pase se conmuta a otro proceso, y cuando la unidad de ejecución invoca una operación de E/S de lectura también se conmuta a otro proceso.

Otra cosa importante es que el programador que haga la implementación de este sistema también tiene que tener en cuenta que este algoritmo respeta el estado actual del proceso (es decir sólo conmuta a procesos que estén en Ready, algunas operaciones bloquean la unidad de ejecución y así)

