---
title: "Generating Swagger/OpenAPI clients"
date: 2021-02-15
draft: false
slug: generating-swagger-openapi-clients
tags: ["Visual Studio", "OpenAPI", "Swagger", "API"]
categories: ["Development"]
---
[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ConnectedServices)

Recently I was developing a .NET Core web application that needs to consume an API developed few years ago using ASP.NET WebAPI. Fortunately, that API have Swagger integrated, so I thought I can save time do not creating manually the instruments required to API calls (DTOs, client, etc.) instead using the "Connected Services" feature of Visual Studio.

I got a big surprise that inspired me to share the experience and the solution in this post. I reproduced a simplified scenario as you can see in figure 1 accessing to Swagger UI through its URL: "http://localhost:5200/swagger".

![FullAPI Swagger UI](/generating-swagger-openapi-clients/01.png)

Figure 1: Swagger UI for API using .NET Framework.

Basically, this API have two controllers providing two methods each with different routes but equivalent intentions.

Accessing to its specification document: "http://localhost:5200/swagger/docs/v1" we have:

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

To simplify the client application, I decided to reproduce it as a .NET Core console application. I added a folder: "OpenAPI" to store the previous specification document as: "FullAPI.json". From figure 2 until figure 5 we can see how to add a connected service.

![Adding a Connected Service](/generating-swagger-openapi-clients/02.png)

Figure 2: Adding a Connected Service.

![Adding an OpenAPI specification](/generating-swagger-openapi-clients/03.png)

Figure 3: Adding an OpenAPI specification.

![Specifying client generation parameters](/generating-swagger-openapi-clients/04.png)

Figure 4: Specifying client generation parameters.

![Client generation results](/generating-swagger-openapi-clients/05.png)

Figure 5: Client generation results.

Right click on client project dependencies and select: "Add Connect Service". Then click on button: "Add". Select "OpenAPI" and click on button: "Next". Type the specification file ("OpenAPI\FullAPI.json") and "FullAPI" as the namespace for the generated client. Finally press button: "Finish" and client will be generated showing the results on next screen. This process adds several lines of codes into the project file (.csproj) as is shown next:

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

Once is compiled, a new file called: "FullAPIClient.cs" is created in folder "obj" inside project folder. At this point, the project stops to build successfully due to this file.

Too many errors, what a surprise! what is happening and why? (figure 6).

![Client generation build errors](/generating-swagger-openapi-clients/06.png)

Figure 6: Client generation build errors.

The cause is being shown in figure 7. The client is generated twice repeating their internal fields.

![Duplicated client generation](/generating-swagger-openapi-clients/07.png)

Figure 7: Duplicated client generation.

I was wonder about why this happens; so, I decided to analyze the source code of the client generator: "NSwag" in this case (remember what was added to the project file). The source code is stored in GitHub at:

[NSwag: The Swagger/OpenAPI toolchain for .NET, ASP.NET Core and TypeScript](https://github.com/RicoSuter/NSwag)

Analyzing the C# client generator, we can find that the default value for parameter: "Operation Generation Mode" is "MultipleClientsFromOperationId" that relies on parameter: "ClassName" that arrives as empty, so the solution is to specify it on the client generation parameters on project file (.csproj).

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
  </OpenApiReference>
</ItemGroup>
```

Class name value is a pattern that includes the controller name followed by the word: "Client". The controller name is taken from the field: "operationId" of the specification file. The value is divided by character: "_" and the first token is taken as client name and the second token is taken as the operation name. Examples: "Second_NumberMethod" creates the client: "SecondClient" with operation: "NumberMethod".

Figure 8 shows the new clients generated. This time free of build error because no duplicated client was generated.

![New generated clients](/generating-swagger-openapi-clients/08.png)

Figure 8: New generated clients.

Now, we can consume the operations from the generated clients as the following code shows (file: "Program.cs"):

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

Problem solved but my curiosity made me to research a little bit more and see what happened with an API created using .NET Core instead of using the .NET Framework. Figure 9 shows the migrated API. Basically, the same functionalities (controllers, methods, routes) but based on .NET Core.

![.NET Core API](/generating-swagger-openapi-clients/09.png)

Figure 9: .NET Core API.

Accessing to its specification document: "http://localhost:5100/swagger/v1/swagger.json" we have:

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

The first thing that is different (and probably the most important one) is the field: "openapi" (previously "swagger") with version: "3.0.1" (previously: "2.0"). Let's save this new specification as: "CoreAPI.json" together with the other one in folder: "OpenAPI". Now, let's generate the client in the same way we did for API developed in .NET Framework and see what happens. Figures from 10 to 14 shows the process.

![Adding a Connected Service](/generating-swagger-openapi-clients/10.png)

Figure 10: Adding a Connected Service.

![Adding an OpenAPI specification](/generating-swagger-openapi-clients/11.png)

Figure 11: Adding an OpenAPI specification.

![Specifying client generation parameters](/generating-swagger-openapi-clients/12.png)

Figure 12: Specifying client generation parameters.

![Client generation results](/generating-swagger-openapi-clients/13.png)

Figure 13: Client generation results.

![Connected Services list](/generating-swagger-openapi-clients/14.png)

Figure 14: Connected Services list.

A new section is added to the project file (.csproj) similar to the previous one:

``` XML
<OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI" />
```

Resulting in:

``` XML
<ItemGroup>
  <OpenApiReference Include="OpenAPI\CoreAPI.json" CodeGenerator="NSwagCSharp" Namespace="CoreAPI" />
  <OpenApiReference Include="OpenAPI\FullAPI.json" Namespace="FullAPI">
    <ClassName>{controller}Client</ClassName>
    <CodeGenerator>NSwagCSharp</CodeGenerator>
  </OpenApiReference>
</ItemGroup>
```

The solution builds including the new generated file: "CoreAPIClient.cs" in folder: "obj" but the generated code is different:

1. There is only one client class.
2. The constructor for the client requires the base URL.
3. Operations have different names.

![Generated client for .NET Core API](/generating-swagger-openapi-clients/15.png)

Figure 15: Generated client for .NET Core API.

Analyzing the specification file ("CoreAPI.json") we can notice that there are no fields: "operationId" and that is important to client and operation names. So, if we add, somehow, the operation identifiers to each operation, should solve the first and third issue.

To add the field: "operationId" to the specification file we need to add the attribute: "SwaggerOperation" to each method of each controller. That attribute is part of NuGet package: "Swashbuckle.AspNetCore.Annotations" that we need to install it first. The following code shows how to use it:

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

With the usage of this attribute, the specification file adds field: "operationId" but then code don't compile due to the same first problem: duplicated clients. We need to modify the generation parameters in the same way we did before. The project file (.csproj) should looks like:

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

But, code doesn't build. Why, if we did exactly what we did generating clients for .NET Framework API? Figure 16 show the errors.

![Errors generating clients for .NET Core API](/generating-swagger-openapi-clients/16.png)

Figure 16: Errors generating clients for .NET Core API.

Notice that errors are in the previous generated clients (file: "FullAPI.cs"). Well, this is because a [well-known bug](https://github.com/dotnet/aspnetcore/issues/18204) in NSwag CSharp Generator that is active and should be resolved in the future and have an easy solution. Let's modify once again the project file (.csproj):

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

We have added the tag: "Options" indicating to not generate exceptions classes because those classes are already generated in another namespace. This is because the order for client generation matters and the first inserted (the .NET Core API client in this case) generates the exception classes. Now the solution build successfully but still we have not solved the second issue (base URL required in the constructor). To solve this, once again we need to modify the project file (.csproj):

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

Notice that we can achieve the same result without modifying manually the project file (.csproj) if we specify the values for different options in the parameter: "Additional code generation options" in the dialog for add new OpenAPI service reference (figures 4 and 12).

This time for the API client developed in .NET Core we have added the option to not use base URL. Now the generated clients are almost identical and can be used in a very similar way:

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

The client project file (.csproj) has commented out another approach to do all we have exposed here without the need of operation identifiers fields. Based on the route we can use the option: "/OperationGenerationMode:MultipleClientsFromPathSegments" together with another interesting parameters like: "OutputPath", and "SourceUri".

### Summary ###

The operation identifier (field: "operationId") is a very important to clients generation. Swagger 2.0 (latest for .NET Framework) generates it for each method while OpenAPI 3.x don't generates it. It is recommended to add it manually using the attribute: "SwaggerOperation". Visual Studio feature "Connected Services" is a really good capability that save us a lot of time once we know how to use it properly as we have shown in this post.

[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ConnectedServices)
