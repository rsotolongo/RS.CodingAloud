---
title: "Firmando ensamblados"
date: 2021-02-08
draft: false
slug: signing-assemblies
tags: ["Visual Studio", "MSBuild", "seguridad"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/SigningAssemblies)

## .NET Core ##

No todos los clientes requieren firmar los ensamblados de sus aplicaciones, pero cuando es necesario, la tarea no es tan fácil como cabría esperar.

En el caso de las aplicaciones .NET Core, podemos lograr esto simplemente editando el archivo del proyecto (.csproj) agregando el siguiente código:

``` XML
<Target Name="BuildSigning" AfterTargets="AfterBuild">
  <Exec Command="SIGNING_COMMAND" />
</Target>
```

Como ejemplo, **SIGNING_COMMAND** podría ser:

*"C\Program Files (x86)\Windows Kits\10\bin\10.0.177630\x64\signtool.exe"* *sign /f "$(SolutionDir)certificate.pfx" /p "pass&1234" "S(TargetDir)*.dll"*

Estamos utilizando el certificado generado en este post anterior: "[Generando certificados de firma](../generating-signature-certificates)".

Esto hace que después de cada compilación (independientemente de la configuración -Debug, Release, etc.-) se firmen los ensamblados al final del proceso. La firma continúa funcionando después de las compilaciones, pero no después de la publicación. El resultado del script de publicación anterior que publica una aplicación .NET Core es "curioso":

Suponiendo que la publicación se basa en la configuración de "Release", los ensamblados en la carpeta "Release" se firman antes de copiarse en la carpeta temporal "Publish". Luego, los artefactos en la carpeta temporal "Publish" se copian en el destino final "Publish". Finalmente, los ensamblados en la carpeta "Release" se vuelven a firmar nuevamente.

Los ensamblados en la carpeta "Release" se firman dos veces porque para la funcionalidad de publicación, la macro $(TargetDir) sigue teniendo el valor para la carpeta "Release" y no la carpeta "Publish". Una solución probada cambió la macro $(TargetDir) por la macro $(PublishDir) pero no funcionó porque esa macro toma el valor de la carpeta temporal "Publish" y no de la carpeta final "Publish". El resultado es que los ensamblados de la carpeta "Release" no se firman dos veces, pero los ensamblados de la carpeta "Publish" tampoco están firmados. En cambio, los ensamblados en la carpeta temporal "Publish" se firman después de haber sido copiados en la carpeta final "Publish".

Con estos antecedentes, la mejor solución es cambiar el script de post compilación para que sea un script posterior a la publicación:

``` XML
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <Exec Command="SIGNING_COMMAND" />
</Target>
```

Cambiando **SIGNING_COMMAND** a:

*"C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64\signtool.exe" sign /f "$(SolutionDir)certificate.pfx" /p "pass&1234" "$(PublishUrl)*.dll"*

Observe la adición de *"publish\"* después de la macro $(TargetDir).

Esta modificación funcionará pero, por supuesto, solo para la publicación y no para todas las compilaciones. Nada nos detiene para agregar ambos scripts de publicación (compilar y/o publicar) para que los ensamblados se firmen en todos los casos, el único problema con esto es que los ensamblados en la carpeta "Release" se firmarán dos veces. Eso no es gran cosa, pero es algo a tener en cuenta.

Otro tema es el tiempo de inactividad durante la firma de ensamblados. Dependiendo de la aplicación que se esté firmando, este proceso puede llevar una cantidad considerable de tiempo que es el doble en caso de doble firma o el triple en caso de utilizar los dos scripts de publicación (Compilar y Publicar). Debido a que "Publicar" no se usa con tanta frecuencia como "Compilar", el tiempo de inactividad no es un gran problema, excepto si incluimos una marca de tiempo en la firma.

Agregar el parámetro */t timeseverURL* to **SIGNING_COMMAND** podría saltar (como ejemplo) de 15 segundos a 2030 segundos (4.5 minutos) en el caso de un servidor de tiempo externo como <http://timestamp.comodoca.com/authenticode> . Si usamos un servidor de tiempo interno, este tiempo podría disminuir significativamente, pero su uso dependerá de si la política de seguridad de ejecución toma o no la marca de tiempo en consideración.

Finalmente, analicemos una muestra para el script de post publicación:

``` XML
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <Message Importance="high" Text="Firmando ensamblados publicados..." />
  <Exec Command="&quot;C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64\signtool.exe&quot; sign /f &quot;$(SolutionDir)certificate.pfx&quot; /p &quot;pass&amp;1234&quot; &quot;$(PublishUrl)*.dll&quot; &quot;$(PublishUrl)*.exe&quot;" />
  <Message Importance="high" Text="Ensamblados publicados firmados..." />
</Target>
```

Incluir mensajes de inicio y finalización en la consola de compilación es una buena práctica. Todas las comillas dobles (") deben codificarse en su equivalente HTML (`&quot;`). El símbolo de conjunción (&) que coincidentemente está presente en la contraseña del certificado, también debe codificarse (`&amp;`). No solo se firman los ensamblados de biblioteca (DLL), los ejecutables (EXE) también se firman.

Finalmente, algo a lo que hay que prestar atención es la versión del SDK de Windows (10.0.17763.0) porque eso depende de lo que esté instalado en la computadora de desarrollo pero podría ser diferente para cada miembro del equipo. Con todo esto en mente, se podría adoptar otro enfoque que aportara claridad y generalidad:

``` XML
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <Exec Command="$(SolutionDir)SignAssemblies.cmd $(SolutionDir) $(PublishUrl)" />
</Target>
```

Donde el archivo: "SignAssemblies.cmd" podría ser:

``` CMD
ECHO Firmando ensamblados...

"C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64\signtool.exe" sign /f "%certificate.pfx" /p "pass&1234" "%2*.dll" "%2*.exe"

ECHO Ensamblados firmados...
```

Esto todavía tiene el problema de la versión del SDK codificada de forma rígida, pero podría generalizarse como:

``` CMD
ECHO Firmando ensamblados...

SET basedir=C:\Program Files (x86)\Windows Kits\10\bin\
SET newestSdkDir=10.0.17763.0
FOR /F "tokens=*" %%a in ('dir /b /on "%baseDir%10*"') DO SET newestSdkDir=%%a
REM "%baseDir%%newestSdkDir%\x64\signtool.exe" sign /?
"%baseDir%%newestSdkDir%\x64\signtool.exe" sign /f "%1certificate.pfx" /p "pass&1234" "%2*.dll" "%2*.exe"

ECHO Ensamblados firmados...
```

La versión del SDK se inicializa como la versión 10.0.17763.0 (porque es la última que he instalado en mi computadora en el momento de escribir esta publicación, pero podría ser cualquier otra versión) y luego itera sobre todas las carpetas que coinciden con el patrón "10*" en orden para obtener la última versión. Hay una instrucción útil que no hace nada, pero si sustituimos **REM** por **ECHO** podemos ver la información del tiempo de ejecución en las trazas de compilación. Observe cómo eliminamos los caracteres codificados porque los trasladamos al archivo de comando (.cmd) y no es necesario usarlos allí.

Por último, creemos un enfoque aún más unificado utilizando más tareas de MSBuild. Necesitamos sustituir el destino de MSBuild "PublishSigning" anterior por el siguiente código:

``` XML
<Import Project="..\SignAssemblies.targets" Condition="Exists('..\SignAssemblies.targets')" />
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(PublishUrl);AssembliesPattern=*.dll" />
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(PublishUrl);AssembliesPattern=*.exe" />
</Target>
```

Básicamente, se incluye otro archivo externo de MSBuild y se llama a su tarea principal pasando los parámetros "AssembliesPath" y "AssembliesPattern". Esto se debe a que no todas las soluciones generan bibliotecas y ensamblados ejecutables al mismo tiempo. Ahora, el nuevo archivo "SignAssemblies.targets" se ve así:

``` XML
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="SignAssemblies">
    <Message Importance="high" Text="Firmando ensamblados..." />
    <Message Importance="high" Text="Camino de los ensamblados: $(AssembliesPath)" />
    <Message Importance="high" Text="Patrón de los ensamblados: $(AssembliesPattern)" />
    <Exec Command="$(SolutionDir)SignAssemblies.cmd $(SolutionDir) $(AssembliesPath)$(AssembliesPattern)" />
    <Message Importance="high" Text="Ensamblados firmados..." />
  </Target>
</Project>
```

El archivo "SignAssemblies.cmd" permanece casi idéntico con solo una pequeña modificación (el patrón de archivos a firmar se pasa desde la llamada a la tarea):

``` CMD
ECHO Firmando ensamblados...

SET basedir=C:\Program Files (x86)\Windows Kits\10\bin\
SET newestSdkDir=10.0.17763.0
FOR /F "tokens=*" %%a in ('dir /b /on "%baseDir%10*"') DO SET newestSdkDir=%%a
REM "%baseDir%%newestSdkDir%\x64\signtool.exe" sign /?
"%baseDir%%newestSdkDir%\x64\signtool.exe" sign /f "%1certificate.pfx" /p "pass&1234" "%2"

ECHO Ensamblados firmados...
```

En caso de que queramos firmar además todos los ensamblados después de la compilación, podemos agregar las siguientes líneas al archivo del proyecto (.csproj) antes o después del objetivo "PublishSigning" anterior:

``` XML
<Target Name="BuildSigning" AfterTargets="AfterBuild">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(TargetDir);AssembliesPattern=*.dll" />
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(TargetDir);AssembliesPattern=*.exe" />
</Target>
```

## .NET Framework ##

Las cosas no cambian drásticamente desde el enfoque de .NET Core porque intentamos ser lo más genéricos posible desde el principio. Sin embargo, hay algunos detalles a tener en cuenta.

En la aplicación .NET Framework podemos editar los eventos de compilación en las propiedades del proyecto como se muestra en la figura 1.

![Project build events](/signing-assemblies/01.png)

Figura 1: Eventos de compilación del proyecto.

Esto modifica el archivo del proyecto (.csproj) agregando las siguientes líneas...

``` XML
<PropertyGroup>
  <PostBuildEvent>
    ECHO Test message...
  </PostBuildEvent>
</PropertyGroup>
```

...donde, por supuesto, podemos reemplazar el comando ECHO por **SIGNING_COMMAND** de la sección anterior. El resultado es muy similar, los ensamblados se firman en la carpeta de configuración (Debug. Release, etc.) pero la función de publicación sigue teniendo problemas. Si revisamos la carpeta publicada, podemos verificar que los ensamblados producidos por la solución se firmen mientras que los ensamblados dependientes de terceros (como los paquetes NuGet) no están firmados por nuestro certificado. Permanecen firmados por su firmante original o no están firmados en absoluto. Esto no es lo que necesitamos, necesitamos todos los ensamblados firmados por nuestro certificado preferiblemente en el momento de la publicación para evitar la firma manual antes de desplegar.

Intenté reutilizar el enfoque de la sección anterior para .NET Core, pero no fue posible sin modificaciones. Terminé de modificar manualmente el archivo del proyecto (.csproj) para agregar las siguientes líneas:

``` XML
<Import Project="..\SignAssemblies.targets" Condition="Exists('..\SignAssemblies.targets')" />
<Target Name="PublishSigning" AfterTargets="GatherAllFilesToPublish">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(ProjectDir)$(WPPAllFilesinSingleFolder)\bin\;AssembliesPattern=*.dll" />
</Target>
```

Aquí estamos especificando que queremos ejecutar target: "PublishSigning" cuando todo esté listo para ser copiado en la carpeta final "Publish" (tarea: "GatherAllFilesToPublish"). Muy similar a lo que hicimos en .NET Core pero cambiando el valor del parámetro "AssembliesPath". Con este enfoque, todos los ensamblados se firman justo antes de que todo esté listo para copiarse en la carpeta "Publish" final; de la carpeta temporal "Publish" a la carpeta final "Publish". No es necesario firmar archivos ejecutables porque las aplicaciones web de .NET Framework no generan archivos .EXE.

En caso de que queramos firmar todos los ensamblados también después de la compilación, podemos agregar las siguientes líneas al archivo del proyecto (.csproj):

``` XML
<Target Name="BuildSigning" AfterTargets="AfterBuild">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(TargetDir);AssembliesPattern=*.dll" />
</Target>
```

Traté de incluir en el archivo "SignAssemblies.targets" el objetivo de llamada "PublishSigning" para simplificar las modificaciones manuales en el archivo del proyecto (.csproj) y reutilizarlo en diferentes soluciones. Desafortunadamente, eso no es una buena idea porque los destinos de llamada dependen de parámetros que son específicos para los proyectos en desarrollo (Atributo: "Properties"). Esto parece más consistente y estandarizado.

El proceso de firma de ensamblados ideal podría verse así:

1. Copiar y agregar el archivo de certificado: "certificate.pfx" a la solución.
2. Copiar y agregar el archivo por lotes: "SignAssemblies.cmd" a la solución.
3. Copiar y agregar el archivo de MSBuild "SignAssemblies.targets" a la solución.
4. Modificar el archivo del proyecto (.csproj) para incluir el archivo: "SingAssemblies.targets".
5. Modificar el archivo del proyecto (.csproj) para agregar la llamada a la tarea: "BuildSigning".
6. Modificar el archivo del proyecto (.csproj) para agregar la llamada a la tarea: "PublishSigning".

Si el archivo: "SignAssemblies.targets" usa el archivo "SignAssemblies.cmd" para encontrar el último SDK instalado, el paso 2 no es opcional.

Con suerte, este documento ayuda a aclarar las diferentes opciones que tenemos a la hora de firmar ensamblados para cumplir con las políticas de seguridad de ejecución de código.

[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/SigningAssemblies)
