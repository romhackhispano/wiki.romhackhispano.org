---
layout: post
title:  "Problemas de alineación"
category: cpu
---

## ¿En qué consiste?

En ocasiones, al hacer romhacking y traducciones, se modifican punteros
o textos que luego van a parar a la memoria. En algunos casos hay datos
que tienen que tener una cierta alineación en memoria. Si no están alineados
correctamente, pueden bloquear el hardware real (la consola) aunque funcionen
correctamente en un emulador.

En este artículo se trata el problema de la alineación en la PSX y más
concretamente en el Tales of Destiny II/Eternia, que es el caso concreto
en el que hemos trabajado.

## ¿Qué operaciones y modificaciones provocan problemas de alineación?

Inicialmente, el problema de alineación ocurre al modificar
**punteros a datos de 2 y 4 bytes** (short *) y (int *) a punteros datos
de 16 y 32 bits conocidos en los procesadores MIPS como "Half" y "Word"
respectivamente. En general, tienen que ver con cualquier operación
relacionada con las instrucciones LH y LW (Load Half y Load Word).

## ¿En qué consiste que una dirección esté alineada?

Una dirección está **alineada a 2 bytes** cuando dicha dirección es:

* múltiplo de 2
* dirección % 2 == 0
* la dirección es par
* bit menos significativo == 0
* dirección & 1 == 0

*Las 5 definiciones son equivalentes.*

Una dirección está **alineada a 4 bytes** cuando dicha dirección es:
* múltiplo de 4
* dirección % 4 == 0
* dos bits menos significativos == 0
* dirección & 3 == 0

*Las 4 definiciones son equivalentes*

## Copia rápida de datos

Al copiar datos de forma rápida se usan punteros de 32 bits y la
instrucción LW, en cuyo caso deben estar alineadas las direcciones de
memoria usadas a 32 bits y tener una longitud múltiplo de 4.

En el Tales of Eternia los datos de las habitaciones están empaquetados
por habitaciones y en cada habitación hay diversos archivos que tienen
una alineación de 32 bits.

## Lectura de valores de 2 y 4 bytes

Los datos de tipo short e int deben estar alineados a 2 y 4 bytes.
Los compiladores suelen alinear todos los datos a 4 bytes en el
ejecutable para que al cargar en memoria sigan alineados y se puedan
acceder igualmente.

Este hecho tiene implicaciones muy importantes que hay que tener en cuenta:

- Cuando busca punteros, tarda bastante en buscar números en las direcciones
múltiples de 4. - Las "strings" o cadenas terminadas en \0 están alineadas a
4 bytes dejando bytes \0 libres para la alineación. Suponiendo que las cadenas
se lean mediante LB (Load Byte) como suele ser normal, se podrá disponer de
espacio adicional.

## Centinelas

En ciertas circunstancias se usan centinelas de 16 y 32 bits para determinar
cuándo termina una secuencia.

En el Tales of Eternia los textos con scripts tinene inserciones de variables
o códigos especiales, como por ejemplo escribir un texto mas rápido o mas
lento, escribir el nombre de un personaje, la cantidad de dinero, etc.

Estos códigos especiales empiezan por un código de control <= 0x20 y tienen
una longitud par. Dicho código puede tener una longitud variable y se utiliza
un centinela (una secuencia fija de bytes para conocer el fin de una sucesión).
El centinela, pese a ser una sucesión de 32 bits, se comprueba cada 16 bits
usando LH un (short *) en C. Para solucionar este problema, en el juego antes
de un código de control se usa el caracter 0x1f (caracter dummy) para alinear
el texto si la dirección de memoria es impar, es decir, no está alineado a 2 bytes.

## ¿Existe algún emulador de PSX que compruebe la alineación?

Hay una modificación del Emulador PCSX con Debugger integrado que permite
comprobar los dos tipos de alineaciones en PSX para saber lo que fallaría
en el hardware real.
