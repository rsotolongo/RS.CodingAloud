---
title: "Validación en el cliente o en el servidor: ¿Llegó .NET? ¡Se acabaron las preocupaciones!"
date: 2020-12-02
draft: false
slug: validación-de-datos
tags: ["seguridad"]
categories: ["Memorias"]
---
Desde los tiempos del inicio del protocolo HTTP (¿Quién iba a imaginar que, a solo 12 años, alguien se refiriera a eso con tal antigüedad?) y el ansiado carácter dinámico que poco a poco se fue haciendo más y más necesario en el mismo; la validación de los datos provenientes del cliente se hizo una necesidad mayor cada vez más.

¿Cuántas veces introducimos código HTML en los campos que se nos pedían en un formulario? ¿Nunca probaron la respuesta del sistema ante la introducción de una consulta a la base de datos en un campo de un formulario? Como por ejemplo: ";DROP ALL DATABASE" o algo similar en dependencia del SQL que se maneje -el punto y coma ";" lo que le dice al sistema es que la consulta se que termina ahí y en adelante hay otra y entonces cuando se trataba de insertar el valor en la base de datos lo que pasaba era que se truncaba el INSERT y se ejecutaba el DROP y por consiguiente sus devastadoras consecuencias (OJO: Todavía hay sistemas vulnerables a este tipo de código malicioso, nunca digan que fue aquí donde lo aprendieron).

Para evitar cosas como estas se realizan (incluso hoy en la actualidad) validaciones en el servidor donde se validaban los datos. Esto por supuesto trae aparejado una ralentización del sistema producto del tráfico en la red; cosa que puede dar al traste con la buena explotación del sistema debido a sus bajas prestaciones en dependencia de la configuración de la red.

En este caso también caían las restricciones propias de los sistemas en cuanto a obligar a sus usuarios a introducir determinados campos de los formularios y al olvido de los clientes de esta restricción.

Para todo esto se comenzó a hacer las validaciones en el cliente, es decir en el navegador web mediante la característica que poseen casi todos de brindar una interface de su documento mediante la programación en un lenguaje de secuencia de comandos (generalmente JavaScript). Esto no fue obstáculo para lo usuarios más avanzados pues comenzaron a salvar las páginas en sus computadoras para modificar los códigos fuentes eliminando dichas validaciones. Una vez introducidos los datos eran enviados al servidor donde eran procesados y de nuevo caían los sistemas.

Si bien es verdad que la filosofía de validaciones no ha cambiado con la introducción de modernas tecnologías de programación como lo constituye la plataforma .NET de Microsoft; es verdad también que el código a escribir se ha minimizado a un extremo que prácticamente es cero la cantidad de líneas que hay que escribir, sobre todo si se hace uso de las facilidades de herramientas RAD como el Visual Studio .NET.

Basta con arrastrar y soltar en nuestro formulario algunos controles y configurar, si acaso, un par de propiedades de los mimos para lograr que el sistema valide los datos tanto en el cliente como en el servidor en caso de que sea necesario. "RequiredFieldValidator" como su nombre bien lo indica hace que el control que se convierta de obligatoriedad la introducción de datos en el control que tiene asociado.

"RangeFieldValidator" es otro control de este tipo. Por si los que vienen con la biblioteca de clases de la plataforma (.NET Framework) no fueran suficientes la misma posee una clase para que se pueda heredar de ella e implementar nosotros mismos nuestros propios validadores respondiendo a las restricciones propias del sistema a implementar.

Evalúe las posibilidades que brindan los validadores de .NET a la seguridad de aplicaciones web y utilícelos cuando los necesite para que vea en la práctica lo que decimos.
