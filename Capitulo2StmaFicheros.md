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



