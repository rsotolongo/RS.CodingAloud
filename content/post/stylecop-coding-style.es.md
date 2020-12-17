---
title: "Estilizando códigos usando StyleCop"
date: 2021-01-04
draft: false
slug: estilo-de-código-usando-stylecop
tags: ["ReSharper", "Visual Studio", "StyleCop", "código fuente", "análisis de código"]
categories: ["Desarrollo"]
---
Un colega me pidió que compartiera mi experiencia programando con ReSharper y, una vez que se enteró de que no uso ReSharper, sintió aún más curiosidad por saber cómo escribo los códigos fuentes. Quien me conoce sabe que me encantan los paquetes NuGet y estar actualizado constantemente, por eso encontré un paquete muy útil que instalo en todos mis proyectos apenas los inicio.

StyleCop.MSBuild

StyleCop solía ser una aplicación independiente con integración con Visual Studio, pero eso cambió hace un tiempo. Lo explicaré con un ejemplo. Comencemos a crear una aplicación de consola desde cero.

![Applicación de consola](/stylecop-coding-style/01.png)

La figura 1 muestra una aplicación de consola recién creada en Visual Studio.

Observe los 0 errores, 0 advertencias y 0 mensajes una vez que se compila. Instalemos el paquete NuGet y veamos qué sucede. La figura 2 muestra la administración de paquetes NuGet para la aplicación.

![Administrador de paquetes de la solución](/stylecop-coding-style/02.png)

Figura 2.

La figura 3 muestra los resultados de la creación de la solución una vez que se instala ese paquete.

![Primeros resultados de compilación](/stylecop-coding-style/03.png)

Figura 3.

Note las 11 advertencias. Mi filosofía personal es entregar productos sin errores (eso es lógico porque los compiladores y administradores lo buscan); cero advertencias (pocas herramientas y personas le impiden implementar con advertencias); y cero mensajes (prácticamente a nadie le importa esto).

La mayoría de las advertencias recibidas se explican por sí mismas y son fáciles de solucionar. Analicémoslsas una por una.

La primera y el última están relacionadas con el hecho de que los archivos "Program.cs" y "AssemblyInfo.cs" no tienen encabezado y se pueden arreglar simplemente agregando un encabezado en formato XML que contenga el nombre del archivo entre otra información útil.

Las siguientes cinco advertencias están relacionadas con el hecho de que las declaraciones de "uso" deben estar dentro del espacio de nombres y no afuera, como dice la plantilla. La solución es obvia, simplemente mover las declaraciones dentro del espacio de nombres.

Las siguientes cuatro advertencias están relacionadas con la falta de documentación en los diferentes instrumentos de código: clases y métodos en este caso. Una vez que se agrega la documentación, las advertencias desaparecen.

La figura 4 muestra el resultado de la compilación después de aplicar las correcciones anteriores.

![Segundos resultados de compilación](/stylecop-coding-style/04.png)

Figura 4.

Observe las dos nuevas advertencias. En este punto es bueno mencionar que este es un procedimiento iterativo hasta la meta (0 errores, 0 advertencias, 0 mensajes). También se eliminaron las declaraciones de "uso" no utilizadas (las que están en gris). La figura 5 muestra el nuevo resultado una vez que arreglamos estas dos advertencias agregando el modificador "público" a la clase y al método.

![Primeros arreglos](/stylecop-coding-style/05.png)

Figura 5.

¡Hurra! ¡Lo hicimos! Pero (siempre hay un pero, ¿no?), Esto es posible gracias al conjunto de reglas que hemos configurado en el proyecto para reportar advertencias. Entonces, seamos más exigentes al cambiar el conjunto de reglas.

La figura 6 muestra las propiedades del proyecto en la opción "Análisis de código". Observe la casilla de verificación no seleccionada: "Habilitar análisis de código al compilar" y las "Reglas recomendadas administradas por Microsoft" en el menú desplegable Conjunto de reglas.

![Opciones de configuración del proyecto](/stylecop-coding-style/06.png)

Figura 6.

Si habilita la casilla de verificación sin instalar el paquete NuGet, no recibirá ninguna advertencia y si también cambia el conjunto de reglas sin instalar el paquete NuGet, recibirá solo 2 advertencias también devueltas por el paquete StyleCop porque lo que hace es agregar muchas más reglas al conjunto de trabajo. Esa es la razón por la que me encanta agregarlo a mis proyectos y no solo usar lo que está incluido en Visual Studio.

La figura 7 muestra las opciones recomendadas.

![Opciones recomendadas](/stylecop-coding-style/07.png)

Figura 7.

Sí, "Microsoft All Rules", más estrictos somos, mejor código obtenemos si seguimos los criterios para obtener al final 0 errores, 0 advertencias y 0 mensajes.

La figura 8 muestra los resultados de la compilación después de las nuevas opciones para el análisis de código.

![Terceros resultados de compilación](/stylecop-coding-style/08.png)

Figura 8.

Seis nuevas advertencias, es hora de empezar de nuevo. Uhmmm... Los dos primeros no tienen un archivo asociado, ¿cómo puedo solucionarlos? Bueno, hay una solución. Haga clic derecho en el código "CA2210" en este caso, seleccione la opción de "Suprimir" y luego la opción "En Archivo de Supresión" como se muestra en La figura 9.

![Opciones de reglas](/stylecop-coding-style/09.png)

Figura 9.

Una vez seleccionada esta opción, se crea un nuevo archivo llamado "GlobalSuppressions.cs" en la raíz del proyecto como se muestra en La figura 10.

![Archivo de supresión de reglas](/stylecop-coding-style/10.png)

Figura 10.

En este punto, me gustaría mencionar que hay ocasiones en las que puede suprimir la advertencia en el archivo de supresión y otras directamente en el código fuente. En La figura 10 todavía hay 5 advertencias más para corregir. Mis opciones son suprimir en el archivo de supresión global la primera advertencia y suprimir en el código fuente la cuarta advertencia. Para suprimir el código fuente, debe hacer casi lo mismo: haga clic con el botón derecho en el código de la advertencia, seleccione la opción "Supresión" y luego seleccione la opción "En fuente".

Esta opción agrega un atributo sobre el método que se informa que contiene el código para suprimir la advertencia, similar al código agregado en el archivo "GlobalSuppressions.cs".

La segunda advertencia se puede arreglar marcando el método "Principal" como "estático"; mientras que el tercero se puede arreglar eliminando el parámetro "args" (y su documentación en los comentarios del método). Finalmente, la quinta advertencia se puede arreglar simplemente corrigiendo la ortografía de la palabra. Sí, StyleCop también incluye corrección ortográfica, ¿no es genial?

Una vez que se abordan todas las advertencias restantes, podemos volver a compilar y ver lo que se informa esta vez, La figura 11 muestra eso.

![Cuartos resultados de compilación](/stylecop-coding-style/11.png)

Figura 11.

¿Qué? ¿Más advertencias? Sí, recuerde que agregó un nuevo archivo: "GlobalSuppressions.cs" y ese archivo ingresa al ciclo para reportar advertencias de compilación también. Entonces, es hora de corregir las nuevas advertencias informadas.

Todas las supresiones a través del código (código fuente del archivo "GlobalSuppressions.cs" o atributos en los instrumentos de código) deben tener una justificación. Utilizo mis iniciales como justificación como una forma de rastrear quién decidió suprimir esta advertencia. Así es como soluciono la primera y tercera advertencias.

La segunda advertencia se puede corregir agregando el encabezado al archivo "GlobalSuppressions.cs".

La cuarta advertencia solicita el uso de "////" en lugar de "//" para iniciar un comentario de código fuente. Si no le gusta esta opción, puede suprimir globalmente en el archivo "GlobalSuppressions.cs", pero prefiero corregirlo cambiando la forma en que escribo los comentarios.

Finalmente, La figura 12 muestra cómo se ve el código al final.

![Código final](/stylecop-coding-style/12.png)

Figura 12.

Primero que se cumpla todo nuestro objetivo: 0 errores, 0 advertencias y 0 mensajes. En segundo lugar, nuestro código resultante es atractivo y se explica por sí mismo para futuras modificaciones, así como menos problemas reportados por otras herramientas de análisis de seguridad como HP Fortify.

Es válido mencionar que debido a que esta disciplina requiere agregar comentarios en todos los instrumentos escribir todos esos comentarios manualmente es una tarea tediosa. Por esa razón también utilizo otra herramienta llamada "GhostDoc" que genera documentación muy precisa dependiendo del contexto del instrumento que se está documentando. El problema con esto es que debe hacerse uno por uno debido a la limitación de la licencia. En realidad, yo uso la versión gratuita, pero la licencia profesional permite generar comentarios para toda una clase, archivo o espacio de nombres con un solo clic o presionando "Shift + Ctrl + D" sobre el instrumento. La buena noticia es que recientemente "GhostDoc" distribuyó una versión usando el paquete de extensión de Visual Studio y no la aplicación independiente como estaba, por lo que la instalación y las actualizaciones se vuelven más fáciles.

Una vez más esta es mi experiencia creando códigos fuentes. Sé que seguir esta disciplina no tiene gracia si el producto no es nuevo pero como dice Robert C. Martin: "… no importa el estado de un proyecto, pero cualquier cosa que cambies trata de hacer tu mejor esfuerzo…" para mí eso significa no generar nuevas advertencias. Al final, piense siempre que el desarrollador que le toque el código en el futuro podría ser un asesino en serie y conoce su dirección (también cita de Robert C. Martin), así que es mejor dejar el código lo más limpio posible.

Todos esto comenzó con una pregunta de ReSharper y terminará con un comentario relacionado con ReSharper. A veces, las advertencias informadas por StyleCop y las informadas por ReSharper entran en conflicto entre ellas. Yo suelo dar prioridad a los reportados por StyleCop porque creo que son más estrictos y compatibles con más herramientas. Además, no hay dinero asociado porque es gratis mientras que ReSharper no lo es.
