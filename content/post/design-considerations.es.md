---
title: "Algunas consideraciones de Diseño"
date: 2020-12-28
draft: false
slug: consideraciones-de-diseño
tags: ["arquitectura"]
categories: ["Memorias"]
---
Dentro de las etapas en que se divide el ciclo de vida del desarrollo de software, el Diseño es considerada como la más importante. Se ha calculado que puede llegar a ocupar hasta un treinta y cinco por ciento del total de tiempo. Es por ello que abordaremos el tema para dar algunas consideraciones generales que siempre son válidas a tener en cuenta, sea cual sea el tipo de aplicación a desarrollar.

Existen tres paradigmas principales de programación: el Estructurado, el Orientado a Objetos y el Funcional. Cada uno tiene sus características propias que están muy relacionadas con la evolución en el tiempo de la programación. Hoy en día el paradigma Estructurado se ve muy ligado a la implementación de algoritmos, el Funcional a la resolución de problemas en la Inteligencia Artificial y el paradigma Orientado a Objetos -el más extendido- a la implementación de aplicaciones.

Es precisamente en este último paradigma donde los profesionales de la computación han hecho la mayor cantidad de aportes ya sean teóricos o prácticos como pueden ser las herramientas que ayudan en su uso. Las herramientas CASE (Ingeniería de Software Asistida por Computadoras, siglas en inglés) son, sin duda alguna, unas de las más representativas. Mediante el modelado visual de la jerarquía de clases a través de UML (Lenguaje Unificado de Modelado, siglas en inglés) puede generarse código altamente confiable y fácilmente modificable, para infinidad de lenguajes de programación; dándole así una elegancia agregada al diseño.

Otro tipo de herramientas son las RAD (Desarrollo Rápido de Aplicaciones, siglas en inglés). En ellas se puede trabajar en el diseño y la implementación a la vez de una forma hasta intuitiva; lográndose con ello disminuir considerablemente el tiempo total de desarrollo del proyecto. El hecho de poder mezclar de una forma muy fácil las reglas del problema dentro de los eventos de los campos visuales de la aplicación (técnica tan difundida en aplicaciones pequeñas y medianas) trae consigo que se afecte directamente la reutilización del código.

Para contrarrestar este problema le proponemos, que una vez haya decidido el paradigma que va a utilizar y el uso o no de herramientas CASE y/o RAD, piense también en separar la lógica de la aplicación de la vista de la misma. La técnica más utilizada para esto es la componentización de la solución ya sea mediante COM (Modelo de Objetos en Componentes, siglas en inglés), Scriptlets, Servicios Web o cualquier otra vía propia de la herramienta de implementación (ejemplo: BPL de Borland Delphi), en dependencia por supuesto de la portabilidad que le quiera dar a la solución.

Nunca descarte el posible uso de la aplicación a ser desarrollada desde la consola de trabajo (stdin, stdout en Linux) sobre todo si la misma realiza tareas programadas. Para ello diseñe adecuadamente los parámetros necesarios que se le deberán pasar.

La creación de nuevos procesos (en Linux) o hilos de aplicación (en Windows) es muy común en las aplicaciones que quieran hacer uso de la multitarea. Si su aplicación es una de ellas tenga en cuenta el uso de las llamadas "piscinas de hilos" que le incorporan un mayor rendimiento y escalabilidad a la solución. Otro aspecto a tener en cuenta en este tema es la cantidad máxima de procesos o hilos que se pueden crear de forma tal que no se afecte el rendimiento final, piense siempre que no debe exceder la decena de unidades.

Para las aplicaciones de mediano o gran tamaño es muy común hacer uso de las facilidades de la Programación Distribuida ya sea mediante la arquitectura Cliente/Servidor o la arquitectura N-capas o SOA (Arquitectura Orientada a Servicios, siglas en inglés). Llamada a Procedimientos Remotos e Invocación de Métodos Remotos (RPC y RMI, siglas en inglés respectivamente) son dos técnicas para implementar el modelo Cliente/Servidor. Mucho es el uso que se hizo y hace en la N-capas, tanto es, que constituye la base fundamental de su evolución: SOA. Esta última se basa en la distribución de Servicios Web como unidades estructurales implementados cada uno como un modelo N-capas, siendo considerada la de más futuro en los venideros años.

Hemos tratado de mantenernos lo más alejado posible de toda clase de decisiones, intentando siempre no mencionar productos o marcas para que no busquen en nosotros una preferencia marcada hacia uno u otra parte de las actuales controversias (IBM-Mac, Windows-Linux, .NET-Java, etc.). Lo que sí quisiéramos es que no utilice tecnologías propietarias que no sean estándares y pueda así evitarse futuros dolores de cabeza sobre todo a la hora de diseñar los mecanismos de comunicación. Nos referimos a DCOM (COM Distribuido), MTS (Servidor de Transacciones de Microsoft, siglas en inglés), COM+ (DCOM + MTS) y CORBA por solo citar algunos ejemplos.

Dentro de este punto quisiéramos tratar el formato de representación de los datos, si bien en el pasado se utilizó mucho los ficheros textos no estructurados y los ficheros estructurados; en la actualidad no son más que historia y añoranzas. XML (Lenguaje Extendido de Marcas, siglas en inglés) está llamado a ser el nuevo ASCII del intercambio de información; dándole así una nueva dimensión hasta ahora dominada por los protocolos binarios.

Los ficheros de configuración (.INI) constituyeron todo un hito como combinación de los dos tipos de ficheros fundamentales (textos y estructurados) y por consiguiente la unión de sus respectivas ventajas. Esto fue así hasta la salida al mercado de Windows 95 donde se introdujo una buena nueva idea, la de centralizar la configuración de las computadoras, en el llamado "registro de configuraciones". Lo malo fue su implementación lo que a la postre se convirtió en una pesadilla para los programadores. Es aquí donde intervine otra vez la mano salvadora de XML que poco a poco se está adueñando de esta funcionalidad también.

El diseño de marcos de trabajo (frameworks) es una buena idea si se quiere hacer más de un trabajo relacionado que compartan jerarquías de clases de forma tal que le sirva como base. Ejemplos tenemos sobrados: VCL, MFC, Java, .NET Framework, BizTalk Framework, etc. 

La aplicación de un determinado patrón de diseño acorde con los requisitos propios de la solución a desarrollar es una de las mejores opciones que tiene el diseñador de software en estos momentos. Un tema relativamente nuevo en esta etapa no deja de fascinarnos con sus buena prácticas recomendadas amén de las dificultades que pueda traer.

Quisiéramos que este material le sirva como introducción a la hora de diseñar una determinada solución de software. En un próximo artículo comentaremos algunas consideraciones de implementación con semejante objetivo.
