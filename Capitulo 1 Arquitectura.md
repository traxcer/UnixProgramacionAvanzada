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


