---
title: "Deploying NuGet packages"
date: 2021-01-25
draft: false
slug: deploying-nuget-packages
tags: ["Visual Studio", "NuGet", "MSBuild"]
categories: ["Development"]
---
[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/DeployPackages)

I a previous post we saw how to: "[Create NuGet packages](../creating_nuget_packages)" but the deployment step was still a manual process. Let's try to automatize this. Figure 1 shows how to generate NuGet package for a .NET Core project (similar to the approach previously shown) using the project's properties.

![Creating NuGet package](/deploying-nuget-packages/01.png)

Figure 1: "Creating NuGet package".

This adds a couple of lines into the project file (.csproj) as figure 2 shows:

![Project file properties to create NuGet package](/deploying-nuget-packages/02.png)

Figure 2: "Project file properties to create NuGet package".

My approach also uses MSBuild targets. The first step is to include the following code into the project file (.csproj):

``` XML
<Import Project="..\DeployPackage.targets" Condition="Exists('..\DeployPackage.targets')" />
<Target Name="DeployingPackage" AfterTargets="GenerateNuspec">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="DeployPackage" Properties="PackageIdentifier=$(PackageId);PackageVersion=$(Version);DeployPackage=true" />
</Target>
```

Basically, we import the target file: "DeployPackage.targets" and calls its task: "DeployPackage" passing two parameters: "PackageIdentifier" and "PackageVersion". Content of file: "DeployPackage.targets" is:

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

This target file makes all the magic. Let's analyze it step by step.

First, we specify a few variables that will be useful for the execution of tasks included in this file. Variable: "NuGetSource" has a value pointing to a local folder but it could be a web address.

The task: "DeployPackage" (called from the project file) will be executed only if variables: "DeployPackage" or "DeployPackages" have value: "true". Variable: "DeployPackage" is passed from the project file call and is designed to deploy only the NuGet package generated for that project. In the other hand, variable: "DeployPackages" is global to this file and is designed to deploy all packages generated in the solution (normally every project generates one different NuGet package). If this task matches the condition to be executed, it will execute its dependent task first.

Task: "DownloadNuget" provides package manager: "NuGet.exe" in order to not deal with installed tools, tool paths, development environment differences between team members, etc. Once this task finish, it returns to the execution of the task: "DeployPackage".

Finally, the package manager tool is called with all required parameters. Figure 3 shows an example of execution.

![Deployment result](/deploying-nuget-packages/03.png)

Figure 3: "Deployment result".

Figure 4 shows the deployed package.

![Deployed package](/deploying-nuget-packages/04.png)

Figure 4: "Deployed package".

I hope these simple instructions help you to automate NuGet deployments.

[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/DeployPackages)
