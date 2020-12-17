---
title: "Creando paquetes NuGet"
date: 2021-01-18
draft: false
slug: creando-paquetes-nuget
tags: ["Visual Studio", "NuGet", "MSBuild"]
categories: ["Desarrollo"]
---
Últimamente he estado involucrado en la creación de varios paquetes NuGet útiles para algunos portales que se están siendo desarrollando. Quiero compartir mis conocimientos en este tema.

Si estuviéramos desarrollando bibliotecas para .NET Core o .NET Standard, la vida sería fácil, como muestra La figura 1 a La figura 8.

![Creación de una nueva biblioteca de .NET Core](/creating-nuget-packages/01.png)

Figura 1: "Creación de una nueva biblioteca de .NET Core".

![Código predeterminado](/creating-nuget-packages/02.png)

Figura 2: “Código predeterminado”.

![Resultados de la primera compilación](/creating-nuget-packages/03.png)

Figura 3: "Resultados de la primera compilación".

![Propiedades del proyecto](/creating-nuget-packages/04.png)

Figura 4: “Propiedades del proyecto”.

![Salida de la segunda compilación](/creating-nuget-packages/05.png)

Figura 5: "Salida de la segunda compilación".

![Resultados de la segunda compilación](/creating-nuget-packages/06.png)

Figura 6: "Resultados de la segunda compilación".

Pero la vida no es tan fácil y casi todos nuestros productos utilizan .NET Framework (normal), lo que significa que Visual Studio no nos ofrece las capacidades que se muestran en las figuras 7 y 8.

![Nueva biblioteca de .NET Framework](/creating-nuget-packages/07.png)

Figura 7: “Nueva biblioteca de .NET Framework”.

![Propiedades del proyecto biblioteca](/creating-nuget-packages/08.png)

Figura 8: "Las propiedades para este tipo de proyectos no incluyen la función Paquete".

Entonces, ¿qué alternativa tenemos? En realidad, algunas de ellas: usan Octopus para generar el paquete, pero esto es complicado debido a que Octopus usa una compilación personalizada de NuGet.exe, por lo que la estructura generada puede no coincidir y su obtención implica muchos pasos.

Otra alternativa es usar "NuGet.exe" manualmente desde la línea de comandos, pero esto puede ser confuso y también implica muchos otros pasos para realizar.

Dije una vez pero me gusta repetirlo, me encantan los paquetes NuGet, entonces, ¿por qué no usar un paquete NuGet para generar paquetes NuGet? Bueno, ya existe y es muy fácil de usar. El primer paso es agregar el paquete: “CreateNewNuGetPackageFromProjectAfterEachBuild” al proyecto que queremos distribuir como un paquete NuGet. La figura 9 muestra un ejemplo.

![Administrador de paquetes de la solución](/creating-nuget-packages/09.png)

Figura 9.

Una vez agregado, este paquete crea un subdirectorio dentro de la raíz del proyecto llamado: "_CreateNewNuGetPackage" y eso es todo lo que necesita. La figura 10 muestra la estructura creada.

![Explorardor de la solución](/creating-nuget-packages/10.png)

Figura 10.

Una vez que compilamos el proyecto, podemos ver el resultado en La figura 11.

![Salida de la tercera compilación](/creating-nuget-packages/11.png)

Figura 11.

La figura 12 muestra los resultados de la compilación.

![Resultados de la tercera compilación](/creating-nuget-packages/12.png)

Figura 12.

La figura 13 muestra el contenido del paquete generado.

![Contenidos del paquete](/creating-nuget-packages/13.png)

Figura 13.

Tenga en cuenta que los metadatos se extraen básicamente de la información del ensamblado o se generan automáticamente (la salida de la compilación también tiene registros sobre esto). Podemos cambiar esto si incluimos un archivo de metadatos como muestra La figura 14.

![Archivo de metadatos del paquete](/creating-nuget-packages/14.png)

Figura 14.

El contenido de este archivo depende de lo que desee para su paquete, pero como ejemplo:

``` XML
<?xml version="1.0" encoding="utf-8"?>
<!-- 
  For more info on this file format visit http://docs.nuget.org/docs/reference/nuspec-reference.
-->
<package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
  <metadata>
    <id>Portals.ClassLibrary1</id>
    <version>1.0.0</version>
    <authors>Portals Development</authors>
    <description>Class library to help portals development.</description>
    <language>en-US</language>
    <releaseNotes></releaseNotes>
  </metadata>
</package>
```

Ahora los resultados de la compilación se parecen a los de La figura 15.

![Primeros paquetes generados](/creating-nuget-packages/15.png)

Figura 15.

La figura 16 muestra el contenido del nuevo paquete generado, esta vez con metadatos más precisos.

![Contenido del primer paquete](/creating-nuget-packages/16.png)

Figura 16.

En este punto pudimos generar el paquete pero lo que realmente necesitamos es la posibilidad de consumirlo desde otro proyecto. Primero necesitamos implementarlo en un repositorio interno de NuGet llamado: "McNuget". Expliquemos cómo hacerlo.

Primero abra “$(ProjectDir)\_CreateNewNuGetPackage\Config1.ps1” y realice las modificaciones resaltadas en color amarillo en La figura 17.

![Archivo de configuración para la generación del paquete](/creating-nuget-packages/17.png)

Figura 17.

Básicamente, estamos especificando que en cada compilación se genera otro paquete que contiene todos los símbolos de depuración, incluidos los códigos fuentes, para facilitar la depuración de este paquete. No agregue la configuración y la plataforma utilizadas para compilar en el nombre del paquete. Además, eventualmente implementaremos este paquete en McNuget (digo eventualmente porque no es automático en todas las compilaciones) y la API para implementar en McNuget. Esta API fue generada por el administrador de McNuget y nos la envió.

La figura 18 muestra el resultado de la compilación.

![Segundos paquetes generados](/creating-nuget-packages/18.png)

Figura 18.

Observe los dos nuevos paquetes sin la configuración y la plataforma; uno sin símbolos de depuración (igual que en La figura 16) y otro con símbolos y código fuente como muestra La figura 19.

![Contenido del segundo paquete](/creating-nuget-packages/19.png)

Figura 19.

Es hora de implementar el paquete en McNuget y para hacer esto necesitamos ejecutar “$(ProjectDir)\_CreateNewNuGetPackage\RunMeToUploadNuGetPackage.cmd” como se muestra en La figura 20.

![Comando para subir paquete](/creating-nuget-packages/20.png)

Figura 20.

Una vez ejecutado, se abrirá una ventana de línea de comandos y un cuadro de diálogo solicitando que el paquete se despliegue, como se muestra en La figura 21.

![Diálogo para subir paquete](/creating-nuget-packages/21.png)

Figura 21.

Una vez que se selecciona el paquete como se muestra en La figura 22...

![Selección para subir paquete](/creating-nuget-packages/22.png)

Figura 22.

...pide una confirmación como muestra La figura 23. Un punto válido para mencionar es que el repositorio de McNuget puede no estar configurado para permitir implementaciones de paquetes que contienen símbolos de código fuente y depuración. Entonces, la única opción que podemos despleguar es el paquete regular.

![Confirmación de subida de paquete](/creating-nuget-packages/23.png)

Figura 23.

Una vez aceptado y si todo ha ido bien, deberíamos poder ver una pantalla similar a la que muestra La figura 24.

![Resultado de subida de paquete](/creating-nuget-packages/24.png)

Figura 24.

Lo que significa que nuestro paquete está en McNuget listo para usarse. Probémoslo.

La figura 25 muestra la gestión de paquetes para otro proyecto.

![Administrador de paquetes de proyecto](/creating-nuget-packages/25.png)

Figura 25.

Una vez que el paquete está instalado, podemos usar sus características. La figura 26 muestra un ejemplo.

![Ejemplo de uso de paquete](/creating-nuget-packages/26.png)

Figura 26.

Una última observación, hemos implementado el paquete regular porque hipotéticamente McNuget no permite implementar paquetes con símbolos de depuración; pero hemos desplegado la versión construida en modo "Debug" y esto no es bueno, debemos desplegar la versión construida en modo "Release" y ganar más rendimiento.

Esto es todo lo que tengo que compartir con ustedes hoy. Espero que esta guía le resulte útil para sus proyectos una vez que desee generar paquetes NuGet.
