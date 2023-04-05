# Concepts

## `PropertyGroup`

### Usage

Define single-value variables.

### Example

```
<PropertyGroup>
  <SourceMyDataDirPath>..\MyData</SourceMyDataDirPath>
  <TargetMyDataDirPath>$(MSBuildProjectDirectory)\MyData</TargetMyDataDirPath>
</PropertyGroup>
```

## `ItemGroup`

### Usage

Define multi-value variables.

Very useful for declaring and assigning data to collections.

### Example

```
<ItemGroup>
    <SourceMyDataFiles Include="$(SourceDataDirPath)\**\*.json" />
</ItemGroup>
```

## `Target`

### Usage

Define a set of tasks to execute. These will mostly be what you want to write.

You'll want to comebine the attribute keys on [this page](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-build-order) with the attribute values on [this page](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-extend-the-visual-studio-build-process) to determine when your tasks execute.

### Example

```
<Target Name="CopyMyData" BeforeTargets="BeforeBuild;BeforeRebuild">
    <Message Text="CopyMyData: $([System.IO.Path]::GetFullPath('$(SourceMyDataDirPath)')) -> $(TargetMyDataDirPath)" Importance="High" />
    <Copy SourceFiles="@(SourceMyDataFiles)"
            DestinationFolder="$(TargetMyDataDirPath)\%(RecursiveDir)"
            SkipUnchangedFiles="true" />
</Target>
```

# Mojo

* You can use the dotnet Base Class Library in your attribute values! Example above
  * e.g. `$([System.IO.Path]::GetFullPath('$(DirPath)'))`
* The `Message` element's `Importance` attribute of must be set to `High` in order to see your output in your IDE's Output window


# Reference

## Learning and HOWTOs

* [Extend the build process](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-extend-the-visual-studio-build-process)
  * This page is a super helpful primer on how to write custom targets.
  * If you read nothing else, then read this!
* [Reserved and Well-known Properties](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-reserved-and-well-known-properties)
  * aka "reserved keywords"
* [Well-known Item Metadata](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-well-known-item-metadata)
  * aka "built-in macros"
* [Target Build Order](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-build-order)

## API Reference

* [Target Element](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-element-msbuild)