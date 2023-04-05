First Published
Apr 4, 2023
Using MSBuild 17.0

# Welcome

MSBuild is the dotnet build system. It works by processing these things called Project Files (`*.csproj`, `*.vcproj`, etc) which tell MSBuild what to do.

Microsoft has put in a lot of good defaults to MSBuild, which is awesome until I need to modify it and forget how to do so. Usually, my use case is copying some custom data or binaries as part of a build.

This document is a cheat sheet of things that are easy to forget about MSBuild Project Files.

# Concepts

MSBuild only has a couple of "nouns"

* Predefined Elements
  * Part of MSBuild's XML Schema
  * Let you group and define Properties
* Properties
  * Fancy word for child XML elements (that aren't Tasks)
  * Are a mix of predefined and user-defined
  * Use user-defined properties

MSBuild only has one "verb"

* Tasks
  * Are how you define custom behavior in MSBuild
  * Are a special case of predefined element that show up under Target elements
  * The order of Tasks underneath a Target determines their sequence
  
# Useful Predefined Elements

## `PropertyGroup`

### Usage

Define single-value variables via xml inner text.

### Example

```
<PropertyGroup>
  <SourceMyDataDirPath>..\MyData</SourceMyDataDirPath>
  <TargetMyDataDirPath>$(MSBuildProjectDirectory)\MyData</TargetMyDataDirPath>
</PropertyGroup>
```

## `ItemGroup`

### Usage

Define multi-value variables via xml attribute values.

Think of this as a way to declare and assign data to a collection.

### Example

```
<ItemGroup>
    <SourceMyDataFiles Include="$(SourceDataDirPath)\**\*.json" />
</ItemGroup>
```

## `Target`

### Usage

Define a set of tasks to execute. These will mostly be what you want to work with.

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


# Reference

## Learning and HOWTOs

* [Extend the build process](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-extend-the-visual-studio-build-process)
  * This page is a super helpful primer on how to write custom targets.
  * If you read nothing else, then read this!
* [Reserved and Well-known Properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-reserved-and-well-known-properties)
  * aka list of "reserved keywords"
* [Common MSBuild Project Properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/common-msbuild-project-properties)
  * list of out of the box properties, along with Project Types that they are available in
* [Well-known Item Metadata](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-well-known-item-metadata)
  * aka list of "built-in macros"
* [Target Build Order](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-build-order)

## API Reference

* [Task Reference](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-task-reference)
* [Target Element](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-element-msbuild)
