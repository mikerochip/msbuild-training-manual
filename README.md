# Mike's MSBuild Training Manual

#### First Published
Apr 4, 2023\
Using MSBuild 17.0

# Welcome

MSBuild is Microsoft's declarative XML-based build system for `dotnet` and C++ projects.

It works by processing XML Project Files (`*.csproj`, `*.vcproj`, etc) which tell MSBuild what to do.

Microsoft has put in a lot of sensible defaults to keep project files from being bloated, which is awesome until I need to modify it and forget how to do so. Usually, my use case is copying some custom data or binaries as part of a build.

This document is a cheat sheet of things that are easy to forget about MSBuild Project Files.

# Concepts

MSBuild only has a couple of "nouns." Targets are where the magic of build steps (called Tasks) happens. Everything outside of Targets is where you configure things for your targets.

* Predefined Elements
  * Make up most of MSBuild's XML Schema
  * Let you group and define Properties
  * Let you sequence `Task`s within a `Target` (more on Targets below)
  * e.g. `Project`, `PropertyGroup`, `ItemGroup`, `Target`, `Content`, etc.
* Properties
  * Fancy word for variables AFAICT
  * Are the child elements of `PropertyGroup` and `ItemGroup`
  * Are a mix of predefined and user-defined

MSBuild only has one "verb" and that's Tasks.

* Tasks
  * Can only be child elements of the predefined element `Target`
  * Are how you make magic happen in MSBuild
  * The order of Tasks inside a Target determines their execution order
  
# Useful Predefined Elements

## `PropertyGroup`

### Usage

Define one or more single-value properties. Values are defined in xml inner text.

### Example

```
<PropertyGroup>
  <SourceMyDataDirPath>..\MyData</SourceMyDataDirPath>
  <TargetMyDataDirPath>$(MSBuildProjectDirectory)\MyData</TargetMyDataDirPath>
</PropertyGroup>
```

## `ItemGroup`

### Usage

Define one or more multi-value properties. Values are defined in xml attributes.

Think of this as a way to define collection variables.

### Example

```
<ItemGroup>
    <SourceMyDataFiles Include="$(SourceDataDirPath)\**\*.json" />
</ItemGroup>
```

## `Target`

### Usage

Define a set of tasks to execute. These will mostly be what you want to work with.

Note that child elements of Target can be a mix of Tasks, PropertyGroups, etc

The main trick to getting your Targets to execute when you want is to combine the attribute keys on [this page](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-build-order) with the attribute values on [this page](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-extend-the-visual-studio-build-process).

### Example

```
<Target Name="CopyMyData" BeforeTargets="BeforeBuild;BeforeRebuild">
    <Message Text="CopyMyData: $([System.IO.Path]::GetFullPath('$(SourceMyDataDirPath)')) -> $(TargetMyDataDirPath)" Importance="High" />
    <Copy SourceFiles="@(SourceMyDataFiles)"
            DestinationFolder="$(TargetMyDataDirPath)\%(RecursiveDir)"
            SkipUnchangedFiles="true" />
</Target>
```

# Assorted Mojo

* You can use the dotnet Base Class Library in your attribute values! Example above
  * e.g. `$([System.IO.Path]::GetFullPath('$(DirPath)'))`
* The `Message` element's `Importance` attribute must be set to `High` in order to see your output in your IDE's Output window
* The "macro" `%(RecursiveDir)` is super useful in the Copy task's `DestinationFolder` value. It will mirror whatever directory structure exists in your `SourceFiles` value
* The `$` symbol evaluates a property into its value (like a shell script)
* The `@` symbol returns the elements of an ItemGroup property (which are multi-value data containers)
* At the top-level (i.e. Project element), PropertyGroups ALL evaluate before ItemGroups. Within a Target it's ordered from top to bottom.

# HOWTO

#### Copy files from an external data folder for builds but also make it work in the IDE
```
<!-- put your source folder in a top-level PropertyGroup so you can set it as a parameter -->
<PropertyGroup>
    <MyExternalDataPath>..\..\MyData</MyExternalDataPath>
    <MyDataDirName>MyData</MyDataDirName>
</PropertyGroup>

<ItemGroup>
    <!-- exclude project data since that's just for running in IDE -->
    <Content Remove="$(MyDataDirName)\**" />

    <!-- copy source data straight into builds -->
    <Content LinkBase="$(MyDataDirName)" Include="$(MyExternalDataPath)\**"
             CopyToOutputDirectory="Always"
             CopyToPublishDirectory="Always" />
</ItemGroup>

<Target Name="CopyDataToProject" BeforeTargets="Build">
    <!-- delete first since the project data might be stale -->
    <RemoveDir Directories="$(MyDataDirName)" />

    <ItemGroup>
        <MyExternalDataFiles Include="$(MyExternalDataPath)\**\*.*" />
    </ItemGroup>

    <Copy SourceFiles="@(MyExternalDataFiles)"
          DestinationFolder="$(MyDataDirName)\%(RecursiveDir)" />
</Target>
```

#### Include a local data directory as reference in your IDE (and exclude from builds)
```
<PropertyGroup>
    <MyLocalDataPath>MyData</MyLocalDataPath>
</PropertyGroup>

<ItemGroup>
    <!-- if MyDataPath is inside your project, then this removes it -->
    <Content Remove="$(MyLocalDataPath)\**" />
    <None LinkBase="FolderNameInIDE" Include="$(MyLocalDataPath)\**" CopyToOutputDirectory="Never" />
</ItemGroup>
```

# Documentation

## Learning

* [Extend the build process](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-extend-the-visual-studio-build-process)
  * This page is a super helpful primer on how to write custom targets.
  * If you read nothing else, then read this!
* [Reserved and Well-known Properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-reserved-and-well-known-properties)
  * aka list of "reserved keywords"
* [Common MSBuild Project Properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/common-msbuild-project-properties)
  * list of out of the box properties, along with Project Types that they are available in
* [MSBuild properties for Microsoft.NET.Sdk](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props)
  * list of `dotnet`-specific extensions to MSBuild (as opposed to C++)
* [Well-known Item Metadata](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-well-known-item-metadata)
  * aka list of "built-in macros"

## API Reference

* [Task Reference](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-task-reference)
  * List of Task-type elements and their parameters
* [Target Element](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-element-msbuild)
* [Choose Element](https://learn.microsoft.com/en-us/visualstudio/msbuild/choose-element-msbuild)
  * conditional execution
