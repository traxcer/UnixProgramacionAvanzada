## Estructura del Sistema

### Niveles en la arquitectura del sistema UNIX

![CAPAS_Unix](https://user-images.githubusercontent.com/4338310/217193279-33506ff1-5aec-477f-8e43-1ab2cad5b45a.gif)

> El nivel más interno (Hardware) no pertenece realmente al SO, es la máquina física sobre la que se monta y cuyos recursos quremos gestionar.

> El núcleo (Kernel) está escrito en C, para recodificar UNIX de forma independiente de la Máquina.

> Programas estándart de cualquier sistema Unix y programas generados por los usuarios. Estos programas no interactuan nunca directamente con el Hardware. El recurso que nos permite operar sobre un determinado recurso del sistema se denomina ***llamada al sistema***, la cual será el objeto principal de nuestro estudio.

> Programas que se valen de otros programas para funcionar. Como el compilador de C (cc).



## Arquitectura del Sistema operativo Unix

La arquitectura Unix esta centrada en dos conceptos, los ficheros y los procesos. El nucleo es el encargado de facilitarnos los servicios relacionados con el control de procesos y los ficheros.
La arquitectura de Unix vamos a dividirla en tres niveles, *Hardware, núcleo y usuario*.  

![bloques_del_nucleo](https://user-images.githubusercontent.com/4338310/218033961-2323069f-8895-4669-977b-ea01cee51cb5.png)

Las llamadas al sistema y su biblioteca asociada representan la frontera entre los programas del usuario y el núcleo. La **biblioteca asociada** a las llamadas es el mecanismo mediante el cual podemos invocar una llamada desde un programa C. Esta biblioteca se enlaza por defecto al compilar cualquier programa C y se encuentra
en el fichero **/usr/lib/libc.a**.

Las llamadas al sistema se ejecutan en modo protegido (modo kernel o supervisor), y para entrar en este modo hay que ejecutar una sentencia en código máquina conocida como interrupción software (trap).

**El núcleo** está dividido en dos subsistemas principales: ***subsistema de ficheros*** y ***subsistema de control de procesos***. 

El subsistema de ficheros controla los recursos del sistema de ficheros y tiene funciones como reservar espacio para los ficheros, administrar el espacio libre, controlar el acceso a los ficheros, permitir el intercambio de datos entre los ficheros y el usuario, etc. Los procesos interaccionan con este subsistema a través de unas llamadas específicas (open, read, write, status, chown, etc.).

- El ***subsistema de ficheros*** se comunica con los dispositivos de almacenamiento secundario —discos duros, unidades de cinta, etc.— a través de los manejadores de dispositivo (device drivers). Los manejadores de dispositivo se encargan de proporcionar el protocolo de comunicación (handshake) entre el núcleo y los periféricos. Se consideran *dos tipos de dispositivos según la forma de acceso*: 
  
  - ***dispositivos modo bloque*** (block devices): El acceso a los dispositivos en modo bloque se lleva a cabo con la intervención de memorias intermedias (buffers) que mejoran enormemente la velocidad de transferencia. 
  
  - ***dispositivos modo carácter*** (row devices): El acceso a dispositivos en modo carácter se lleva a cabo de forma directa, sin la intervención de estas memorias.

Un mismo dispositivo físico puede ser manejado **tanto en modo bloque como en modo carácter**, dependiendo de qué manejador usemos para acceder a él.

- El ***subsistema de control de procesos*** es el responsable de la planificación de los procesos —scheduling o dispatching—, su sincronización, comunicación entre los mismos (IPC-inter process comunication) y del control de la memoria principal. Algunas de las llamadas usadas para controlar procesos son: fork, exec, exit, wait, brk, signal, etc.

  - El ***módulo de gestión de memoria*** se encarga de controlar qué procesos están cargados en la memoria principal en cada instante. Si en un momento determinado no hay suficiente memoria principal para todos los procesos que lo solicitan, el gestor de memoria debe recurrir a mecanismos de intercambio (swapping) para que todos los procesos tengan derecho a un tiempo mínimo de ocupación de la memoria y se puedan ejecutar. El intercambio consiste en llevar los procesos cuyo tiempo de ocupación de la memoria expira a una memoria secundaria en el disco (área de swap) que se monta como un sistema de ficheros aparte, y traer de esa memoria secundaria los procesos a los que se les asigna tiempo de ocupación de la memoria principal. Al módulo gestor de memoria se le conoce también como *intercambiador* (swapper).

  - El ***distribuidor*** (también despachador o scheduler) se encarga de gestionar el tiempo de CPU que tiene asignado cada proceso. El distribuidor entra en ejecución controlado por la interrupción del reloj y decide si el proceso actual tiene derecho a seguir ejecutándose (esto depende de su prioridad y de sus privilegios) o ha de conmutarse de contexto (asignarle la CPU a otro proceso). La comunicación entre procesos puede realizarse de forma asíncrona (señales) o
síncrona (colas de mensajes, semáforos).

  - El ***módulo de control del hardware*** es la parte del núcleo encargada del manejo de las interrupciones y de la comunicación con la máquina. Los dispositivos pueden interrumpir a la CPU mientras está ejecutando un proceso. Si esto ocurre, el núcleo debe reanudar la ejecución del proceso después de atender a la interrupción. Las interrupciones no son atendidas por procesos, sino por funciones especiales, codificadas en el núcleo, que son invocadas durante la ejecución de cualquier proceso.




## Interfaz de las llamas al Sistema

Las llamadas al sistema de unix tienen un formato estándar, la documentación de las llamadas se encuentra en la **sección 2 de las páginas
del manual de unix*.

La forma de averiguar si la llamada a la función ha fallado es analizar el valor que devuelve. Este valor es -1 de forma estándar para todas las llamadas cuando se produce un error. Para averiguar cuál es el error producido, hemos de consultar el valor que toma la variable externa errno. 
En el fichero de cabecera <errno.h> hay una descripción de todos los valores que puede tomar errno. Los códigos de error tendrán un significado u otro según la llamada que los haya generado, por ello es recomendable consultar el manual para clarificar su significado.

A continuación se muestra un programa que visualiza por pantalla todos los códigos de error del sistema y una descripción asociada a cada uno de ellos.

Programa 1.1: Listado de códigos de error (errno.c)
```
#include <stdio.h>
  main (){
    int i;
    /∗ En el array sys_errlist hay una descripción corta asociada a cada número
       de error, sys_nerr es el total de elementos del array sys_errlist. ∗/
    for (i = 0; i < sys_nerr; i++)
      printf ("%d: %s\n", i, sys_errlist [i]);
}
```

Existen dos formas de obtener el mensaje de error asociado a la variable errno: 
- la primera es usar errno como índice para acceder al array sys_errlist y obtener una cadena de caracteres descriptiva del error; 
- la segunda es usar la función perror cuya declaración es:
  ```
    #include <stdio.h>
    void perror (char ∗str);
    ```
    y la forma de invocarla es ***perror ("Mi mensaje")***; 
    Si, por ejemplo, errno vale 2, por pantalla se desplegará el mensaje, **Mi mensaje: No such file or directory** donde '*No such file or directory*' es el mensaje asociado al código de error número 2.

