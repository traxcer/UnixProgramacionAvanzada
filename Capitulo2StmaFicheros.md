# Capitulo 2 Arquitectura del Sistema de Ficheros

## Características del sistema de ficheros

Un sistema de ficheros es un mecanismo de abstracción de los dispositivos físicos de almacenamiento que nos permite manejarlos a un nivel lógico sin la necesidad de conocer 
su arquitectura hardware particular. El sistema de ficheros de unix se caracteriza por:

- Poseer una estructura jerárquica.
- Realizar un tratamiento consistente de los datos de los ficheros.
- Poder crear y borrar ficheros.
- Permitir un crecimiento dinámico de los ficheros.
- Proteger los datos de los ficheros.
- Tratar los dispositivos y periféricos —terminales, unidades de cinta, discos, etc... como si fuesen ficheros.

El sistema de ficheros está organizado, a nivel lógico, en forma de árbol invertido, con un nodo principal conocido como nodo raíz *—/—*. 
Cada nodo dentro del árbol es un directorio y puede contener a su vez otros nodos *—subdirectorios—*, ficheros normales o ficheros de dispositivos.

Los nombres de los ficheros se especifican mediante la ruta *path name*, que describe cómo localizar un fichero dentro de la jerarquía del sistema. 
La ruta de un fichero puede ser absoluta *—referida al nodo raíz—* o relativa *—referido al directorio de trabajo actual-* **CWD** current work directory—.
Los programas que se ejecutan en unix no conocen el formato interno con el que el núcleo almacena los datos. Cuando accedemos al contenido de un fichero 
mediante una llamada al sistema —read—, el sistema nos lo va a presentar como una secuencia de bytes sin formato. Nuestro programa es el encargado de 
interpretar la secuencia de bytes y darle significado según sus necesidades. Por lo tanto, la sintaxis *—forma—* del acceso a los datos de un fichero 
viene impuesta por el sistema y es la misma para todos los programas, y la semántica  *—significado—* de los datos es responsabilidad del programa que
trabaja con el fichero.

