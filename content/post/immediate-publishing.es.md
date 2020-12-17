---
title: "Publicación inmediata"
date: 2021-02-22
draft: false
slug: immediate-publishing
tags: ["Visual Studio", "MSBuild"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ImmediatePublishing)

Érase una vez una solución, alojada en un proyecto de equipo en Team Foundation Services (TFS) que fue descargada por primera vez. Sabía que la solución compilaba y podía publicarse sin problemas de inmediato. Pero me llevé una sorpresa porque ambos procesos fallaron. La figura 1 muestra una estructura simplificada del proyecto.

![Estructura del proyecto](/immediate-publishing/01.png)

Figura 1: "Estructura del proyecto".

La figura 2 muestra cómo se configuró el proyecto para generar su documentación:

![Generación de documentación del proyecto](/immediate-publishing/02.png)

Figura 2: "Generación de documentación del proyecto".

Una vez que intentamos construirlo obtenemos el error que se muestra en la figura 3.

![Resultado de la compilación del proyecto](/immediate-publishing/03.png)

Figura 3: "Resultado de la compilación del proyecto".

El error se debe a que el archivo de documentación: "RS.Blog.Projects.ImmediatePublishing.Web.xml" es de solo lectura y no se puede sobrescribir. Este problema se puede resolver si pasamos por el sistema de archivos y eliminamos el atributo: "Solo lectura" de ese archivo. Esa es la solución a corto plazo porque si usted u otro miembro del equipo descargan la solución por primera vez, el atributo volverá a impedir que la solución compilación. La solución a largo plazo es excluir ese archivo del proyecto como se muestra en la figura 4.

![Exclusión del archivo de documentación del proyecto](/immediate-publishing/04.png)

Figura 4: "Exclusión del archivo de documentación del proyecto".

Este paso agrega un par de líneas al archivo del proyecto (.csproj) como muestra la figura 5:

![Contenido del archivo del proyecto](/immediate-publishing/05.png)

Figura 5: "Contenido del archivo del proyecto".

Ahora, la solución compila pero una vez que intentamos publicarla; no funciona. La figura 6 muestra el error.

![Error de pubicación](/immediate-publishing/06.png)

Figura 6: "Error de pubicación".

En este caso, el error también está relacionado con el atributo: "Solo lectura" pero del archivo: "web.config". Por lo general, publicamos aplicaciones en configuración: "Release" y las transformaciones de configuración para "web.config" no se pueden ejecutar. Una vez más, eliminando el atributo del archivo: "web.config" (figura 7) resolverá el problema y finalmente podremos publicar la aplicación de inmediato (figura 8).

![Quitando el atributo 'Solo lectura'](/immediate-publishing/07.png)

Figura 7: "Quitando el atributo 'Solo lectura'".

![Resultados de publicación](/immediate-publishing/08.png)

Figura 8: "Resultados de publicación".

### Resumen ###

La publicación de una aplicación inmediatamente después de descargarla por primera vez desde TFS requiere eliminar el atributo: "Solo lectura" del archivo: "web.config". Si el proyecto también genera documentación, es mejor excluirlo del proyecto en lugar de eliminar el mismo atributo del archivo de documentación.

[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ImmediatePublishing)
