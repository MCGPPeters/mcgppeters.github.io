---
title: GitVersion with Cake build and .csproj (msbuild) based dotnet core projects
---

Just a little shout out (an note to self) to people who wonder how to use [GitVersion](https://github.com/GitTools/GitVersion) in [Cake](http://cakebuild.net/) build scripts to [semantically versioning](http://semver.org/) the NuGet packages that are the result of building a .csproj (MSBuild) based dotnet core project.

Cake has support for GitVersion. Determining the correct version for the current build could look something like this:

```c#
Task("SetVersionInfo")
    .IsDependentOn("Clean")
    .Does(() =>
{
    versionInfo = GitVersion(new GitVersionSettings {
        RepositoryPath = "."
    });
});
```

That I figured out pretty quickly while writing my first Cake build script. But I had to do a little digging before I had found a way to actually version the build output properly, because there is no default parameter in [MSBuildSettings](http://cakebuild.net/api/Cake.Common.Tools.MSBuild/MSBuildSettings/) or [DotNetCoreBuildSettings](http://cakebuild.net/api/Cake.Common.Tools.DotNetCore.Build/DotNetCoreBuildSettings/) that enables setting the version (only a version-suffix).

Turns out that .csproj allows for setting custom properties, like in this example ```$(SemVer)``` is used:

```MSBuild
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp1.1</TargetFramework>
    <GeneratePackageOnBuild>True</GeneratePackageOnBuild>
    <PackageRequireLicenseAcceptance>True</PackageRequireLicenseAcceptance>
    <Authors>Maurice CGP Peters</Authors>
    <PackageProjectUrl>https://github.com/MCGPPeters/AspNetCoreHttpMessageHandler</PackageProjectUrl>
    <PackageLicenseUrl>https://github.com/MCGPPeters/AspNetCoreHttpMessageHandler/blob/master/LICENSE</PackageLicenseUrl>
    <RepositoryUrl>https://github.com/MCGPPeters/AspNetCoreHttpMessageHandler.git</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
    <PackageTags>asp.net core httpmessagehandler test</PackageTags>
    <Description />
    <Version>$(SemVer)</Version>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Http" Version="1.1.0" />
    <PackageReference Include="Microsoft.AspNetCore.Http.Abstractions" Version="1.1.1" />
    <PackageReference Include="System.Runtime.Serialization.Primitives" Version="4.3.0" />
  </ItemGroup>

</Project>
```

So to set the custom property, one can add a parameter to MSBuild referencing this custom property with ```/p:SemVer={value}```. In the Cake file you must add an ```ArgumentCustomization``` property value to the ```MSBuildSettings``` like so:

```c#
Task("Build")
    .IsDependentOn("RestorePackages")
    .Does(() =>
{
    var buildSettings = new DotNetCoreBuildSettings
     {
         Framework = "netcoreapp1.1",
         Configuration = configuration,
         ArgumentCustomization = args => args.Append("/p:SemVer=" + versionInfo.NuGetVersionV2)
     };

    DotNetCoreBuild(solution, buildSettings);
});

Task("CopyPackages")
    .IsDependentOn("Build")
    .Does(() =>
{
    var files = GetFiles("./src/**/*.nupkg");
    CopyFiles(files, "./artifacts");

});
```

Please note that I'm not using ```DotNetCorePack``` as it doesn't seem to pick up the version number.... Copy the output of the build, which includes the ```.nupkg```.

This will add the correct version to the build output, like for instance:

```AspNetCoreHttpMessageHandler.0.1.0.nupkg```

For the full source code including the samples used here, visit [this](https://github.com/MCGPPeters/AspNetCoreHttpMessageHandler) repository on GitHub.

Hope it helps someone :)