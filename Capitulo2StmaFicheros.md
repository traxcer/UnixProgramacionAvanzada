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

### Directorios
Los directorios son los ficheros que nos permiten darle una estructura jerárquica a los sistemas de ficheros de unix. Su función fundamental consiste en establecer la 
relación que existe entre el nombre de un fichero y su nodo-i correspondiente.

#### Estructura de un directorio en el unix System V

En esta versión de unix, un directorio es un fichero cuyos datos están organizados como una secuencia de entradas, cada una de las cuales contiene un número de nodo-i 
y el nombre de un fichero que pertenece al directorio. Al par [nodo-i]↔[nombre de fichero] se le conoce como enlace y puede haber varios nombres de ficheros —
distribuidos por la jerarquía de directorios— que estén enlazados con un mismo nodo-i.

El tamaño de cada entrada del directorio es de 16 bytes; dos dedicados al nodo-i y 14 dedicados al nombre del fichero. En la figura 2.5 podemos ver la estructura 
típica de un directorio.

![Captura de pantalla 2023-02-20 a las 7 51 19](https://user-images.githubusercontent.com/4338310/220033784-e55b85f7-6c82-496f-bee2-8a76b8f92653.png)

Las dos primeras entradas de un directorio reciben los nombres (.) y (..). El fichero . tiene asociado el nodo-i correspondiente al directorio actual y al fichero .. 
se le asocia el nodo-i del directorio padre del actual. Estas dos entradas están presentes en todo directorio, y en el caso del directorio raíz (/), el programa mkfs 
make file system, programa mediante el cual se crea un sistema de ficheros— se encarga de que el fichero (..) se refiera al propio directorio raíz.
El núcleo maneja los datos de un directorio con los mismos procedimientos empleados para acceder a los datos de ficheros ordinarios, usando la estructura nodo-i y los 
bloques de acceso directos e indirectos.

Los procesos pueden leer el contenido de un directorio como si se tratase de un fichero de datos, sin embargo no pueden modificarlos. El derecho de escritura en un 
directorio está reservado al núcleo. Esto es una medida de seguridad tomada para evitar que una manipulación incorrecta de un directorio pueda destruir un sistema de ficheros.

No hay razones estructurales para impedir la creación de múltiples enlaces a un directorio, sin embargo, esta posibilidad complicaría bastante la escritura de los 
programas que recorren el sistema de ficheros, por lo que el núcleo lo prohibe.

Los permisos de acceso a un directorio tienen los siguientes significados:
- Lectura, permite que un proceso lea ese directorio.
- Escritura, permite a un proceso crear una nueva entrada en el directorio o borrar alguna ya existente. Esto deberá hacerlo a través de las llamadas: creat, mknod,
link o unlink.
- Ejecución, autoriza a un proceso para buscar el nombre de un fichero dentro del directorio.

#### Estructura de un directorio en el sistema bsd

El concepto de directorio desempeña la misma función en el unix de Berkeley que en el de AT&T. La función principal es la de establecer los enlaces entre los nombres 
de los ficheros y los nodos-i. La diferencia entre los directorios de las versiones System V y bsd es que en ésta los nombres de los ficheros pueden ser más largos, 
hasta 255 caracteres, y no se reserva un espacio fijo de bytes para cada entrada del directorio.

Los directorios se encuentran situados en unidades conocidas como bloques de directorio (*chunks*) tal y como se muestra en la figura.

![Captura de pantalla 2023-02-20 a las 7 55 27](https://user-images.githubusercontent.com/4338310/220034485-aadfee29-f7a9-4533-800a-28ad56109f9b.png)

El tamaño de un bloque de directorio se elige de tal forma que pueda ser transferido en una sola operación con el disco. Esto permite actualizar los directorios en 
accesos atómicos —ningún proceso puede interrumpir para actualizar un directorio mientras haya otro que lo esté haciendo—.

Cada bloque de directorio se compone de entradas de directorio de tamaño variable. No se permite que una entrada esté distribuida en más de un bloque. Los tres primeros campos de una entrada de directorio son de tamaño fijo y contienen:
1. El tamaño de la entrada.
2. La longitud del nombre del fichero al que se refiere la entrada.
3. El número del nodo-i asociado al fichero.

El resto de la entrada es un campo de longitud variable que contiene una cadena de caracteres terminada con el carácter nulo. Esta cadena es el nombre del fichero y su
longitud máxima permitida es de 255 caracteres.

El espacio libre en un directorio se registra bajo en una o varias entradas que lo acumulan en su campo tamaño de la entrada. Estas entradas se reconocen porque su
tamaño es mayor que el necesario para almacenar sus campos de tamaño fijo más el campo nombre del fichero.

Cuando se borra una entrada de un directorio, el sistema añade el espacio que queda libre a la entrada anterior, si ésta pertenece al mismo bloque de directorio que la 
borrada, aumentado así el tamaño de la entrada. Si la primera entrada de un bloque de directorio está libre, el número de nodo-i que almacena esa entrada es cero, para 
indicar que no está reservada por ningún fichero.

#### Acceso al contenido de un directorio

Como ya hemos indicado, los procesos pueden leer el contenido de un directorio, pero no pueden modificarlo. Este privilegio está reservado exclusivamente al núcleo del 
sistema. 

Para leer un directorio podemos utilizar las mismas llamadas que empleamos para los ficheros ordinarios —open, read, lseek, close, etc.— El sistema bsd ofrece una 
interfaz más cómoda para movernos por el interior de la jerarquía de directorios. Esta interfaz también ha sido adoptada por el unix System V, y sus funciones son: 
opendir, readdir, rewindir, closedir, seekdir y telldir. Estas funciones no son parte del conjunto de llamadas al sistema, sino de la biblioteca estándar de E/S —
consulte directory(3C) y el § 4.1.5 pág. 122—.

Las funciones anteriores pueden codificarse a partir de las llamadas de manejo de ficheros [Kernighan & Ritchie, ], lo cual ofrece la ventaja de poder ser emuladas 
sobre una red o incluso por un sistema no unix.

#### Conversión de ruta de acceso a nodo-i

Desde el punto de vista del usuario, los ficheros se sitúan en la jerarquía de directorios y se nombran mediante su ruta de acceso. Llamadas como open, chdir o link 
reciben como parámetro de entrada la ruta de acceso de un fichero y no su nodo-i. El núcleo es quien se encarga de traducir la ruta de acceso de un fichero a su nodo-i 
correspondiente.

El algoritmo que realiza la transformación —llamado namei— se encarga de analizar los componentes de la ruta de acceso y de leer los nodos-i intermedios necesarios 
para verificar que se trata de una ruta correcta y que el fichero realmente existe.

Si la ruta es absoluta, la búsqueda del nodo-i del fichero se iniciará en el directorio raíz. Si la ruta es relativa, la búsqueda se iniciará en el directorio de 
trabajo actual que tiene asociado el proceso que quiere acceder al fichero.

A medida que se van recorriendo los nodos-i intermedios, se verifican los permisos para comprobar si el proceso tiene derechos de acceso a los directorios intermedios.

Como ejemplo, vamos a suponer que un proceso quiere abrir el fichero /etc/passwd que tiene una entrada por cada usuario del sistema y donde, entre otra información, 
figura la contraseña del usuario cifrada. Cuando el núcleo inicia el análisis de la ruta, encuentra que empieza por / y pasa a leer el nodo-i asociado al directorio 
raíz. Este nodo-i pasa a ser el nodo-i de trabajo y el núcleo verifica si corresponde a un directorio y si el proceso tiene permiso para buscar ficheros dentro de él. 
Suponiendo que los permisos están en regla, el núcleo toma el siguiente elemento de la ruta, que es etc, y busca, dentro del directorio raíz, alguna entrada cuyo 
nombre de fichero sea etc. Cuando el núcleo la encuentra, libera el nodo-i del directorio raíz y lee el nodo-i del fichero etc. En este nodo-i se refleja
que etc es otro directorio y por lo tanto en él podemos buscar el fichero passwd, tercer elemento de la ruta. El núcleo, tras verificar los permisos de acceso a etc, 
repite el mismo proceso de búsqueda para el nombre de fichero passwd y tras encontrarlo y verificar que es un fichero y no un directorio, como se esperaba a la vista 
de su ruta, y que tenemos permiso para acceder a él, nos habilita una estructura para poder trabajar con ese fichero.

### Ficheros especiales

Los ficheros especiales, o ficheros de dispositivo, se utilizan para que los procesos se comuniquen con los dispositivos periféricos: discos, cintas, impresoras, 
terminales, redes, etc.

Hay dos familias de ficheros de dispositivo: ficheros modo bloque y ficheros modo carácter.

**Los ficheros de dispositivo modo bloque** se ajustan a un modelo concreto: el dispositivo contiene un array de bloques de tamaño fijo —generalmente múltiplo de 512 bytes
— y el núcleo gestiona una memoria intermedia —buffer caché, también traducido como antememoria— que acelera la velocidad de transferencia de los datos. La 
transferencia de información entre el dispositivo y el núcleo se efectúa con mediación de la antememoria y el bloque es la unidad mínima que se transfiere en cada 
operación de entrada/salida. Esta memoria intermedia se implementa vía software y no hay que confundirla con las memorias caché de acceso rápido de que disponen los 
microprocesadores actuales. Ejemplos típicos de dispositivos modo bloque son los discos y las unidades de cinta.

**En los ficheros de dispositivo modo carácter** la información no se organiza según una estructura concreta y es vista por el núcleo, o por el usuario, como una 
secuencia lineal de bytes. En la transferencia de datos entre el núcleo y el dispositivo no participa la memoria intermedia y por lo tanto se realiza a menor 
velocidad. Ejemplos típicos de dispositivos modo carácter son los terminales serie y las líneas de impresora. Un mismo dispositivo físico puede soportar los dos modos 
de acceso: bloque y carácter, y de hecho esto suele ser habitual en el caso de los discos.

Por otro lado, en el caso de los discos, si queremos realizar un acceso más organizado, debemos recurrir a las facilidades que brinda el sistema de ficheros y la 
estructura de directorios que hay creada sobre él. Esto no nos impide un acceso de más bajo nivel a través de los ficheros de dispositivo asociados al disco. Este tipo 
de acceso pasa por encima de la estructura del sistema de ficheros y trata directamente con el manejador del
disco. Para Leffler [Leffler et al., ], el sistema de ficheros es un apartado más dentro del subsistema de entrada/salida.

Los módulos del núcleo que gestionan la comunicación con los dispositivos se conocen como manejadores de dispositivo. Lo normal es que cada dispositivo tenga su 
manejador propio, aunque puede haber manejadores que controlen a toda una familia de dispositivos con características comunes, por ejemplo, el manejador que controla 
los terminales.

El sistema también puede soportar dispositivos software —o seudodispositivos— que no tienen asociados un dispositivo físico. Por ejemplo, si una parte de la memoria 
del sistema se gestiona como un dispositivo, los procesos que quieran acceder a esa zona de memoria tendrán que usar las mismas llamadas al sistema que hay para el 
manejo de ficheros, pero sobre el fichero de dispositivo /dev/mem —fichero de dispositivo genérico para acceder a memoria—. En esta situación, la memoria es tratada 
como un periférico más.

Como ya hemos visto en epígrafes anteriores, los ficheros de dispositivo, al igual que el resto de los ficheros, tienen asociado un nodo-i. En el caso de los ficheros 
ordinarios o de los directorios, el nodo-i nos indica los bloques donde se encuentran los datos del fichero, pero en el caso de los ficheros de dispositivo no hay 
datos a los que referenciar. En su lugar, el nodo-i contiene dos números conocidos como major number y minor number.

El major number indica el tipo de dispositivo de que se trata —disco, cinta, terminal, etc.— y el minor number indica el número de unidad dentro del dispositivo. En 
realidad, estos números los utiliza el núcleo para buscar dentro de unas tablas —block device switch table y character device switch table— una colección de rutinas 
que permiten manejar el dispositivo. Esta colección de rutinas constituyen realmente el manejador del dispositivo.
Cuando se invoca a una llamada al sistema para realizar una operación de entrada/salida sobre un fichero especial, el núcleo se encarga de llamar al manejador de 
dispositivo adecuado. Lo que ocurre a continuación es algo que compete al diseñador del manejador y es completamente transparente para el usuario.

Naturalmente, nosotros también podemos convertirnos en diseñadores de manejadores y añadir a una arquitectura hardware determinada los dispositivos que necesitemos.

### Tuberías con nombre

Una tubería con nombre es un fichero con una estructura similar a la de un fichero ordinario. La diferencia principal con éstos es que los datos de una tubería son
transitorios.

Esto quiere decir que los datos desaparecen de la tubería a medida que son leídos. Las tuberías se utilizan para comunicar procesos. Lo normal es que un proceso abra 
la tubería para escribir en ella y el otro para leer de ella. Los datos escritos en la tubería se leen en el mismo orden en el que fueron escritos, siguiendo la 
disciplina de una hilera: el primer dato en entrar es el primero en salir2. La sincronización del acceso a la tubería es algo de lo que se encarga el núcleo.
El almacenamiento de los datos en una tubería se realiza de la misma forma que en un fichero ordinario, excepto que el núcleo sólo utiliza las entradas directas de la 
tabla de direcciones de bloque del nodo-i de la tubería. Por lo tanto, una tubería con nombre podrá almacenar 10 Kbytes a lo sumo —suponiendo bloques de 1.024 bytes—. 
Esto no supone un problema grave, ya que los datos de la tubería son transitorios; sin embargo, si la velocidad a la que se introducen datos es mayor que la velocidad 
de extracción, la tubería podría rebosar y perderse información.

Los accesos de lectura/escritura en las tuberías se realizan con las mismas llamadas que empleamos para los ficheros ordinarios —open, close, read, write, etc.— 
Además, estos accesos son de tipo atómico3, con lo que se garantiza el sincronismo en el caso de que varios procesos compitan concurrentemente por el uso de la misma 
tubería. En contraposición a las tuberías con nombre tenemos las tuberías sin nombre que no tienen asociado ningún nombre de fichero y que sólo existen mientras algún 
proceso está unido a ellas.

Las tuberías sin nombre también actúan como un canal de comunicación entre dos procesos y tienen asociado un nodo-i mientras existen. Un ejemplo típico de tubería sin
nombre es la que se crea desde el intérprete de órdenes a través del carácter de tubería |.

```
Por ejemplo,
$ ls | sort -r
```
La línea de órdenes anterior le indica al intérprete que debe arrancar dos procesos: uno para ejecutar ls —listar por pantalla el contenido del directorio actual— y 
otro para sort -r —filtro que lee líneas de texto de la entrada estándar y las presenta por pantalla en orden alfabético inverso—. Además, el intérprete debe crear una 
tubería sin nombre a la que se unirán los procesos creados para ejecutar ls y sort. ls dirigirá su salida hacia la tubería y sort leerá datos de la tubería.

Las tuberías sin nombre son creadas desde un proceso con la llamada pipe, mientras que las tuberías con nombre se crean con la orden mknod desde la línea de órdenes 
del sistema o con la llamada mknod desde un proceso.


## Extensiones del sistema bsd

La arquitectura del sistema de ficheros y los tipos de ficheros tal y como los hemos descrito hasta ahora se ajustan al modelo del unix System V de AT&T y a las 
primeras versiones del unix de Berkeley.

En la organización de los discos de la versión 4.4 de bsd, al igual que en versiones anteriores, cada disco puede contener uno o más sistemas de ficheros. Cada sistema 
está definido mediante su superbloque, localizado al principio de la partición de disco dedicada al sistema. Dada la importancia de los datos contenidos en el 
superbloque, éste está duplicado para evitar pérdidas de información en situaciones límite. Generalmente, la copia del superbloque no es referenciada, a menos que se 
estropee el original.

Para asegurar que los ficheros de tamaño 232 bytes se pueden manipular sin necesidad de recurrir a la entrada indirecta triple del nodo-i, el tamaño mínimo del bloque 
pasa a ser de 4.096 bytes. Como sabemos, el tamaño del bloque se especifica en el superbloque y es posible manejar simultáneamente sistemas de ficheros con diferentes 
tamaños de bloque. No obstante, el tamaño del bloque no puede cambiarse una vez que el sistema ha sido creado, a menos que se decida reinstalarlo de nuevo.

En el sistema bsd se introduce una mejora en la organización del disco basada en el concepto de grupo de cilindros.

### Grupo de cilindros

En la organización de los sistemas de ficheros del bsd, cada disco físico se divide en una o más áreas, conocidas como grupo de cilindros. La figura 2.7 muestra un 
disco dividido en 4 grupos de cilindros, cada uno de los cuales puede agrupar a uno o más cilindros físicos del disco. Cada grupo de cilindros contiene información de 
control, que incluye una copia del superbloque, espacio para los nodos-i, un mapa de bits que refleja la ocupación de bloques dentro del grupo e información 
estadística que describe el uso de los bloques dentro del grupo de cilindros. El número de nodos-i que tiene ese grupo debe ser asignado cuando éste es creado.

El motivo de dividir el disco en grupos de cilindros es crear grupos de nodos-i que estén distribuidos por todo el disco, en lugar de colocarlos todos al principio del 
sistema, como ocurre en otras versiones de unix. Así, los nodos-i estarán en bloques situados físicamente cerca de los bloques de fichero a los que hacen referencia.
Con esto ganamos en velocidad de acceso a los datos del fichero, ya que los desplazamientos de las cabezas lectoras para pasar de los bloques donde están situados los 
nodos-i a los bloques de datos serán más cortos. Además, la seguridad del sistema ante pérdidas accidentales de información se ve incrementada, ya que si se dañan los 
bloques físicos de algunos nodos-i, no se habrán perdido todos los nodos-i.

Toda la información de control sobre el grupo de cilindros puede situarse al principio del mismo; sin embargo, ésta no es la solución más segura, ya que la información 
de control de todos los grupos de cilindros estaría situada sobre el mismo plato del disco. Si se daña ese plato se perderá toda la información de configuración del 
disco y por lo tanto el resto de la información del mismo. Esto justifica que la información de control se almacene en una posición, con respecto al inicio del grupo 
de cilindros, diferente para cada grupo. La posición de cada grupo sucesivo se calcula para que esté alejada al menos un sector con respecto al grupo anterior. Así, la 
información del disco estará distribuida en diferentes platos y sectores, por lo que la pérdida de uno de ellos no va ocasiona el daño de todo el disco.

El espacio que hay entre el inicio del grupo de cilindros y el inicio de la información de control del grupo se aprovecha para almacenar datos. Sólo en el caso del 
primer grupo la información de control está situada al principio del mismo. Como vemos, el diseño de los sistemas de ficheros del bsd responde a mejoras en tiempos de 
acceso y seguridad.

![Captura de pantalla 2023-02-21 a las 17 41 38](https://user-images.githubusercontent.com/4338310/220406615-bafd88bd-c5ec-42a3-a66b-771cbd40c4ce.png)


## Tablas de control de acceso a los ficheros

Además de la tabla de nodos-i, el núcleo mantiene en memoria otras dos tablas que contienen información necesaria para poder manipular un fichero: la tabla de ficheros 
y la tabla de descriptores de fichero.

### Tabla de nodos-i

Como ya hemos visto, la tabla de nodos-i es una copia en memoria de la lista de nodos-i que hay en el disco, a la que se le añade información adicional. Esta tabla se 
copia del disco para conseguir un acceso más rápido a sus componentes.

### Tabla de ficheros

La tabla de ficheros es una estructura global del núcleo y en ella hay una entrada por cada fichero distinto que los procesos del núcleo o los procesos del usuario 
tienen abiertos. Cada vez que un proceso abre o crea un fichero nuevo, se reserva una nueva entrada en la tabla.

![Captura de pantalla 2023-02-21 a las 17 43 36](https://user-images.githubusercontent.com/4338310/220407070-1d743e4b-871d-4300-b17a-811a32fd9ba2.png)

La tabla de ficheros es una estructura de datos orientada a objetos [Leffler et al]. Cada entrada de la tabla contiene un bloque de datos y un array de punteros a 
funciones que traducen las operaciones genéricas que se pueden realizar con los ficheros: leer, escribir, situar el puntero de lectura/escritura, cerrar, posicionar, 
etc., en acciones concretas asociadas a cada tipo de fichero. Las operaciones que se deben implementar por cada tipo son:
- Funciones para leer —read— y escribir —write— en un fichero.
- Una función para realizar la multiplexación síncrona de la E/S —select—.
- Una función para controlar los modos de E/S —ioctl—.
- Una función para cerrar el fichero —close—.

Se puede observar que no hay ninguna rutina de apertura de ficheros en esta definición de objeto. Esto es debido a que el sistema sólo empieza a tratar los elementos 
de esta tabla como objetos a partir de que se haya abierto el fichero.

Cada entrada de la tabla de ficheros tiene también un puntero a una estructura de datos que contiene información del estado actual del fichero. Entre otros, se deben 
tener en cuenta los siguientes campos:
- Nodo-i asociado a la entrada de la tabla.
- Desplazamientos de lectura y escritura que indican sobre qué byte del fichero van a tener efecto las siguientes operaciones de lectura o escritura. Estos desplazamientos quedan actualizados cada vez que se realiza una de las operaciones anteriores.
- Permisos de acceso para el proceso que ha abierto el fichero.
- Indicadores del modo de apertura del fichero, que se verificarán cada vez que se realice una operación sobre el mismo para ver si es congruente. Por ejemplo, si un 
fichero se abre para ser leído solamente, el núcleo no permitirá que se realicen operaciones de escritura sobre él.
- Un contador para indicar cuántas entradas de la tabla de descriptores tiene asociadas esta entrada de la tabla de ficheros.

### Tabla de descriptores de fichero

La tabla de descriptores de fichero es una estructura local a cada proceso. Esta tabla identifica todos los ficheros abiertos por un proceso. Cuando utilizamos las 
llamadas open, creat, dup o link, el núcleo devuelve un descriptor de fichero, que es un índice para poder acceder a las entradas de la tabla anterior. En cada una de 
las entradas de la tabla hay un puntero a una entrada de la tabla de ficheros del sistema.

Los procesos no manipulan directamente ninguna de las tablas anteriores —esto lo hace el núcleo—, sino que acceden a los ficheros a través de un descriptor asociado, 
que es un número. Cuando un proceso utiliza una llamada para realizar una operación sobre un fichero, le pasa al núcleo el descriptor de ese fichero. El núcleo usa 
este número para acceder a la tabla de descriptores de fichero del proceso y buscar en ella cuál es la entrada de la tabla de ficheros que le da acceso a su nodo-i.

Este mecanismo puede parecer artificioso, pero ofrece una gran flexibilidad cuando queremos que un proceso acceda simultáneamente a un mismo fichero en modos 
distintos, o que varios procesos compartan ficheros.

El tamaño de la tabla de descriptores tiene un valor limitado para cada proceso. Este valor depende de la configuración del sistema —consulte limits, constante 
OPEN_MAX—, aunque está bastante extendido que sea 20. Así, un proceso no puede tener abiertos más de OPEN_MAX ficheros distintos en un instante determinado. Si algún 
proceso necesita manejar más de OPEN_MAX ficheros, será obligatorio cerrar algunos —por ejemplo, los menos usados— para poder abrir otros, ya que todos no pueden estar 
abiertos simultáneamente. Esta situación se plantea, por ejemplo, cuando intentamos compilar un proyecto que consta de más de OPEN_MAX ficheros distribuidos entre 
ficheros de cabecera, ficheros fuente, ficheros objeto y bibliotecas. El compilador debe arreglárselas para generar el fichero ejecutable sin imponer límites al número 
de ficheros de nuestro proyecto.

Cuando se arranca un proceso en unix, el sistema le asigna tres ficheros que ocupan las tres primeras entradas de la tabla de descriptores. Estos ficheros se conocen 
como fichero estándar de entrada —por lo general es el teclado de un terminal— que tiene asociado el descriptor número 0, el fichero estándar de salida —la pantalla de 
un terminal— que tiene asociado el descriptor número 1, y el fichero estándar de salida de mensajes de error (también suele estar asociado a la pantalla de un terminal 
que tiene asociado el descriptor número 2).

![Captura de pantalla 2023-02-21 a las 17 48 45](https://user-images.githubusercontent.com/4338310/220408331-d91252a0-10c4-4f31-a02d-af647aa212e2.png)

Estos ficheros están asociados por defecto a todo proceso y pueden cambiarse para que tomen otros valores. Uno de los procedimientos consiste en intercalar en el 
código del programa las llamadas necesarias para modificar la tabla de descriptores. Este procedimiento lo podremos emplear sólo cuando dispongamos del código fuente 
del programa y aun así no es muy recomendable. El otro procedimiento lo facilita el sistema a través  de la redirección y las tuberías. Con los caracteres de 
redirección **<** y **>** podemos hacer que un programa codificado para trabajar con los ficheros estándar pase a trabajar con los ficheros que nos interesen. Por 
ejemplo, la orden:
```
$ ls > directorio
```
hace que la salida estándar de ls —la pantalla— se redirija hacia el fichero directorio, y la orden:
```
$ cc programa.c 2> errores
```
redirige la salida estándar de errores del compilador —descriptor 2 que por defecto tiene asociada la pantalla— hacia el fichero errores.
Como vimos en el párrafo dedicado a las tuberías con nombre, el carácter |, usado en la línea de órdenes, también permite modificar los ficheros estándar asociados a 
un programa. Así, la línea:
```
$ ls | grep root
```
hace que **ls** dirija su salida hacia una tubería sin nombre y que **grep** lea líneas de la misma tubería.


## Administración de los sistemas de ficheros

Comentaré a continuación algunas de las tareas que se deben realizar para crear un sistema de ficheros y asegurar su correcto funcionamiento. Estos trabajos no 
corresponden al usuario del sistema, sino al administrador, pero es útil conocerlos para tener una idea más adecuada del funcionamiento del sistema unix. Sobre todo, 
el usuario neófito podrá obtener respuesta a algunas de las preguntas que se le plantean cuando se enfrenta al sistema por primera vez. Hay que indicar también que las 
órdenes y procesos que se describirán a continuación varían de un sistema a otro, por lo que la última palabra la tiene siempre la documentación técnica del sistema, a 
la que deberá acudir el lector en caso de duda.

### Particiones del disco

La primera idea que se debe manejar es la posibilidad de que en un mismo disco físico coexistan varios sistemas operativos sin que interfieran unos con otros. Esto se 
consigue dedicando una partición del disco a cada uno de los sistemas. Un sistema puede ocupar una o varias particiones, dependiendo de cómo configuremos el disco. 
***Las particiones son, por lo tanto, divisiones del disco independientes unas de las otras y compete al administrador del sistema decidir qué va a contener cada una 
de ellas***.

Las particiones deben crearse antes de hacer ninguna instalación sobre el disco, ya que toda la información que éste contenga dejará de estar accesible. Cada sistema 
dispone de un programa que permite crear la tabla de particiones. En el caso del sistema Dos o de algunas versiones de unix para PC, el programa es fdisk. Este 
programa presenta un menú que permite definir el tamaño dedicado a cada partición, visualizar el total de particiones que tenemos definidas, definir una partición como 
partición activa, etc. 

No es necesario trocear todo el disco para poder trabajar con él. Si sólo necesitamos una parte, podemos crear una partición y posponer la creación de nuevas 
particiones para cuando tengamos necesidad de ellas. Igualmente, podemos dedicar la totalidad del disco a una sola partición.

Si en el disco hay instalados varios sistemas operativos, el usuario puede preguntarse cuál es el sistema que se carga al arrancar el ordenador. Para resolver este 
problema, fdisk permite definir una partición activa; es decir, una partición en la que se busca el sistema operativo en el momento de arranque. Naturalmente, para que 
una partición sea autoarrancable, debe tener un sector o bloque de boot y el archivo o archivos del sistema. Esto lo especificamos al dar formato a la partición.

### Formato del disco

Una vez establecidas las particiones del disco, podemos pasar a instalar sistemas de ficheros sobre ellas. __Dependiendo del sistema operativo__ que vaya a albergar 
cada partición, será necesario darle un tipo de formato u otro. 

Cada sistema tiene su propio programa para formatear; así, **dos** dispone del programa format que se utiliza para dar formato a discos duros y disquetes. En unix no 
hay un programa estándar para formatear; cada fabricante suministra un conjunto de herramientas de administración del sistema y los aspectos de formateo pueden estar 
incluidos dentro de ellas. Así, por ejemplo, **sun** ofrece a sus usuarios el programa diag, que  incluye la opción format para dar formato a una partición. **sco** da 
formato a las particiones en el momento de instalar el sistema. 

Otra de las verificaciones que debemos realizar antes de empezar a instalar sistemas de ficheros sobre el disco es revisar el estado de sus sectores y generar una 
tabla de sectores defectuosos. El programa que realiza esta operación también depende de la versión del sistema que estamos manejando. Algunos ejemplos son: bad144 y badsect para el unix de Berkeley, badblk para AT&T, badtrk para sco, etc.

### Ficheros de dispositivo del disco

Con objeto de construir un sistema de ficheros sobre alguna de las particiones de disco, es necesario que en el directorio /dev esté presente el fichero de dispositivo 
que nos dé acceso a esa partición. Si estamos haciendo la primera instalación de unix, no es necesario llevar a cabo esta tarea, puesto que el programa de instalación 
se encarga de crear los ficheros de dispositivo necesarios. Sin embargo, en ampliaciones de una instalación ya realizada, hay que tener presentes estos aspectos.
La forma de crear los ficheros de dispositivo es mediante la orden mknod. Su sintaxis es la siguiente:
´´´
$ /etc/mknod nombre c|b major minor
´´´
**nombre** es el nombre del fichero de dispositivo que se va a crear; **c|b** indica el tipo de acceso: modo bloque o modo carácter, sólo uno debe estar presente; 
**major** es el major number del dispositivo y **minor** su minor number.

Para usar mknod debemos conocer los major y minor number de los dispositivos que puede manejar nuestro sistema. Esta numeración tampoco es algo estándar, por lo que
debemos consultar la documentación técnica del fabricante. De forma estándar, en la sección 7 del manual se documentan los ficheros de dispositivo que maneja el 
sistema, así como el significado de sus números asociados.

De cara a **los nombres de los dispositivos** se suelen utilizar algunos convenios. Así, los ficheros de dispositivo de los discos duros se llaman hd##, donde ##  
representa dos números que indican la unidad de disco y la partición a la que se refieren. Por ejemplo, hd00 se refiere a la totalidad del primer disco físico, hd01 a 
la primera partición del primer disco físico, hd02 a la segunda partición, etc. Para referirnos a la totalidad del segundo disco, emplearemos el fichero de dispositivo 
hd10, la primera partición del segundo disco será hd11, y así sucesivamente.

Podemos ver los ficheros de dispositivo destinados a disco mediante la orden:
´´´
$ ls -al /dev/hd*
brw------- 2 sysinfo sysinfo 1, 0 Mar 14 1989 /dev/hd00
brw------- 2 sysinfo sysinfo 1, 15 Mar 14 1989 /dev/hd01
brw------- 2 sysinfo sysinfo 1, 23 Mar 14 1989 /dev/hd02
brw------- 2 sysinfo sysinfo 1, 31 Mar 14 1989 /dev/hd03
brw------- 2 sysinfo sysinfo 1, 39 Mar 14 1989 /dev/hd04
brw------- 2 sysinfo sysinfo 1, 47 Mar 14 1989 /dev/hd0a
brw-r----- 2 dos 
sysinfo 1, 55 Mar 14 1989 /dev/hd0d
´´´
En las columnas 5 y 6 de la salida que produce la orden ls podemos ver cuáles son los major y minor number de los distintos ficheros de dispositivo. En el ejemplo 
anterior, todos los ficheros de disco tienen el major number 1 y los minor number 0, 15, 23, 31, 39,
47 y 55, respectivamente.

También podemos ver que estos ficheros corresponden a dispositivos modo bloque, el carácter b de la primera columna así lo indica. Como vimos en apartados anteriores, 
hay dispositivos que pueden ser referenciados a través de ficheros de dispositivo modo bloque o modo carácter. En concreto, los discos son de ese tipo de dispositivos, 
por lo que existe toda una familia de ficheros paralela a la anterior. Los nombres de estos ficheros responden al esquema rhd##, donde ## representa dos números con el 
significado ya explicado. Para visualizar estos dispositivos podemos escribir:
´´´
$ ls -al /dev/rh*
crw------- 2 sysinfo sysinfo 1, 0 Mar 14 1989 /dev/rhd00
crw------- 2 sysinfo sysinfo 1, 15 Mar 14 1989 /dev/rhd01
crw------- 2 sysinfo sysinfo 1, 23 Mar 14 1989 /dev/rhd02
crw------- 2 sysinfo sysinfo 1, 31 Mar 14 1989 /dev/rhd03
crw------- 2 sysinfo sysinfo 1, 39 Mar 14 1989 /dev/rhd04
crw------- 2 sysinfo sysinfo 1, 47 Mar 14 1989 /dev/rhd0a
crw-r----- 2 dos 
sysinfo 1, 55 Mar 14 1989 /dev/rhd0d
´´´
Se puede apreciar que los números asociados a estos ficheros son los mismos que los del modo bloque. La única diferencia es que el acceso a través de estos nuevos 
ficheros se va a realizar carácter a carácter, sin la intervención de la memoria intermedia, por lo que responderán de una forma más lenta.

