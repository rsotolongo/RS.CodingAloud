---
title: "Creando y desplegando aplicaciones usando Octopus"
date: 2021-01-11
draft: false
slug: usando-octopus
tags: ["Visual Studio", "Octopus"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/UsingOctopus)

Desde hace un tiempo quería compartir mis experiencias automatizando los despliegues de productos con Octopus desde Visual Studio. Nuevamente, hagámoslo con un ejemplo.

Como no podía ser de otra manera, La figura 1 muestra el primer paquete que debes agregar a tu proyecto para poder crear paquetes Octopus listos para ser desplegados: "OctoPack". La última versión del paquete disponible en el momento de este tutorial era la "3.6.1", pero instalamos la "3.6.0" a propósito para mostrar el proceso de actualización.

![Paquete OctoPack](/using-octopus/01.png)

Figura 1.

La figura 2 muestra el segundo paquete que debería instalarse: "OctopusTool". Lo mismo aquí, la última versión disponible fue la "4.17.0" pero instalamos la "4.15.7" con el mismo propósito.

![Paquete OctopusTool](/using-octopus/02.png)

Figura 2.

Una vez instalados ambos paquetes, podemos comprobar el subdirectorio "paquetes" dentro del subdirectorio de la solución (C:\Temp\WebApplication1\packages) en este caso. La figura 3 muestra el contenido del subdirectorio.

![Paquetes instalados](/using-octopus/03.png)

Figura 3.

Ahora comienza los pasos divertidos. Primero, necesitamos abrir el archivo del proyecto (WebApplication1.csproj) en este caso. Al final del archivo, debemos eliminar la tarea "Importar" de "OctoPack.targets" del subdirectorio del paquete NuGet. Este es un paso MUY importante porque debe repetirse cada vez que se actualiza el paquete "OctoPack". La figura 4 muestra la línea que debe eliminarse.

![Contenidos del archivo de proyecto](/using-octopus/04.png)

Figura 4.

Expliquemos un poco más aquí. MSBuild importará y ejecutará los destinos de "OctoPack" en cada compilación. Si mantenemos esa línea, el archivo que debe modificarse es el que se encuentra en el subdirectorio del paquete, lo que significa que normalmente no forma parte de los elementos controlados por el sistema de control de versiones (TFS ó Git) y estará suelto en cada actualización del paquete. NuGet elimina el subdirectorio del paquete anterior una vez que se descarga la nueva versión después de una actualización. Para evitar estos retrocesos y mantener actualizados los paquetes al mismo tiempo, es una buena práctica eliminar esta línea y continuar el proceso como recomiendo en los siguientes pasos.

Cree un nuevo subdirectorio dentro del proyecto a implementar y copie el archivo "OctoPack.targets" en ella. Normalmente lo llamo "Implementación" y guardo el archivo con el mismo nombre. Quiero decir, copiar "$(SolutionDir)\packages\OctoPack.X.Y.Z\build\OctoPack.targets" a "$(ProjectDir)\Deployment\OctoPack.targets" La figura 5 muestra este paso.

![Tareas del archivo OctoPack](/using-octopus/05.png)

Figura 5.

En el archivo: "OctoPack.targets" dentro del subdirectorio "Implementación", necesitamos modificar algunos campos y agregar algunos otros.

Los dos primeros campos que necesitamos modificar son los que apuntan a las bibliotecas de Octopus para ejecutar las tareas especificadas. La figura 6 lo muestra.

![Tareas importadas](/using-octopus/06.png)

Figura 6.

Debido a que hemos movido el archivo de destino, necesitamos actualizar esas rutas a las bibliotecas dentro del subdirectorio del paquete. Debido a que estamos dentro del subdirectorio "Implementación", necesitamos subir dos niveles hasta el subdirectorio de la solución donde está el subdirectorio "Paquete". Tenga en cuenta que en cada actualización del paquete "OctoPack", necesitamos actualizar estas rutas.

Después, necesitamos especificar el valor de "OctoPackNuGetExePath". Me gusta usar lo que está incluido en el paquete que se está usando porque Octopus usa una versión personalizada de NuGet.exe, por lo que es mejor usar lo que está incluido en el paquete, por esa razón: "..\packages\OctoPack.3.6.0\build\NuGet.exe" es un buen valor (tenga en cuenta que también depende de la versión del paquete y subimos solo un nivel porque esto se ejecuta desde el subdirectorio raí del proyecto). La última variable para modificar aquí es "OctoPackPublishPackageToFileShare" que debe apuntar al subdirectorio compartido donde se copiará el paquete para implementarlo en Octopus. Normalmente usamos "\\rant.com\Nuget", por lo que "\\rant.com\Nuget\Portals.WebApplication1" es una buena opción. La figura 7 muestra las modificaciones.

![Valores de las variables de configuración](/using-octopus/07.png)

Figura 7.

Ahora lo que tenemos que hacer es configurar el proyecto para ejecutar la tarea una vez que esté construida. ¿Recuerda la línea eliminada en la figura 4? Necesitamos agregar una línea allí como se muestra en la figura 8.

![Tareas importadas en el archivo de proyecto](/using-octopus/08.png)

Figura 8.

``` XML
<Import Project=".\Deployment\OctoPack.targets" Condition="Exists('.\Deployment\OctoPack.targets')" />
```

Una vez agregada esta línea podemos construir la solución pero "\\rant.com\Nuget\Portals.WebApplication1" sigue estando vacío. ¿Por qué? Porque necesitamos cambiar el valor de la variable "RunOctoPack" a "verdadero" como muestra la figura 9.

![Valor de configuración para tarea de creación](/using-octopus/09.png)

Figura 9.

Ahora, si construimos nuevamente la solución obtenemos los resultados que se muestran en la figura 10 y un archivo de paquete en "\\rant.com\Nuget\Portals.WebApplication1" como muestra la figura 11.

![Resultados de la primera compilación](/using-octopus/10.png)

Figura 10.

![Primer paquete generado](/using-octopus/11.png)

Figura 11.

Hasta ahora todo va bien, excepto que no importa cuántas veces compilemos, el paquete generado será la misma versión: "1.0.0.0" que no es buena para las versiones automáticas de Octopus. Entonces, arreglemos esto abriendo el archivo "AssemblyInfo.cs" dentro del subdirectorio del proyecto "Propiedades" y cambiemos el atributo "AssemblyVersion" de "1.0.0.0" a "1.0. *" Como se muestra en la figura 12.

![Archivo de información del ensamblado](/using-octopus/12.png)

Figura 12.

Al hacer eso, nos aseguramos de que cada compilación tenga una versión de ensamblado diferente, es decir, una versión de Octopus diferente que configuraremos en los siguientes pasos. Consulte las figuras 13 y 14 para observar los cambios.

![Resultados de la segunda compilación](/using-octopus/13.png)

Figura 13.

![Segundo paquete generado](/using-octopus/14.png)

Figura 14.

Vayamos de nuevo al archivo: "OctoPack.targets" y agreguemos la sección resaltada en la figura 15.

Esta región es una sección totalmente nueva que debemos agregar para configurar (no ejecutar) despliegues automáticos de Octopus. El codigo es:

``` XML
<OctoExePath Condition="'$(OctoExePath)' == ''">..\packages\OctopusTools.4.15.7\tools\Octo.exe</OctoExePath>
<OctopusProject Condition="'$(OctopusProject)' == ''">Portals.WebApplication1</OctopusProject>
<OctopusEnvironment Condition="'$(OctopusEnvironment)' == ''">Development</OctopusEnvironment>
<OctopusPortal Condition="'$(OctopusPortal)' == ''">http://octopus.rant.com/Web</OctopusPortal>
<OctopusApiKey Condition="'$(OctopusApiKey)' == ''">API-L6NRPGPZB4V1EYDVUX23KDXUJU</OctopusApiKey>
<OctopusNotes Condition="'$(OctopusNotes)' == ''">New release sample notes.</OctopusNotes>
```

![Valores de configuración para tarea de despliegue](/using-octopus/15.png)

Figura 15.

Debido a que esto se ejecuta desdel subdirectorio raíz del proyecto, solo necesitamos subir un nivel. "OctoExePath" es la única variable que depende de la ruta del paquete "OctopusTools". Las variables "OctopusProject", "OctopusEnvironment", "OctopusPortal" y "OctopusApiKey" dependen de la configuración de Octopus de tu proyecto, pero lo que puedo decir aquí es que normalmente solo desplegamos al entorno de "Desarrollo" y hablaré de la variable API después en este documento. Por último, especificamos el valor de las notas de la versión en la variable "OctopusNotes" que normalmente se incluyen en el correo electrónico que envía el servidor Octopus una vez que una nueva versión termina de implementarse.

Una vez más, esa región es solo para configurar variables relacionadas con Octopus para crear despliegues. Para hacer realmente los despliegues, necesitamos agregar las siguientes líneas al final de ese archivo. La figura 16 muestra los resultados.

``` XML
<Message Text="Creating release in Octopus Portal: $(OctoPackPackageVersion)" Condition="'$(OctoExePath)' != ''" Importance="Normal" />
<Exec Command='"$(OctoExePath)" create-release --server="$(OctopusPortal)" --project="$(OctopusProject)" --apikey="$(OctopusApiKey)" --deployto="$(OctopusEnvironment)" --version=$(OctoPackPackageVersion) --packageversion=$(OctoPackPackageVersion)' Condition="'$(OctoExePath)' != ''" />
```

![Configuración de tarea de despliegue](/using-octopus/16.png)

Figura 16.

La Figura 17 muestra el resultado de la compilación, observe la sección resaltada después del último mensaje recibido en la figura 10.

![Salida de tercera compilación](/using-octopus/17.png)

Figura 17.

Si abre el portal Octopus en su proyecto después de la compilación, puede obtener algo como lo que se muestra en la figura 18.

![Portal del proyecto en Octopus](/using-octopus/18.png)

Figura 18.

Entonces, con solo un cambio de una palabra: "falso" por "verdadero" en la variable "RunOctoPack" logramos muchos resultados: un paquete Octopus listo para ser desplegado y una versión del producto desplegado en el entorno especificado en Octopus. El riesgo aquí es dejar esa variable como verdadera y/o registrarla como verdadera y hacer un despliegue después de cada compilación. Eso sucede, pero generalmente no es lo que me gusta hacer, por lo que presto doble atención antes de registrar los cambios en este archivo.

Pasemos entonces a la actualización de paquetes. La Figura 19 muestra las actualizaciones disponibles.

![Actualización de paquetes](/using-octopus/19.png)

Figura 19.

Después de actualizar ambos paquetes, Visual Studio elimina las versiones anteriores como muestra la Figura 20.

![Subdirectorios de paquetes actualizados](/using-octopus/20.png)

Figura 20.

Gracias a que estamos usando los objetivos no desde la ubicación del paquete sino desde la ubicación del proyecto, lo que tenemos que hacer es actualizar todas las referencias de la versión anterior a las referencias de la nueva versión, es decir, todos "3.6.0" por "3.6.1" y todos "4.15.7" por "4.17.0". Hay 3 lugares en "OctoPack.targets" para los primeros cambios y solo un lugar para el segundo cambio.

Las figures 21 y 22 muestran los respectivos lugares.

![Configuración actualizada para tarea de creación](/using-octopus/21.png)

Figura 21.

![Configuración actualizada para tarea de despliegue](/using-octopus/22.png)

Figura 22.

Una cosa más, el valor de API para las versiones de Octopus se puede generar accediendo al portal de Octopus...

<http://octopus.rant.com/Web/>

...y siga los pasos de la figura 23 a la figura 26.

![Opciön del perfil en Octopus](/using-octopus/23.png)

Figura 23.

![Configuración del perfil en Octopus](/using-octopus/24.png)

Figura 24.

![Opción de generación de llave API en Octopus](/using-octopus/25.png)

Figura 25.

![Resultado de generación de llave API en Octopus](/using-octopus/26.png)

Figura 26.

Es importante que guarde esta clave porque tan pronto como cierre este cuadro de diálogo, no podrá volver a ver ese valor.

Una última cosa, algunos productos pueden introducir nuevos problemas a la mesa porque pueden estar siendo implementandos por países: es decir, diferentes proyectos de Octopus, entornos, ruta física, etc. Entonces, la idea es tener varios archivos ".target" en el subdirectorio "Implementación" y agregado de la misma manera que lo hicimos en el archivo ".csproj". Las figuras 27 y 28 muestran un ejemplo.

![Archivos multi objetivo](/using-octopus/27.png)

Figura 27.

Se agregaron objetivos para Chile y Brasil para implementar esas respectivas instancias.

![Archivo del proyecto using multi objetivo](/using-octopus/28.png)

Figura 28.

Aquí en el ".csproj" es importante tener el primer objetivo creado (llamémoslo de ahora en adelante el "general" o "genérico" -OctoPack.targets-) como el último en la lista de objetivos importados.

Las figuras 29 y 30 muestran el contenido de esos nuevos objetivos creados.

![Primer archivo de objetivo específico](/using-octopus/29.png)

Figura 29.

![Segundo archivo de objetivo específico](/using-octopus/30.png)

Figura 30.

Básicamente, ambos nuevos destinos son archivos muy pequeños que comparten la misma estructura pero con valores muy específicos para la instancia del país.

RunOctoPack
OctopusProject
OctopusNotas

Son las variables específicas de la instancia que deben cambiar el valor en el momento del despliegue. Todos se controlan ahora mediante una nueva condición que depende del valor de una nueva variable introducida: "Desploy..." ("DeployBrasil" y "DeployChile" en las figuras). Es válido mencionar que es posible incluir más variables como la versión que queremos usar, etc.

Esta nueva variable es la única que deberíamos cambiar si decidimos desplegar esa instancia. Por ejemplo, si quiero desplegar una instancia de Brasil, puse el valor de la variable "DeployBazil" como "verdadero" en el archivo "OctoPack (Brasil).targets" y eso hará que "RunOctoPack" sea verdadero y "OctopusProject" el nombre de Proyecto Octopus para Brasil. Debido a que el destino para Brasil se importa antes que el genérico en el archivo ".csproj", una vez que ese destino se ejecuta en el tiempo de compilación, se implementará utilizando los valores especificados.

Por último, pero no menos importante, debemos modificar el objetivo genérico para eliminar el valor de "OctopusProject". Ver figura 31.

![Configuración actualizada para multi objetivo](/using-octopus/31.png)

Figura 31.

Espero que haya disfrutado de esta guía y la encuentre útil.

[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/UsingOctopus)
