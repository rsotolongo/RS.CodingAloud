---
title: "La delgada línea entre reescribir y reutilizar código"
date: 2020-12-18
draft: false
slug: reescribir-ó-reusar
tags: ["arquitectura"]
categories: ["Memorias"]
---
Hay veces que no nos damos cuenta de la existencia de delgadas líneas que existen entre cosas. Por otra parte, no sé por qué tenemos esa desconfianza en lo que otro hace. La reticencia al cambio es algo siempre presente en toda persona y es un mal muy difícil de arraigar, hay que ser un experto para lograr que se entiendan los cambios.

Serán por estas razones por las que nosotros los profesionales de la computación cada vez que tenemos que desarrollar una aplicación pensamos seriamente codificarla desde cero, incluso a veces lo hacemos. De nada vale que nos hayamos leído cientos de artículos que resalten la importancia de hacer código reusable para utilizarlo posteriormente. En el mejor de los casos utilizamos componentes desarrollados por nosotros mismos en otro momento, pero rechazamos todo en lo que no hayamos puesto nuestras manos.

Yo se lo achaco a problemas de cultura, si interiorizáramos estas cuestiones a profundidad y pensamos solamente un poquito que efectivamente esto es cierto, no tendríamos necesidad alguna de gastar horas y horas "reinventando la rueda" o "descubriendo el agua tibia". Es cierto que al utilizar código (dígase componentes) tenemos que estudiar su referencia, que la mayoría de las veces no están a la altura de la documentación de una API o Framework profesional, pero les aseguro que es mucho menor el tiempo que se puede emplear en esto que el que se puede emplear para implementarlo desde cero.

Por lo general los desarrolladores de componentes hacen sus trabajos de forma impecable, funcionamiento correcto, comportamiento más que probado y referencia completa. Somos nosotros los supuestos clientes los que no le otorgamos la confianza suficiente como para utilizarlos.

En el caso particular de la carrera de Ingeniería Informática su currículo está muy pensado sobre todo a desarrollar desde cero (otro factor que influye en lo antes expuesto); por suerte los encargados de rediseñar su Plan de Estudios se dieron cuenta y entonces el nuevo Plan de Estudios "D" se enfocará más al reuso que a la reescritura de código. Otorgarle la confianza a lo que se lo merece en aras de ganar tiempo para otras cuestiones fundamentales del proceso de desarrollo de software. Las mentes tienen que cambiar, de lo contrario, jamás podremos hacer algo que esté al nivel de los estándares mundiales de hoy en día.

Para que se tenga un ejemplo de esto, alguien se ha preguntado el ¿por qué en Windows existe el Worpad si Microsoft Word es mejor por millas y millas de funcionalidad? Sencillamente por problemas de compatibilidad (al igual que hay dos copias del Notepad.exe) y alguien se ha preguntado ¿Por qué su tamaño es despreciable con respecto al Word, por qué tardó tanto en ser actualizado? Pues bien, en su momento era otra compañía su desarrolladora y decidieron hacerlo en Ensamblador a pesar de ya existir la API de Win32, cierto fue que permitía un buen número de funcionalidades de una manera muy eficaz (rapidez y consumo de memoria) pero su mantenimiento se hizo insoportable al punto de que tuvo que ser vendido a Microsoft.

Con esto no quiero decir que las cosas no se hagan eficientes, sino que hay que llegar a un balance entre lo eficiencia y la eficacia del proyecto, y me despido porque me estoy yendo del tema.
