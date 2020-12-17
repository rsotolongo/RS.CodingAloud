---
title: "Algunas consideraciones de Implementación"
date: 2020-12-29
draft: false
slug: consideraciones-de-implementación
tags: ["arquitectura"]
categories: ["Memorias"]
---
Creemos que es necesario comenzar este artículo sugiriendo la introducción de tres técnicas que, si bien no son nuevas, sí constituyen buenas prácticas dentro del desarrollo organizado de software.

El acoplamiento de módulos, la compilación, la producción de ejecutables, la generación de reportes y la creación y restauración de copias de seguridad en la producción de sistemas informáticos constituyen tareas perfectamente automatizables. Las ventajas que puede traer el uso de esta técnica están estrechamente relacionadas con el tiempo total de desarrollo de los sistemas informáticos. Cuanto menor sea el tiempo que se dedique a este tipo de actividades mayor será el que se pueda invertir en hacer cumplir los requisitos funcionales del sistema. Dentro de los métodos más utilizados en esta técnica están la creación de archivos de ejecución por lotes combinados con los eventos calendariables que brindan los sistemas operativos modernos. FinalBuilder es una de las herramientas que brinda por igual estas facilidades de una forma visual y amena.

Un CVS (Sistema de Control de Versiones, siglas en inglés) implantado desde el inicio del proceso de desarrollo de software permite que se tenga un rastro exacto del estado del proyecto en cualquier momento del mismo. Permitiendo además que se los programadores puedan trabajar simultáneamente en un mismo fichero de forma segura. CVS es la herramienta más utilizada por los programadores de Linux, mientras que WinCVS, Borland TeamWork y Microsoft Visual Source Safe son las más utilizadas en Windows.

La tercera técnica que debe tenerse en cuenta es la explotación de una Base de Datos de Errores. En ella se irán reflejando los errores que vayan surgiendo, quién de los miembros lo descubrió, causas que lo originaron, así como quién es el encargado de corregirlo y cómo lo hizo. Cada error debe permanecer abierto (no corregido) hasta una vez no se haya probado su solución. Esta técnica constituye para los desarrolladores una fuente extra de información y experiencia.

De más está decir que el hecho de implementar un producto informático en un lenguaje y/o ambiente u otro está estrechamente relacionado con las características propias del producto a desarrollar. Por ello es que nos referiremos a tres grandes lenguajes de programación con sus respectivos ambientes de trabajo más comunes que, si bien no son los únicos, sí podemos afirmar que son los más extendidos en nuestro país y en el mundo.

##### Object Pascal / Delphi #####

Object Pascal, lógica evolución de Pascal y evangelizado por Borland, será nuestro primer lenguaje. Cuando en 1995 Borland sacó al mercado Delphi 1.0 la comunidad de profesionales de la computación enseguida vio un futuro muy próspero para este ambiente que reemplazaba a los antiguos Turbo con la introducción de un nuevo lenguaje: Object Pascal. Desde su primera versión soportó las características fundamentales del paradigma Orientado a Objetos como son la herencia y el polimorfismo, en versiones posteriores se le incorporarían los soportes para interfaces y meta clases.

Borland Delphi -nombre oficial de este ambiente de desarrollo- ya va por ocho versiones y en los próximos años verán la luz dos más. Considerada por muchos la mejor herramienta RAD (Desarrollo Rápido de Aplicaciones) no las ha tenido todas consigo en las herramientas CASE (Ingeniería de Software Asistida por Computadora, siglas en inglés) pues no todas generan código Object Pascal de forma nativa, lográndose en la mayoría de los casos solo a través de la instalación de "pluggings" o extensiones.

No es difícil de demostrar el hecho de que para Delphi es la herramienta que más componentes se han implementado. Tal vez esto se deba a que son varias las formas de extensión que permite, BPL como propio, pero permite además DLL (Bibliotecas de Enlace Dinámico), COM (Modelo de Objetos de Componentes), DCOM (COM Distribuido, siglas en inglés en todos los casos) y la gran facilidad con que maneja cada una de ellas.

Los ficheros de configuración INI, el registro de Windows o los modernos XML (Lenguaje Extendido de Marcas, siglas en inglés) son soportados de forma nativa por la biblioteca de clases VCL (Biblioteca de Componentes Visuales, siglas en inglés) que es parte de Delphi.

Un paso muy importante en la portabilidad de este lenguaje hacia otras plataformas lo dio precisamente Borland cuando en 1995 saca al mercado Kylix. Herramienta RAD que vino a cubrir un espacio vacío en la comunidad de desarrolladores de Linux. Se puede llegar hasta un 90% del total del proyecto sin tener que hacer ninguna modificación. Lamentamos que este potencial se esté perdiendo y Kylix haya sido relegado por Borland a planos secundarios.

Los costos dependen mucho de la versión que se desee comprar y esta del para qué se quiera la herramienta. Por lo general de una misma versión hay varias ediciones, una para el usuario medio, una para el desarrollador y una para un nivel corporativo (esto es, por supuesto, lo más clásico) que en ese mismo orden van soportando y facilitando el desarrollo de diferentes tipos de aplicaciones en dependencia de su complejidad y composición.

##### C++ / Visual C++ #####

C++ ó C plus plus es una extensión del C original para soportar el paradigma Orientado a Objetos. Ambientes de desarrollo RAD para este lenguaje son muy variados desde la misma Borland hasta Microsoft. La primera con su serie C++ Builder no ha logrado consolidarse tal como lo hizo con Delphi, mientras que la segunda ha progresado enormemente con su Visual C++ al incorporarlo a la herramienta de desarrollo más completa que existe hasta estos momentos: Visual Studio.

C++ soporta todas las características de la orientación a objetos excepto las meta clases. La herencia se hace múltiple en vez de simple, las interfaces se logran solo mediante algunos "trucos" de sintaxis, lo que le quita elegancia al lenguaje, no obstante, sigue siendo elegido para aplicaciones dentro de este paradigma por muchos programadores.

Desde las primeras implementaciones de las herramientas CASE, estas fueron capaces de generar código para este lenguaje ya que su comunidad lideraba a las demás. Generalización es un tópico que está presente en este lenguaje y el hecho de poder hacer clases genéricas (sin que medie el tipo hasta la pre compilación). Los macros son otra extensión del lenguaje, aunque en estos días tenga más detractores que seguidores (algo similar a la sentencia goto).

La VCL es la biblioteca de clases que incorpora C++ Builder y la MFC (Fundación de Clases de Microsoft, siglas en inglés) el Visual C++. Esta última cuenta con más de 300 clases para su más variado uso. La gran ventaja del C++ Builder es el hecho de estar concebido sobre la VCL lo que lo hace fácil la translación de los códigos a Delphi y por otra parte Borland trabaja en un producto integrador del lenguaje C++ completo, llamado C++ BuilderX que no solo hará posible portar códigos a Linux, sino que a cualquier o desde cualquier otra plataforma y/o compilador de este lenguaje.

¿Dónde está entonces la portabilidad de Visual C++? En que este ambiente está girando alrededor de la generación de código intermedio administrado y con el mismo es posible ejecutarlo en cualquier máquina virtual. La estrategia .NET de Microsoft ya ha tenido resultados en Linux con la implementación de Mono, el traspaso de código hasta el momento solo es posible en C# (C sharp ó C almohadilla) que constituye a su vez una combinación mejora de los tres lenguajes que hemos tomado como casos de estudio.

Estos dos ambientes analizados centran su atención en el paradigma Orientado a Objetos soportando el paradigma Estructurado gracias a la compatibilidad que comparten con su lenguaje de programación base. La mayoría de los lenguajes propios del paradigma Funcional brindan sus potencialidades a través de su encapsulamiento en DLL (Bibliotecas de Enlace Dinámico, siglas en inglés) o cualquier tipo de módulo propio del ambiente. Este es el caso también del tercer lenguaje Java.

Visual Studio, si bien coincidimos en que es la herramienta de desarrollo más integradora que existe, sus costos son en la actualidad lucrativos para la mayoría de entidades productoras de software; independientemente del hecho de que Microsoft esté llevando a cabo algunas ofensivas comercializadoras muy competitivas.

##### Java / Eclipse #####

Descendiente del Oak y con el propósito de que se pudiera insertar en cualquier equipo electrodoméstico estudiaremos el tercer lenguaje: Java. Original de Sun Microsystems es Java el lenguaje más extendido sobre las plataformas de teléfonos celulares. Es muy rico en su sintaxis contando con un número considerable de palabras reservadas y reglas gramaticales. Soporta todas las características del paradigma Orientado a Objetos.

Fruto de su amplia extensión en cuanta plataforma de hardware existe enseguida las herramientas CASE fueron capaces de generar código para él. Los paquetes de componentes constituyen su base fundamental de componentización nativa. Las demás formas de reutilización de componentes no es algo que lo lleve implícito este lenguaje.

Como podemos inferir la ejecución del código que se produce de la compilación de una aplicación programada en Java (que no es más que un código intermedio llamado "ByteCode") debe realizarse bajo la máquina virtual propia para esa plataforma. Aprovechando esto Microsoft introdujo cambios en la máquina virtual que adjuntaba con su Internet Explorer haciendo que existieran ciertos problemas de compatibilidad, por suerte esto ha cambiado y ya casi no existen. Es el mismo caso para los diferentes Sistemas Operativos por lo que para cada una existen diferentes implementaciones y extensiones.

Preferimos hablar del lenguaje en sí y no del ambiente de desarrollo puesto que en este tópico la competencia es muy reñida entre un producto u otro. Ejemplo de ello tenemos a JBuilderX de Borland; Visual J# (extensión de Java soportando código administrado e incorporado como parte del Microsoft Visual Studio); VisualAge de IBM y Eclipse de Eclipse Foundation que es el más extendido.

Los costos son elevados en caso de los ambientes de desarrollo pues algunas máquinas virtuales y la biblioteca de clases JFC (Fundación de Clases de Java, siglas en inglés) son gratuitas e incluso incluyen su código fuente.

Todos los lenguajes se pueden utilizar para el desarrollo de cualquier tipo de aplicaciones desde las de consola y las que llevan bases de datos hasta las que hacen uso de la multitarea o son concebidas de una forma distribuida (sin dejar de tener en cuenta las de redes y las "standalone"). Dependiendo solo de las restricciones propias de los ambientes desde los que son utilizados. Poco a poco se van creando los artefactos necesarios para que cada uno soporte la última tendencia o tecnología moderna en cuanto a la creación de productos informáticos. Algunos son más rápidos que otros en responder dependiendo del estado del ambiente, su comunidad de seguidores y compañías que lo desarrolla.

Quisiéramos que este material le sirva como introducción a la hora de tomar decisiones críticas en el proceso de implementación de una determinada solución de software. Nos hemos basado en la experiencia y los consejos de muchos profesionales que han dedicado miles de horas al incansable arte de la programación.
