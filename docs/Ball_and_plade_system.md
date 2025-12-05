#Proyecto Final

###Nombre del proyecto: Ball and Plate system

###Autores
- Barrientos Miguel Leonardo
- Zermeño Cervantes Rodrigo

###Asignatura: Introducción a la Mecatronica

###Fecha: 5 de Diciembre del 2025

###Descripción
En este proyecto final desarrollamos un Ball and Plate System, un mecanismo mecatrónico capaz de mantener una pelota equilibrada sobre una plataforma mediante la variación de ángulos en una placa superior.

El sistema utiliza cuatro servomotores, los cuales modifican la inclinación de la placa en los ejes X y Y. A través del control coordinado de estos servomotores, la plataforma es capaz de moverse en diferentes direcciones para dirigir o estabilizar la pelota en una posición deseada.

Este mecanismo es una base importante para sistemas de control avanzados, como plataformas estabilizadoras, robots equilibristas o sistemas de visión con retroalimentación.
El proyecto se encuentra documentado y versionado en GitHub, donde se registran los avances, diagramas y códigos implementados.

                         
##Objetivos


###General
Diseñar y programar un sistema mecatrónico de control de inclinación capaz de mover una plataforma en dos ejes mediante servomotores para lograr el equilibrio dinámico de una pelota.

###Especificos

- Implementar la comunicación y control de 4 servomotores utilizando un microcontrolador (Arduino/ESP32)..  
- Diseñar la estructura mecánica para mover la placa superior mediante un sistema articulado. 
- Desarrollar el código para mover la plataforma en diferentes direcciones variando los ángulos de los servos.
- Comprender la relación entre los movimientos de los servomotores y la inclinación resultante de la placa.
- Documentar el proceso mecánico, electrónico y de programación del sistema.

##Alcance y Exclusiones
 -**Incluye**:

- Montaje mecánico con una placa móvil y cuatro servos.
- Código para controlar la inclinación de la placa en ambos ejes. 
- Pruebas manuales de inclinación mediante valores programados.
- Calibración básica del rango de movimiento de los servos.


 -**No Incluye**: 

##Procedimiento

###Diseño Mecánico Del Sistema

Se construyó una plataforma con dos niveles:

1- Placa inferior fija, donde se colocan los servomotores.

2- Placa superior móvil, unida mediante rótulas o uniones articuladas.


Cada uno de los cuatro servomotores está conectado a un punto distinto de la placa móvil. Al variar el ángulo de los servos, se altera la altura relativa de cada esquina de la plataforma, generando inclinación en dos ejes.

Tambien se incluye una webcam mediante la cual recibimos la imagen del de la parte superior de la plataforma donde es colocada la pelota. El objetivo de esto es que mediante la retroaliemtación visual recibida por parte de la webcam , el sistema pueda compensar perturbaciones y mantener la pelota en la plataforma sin que se caiga de esta.

###Programación del código

Por otro lado y probablemente la parte más importante para que funionara el mecanismo, es el codigo, el cual consistio en 2 partes, el codigo realizado en python mediante Visual Studio.

```cpp

```

Y el codigo bluetooth para el arduino y que este pudiese recibir señales de la webcam.


```cpp

```

