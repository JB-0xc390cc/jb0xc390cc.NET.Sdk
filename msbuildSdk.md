# MSBuild and Sdk style projects in .NET

This page gives a basic introduction about how msbuild compiles your .NET project files, and how also explains the `Sdk`
styled projects. The information is mostly gathered from  the original MSDoc pages, so you could read them instead too.

## MSBuild (Microsoft Build Engine)
From [MSDoc](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild?view=visualstudiohttps://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild?view=visualstudio): 
MSBuild "_provides an XML schema for a project file that controls how the build platform processes and builds 
software_"   
Basically all your **.NET project files**(`.csproj`, `.fsproj` etc...) 
are these [MSBuild files](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild?view=visualstudio). In 
short these can contain `Properties`, `Items`, `Tasks` and `Targets`. 




## Sdk style projects
At first a Sdk style project is what's project files is referring to a Sdk. This could look something like that:
```xml
<Project Sdk="MySdk">
  ...
</Project>
```
```xml
<Project Sdk="MySdk/1.0.0">
  ...
</Project>
```
```xml
<Project Sdk="">
    <Sdk Name="MySdk" Version="1.0.0" />
    ...
</Project>
```

_(For example in case of a `.NET C#` project the most commonly used **Sdk** is `Microsoft.NET.Sdk`)_

What **`SDK`** style project does in regard of a **`NOT SDK style`** project, is importing(adding), predefined 
`Properties`, `Items`,`Tasks` and `Targets` to your project. **MSBuild** does this by adding implicit imports to the Sdk
files when evaluating the project file.

```xml
<Project>
    <!-- Implicit top import -->
    <Import Project="Sdk.props" Sdk="Microsoft.NET.Sdk" />
    ...
    <!-- Implicit bottom import -->
    <Import Project="Sdk.targets" Sdk="Microsoft.NET.Sdk" />
</Project>

```
_(This basically just imports this two MSBuild files like  `<Import Project="path/Sdk.props"/>`
The difference is with that approach you get search assist across nuget repositories etc... to find the `Sdk.props `
and `Sdk.targets `files)_    

You could do this import manually like the example above, and you'll get the exact same behaviour, 
but you need to take care of the correct positions(top, bottom)

### Directory properties
If you have a `Microsoft.NET.Sdk`(_or equivalent_) styled project, _(or you import`Microsoft.Common.props` and `Microsoft.
Common.targets` in your project file)_ then **MSBuild**(_basically the mentioned MSBuild files in the Sdk_) scans 
the directory tree upwards and searches for `Directory.build.props` and`Directory.build.targets`,  then it loads
(import) them.

Now assume that you have a "standard" [Microsoft.NET.Sdk style project](#Sdk-style-projects), or you included 
`Sdk.props` at the top and `Sdk.targets` (_from Microsoft.NET.Sdk_) at the bottom of your project file. Then the MSBuild 
load order will look like this:

1. It starts to load your project file
2. At the very top you included a `Microsoft.NET.Sdk`/`Microsoft.Common.props`   
It loads the Sdk's MSBuild files, and your  `Directory.build.props` 
_(NOTE!, Directory.build.props gets imported early in the Microsoft.NET.Sdk's MSBuild files so you won't have much 
   thing defined yet  when loading the Directory.build.props)_ 
3. Continues to load` your project file` (basically now it loads it, because before it stopped at the top)
4.  Loads the `Sdk.targets` (and for this reason your `Directory.build.targets`)

 #### Note: you see the load order is like:   
- Get the default things (**_in reality mostly Properties and Items_**) from   the **Sdk**
- Get the user defined things from the `Directory.build.props` and his `project file`
- Load the `Sdk's .targets` files (here you have the option to modify the build process based on user info)
- Load the `user's .targets` files (He still has full power to decide what he wants to do...)

From this info creating your custom Sdk is sufficient for creating some kind of standard for your 
applications build process (**It not just the build process, because in the build process you decide what `language`, 
what `.NET version`, etc... you want to use**).