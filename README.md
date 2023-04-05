First Published
Apr 4, 2023
Using MSBuild 17.0

# Welcome

This is basically my cheat sheet of things I probably will forget and need a periodic reminder of for making MSBuild Targets.

Usually I do this when I want to copy files as part of a build process. Maybe you need to do the same thing!

# Concepts

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
* [Well-known Item Metadata](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-well-known-item-metadata)
  * aka list of "built-in macros"
* [Target Build Order](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-build-order)

## API Reference

* [Target Element](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-element-msbuild)
