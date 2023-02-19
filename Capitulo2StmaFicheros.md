# Capitulo 2 Arquitectura del Sistema de Ficheros

## Características del sistema de ficheros

En Linux y Unix todo es un fichero. Los directorios son ficheros, los ficheros son ficheros, y los dispositivos son ficheros. A veces a los dispositivos se les llama nodos, pero siguen siendo ficheros.

Un sistema de ficheros es una abstracción lógica de los dispositivos físicos de almacenamiento que nos permite manejarlos sin necesitar conocer 
su arquitectura hardware. Caracterizado por:

- Tener estructura jerárquica.
- Tratamiento consistente de los datos en los ficheros.
- Posibilidad de crear y borrar ficheros.
- Permitir un crecimiento dinámico de los ficheros.
- Proteger los datos de los ficheros.
- Tratar los dispositivos y periféricos como si fuesen ficheros.

A nivel lógico el sistema de ficheros está organizado en forma de árbol invertido, con un nodo principal conocido como nodo raíz *(/)*. 
Cada nodo dentro del árbol es un directorio y puede contener a su vez otros nodos, *subdirectorios*, ficheros normales o ficheros de dispositivos.

Los nombres de los ficheros se especifican mediante la ruta *path name*, que describe cómo localizar un fichero dentro de la jerarquía del sistema. 
La ruta de un fichero puede ser absoluta *(referida al nodo raíz)* o relativa *(referido al directorio de trabajo actual)* **CWD** (current work directory).
Los programas que se ejecutan en unix no conocen el formato interno con el que el núcleo almacena los datos. Cuando accedemos al contenido de un fichero 
mediante una llamada al sistema ***read***, el sistema nos lo va a presentar como una **secuencia de bytes sin formato**. Nuestro programa es el encargado de 
interpretar la secuencia de bytes y darle significado según sus necesidades. Por lo tanto, la sintaxis *(forma)* del acceso a los datos de un fichero 
viene impuesta por el sistema y es la misma para todos los programas, y la semántica  *(significado)* de los datos es responsabilidad del programa que
trabaja con el fichero.


## Estructura del sistema de ficheros

Los sistemas de ficheros suelen estar situados en dispositivos de almacenamiento modo bloque, tales como cintas o discos. Las cintas tienen un tiempo de acceso mucho
más alto que los discos, por ello es poco práctico instalar un sistema de ficheros sobre ellas. Sin embargo, son muy útiles para realizar copias de seguridad 
(backup)* de un sistema ya instalado en disco.

Lo normal es que un sistema unix se arranque —boot— desde cinta o CDROM cuando queremos instalarlo por primera vez sobre un disco o cuando, tras producirse una quiebra
*crash* del sistema, queremos restaurarlo a su situación anterior *proceso conocido como *(**recovery system**)*. En el resto de nuestra discusión vamos a considerar 
que los sistemas de ficheros están instalados sobre discos.
Un sistema unix puede manejar **uno o varios discos físicos**, cada uno de los cuales puede contener **uno o varios sistemas de ficheros**. ***Los sistemas de ficheros son particiones lógicas del disco***.

Hacer que un disco físico contenga varios sistemas de ficheros permite una administración más segura, ya que si uno de los sistemas de ficheros se daña, perdiéndose la
información que hay en él, este accidente no se habrá transmitido al resto de los sistemas de ficheros que hay en el disco y podremos seguir trabajando con ellos para 
intentar una restauración o una reinstalación.

**El núcleo del sistema trabaja con el sistema de ficheros a un nivel lógico y no trata directamente con los discos a nivel físico**. Cada disco es considerado como un 
dispositivo lógico que tiene asociados unos números de dispositivo *(minor number y major number)*. Estos números se utilizan para indexar, dentro de una tabla de 
funciones, la que tenemos que emplear para acceder al manejador del disco. El manejador del disco se va a encargar de transformar las direcciones lógicas *(núcleo)* de 
nuestro sistema de ficheros a direcciones físicas del disco.

**Un sistema de ficheros se compone de una secuencia de bloques lógicos**, cada uno de los cuales tiene un tamaño fijo. **El tamaño de cada bloque es el mismo para 
todo el sistema de ficheros y suele ser múltiplo de 512 bytes**. A pesar de que el tamaño de un bloque es homogéneo en un sistema de ficheros, puede variar de un 
sistema a otro dentro de una misma configuración unix con varios sistemas de ficheros. **El tamaño elegido para el bloque va a influir en las prestaciones globales** del sistema. Por un lado, interesa que los bloques sean grandes para que la velocidad de transferencia entre el disco y memoria sea grande. Sin embargo, si los bloques
lógicos son demasiado grandes, la capacidad de almacenamiento del disco se puede ver desaprovechada cuando abundan los ficheros pequeños que no llegan a ocupar un 
bloque completo. Valores típicos en bytes para el tamaño de un bloque son: 512, 1024 y 2048.
En la figura podemos ver, en un primer nivel de análisis, la estructura que tiene un sistema de ficheros en el unix System V.

![estructurastmaarchivos1](https://user-images.githubusercontent.com/4338310/219594519-e1ee5115-e0dc-4a78-8b3a-8afafdc1458d.png)

En la figura podemos ver cuatro partes:
- **El bloque de arranque *(boot)**. Ocupa la parte del principio del sistema de ficheros, típicamente el primer sector, y puede contener el código de arranque. Este 
código es un pequeño programa que se encarga de buscar el sistema operativo y cargarlo en memoria para inicializarlo.
- **El superbloque** describe el estado de un sistema de ficheros. Contiene información acerca de su tamaño, el número total de ficheros que puede contener, qué 
espacio queda libre, etc.
- **La lista de nodos índice (***nodos-i***)** . Se encuentra a continuación del superbloque. Esta lista tiene una entrada por cada fichero, donde se guarda una 
descripción del mismo: situación del fichero en el disco, propietario, permisos de acceso, fecha de actualización, etc. El administrador del sistema es quien 
especifica el tamaño de la lista de nodos índice al configurar el sistema.
- **Los bloques de datos** empiezan a continuación de la lista de nodos índice y ocupan el resto del sistema de ficheros. En esta zona es donde se encuentra situado el
contenido de los ficheros a los que hace referencia la lista de nodos-i. Cada unode los bloques destinados a datos sólo puede ser asignado a un fichero, tanto si lo 
ocupa totalmente como si no.



### El superbloque

Como hemos visto anteriormente, en el superbloque está la descripción del estado del sistema de ficheros. En el fichero de cabecera **<sys/filsys.h>** hay declarada una estructura en C que describe el significado del contenido del superbloque. El superbloque contiene, entre otras, la siguiente información:

- Tamaño del sistema de ficheros.
- Lista de bloques libres disponibles.
- Índice del siguiente bloque libre en la lista de bloques libres.
- Tamaño de la lista de nodos-i.
- Total de nodos-i libres.
- Lista de nodos-i libres.
- Índice del siguiente nodo-i libre en la lista de nodos-i libres.
- Campos de bloqueo de elementos de las listas de bloques libres y de nodos-i libres. Estos campos se emplean cuando se realiza una petición de bloque o de nodo-i libre.
- Indicador que informa si el superbloque ha sido modificado o no.

Cada vez que, desde un proceso, se accede a un fichero, es necesario consultar el superbloque y la lista de nodos-i. Como el acceso a disco suele degradar bastante el
tiempo de ejecución de un programa, lo normal es que el núcleo realice la E/S con el disco a través de un buffer caché y que el sistema tenga en memoria una copia del superbloque y de la lista de nodos-i. Esto puede plantear problemas de consistencia de los datos, ya que una actualización en memoria del superbloque y de la tabla de nodos-i no implica una actualización inmediata en disco. La solución a este problema consiste en realizar periódicamente una actualización en disco de los datos de administración que mantenemos en memoria. De esta tarea se encarga un demonio que se arranca al inicializar el sistema.
Naturalmente, antes de apagar el sistema también hay que actualizar el superbloque y las tablas de nodos-i del disco. El programa shutdown se encarga de esta tarea y ***es fundamental tener presente que no se debe realizar una parada del sistema sin haber invocado previamente al programa shutdown***, porque de lo contrario el sistema de ficheros podría quedar seriamente dañado.



### Nodos índice (inodes)

Cada fichero en un sistema unix tiene asociado un nodo-i. El nodo-i contiene la información necesaria para que un proceso pueda acceder al fichero. 
Esta información incluye: propietario, derechos de acceso, tamaño, localización en el sistema de ficheros, etc.

La lista de nodos-i se encuentra situada en los bloques que hay a continuación del superbloque. Durante el proceso de arranque del sistema, el núcleo lee la lista de 
nodos-i del disco y carga una copia en memoria, conocida como tabla de nodos-i.

Las manipulaciones que haga el subsistema de ficheros (*parte del código del núcleo*) sobre los ficheros van a involucrar a la tabla de nodos-i pero no a la lista de 
nodos-i. Mediante este mecanismo se consigue una mayor velocidad de acceso a los ficheros, ya que la tabla de nodos-i está cargada siempre en memoria. En el párrafo 
anterior vimos que hay un demonio del sistema (*syncer*) que se encarga de actualizar periódicamente el contenido de la lista de nodos-i con la tabla de nodos-i.

En el fichero de cabecera **<sys/ino.h>** se declara la estructura C ***struct dinode*** que describe la información de un nodo-i. Los campos que componen un nodo-i son los siguientes:
- Identificador del propietario del fichero. La posesión se divide entre un propietario individual y un grupo de propietarios y define el conjunto de usuarios que 
tienen derecho de acceso al fichero. El superusuario tiene derecho de acceso a todos los ficheros del sistema.
- Tipo de fichero. Los ficheros pueden ser ordinarios de datos, directorios, especiales de dispositivo (*en modo carácter o en modo bloque*) y tuberías (*o FIFO*).
- Tipo de acceso al fichero. El sistema protege los ficheros estableciendo tres niveles de permisos: permisos del propietario, del grupo de usuarios al que pertenece 
el propietario y del resto de los usuarios (*conocido también como el mundo*). Cada clase de usuarios puede tener habilitados o deshabilitados los derechos de lectura,
escritura y ejecución. Para los directorios, el derecho de ejecución significa poder acceder o no a los ficheros que contiene.
- Tiempos de acceso al fichero. Dan información sobre la fecha de la última modificación del fichero, la última vez que se accedió a él y la última vez que se 
modificaron los datos de su nodo-i.
- Número de enlaces del fichero. Representa el total de los nombres que el fichero tiene en la jerarquía de directorios. Como veremos más adelante, un fichero puede
- tener asociados diferentes nombres que correspondan a diferentes rutas y a través de los cuales accedamos a un mismo nodo-i y por consiguiente a los mismos bloques 
de datos.
- Entradas para los bloques de dirección de los datos de un fichero. Si bien los usuarios ven los datos de un fichero como si fuesen una secuencia de bytes contiguos, 
el núcleo puede almacenarlos en bloques que no tienen por qué ser contiguos. En los bloques de dirección es donde se especifican los bloques de disco que contienen los datos del fichero.
- Tamaño del fichero. Los bytes de un fichero se pueden direccionar indicando un desplazamiento a partir de la dirección de inicio del fichero —desplazamiento 0—.
El tamaño del fichero es igual al desplazamiento del byte más alto, incrementado en una unidad. Por ejemplo, si un usuario crea un fichero y escribe en él un solo byte
en la posición 2.000, el tamaño del fichero es 2.001 bytes.

Hay que hacer notar que el **nombre del fichero no queda especificado en su nodo-i**. Como veremos más adelante, es en los ficheros de tipo directorio donde a cada 
nombre de fichero se le asocia su nodo-i correspondiente. 
También hay que reseñar la diferencia entre escribir el contenido de un nodo-i en disco y escribir el contenido del fichero. El contenido del fichero (*sus datos*) 
cambia sólo cuando se escribe en él. El contenido de un nodo-i cambia cuando se modifican los datos del fichero o la situación administrativa del mismo (*propietario, 
permisos, enlaces, etc.*)
La tabla de nodos-i contiene la misma información que la lista de nodos-i, junto con la siguiente información adicional:
- El estado del nodo-i, que indica:
  - si el nodo-i está bloqueado;
  - si hay algún proceso esperando a que el nodo-i quede desbloqueado;
  - si la copia del nodo-i que hay en memoria difiere de la que hay en el disco;
  - si la copia de los datos del fichero que hay en memoria difiere de los datos que hay en el disco (*caso de la escritura en el fichero a través del buffer caché*).
- El número de dispositivo lógico del sistema de ficheros que contiene al fichero.
- El número de nodo-i. Como los nodos-i se almacenan en el disco en un array lineal, al cargarlo en memoria, el núcleo le asigna un número en función de su posición en 
el array. El nodo-i del disco no necesita esta información.
- Punteros a otros nodos-i cargados en memoria. El núcleo enlaza los nodos-i sobre una cola dispersa —cola hash— y sobre una lista libre. Las claves de acceso a la 
cola dispersa nos las dan el número de dispositivo lógico del nodo-i y el número de nodo-i.
- Un contador que indica el número de copias del nodo-i que están activas (*por ejemplo, porque el fichero está abierto por varios procesos*).



### Los bloques de datos. Estructura de un fichero ordinario

Los bloques de datos están situados a partir de la lista de nodos-i. Como se mencionó anteriormente, cada nodo-i tiene unas entradas (*bloques de direcciones*) para 
localizar dónde están los datos de un fichero en el disco. Como cada bloque del disco tiene asociada una dirección, las entradas de direcciones consisten en un 
conjunto de direcciones de bloques del disco.

Si los datos de un fichero se almacenasen en bloques consecutivos, para poder acceder a ellos nos bastaría con conocer la dirección del bloque inicial y el tamaño del
fichero. Sin embargo, esta política de gestión del disco produce a la larga un desaprovechamiento del mismo, ya que tenderán a proliferar áreas libres demasiado 
pequeñas para poder ser usadas. Por ejemplo, supongamos que un usuario crea tres ficheros, A, B y C, cada uno de los cuales ocupa 10 bloques en el disco. Supongamos 
que el sistema sitúa estos tres ficheros en bloques contiguos —figura 2.2 (a)—. Si el usuario necesita añadir 5 bloques al fichero central —fichero B— porque ha 
aumentado de tamaño, el sistema tendrá que buscar 15 bloques contiguos que se encuentren libres para copiar el fichero B en esa zona —figura 2.2 (b)—. Esto plantea dos 
inconvenientes, el primero es el tiempo que se pierde en copiar los 10 bloques de B que no varían de contenido, el segundo es que los 10 bloques que quedan libres sólo 
pueden contener ficheros con un tamaño inferior o igual a 10 bloques. Esto provoca a la larga una microfragmentación del disco que lo va a dejar inservible.

![fragmentacion](https://user-images.githubusercontent.com/4338310/219972384-8fd75f8d-c9c2-4642-821d-c77dbe85d26c.png)

El núcleo puede minimizar la fragmentación del disco ejecutando periódicamente procesos para compactarlo, pero esto produce una degradación de las prestaciones del 
sistema en cuanto a velocidad.

Para una mayor flexibilidad, el núcleo reserva los bloques para un fichero de uno en uno, y permite que los datos de un fichero estén esparcidos por todo el sistema de 
ficheros. Este esquema de reserva complica la tarea de localizar los datos. Las entradas de direcciones del nodo-i consisten en una lista de direcciones de los
bloques que contiene los datos del fichero. Un cálculo simple nos muestra que almacenar la lista de bloques del fichero en un nodo-i es algo difícil de implementar. 
Por ejemplo, si los bloques son de 1 Kbyte y el fichero ocupa 10 Kbytes —10 bloques—, vamos a necesitar almacenar 10 direcciones en el nodo-i. Pero si el fichero es de 
100 Kbytes (*100 bloques*), necesitaremos 100 direcciones para acceder a todos los datos. Así pues, esta gestión nos impone que el tamaño del nodo-i sea variable, ya 
que, si lo fijamos en un valor concreto, estamos fijando el tamaño máximo de los ficheros que podemos manejar. Debido a lo poco manejable que resulta, la idea de nodos-i de tamaño variable es algo que no implementa ningún sistema.

Para conseguir que el tamaño de un nodo-i sea pequeño y a la vez podamos manejar ficheros grandes, las entradas de direcciones de un nodo-i se ajustan al esquema que
muestra la figura 2.3. En el unix System V, los nodos-i tienen una tabla con 13 entradas. Las 10 entradas marcadas como directas en la figura contienen direcciones de 
bloques en los que hay datos del fichero. La entrada marcada como indirecta simple direcciona un bloque de datos que contiene una tabla de direcciones de bloques de 
datos. Para acceder a los datos a través de una entrada indirecta, el núcleo debe leer el bloque cuya dirección nos indica la entrada indirecta y buscar en él la 
dirección del bloque donde realmente está el dato, para a continuación leer ese bloque y acceder al dato. La entrada marcada como indirecta doble contiene la dirección 
de un bloque cuyas entradas actúan como entradas indirectas simples, y la entrada indirecta triple direcciona un bloque cuyas entradas son indirectas dobles.

![direcciones_en_inode](https://user-images.githubusercontent.com/4338310/219972561-215da635-6d86-41f1-a98c-0400cb68afbf.png)

Este método puede extenderse para soportar entradas indirectas cuádruples, quíntuples, etc., pero en la práctica tenemos más que suficiente con una indirección triple.
Vamos a ver el caso práctico en el que los bloques son de 1 Kbyte y el bloque se direcciona con 32 bits —232 = 4 Gdirecciones posibles—. En esta situación, un bloque 
de datos puede almacenar 256 direcciones de bloques y un fichero podría llegar a tener un tamaño del orden de 16 Gbytes —consulte la tabla 2.1—, pero teniendo en 
cuenta que el campo tamaño del fichero del nodo-i es de 32 bits, el tamaño máximo de un fichero no es del orden de los 16 Gbytes, como indica la tabla, sino de 4 
Gbytes. Este tamaño, en la práctica, resulta más que suficiente.

Los procesos acceden a los datos de un fichero indicando la posición, con respecto al inicio del fichero, del byte que queremos leer o escribir. El fichero es visto 
como una secuencia de bytes que empieza en el byte número 0 y llega hasta el byte cuya posición, con respecto al inicial, coincide con el tamaño del fichero menos uno. 
El núcleo se encarga de transformar las posiciones de los bytes, tal y como las ve el usuario, a direcciones de los bloques de disco.

![capacidad_direccionamienot_inode](https://user-images.githubusercontent.com/4338310/219972618-219adbd1-ced6-4ad5-8249-d825b298116c.png)

Veamos un ejemplo para clarificar estos conceptos. Supongamos un fichero cuyos datos están en los bloques que nos indican las entradas de direcciones del nodo-i 
descrito en la figura 2.4. Vamos a seguir suponiendo que el bloque tiene un tamaño de 1.024 bytes. Si un proceso quiere acceder a un byte que se encuentra en la 
posición 9.125 del fichero, el núcleo calcula que ese byte está en el bloque número 8 del fichero —empezando a numerar los bloques lógicos del fichero
En efecto,

![ej1_cap2](https://user-images.githubusercontent.com/4338310/219972650-00bfaaac-1363-4737-83f2-a5feeadb12b6.png)

La división la consideramos como división entera. Para ver a qué bloque del disco corresponde el bloque número 8 del fichero, hay que consultar el número de bloque que
almacena la entrada número 8 —entrada de dirección directa— de la tabla de direcciones del nodo-i. En el caso de la figura, el bloque de disco buscado es el 412. 
Dentro de este bloque, el byte 9.125 del fichero se corresponde con el byte 933 con respecto al inicio del bloque —los bytes del bloque se numeran desde 0 a 1.023—.
En efecto,

![ej2Cap2](https://user-images.githubusercontent.com/4338310/219972691-b3806336-5a47-4bab-8715-ed6500ed3a7b.png)

El operador % indica resto de la división entera. 
En este ejemplo el cálculo ha sido sencillo porque el byte buscado era accesible desde una entrada directa del nodo-i. Veamos qué ocurre si queremos localizar en el 
disco el byte que se encuentra en la posición 425.000 del fichero. Si calculamos su bloque lógico de fichero, veremos que se encuentra en el bloque 415,


![ej3cap2](https://user-images.githubusercontent.com/4338310/219972832-2d36c936-953b-4cb4-8803-2df4a7896a6f.png)

![ej4cap2](https://user-images.githubusercontent.com/4338310/219972839-be0e5a05-3c43-463a-a4d2-02a25c794b55.png)


El total de bloques al que podemos acceder con las entradas directas es de 10. Con la entrada indirecta simple podemos acceder a 256 bloques y, con la indirecta doble, 
a 65.536. Luego el byte buscado estará direccionado por la entrada indirecta doble. A esta entrada pertenecen los bloques comprendidos entre los números lógicos de 
fichero 266 y 65.581 —ambos inclusive— y el bloque buscado es el 415. Si nos fijamos en la figura, el número de bloque que contiene la entrada indirecta doble es el 
12.456, que es el bloque de disco donde están las direcciones de los bloques con entradas indirectas simples.

La entrada número 0 del bloque indirecto doble nos da acceso a los bloques de fichero comprendidos entre el 266 y el 521, ya que cada entrada actúa de indirección 
simple y da acceso a 256 bloques de datos. En la entrada 0 vemos que el número de bloque de disco del bloque indirecto simple que buscamos es 158. Dentro del bloque 
indirecto simple, la entrada que nos interesa es la diferencia entre 415 —bloque lógico del fichero— y 266 — bloque inicial al que da acceso el bloque indirecto doble
—; luego es 149. Según la figura, la entrada 149 del bloque de disco 158 contiene el número 9.126. Es en el bloque de disco 9.126 donde se encuentra el dato que buscamos, y en el byte 40 de este bloque está el byte 425.000 de nuestro fichero,

![ej5cap2](https://user-images.githubusercontent.com/4338310/219972904-30508b21-92f7-424f-ba0d-18cd136f99bb.png)

![ej6cap2](https://user-images.githubusercontent.com/4338310/219972916-accb38ec-ec11-4920-ab34-77ccfb8920db.png)

Si observamos la figura 2.4 con más detenimiento, vemos que hay algunas entradasdel nodo-i que están a 0. Esto significa que no referencian a ningún bloque del disco y 
que los bloques lógicos correspondientes del fichero no tienen datos. Esta situación se da cuando se crea un fichero y nadie escribe en los bytes correspondientes a 
estos bloques, por lo que permanecen en su valor inicial 0. Al no reservar el sistema bloques de disco para estos bloques lógicos, se consigue un ahorro de los 
recursos del disco. 

Imaginemos que creamos un fichero y sólo escribimos un byte en la posición 11048.276, esto significa que el fichero tiene un tamaño de 1 Mbyte. Si el sistema reservase 
bloques de disco para este fichero en función de su tamaño y no en función de los bloques lógicos que realmente tiene ocupados, nuestro fichero ocuparía 1.024 bloques 
de disco en lugar de 1, como en realidad ocupa.

La conversión de un desplazamiento de byte de fichero a su dirección de disco es un proceso laborioso, sobre todo si se ve involucrada una indirección triple. En este 
caso, el núcleo necesita acceder a tres bloques de direcciones, además de al nodo-i y al bloque de datos. Aun en el caso de que el núcleo encuentre los bloques en el 
buffer caché, la operación sigue siendo costosa porque debe hacer varios accesos al buffer y puede que tenga que esperar a que algunos buffers queden desbloqueados. La 
efectividad de este algoritmo queda limitada en la práctica por la frecuencia de uso de los ficheros grandes.


## Tipos de ficheros en unix

Como vimos en el epígrafe anterior, en cada nodo-i hay un campo que nos indica el tipo de fichero al que se refiere ese nodo-i. En el sistema unix hay cuatro tipos de 
ficheros: ordinarios (*también llamados regulares o de datos*), directorios, ficheros de dispositivos (*conocidos también como ficheros especiales*) y tuberías (*en 
inglés pipes o FIFOS*).

### Ficheros ordinarios

Los ficheros ordinarios contienen bytes de datos organizados como un array lineal. Esto no es nuevo para nosotros. Las operaciones que se pueden hacer con los datos de 
un fichero son:
- Leer o escribir cualquier byte del fichero.
- Añadir bytes al final del fichero, con lo que aumenta su tamaño.
- Truncar el tamaño de un fichero a cero bytes. Esto es como si borrásemos el contenido del fichero.

Se debe tener presente que las siguientes operaciones no están permitidas con los ficheros:
- Insertar bytes en un fichero, excepto al final. No hay que confundir la inserción de bytes con la modificación de los que ya existen (*proceso de escritura*).
- Borrar bytes de un fichero. No hay que confundir el borrado de bytes con la puesta a cero de los que ya existen.
- Truncar el tamaño de un fichero a un valor distinto de cero.

Dos o más procesos pueden leer y escribir concurrentemente sobre un mismo fichero. Los resultados de esta operación dependerán del orden de las llamadas de 
entrada/salida individuales de cada proceso y de la gestión que el planificador haga de los procesos, y en general son impredecibles.

Hasta hace poco, unix no poseía mecanismos eficientes para controlar el acceso concurrente a los ficheros. Recientemente, algunas versiones de unix proporcionan 
facilidades de bloqueo de ficheros y de gestión de semáforos, para controlar el acceso a los ficheros. 

Los ficheros ordinarios, como tales, no tienen nombre y el acceso a ellos se realiza a través de los nodos-i.

