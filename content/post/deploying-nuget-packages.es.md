---
title: "Desplegando paquetes NuGet"
date: 2021-01-25
draft: false
slug: deploying-nuget-packages
tags: ["Visual Studio", "NuGet", "MSBuild"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/DeployPackages)

En una publicación anterior vimos cómo: "[Crear paquetes NuGet](../create_nuget_packages)" pero el paso de despliegue aún era un proceso manual. Intentemos automatizar esto. La figura 1 muestra cómo generar un paquete NuGet para un proyecto de .NET Core (similar al enfoque mostrado anteriormente) utilizando las propiedades del proyecto.

![Creando un paquete NuGet](/deploying-nuget-packages/01.png)

Figura 1: "Creando un paquete NuGet".

Esto agrega un par de líneas al archivo del proyecto (.csproj) como muestra la figura 2:

![Propiedades del archivo de proyecto para crear el paquete NuGet](/deploying-nuget-packages/02.png)

Figura 2: "Propiedades del archivo de proyecto para crear el paquete NuGet".

Mi enfoque también usa tagets de MSBuild. El primer paso es incluir el siguiente código en el archivo del proyecto (.csproj):

``` XML
<Import Project="..\DeployPackage.targets" Condition="Exists('..\DeployPackage.targets')" />
<Target Name="DeployingPackage" AfterTargets="GenerateNuspec">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="DeployPackage" Properties="PackageIdentifier=$(PackageId);PackageVersion=$(Version);DeployPackage=true" />
</Target>
```

Básicamente, importamos el archivo de tareas: "DeployPackage.targets" y llamamos a su tarea: "DeployPackage" pasando dos parámetros: "PackageIdentifier" y "PackageVersion". El contenido del archivo: "DeployPackage.targets" es:

``` XML
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <DeployPackages Condition="'$(DeployPackages)' == ''">false</DeployPackages>
    <NuGetPath>C:\Temp\</NuGetPath>
    <NuGetUrl>https://dist.nuget.org/win-x86-commandline/latest/nuget.exe</NuGetUrl>
    <NuGetSource>C:\Temp\Packages\</NuGetSource>
    <ApiKey>45606a77-19e6-4de3-9302-e651a507c785</ApiKey>
  </PropertyGroup>
  <Target Name="DownloadNuget" Condition="!Exists('$(NuGetPath)nuget.exe')">
    <Message Importance="high" Text="Downloading NuGet..." />
    <DownloadFile SourceUrl="$(NuGeturl)" DestinationFolder="$(NuGetPath)" />
    <Message Importance="high" Text="NuGet downloaded..." />
  </Target>
  <Target Name="DeployPackage" Condition="$(DeployPackage) OR $(DeployPackages)" DependsOnTargets="DownloadNuget">
    <Message Importance="high" Text="Deploying package..." />
    <Exec Command="&quot;$(NuGetPath)nuget.exe&quot; push &quot;$(ProjectDir)bin\$(Configuration)\$(PackageIdentifier).$(PackageVersion).nupkg&quot; -Source $(NuGetSource) -ApiKey $(ApiKey)" />
    <Message Importance="high" Text="Package deployed..." />
  </Target>
</Project>
```

Este archivo de tareas hace toda la magia. Analicémoslo paso a paso.

Primero, especificamos algunas variables que serán útiles para la ejecución de las tareas incluidas en este archivo. La variable: "NuGetSource" tiene un valor correspondiente a un subdirectorio local pero pudiera ser una dirección web.

La tarea: "DeployPackage" (llamada desde el archivo del proyecto) se ejecutará solo si las variables: "DeployPackage" o "DeployPackages" tienen el valor: "true". La variable: "DeployPackage" se pasa desde la llamada en el archivo del proyecto y está diseñada para desplegar solo el paquete NuGet generado para ese proyecto. Por otro lado, la variable: "DeployPackages" es global para este archivo y está diseñada para desplegar todos los paquetes generados en la solución (normalmente cada proyecto genera un paquete NuGet diferente). Si esta tarea cumple con la condición de ejecución, se ejecutará primero su tarea dependiente.

La tarea: "DownloadNuget" proporciona un administrador de paquetes: "NuGet.exe" para no lidiar con herramientas instaladas, rutas de herramientas, diferencias en el entorno de desarrollo entre los miembros del equipo, etc. Una vez finalizada esta tarea, vuelve a la ejecución de la tarea: "DeployPackage".

Finalmente, se llama a la herramienta del administrador de paquetes con todos los parámetros requeridos. La figura 3 muestra un ejemplo de ejecución.

![Resultado del despliegue](/deploying-nuget-packages/03.png)

Figura 3: "Resultado del despliegue".

la figura 4 muestra el paquete desplegado.

![Paquete desplegado](/deploying-nuget-packages/04.png)

Figura 4: "Paquete desplegado".

Espero que estas siemples instrucciones le ayuden a automatizar los despliegues de paquetes NuGet.

[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/DeployPackages)
