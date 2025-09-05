# Multiplexado

## Prefacio

Esta nota es una de las más cortas, está basada en la lectura del libro "Sistemas operativos modernos" y probablemente, mientras vas vaya leyendo este libro, esta nota crezca incluso hasta cambiar de nombre.

## ¿Qué es la multiplexación?

La multiplexación es compartir un recurso, pero empecemos por lo básico: definamos un recurso. Un recurso es cualquier elemento. Realmente, por ejemplo, la CPU es un recurso, pero también un archivo es un recurso.

Entonces cuando tenemos un recuso y varios elementos que quieren consumirlo se debe de multiplexar, un ejemplo claro es cuando tienes 2 procesos y la RAM (que vendría siendo el recurso) multiplexas este recurso entre los procesos para hacerlos creer que tienen su propia RAM cuando en verdad la comparten.

## Multiplexacion por tiempo

La multiplexación por tiempo es compartir un recurso por un tiempo determinado, en ese tiempo el elemento/proceso tiene acceso total al recurso, una vez que este tiempo acaba pierde el acceso al recurso para que otro proceso/elemento pueda acceder a este recurso, y comúnmente antes de la transición total se guarda el estado del proceso/elemento para cuando vuelva a tener el acceso al recurso entonces no parezca que tenga la enfermedad de Alzhéimer.

Un ejemplo claro de esto es la CPU, si tengo una sola CPU y 3 procesos necesito multiplexarla de tal forma de que los 3 procesos parezcan que tienen su propia CPU cuando en verdad están compartiendo el mismo recurso, siguiendo con el ejemplo del procesador el primer proceso se ejecuta (que es darle acceso al recurso al proceso) después de una N cantidad de tiempo o algún evento (ejemplo una llama E/S) el proceso pierde el acceso al recurso se guarda su estado y se ejecuta el siguiente proceso (para ejecutar un proceso comúnmente también se carga su estado).

Una analogía para explicar lo mismo (aunque siento que ya tengo 2 párrafos explicando lo mismo) es como si tuviéramos 3 hermanos y una madre, cada hermano puede jugar por 1 hora (aunque realmente los procesos se ejecutan por milisegundos) y cada vez que pasa la hora la madre le quita la consola de videojuegos guarda su partida y carga la partida del otro hermano y así sucesivamente hasta que un hermano termine el juego (lo que el equivalente en procesos  sería a que el proceso termina comúnmente esto se le notifica al sistema con la llamada exit en sistemas UNIX-like) entonces ese hermano deja de jugar y ahora la madre reparte entre los otros dos hermanos.

Ahora imagina eso, solo que en vez de hermanos son procesos y en vez de una madre es el sistema operativo.


## Multiplexacion por espacio

Imagina que tienes un recurso como la RAM y lo tienes que multiplexar entre 3 procesos. Si usáramos la multiplexación por tiempo, sería horrible, además de casi imposible de implementar.

En la multiplexación por espacio, tú les das un "espacio" que pueden consumir. Una cosa es que el sistema operativo debe de asegurar que el espacio que le esté dando a un proceso/elemento esté aislado de otros. En el caso de la memoria en los procesos, sí se suele a veces compartir regiones de memoria.

Vamos con un ejemplo técnico imagina que tenemos 2 procesos para que estos dos procesos parezcan que están corriendo de forma simultánea tenemos que multiplexar por tiempo la CPU y multiplexar por espacio la RAM en este caso el hardware tiene algo que se llama MMU y se separa la RAM por espacio usando tablas de páginas que contienen el espacio de dirección del proceso así cada proceso tiene su propio espacio direcciones que permite hacerle parecer que tiene su propia RAM cuando en verdad está recibiendo un pedazo de esta RAM.

Ahora, con el ejemplo informal, imagina que tienes un pan y 3 personas a las que repartirlo. La forma en que lo repartirías entre las 3 personas es cortar el pan entre 3 pedazos.

# Recursos

Algo importante que quiero aclarar de los recursos es que la información es mayormente el resultado de la lectura del libro de Tanenbaum. Las páginas de la Osdev las dejo para si quieren investigar en algunos conceptos que se mencionan con pinzas

- https://wiki.osdev.org/Paging#Page_Table
- https://wiki.osdev.org/Memory_Management_Unit
- Sistemas operativos modernos, Andrew S. Tanenbaum
