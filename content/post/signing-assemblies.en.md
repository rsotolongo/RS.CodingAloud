---
title: "Signing assemblies"
date: 2021-02-08
draft: false
slug: signing-assemblies
tags: ["Visual Studio", "MSBuild", "security"]
categories: ["Development"]
---
[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/SigningAssemblies)

## .NET Core ##

Not all clients require to sign the assemblies of their applications but when its required, the task is not so easy as you can expect.

In the case of .NET Core applications, we can achieve this just editing the project file (.csproj) adding the following code:

``` XML
<Target Name="BuildSigning" AfterTargets="AfterBuild">
  <Exec Command="SIGNING_COMMAND" />
</Target>
```

As example **SIGNING_COMMAND** could be:

*"C\Program Files (x86)\Windows Kits\10\bin\10.0.177630\x64\signtool.exe"* *sign /f "$(SolutionDir)certificate.pfx" /p "pass&1234" "S(TargetDir)*.dll"*

We are using the certificate generated in this previous post: "[Generating signature certificates](../generating-signature-certificates)".

This makes that after every build (regardless of configuration -Debug, Release, etc.-) library assemblies are signed at the end of the process. The post script signing continues working after builds but not after publishing. The result of the previous post script publishing a .NET Core application is "curious":

Assuming that publishing is based in "Release" configuration, the assemblies in "Release" folder are signed before copied into "Publish" temporary folder. Then, artifacts in "Publish" temporary folder are copied into "Publish" final destination. Finally, the assemblies in "Release" folder are re-signed again.

Assemblies in "Release" folder are signed twice because for publishing functionality the macro S(TargetDir) continues having the value for the "Release" folder and not the "Publish" folder. One solution tested was changed the macro $(TargetDir) by the macro $(PublishDir) but it didn't work because that macro takes the value of the "Publish" temporary folder and not of the "Publish" final folder. The result is that assemblies in "Release" folder are not signed twice but assemblies in "Publish" folder are not signed either. Instead, assemblies in "Publish" temporary folder are signed after being already copied into "Publish" final folder.

With this background, the best solution is to change the post build script to be a post publish script:

``` XML
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <Exec Command="SIGNING_COMMAND" />
</Target>
```

Changing **SIGNING_COMMAND** to:

*"C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64\signtool.exe" sign /f "$(SolutionDir)certificate.pfx" /p "pass&1234" "$(PublishUrl)*.dll"*

Notice the addition of *"publish\"* after $(TargetDir) macro.

This modification will work but of course only for publishing and not for every build. Nothing stops us to add both post scripts (build and/or publish) so assemblies will be signed in all cases, the only problem with this is that assemblies in "Release" folder will be signed-twice. That is not a big deal but is something to keep in mind.

Another topic is the idle time while signing assemblies. Depending of the application being signed this process could takes considerable amount of time that is double in case of double signing or triple in case of using the two post scripts (Build and Publish). Because "Publish" is not used so often as "Build" that idle time is not a big deal except, we include a timestamp in the signature.

Adding parameter */t timeseverURL* to **SIGNING_COMMAND** could jump (as example) from 15 seconds to 2030 seconds (4.5 minutes) in case of an external time server like <http://timestamp.comodoca.com/authenticode>. If we use an internal time server this time could decrease significantly but its usage will depend if the execution security policy takes or not the timestamp in consideration.

Finally let's analyze a sample for post publish script:

``` XML
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <Message Importance="high" Text="Signing published assemblies..." />
  <Exec Command="&quot;C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64\signtool.exe&quot; sign /f &quot;$(SolutionDir)certificate.pfx&quot; /p &quot;pass&amp;1234&quot; &quot;$(PublishUrl)*.dll&quot; &quot;$(PublishUrl)*.exe&quot;" />
  <Message Importance="high" Text="Published assemblies signed..." />
</Target>
```

To include starting and ending messages on build console is a good practice. All double quotes (") should be encoded to its HTML equivalent (`&quot;`). As the symbol ampersand (&) that coincidently is present on the certificate password, needs to be encoded as well (`&amp;`). Not only library assemblies (DLL) are signed, executables (EXE) are signed too.

Finally, something that requires to pay attention is the Windows SDK version (10.0.17763.0) because that depends of what is installed in the development computer but could be different for every team member. With all of this in mind other approach could be adopted bringing clarity and generality:

``` XML
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <Exec Command="$(SolutionDir)SignAssemblies.cmd $(SolutionDir) $(PublishUrl)" />
</Target>
```

Where file: "SignAssemblies.cmd" could be:

``` CMD
ECHO Signing assemblies...

"C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64\signtool.exe" sign /f "%certificate.pfx" /p "pass&1234" "%2*.dll" "%2*.exe"

ECHO Assemblies signed...
```

This still have the problem of the hard-coded SDK version but could be generalized as:

``` CMD
ECHO Signing assemblies...

SET basedir=C:\Program Files (x86)\Windows Kits\10\bin\
SET newestSdkDir=10.0.17763.0
FOR /F "tokens=*" %%a in ('dir /b /on "%baseDir%10*"') DO SET newestSdkDir=%%a
REM "%baseDir%%newestSdkDir%\x64\signtool.exe" sign /?
"%baseDir%%newestSdkDir%\x64\signtool.exe" sign /f "%1certificate.pfx" /p "pass&1234" "%2*.dll" "%2*.exe"

ECHO Assemblies signed...
```

The SDK version is initialized as version 10.0.17763.0 (because is the latest I have installed in my computer at the time to be writing this post but could be any other version) and then iterates over all folders matching pattern "10*" in order to get latest version. There is a useful instruction that do nothing but if we substitute the **REM** by a **ECHO** we can see runtime information in the compilation traces. Notice how we removed the encoded characters because we moved into the command file (.cmd) and there is no need to used them there.

Last, let's build an even more unified approach using more MSBuild tasks. We need to substitute the previous "PublishSigning" MSBuild target by the following code:

``` XML
<Import Project="..\SignAssemblies.targets" Condition="Exists('..\SignAssemblies.targets')" />
<Target Name="PublishSigning" AfterTargets="AfterPublish">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(PublishUrl);AssembliesPattern=*.dll" />
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(PublishUrl);AssembliesPattern=*.exe" />
</Target>
```

Basically, including another external MSBuild target file and calling its main target passing "AssembliesPath" and "AssembliesPattern" parameters. This is because not all solutions generate library and executable assemblies at the same time. Now, the new "SignAssemblies.targets" file looks like:

``` XML
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="SignAssemblies">
    <Message Importance="high" Text="Signing assemblies..." />
    <Message Importance="high" Text="Assemblies path: $(AssembliesPath)" />
    <Message Importance="high" Text="Assemblies pattern: $(AssembliesPattern)" />
    <Exec Command="$(SolutionDir)SignAssemblies.cmd $(SolutionDir) $(AssembliesPath)$(AssembliesPattern)" />
    <Message Importance="high" Text="Assemblies signed..." />
  </Target>
</Project>
```

File "SignAssemblies.cmd" remains almost identical with just one little modification (the pattern of files to be signed is passed from the calling task):

``` CMD
ECHO Signing assemblies...

SET basedir=C:\Program Files (x86)\Windows Kits\10\bin\
SET newestSdkDir=10.0.17763.0
FOR /F "tokens=*" %%a in ('dir /b /on "%baseDir%10*"') DO SET newestSdkDir=%%a
REM "%baseDir%%newestSdkDir%\x64\signtool.exe" sign /?
"%baseDir%%newestSdkDir%\x64\signtool.exe" sign /f "%1certificate.pfx" /p "pass&1234" "%2"

ECHO Assemblies signed...
```

In case we want to sign all assemblies also after build, we can add the following lines to the project file (.csproj) before or after previous "PublishSigning" target:

``` XML
<Target Name="BuildSigning" AfterTargets="AfterBuild">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(TargetDir);AssembliesPattern=*.dll" />
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(TargetDir);AssembliesPattern=*.exe" />
</Target>
```

## .NET Framework ##

Things don't change drastically from .NET Core approach because we tried to be as generic as possible from the beginning. However, there are a few details to take in consideration.

In .NET Framework application we can edit the Build Events in the project properties as is shown in figure 1.

![Project build events](/signing-assemblies/01.png)

Figure 1: Project build events.

This modifies the project file (.csproj) adding the following lines...

``` XML
<PropertyGroup>
  <PostBuildEvent>
    ECHO Test message...
  </PostBuildEvent>
</PropertyGroup>
```

...where of course we can replace the ECHO command by the **SIGNING_COMMAND** from previous section. The result is very similar, assemblies get signed in the configuration folder (Debug. Release, etc.) but publishing feature continue having issues. If we check the published folder, we can verify that assemblies produced by the solution get signed while depending third party assemblies (like NuGet packages) are not signed by our certificate. They remain signed by their original signer or they are not signed at all. This is not what we need, we need all assemblies signed by our certificate preferably at publishing time in order to avoid manual signing before deployments.

I tried to reuse the approach in the previous section for .NET Core but was not possible without modifications. I ended modifying manually the project file (.csproj) to add the following lines:

``` XML
<Import Project="..\SignAssemblies.targets" Condition="Exists('..\SignAssemblies.targets')" />
<Target Name="PublishSigning" AfterTargets="GatherAllFilesToPublish">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(ProjectDir)$(WPPAllFilesinSingleFolder)\bin\;AssembliesPattern=*.dll" />
</Target>
```

Here we are specifying that we want to execute target: "PublishSigning" when everything is ready to be copied into the final "Publish" folder (target: "GatherAllFilesToPublish"). Very similar to what we did in .NET Core but changing the value for "AssembliesPath" parameter. With this approach all assemblies are signed just before everything is ready to be copied into the final "Publish" folder; from the "Publish" temporary folder to "Publish" final folder. There is no need to sign executable files because .NET Framework web applications don't generate .exe files.

In case we want to sign all assemblies also after build, we can add the following lines to the project file (.csproj):

``` XML
<Target Name="BuildSigning" AfterTargets="AfterBuild">
  <MSBuild Projects="$(MsBuildThisFile)" Targets="SignAssemblies" Properties="AssembliesPath=$(TargetDir);AssembliesPattern=*.dll" />
</Target>
```

I tried to include in file "SignAssemblies.targets" the calling "PublishSigning" target to simplify the manual modifications in the project fie (.csproj) and reuse across different solutions. Unfortunately, that is not a good idea because calling targets depends on parameters that are specific for the projects in development (Attribute: "Properties"). This looks more consistent and standardized.

Ideal assemblies signing process could looks like:

1. Copy and add certificate file: "certificate.pfx" to the solution.
2. Copy and add batch fie: "SignAssemblies.cmd" To the solution.
3. Copy and add MSBuild file "SignAssemblies.targets" to the solution.
4. Modify project file (.csproj) to include file: "SingAssemblies.targets".
5. Modify project file (.csproj) to add calling targets: "BuildSigning".
6. Modify project file (.csproj) to add calling targets: "PublishSigning".

If file: "SignAssemblies.targets" uses file "SignAssemblies.cmd" in order to find the latest SDK installed, step 2 is not optional.

Hopefully, this document helps to clarify the different options we have at the time to sign assemblies to comply with code execution security policies.

[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/SigningAssemblies)
