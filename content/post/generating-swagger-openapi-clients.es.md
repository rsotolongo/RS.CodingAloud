---
title: "Generando clientes de Swagger/OpenAPI"
date: 2021-02-15
draft: false
slug: generating-swagger-openapi-clients
tags: ["Visual Studio", "OpenAPI", "Swagger", "API"]
categories: ["Desarrollo"]
---
[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ConnectedServices)

Recientemente, estaba desarrollando una aplicación web .NET Core que necesita consumir una API desarrollada hace unos años usando ASP.NET WebAPI. Afortunadamente, esa API tenía Swagger integrado, así que pensé que podía ahorrar tiempo al no crear manualmente los instrumentos necesarios para las llamadas a la API (DTO, cliente, etc.) en su lugar podría utilizar la función "Servicios Conectados" de Visual Studio.

Me llevé una gran sorpresa que me inspiró a compartir la experiencia y la solución en este post. Reproduje un escenario simplificado como se puede ver en la figura 1 accediendo a Swagger UI a través de su URL: "http://localhost:5200/swagger".

![IU de Swagger de FullAPI](/generating-swagger-openapi-clients/01.png)

Figura 1: IU de Swagger para API usando .NET Framework.

Básicamente, esta API tiene dos controladores que proporcionan dos métodos, cada uno con rutas diferentes pero intenciones equivalentes.

Accediendo a su documento de especificación: "http://localhost:5200/swagger/docs/v1" tenemos:

``` JSON
{
  "swagger": "2.0",
  "info": {
    "version": "v1",
    "title": "RS.Blog.Projects.ConnectedServices.FullAPI"
  },
  "host": "localhost:5200",
  "schemes": [
    "http"
  ],
  "paths": {
    "/api/first/number": {
      "get": {
        "tags": [
          "First"
        ],
        "operationId": "First_NumberMethod",
        "consumes": [],
        "produces": [
          "application/json",
          "text/json",
          "application/xml",
          "text/xml"
        ],
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "format": "int32",
              "type": "integer"
            }
          }
        }
      }
    },
    "/api/first/string": {
      "get": {
        "tags": [
          "First"
        ],
        "operationId": "First_StringMethod",
        "consumes": [],
        "produces": [
          "application/json",
          "text/json",
          "application/xml",
          "text/xml"
        ],
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "type": "string"
            }
          }
        }
      }
    },
    "/api/second/number": {
      "get": {
        "tags": [
          "Second"
        ],
        "operationId": "Second_NumberMethod",
        "consumes": [],
        "produces": [
          "application/json",
          "text/json",
          "application/xml",
          "text/xml"
        ],
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "format": "int32",
              "type": "integer"
            }
          }
        }
      }
    },
    "/api/second/string": {
      "get": {
        "tags": [
          "Second"
        ],
        "operationId": "Second_StringMethod",
        "consumes": [],
        "produces": [
          "application/json",
          "text/json",
          "application/xml",
          "text/xml"
        ],
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "type": "string"
            }
          }
        }
      }
    }
  },
  "definitions": {}
}
```

Para simplificar la aplicación cliente, decidí reproducirla como una aplicación de consola .NET Core. Agregué una carpeta: "OpenAPI" para almacenar el documento de especificación anterior como: "FullAPI.json". Desde la figura 2 hasta la figura 5 podemos ver cómo agregar un servicio conectado.

![Agregar un servicio conectado](/generate-swagger-openapi-clients/02.png)

Figura 2: Agregar un servicio conectado.

![Adición de una especificación OpenAPI](/generate-swagger-openapi-clients/03.png)

Figura 3: Agregar una especificación de OpenAPI.

![Especificando los parámetros de generación del cliente](/generate-swagger-openapi-clients/04.png)

Figura 4: Especificación de los parámetros de generación de clientes.

![Resultados de generación de clientes](/generate-swagger-openapi-clients/05.png)

Figura 5: Resultados de la generación de clientes.

Haga clic con el botón derecho en las dependencias del proyecto del cliente y seleccione: "Agregar Servicio Conectado". Luego haga clic en el botón: "Agregar". Seleccione "OpenAPI" y haga clic en el botón: "Siguiente". Escriba el archivo de especificación ("OpenAPI\FullAPI.json") y "FullAPI" como espacio de nombres para el cliente generado. Finalmente presione el botón: "Finalizar" y se generará el cliente mostrando los resultados en la siguiente pantalla. Este proceso agrega varias líneas de códigos al archivo del proyecto (.csproj) como se muestra a continuación:

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <CodeGenerator>NSwagCSharp</CodeGenerator>
  </OpenApiReference>
</ItemGroup>

<ItemGroup>
  <PackageReference Include="Microsoft.Extensions.ApiDescription.Client" Version="3.0.0">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  </PackageReference>
  <PackageReference Include="Newtonsoft.Json" Version="12.0.2" />
  <PackageReference Include="NSwag.ApiDescription.Client" Version="13.0.5">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  </PackageReference>
</ItemGroup>
```

Una vez compilado, se crea un nuevo archivo llamado: "FullAPIClient.cs" en la carpeta "obj" dentro de la carpeta del proyecto. En este punto, el proyecto deja de compilarse correctamente debido a este archivo.

Demasiados errores, ¡Qué sorpresa! ¿Qué esta pasando y por qué? (figura 6).

![Errores de compilación de generación de clientes](/generate-swagger-openapi-clients/06.png)

Figura 6: Errores de compilación de generación de clientes.

La causa se muestra en la figura 7. El cliente se genera dos veces repitiendo sus campos internos.

![Generación de clientes duplicados](/generate-swagger-openapi-clients/07.png)

Figura 7: Generación de clientes duplicados.

Me preguntaba por qué sucede esto; así que decidí analizar el código fuente del generador del cliente: "NSwag" en este caso (recuerda lo que se agregó al archivo del proyecto). El código fuente se almacena en GitHub en:

[NSwag: La cadena de herramientas Swagger/OpenAPI para .NET, ASP.NET Core y TypeScript](https://github.com/RicoSuter/NSwag)

Al analizar el generador de clientes para C#, podemos encontrar que el valor predeterminado para el parámetro: "Operation Generation Mode" es "MultipleClientsFromOperationId" que se basa en el parámetro: "ClassName" que llega como vacío, por lo que la solución es especificarlo en los parámetros de generación del cliente en el archivo de proyecto (.csproj).

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
  </OpenApiReference>
</ItemGroup>
```

El valor del nombre de clase es un patrón que incluye el nombre del controlador seguido de la palabra: "Client". El nombre del controlador se toma del campo: "operationId" del archivo de especificación. El valor se divide por el carácter: "_" y el primer token se toma como nombre de cliente y el segundo token se toma como nombre de operación. Ejemplos: "Second_NumberMethod" crea el cliente: "SecondClient" con la operación: "NumberMethod".

La Figura 8 muestra los nuevos clientes generados. Esta vez sin errores de compilación porque no se generó ningún cliente duplicado.

![Nuevos clientes generados](/generate-swagger-openapi-clients/08.png)

Figura 8: Nuevos clientes generados.

Ahora, podemos consumir las operaciones de los clientes generados como muestra el siguiente código (archivo: "Program.cs"):

``` C#
using System;
using System.Net.Http;

namespace RS.Blog.Projects.ConnectedServices.Client
{
    internal class Program
    {
        private static void Main()
        {
            var fullClient = new HttpClient
            {
                BaseAddress = new Uri("http://localhost:5200"),
            };
            FullAPI.FirstClient operationFullFirstClient = new FullAPI.FirstClient(fullClient);
            int operationFullFirstNumberResult = operationFullFirstClient.NumberMethodAsync().Result;
            string operationFullFirstStringResult = operationFullFirstClient.StringMethodAsync().Result;
            FullAPI.SecondClient operationFullSecondClient = new FullAPI.SecondClient(fullClient);
            int operationFullSecondNumberResult = operationFullSecondClient.NumberMethodAsync().Result;
            string operationFullSecondStringResult = operationFullSecondClient.StringMethodAsync().Result;
        }
    }
}
```

Problema resuelto, pero mi curiosidad me hizo investigar un poco más y ver qué sucedía con una API creada usando .NET Core en lugar de usar .NET Framework. La Figura 9 muestra la API migrada. Básicamente, las mismas funcionalidades (controladores, métodos, rutas) pero basadas en .NET Core.

![.NET Core API](/generate-swagger-openapi-clients/09.png)

Figura 9: API en .NET Core.

Accediendo a su documento de especificación: "http://localhost:5100/swagger/v1/swagger.json" tenemos:

``` JSON
{
    "openapi": "3.0.1",
    "info": {
      "title": "ConnectedServices API",
      "description": "Description for ConnectedServices API.",
      "version": "v1"
    },
    "paths": {
      "/api/first/number": {
        "get": {
          "tags": [
            "First"
          ],
          "responses": {
            "200": {
              "description": "Success",
              "content": {
                "text/plain": {
                  "schema": {
                    "type": "integer",
                    "format": "int32"
                  }
                },
                "application/json": {
                  "schema": {
                    "type": "integer",
                    "format": "int32"
                  }
                },
                "text/json": {
                  "schema": {
                    "type": "integer",
                    "format": "int32"
                  }
                }
              }
            }
          }
        }
      },
      "/api/first/string": {
        "get": {
          "tags": [
            "First"
          ],
          "responses": {
            "200": {
              "description": "Success",
              "content": {
                "text/plain": {
                  "schema": {
                    "type": "string"
                  }
                },
                "application/json": {
                  "schema": {
                    "type": "string"
                  }
                },
                "text/json": {
                  "schema": {
                    "type": "string"
                  }
                }
              }
            }
          }
        }
      },
      "/api/second/number": {
        "get": {
          "tags": [
            "Second"
          ],
          "responses": {
            "200": {
              "description": "Success",
              "content": {
                "text/plain": {
                  "schema": {
                    "type": "integer",
                    "format": "int32"
                  }
                },
                "application/json": {
                  "schema": {
                    "type": "integer",
                    "format": "int32"
                  }
                },
                "text/json": {
                  "schema": {
                    "type": "integer",
                    "format": "int32"
                  }
                }
              }
            }
          }
        }
      },
      "/api/second/string": {
        "get": {
          "tags": [
            "Second"
          ],
          "responses": {
            "200": {
              "description": "Success",
              "content": {
                "text/plain": {
                  "schema": {
                    "type": "string"
                  }
                },
                "application/json": {
                  "schema": {
                    "type": "string"
                  }
                },
                "text/json": {
                  "schema": {
                    "type": "string"
                  }
                }
              }
            }
          }
        }
      }
    },
    "components": {}
}
```

Lo primero que es diferente (y probablemente lo más importante) es el campo: "openapi" (anteriormente "swagger") con la versión: "3.0.1" (anteriormente: "2.0"). Guardemos esta nueva especificación como: "CoreAPI.json" junto con la otra en la carpeta: "OpenAPI". Ahora, generemos el cliente de la misma manera que lo hicimos para la API desarrollada en .NET Framework y veamos qué sucede. Las figuras del 10 al 14 muestran el proceso.

![Agregar un servicio conectado](/generate-swagger-openapi-clients/10.png)

Figura 10: Agregar un servicio conectado.

![Agregar una especificación de OpenAPI](/generate-swagger-openapi-clients/11.png)

Figura 11: Agregar una especificación de OpenAPI.

![Especificando los parámetros de generación del cliente](/generate-swagger-openapi-clients/12.png)

Figura 12: Especificación de los parámetros de generación de clientes.

![Resultados de la generación de clientes](/generate-swagger-openapi-clients/13.png)

Figura 13: Resultados de la generación de clientes.

![Lista de servicios conectados](/generate-swagger-openapi-clients/14.png)

Figura 14: Lista de servicios conectados.

Se agrega una nueva sección al archivo del proyecto (.csproj) similar a la anterior:

``` XML
<OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI" />
```

Resultando en:

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI" />
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
  </OpenApiReference>
</ItemGroup>
```

La solución se compila incluyendo el nuevo archivo generado: "CoreAPIClient.cs" en la carpeta: "obj" pero el código generado es diferente:

1. Solo hay una clase de cliente.
2. El constructor del cliente requiere la URL base.
3. Las operaciones tienen diferentes nombres.

![Cliente generado para .NET Core API](/generate-swagger-openapi-clients/15.png)

Figura 15: Cliente generado para una API en .NET Core.

Analizando el archivo de especificación ("CoreAPI.json") podemos notar que no hay campos: "operationId" y eso es importante para los nombres de clientes y operaciones. Entonces, si agregamos, de alguna manera, los identificadores de operación a cada operación, deberíamos resolver el primer y tercer problema.

Para agregar el campo: "operationId" al archivo de especificación, necesitamos agregar el atributo: "SwaggerOperation" a cada método de cada controlador. Ese atributo es parte del paquete NuGet: "Swashbuckle.AspNetCore.Annotations" que necesitamos instalar primero. El siguiente código muestra cómo usarlo:

``` C#
using Microsoft.AspNetCore.Mvc;
using Swashbuckle.AspNetCore.Annotations;

namespace RS.Blog.Projects.ConnectedServices.CoreAPI.Controllers
{
    public class FirstController : ControllerBase
    {
        [HttpGet, Route("api/first/number")]
        [SwaggerOperation(OperationId = "First_NumberMethod")]
        public int NumberMethod()
        {
            return 211;
        }

        [HttpGet, Route("api/first/string")]
        [SwaggerOperation(OperationId = "First_StringMethod")]
        public string StringMethod()
        {
            return "Framework: Core, Controller: First, Method: String";
        }
    }
}
```

Con el uso de este atributo, el archivo de especificación agrega el campo: "operationId" pero luego el código no se compila debido al mismo primer problema: clientes duplicados. Necesitamos modificar los parámetros de generación de la misma manera que lo hicimos antes. El archivo del proyecto (.csproj) debería verse así:

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI">
    <ClassName>{controller}Client</ClassName>
  </OpenApiReference>
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
  </OpenApiReference>
</ItemGroup>
```

Pero el código no compila. ¿Por qué, si hicimos exactamente lo que hicimos al generar clientes para la API en .NET Framework? La figura 16 muestra los errores.

![Errores al generar clientes para .NET Core API](/generate-swagger-openapi-clients/16.png)

Figura 16: Errores al generar clientes para la API en .NET Core.

Tenga en cuenta que los errores están en los clientes generados anteriormente (archivo: "FullAPI.cs"). Bueno, esto se debe a un [error bien conocido](https://github.com/dotnet/aspnetcore/issues/18204) en NSwag CSharp Generator que está activo y debería resolverse en el futuro y tener una solución fácil. Modifiquemos una vez más el archivo del proyecto (.csproj):

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI">
    <ClassName>{controller}Client</ClassName>
  </OpenApiReference>
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
    <Options>/GenerateExceptionClasses:false /ExceptionClass:"CoreAPI.ApiException"</Options>
  </OpenApiReference>
</ItemGroup>
```

Hemos agregado la etiqueta: "Options" indicando no generar clases de excepciones porque esas clases ya están generadas en otro espacio de nombres. Esto se debe a que el orden para la generación de los clientes es importante y el primero que se inserta (el cliente de la API de .NET Core en este caso) genera las clases de excepción. Ahora la solución se compila con éxito, pero aún no hemos resuelto el segundo problema (se requiere la URL base en el constructor). Para solucionar esto, una vez más necesitamos modificar el archivo del proyecto (.csproj):

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI">
    <ClassName>{controller}Client</ClassName>
    <Options>/UseBaseUrl:false</Options>
  </OpenApiReference>
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
    <Options>/GenerateExceptionClasses:false /ExceptionClass:"CoreAPI.ApiException"</Options>
  </OpenApiReference>
</ItemGroup>
```

Observe que podemos lograr el mismo resultado sin modificar manualmente el archivo del proyecto (.csproj) si especificamos los valores para diferentes opciones en el parámetro: "Opciones adicionales de generación de código" en el cuadro de diálogo para agregar una nueva referencia del servicio OpenAPI (figuras 4 y 12).

En esta ocasión para el cliente de la API desarrollada en .NET Core hemos agregado la opción de no usar URL base. Ahora los clientes generados son casi idénticos y se pueden utilizar de forma muy similar:

``` C#
using System;
using System.Net.Http;

namespace RS.Blog.Projects.ConnectedServices.Client
{
    internal class Program
    {
        private static void Main()
        {
            var fullClient = new HttpClient
            {
                BaseAddress = new Uri("http://localhost:5200"),
            };
            var operationFullFirstClient = new FullAPI.FirstClient(fullClient);
            int operationFullFirstNUmberResult = operationFullFirstClient.NumberMethodAsync().Result;
            string operationFullFirstStringResult = operationFullFirstClient.StringMethodAsync().Result;
            var operationFullSecondClient = new FullAPI.SecondClient(fullClient);
            int operationFullSecondNumberResult = operationFullSecondClient.NumberMethodAsync().Result;
            string operationFullSecondStringResult = operationFullSecondClient.StringMethodAsync().Result;

            var coreClient = new HttpClient
            {
                BaseAddress = new Uri("http://localhost:5100"),
            };
            var operationCoreFirstClient = new CoreAPI.FirstClient(coreClient);
            int operationCoreFirstNumberResult = operationCoreFirstClient.NumberMethodAsync().Result;
            string operationCoreFirstStringResult = operationCoreFirstClient.StringMethodAsync().Result;
            var operationCoreSecondClient = new CoreAPI.SecondClient(coreClient);
            int operationCoreSecondNumberResult = operationCoreSecondClient.NumberMethodAsync().Result;
            string operationCoreSecondStringResult = operationCoreSecondClient.StringMethodAsync().Result;

        }
    }
}
```

El archivo de proyecto del cliente (.csproj) hay comentado otro enfoque para hacer todo lo que hemos expuesto aquí sin la necesidad de campos de identificadores de operación. Basándonos en la ruta podemos utilizar la opción: "/OperationGenerationMode:MultipleClientsFromPathSegments" junto con otros parámetros interesantes como: "OutputPath", y "SourceUri".

### Resumen ###

El identificador de operación (campo: "operationId") es muy importante para la generación de clientes. Swagger 2.0 (más reciente para .NET Framework) lo genera para cada método, mientras que OpenAPI 3.x no lo genera. Se recomienda agregarlo manualmente usando el atributo: "SwaggerOperation". La función "Servicios Conectados" de Visual Studio es una capacidad realmente buena que nos ahorra mucho tiempo una vez que sabemos cómo usarla correctamente, como hemos mostrado en esta publicación.

[Código fuente de apoyo disponible en GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ConnectedServices)
