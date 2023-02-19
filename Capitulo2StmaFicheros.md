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



