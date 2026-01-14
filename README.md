# jb0xc390cc.NET.Sdk

**Build standardization for distributed .NET repositories, with modern standards.**

Before reading this page better to read a bit about [MSBuild](msbuildSdk.md) things specially used for this project.
([msbuildSdk.md](msbuildSdk.md))

From now refer to all the things that can be set in the `MSBuild Schema XML` files as **build settings**.
And the files that contains these as **build files**

## The Problem
Maintaining multiple projects with the same standards(_basically you can create a standard with **build settings**_) can be 
time-consuming and error-prone.
- First idea could be copy-pasting the **build settings** across the **project files**(ex: .csproj files in C# applications)  
- - Issue:  
But this is a very naive approach that's why have `Directory.build.props` and `Directory.build.targets` files.   
And these are very useful, but just if you have a monorepo, and all your projects lives in the same repository.
- You could create templates with `.NET template` engine that plants your **build files** in your repositories.
- - Issue:  
You should still manually apply the templates at each project creation, and you won't get a notification if 
  you have a newer standard, you'll probably just leave it with the older version. Moreover, you need to take care of 
  referencing these **build files** in your `Directory.build.props/.targets` or `.csproj` files.
- You could plant the **build files** in git submodules
- - Issue:
Same as with templates, but it's easier to update
- Creating a nuget package that contains the **build settings** as your project will load the `.props`/`.target` files 
  from nuget packages too.
- - Issue:
You can't set all settings there, because the nuget package import could only happen if you already set the 
    `TargetFramework` property for example, so you can't set some properties with your standard.

## Creating your custom Sdk
Just as like in a standard .NET sdk styled project
```xml
<Project Sdk="Microsoft.NET.Sdk">
  ...
</Project>
```
you could use your own Sdk too
```xml
<Project Sdk="MySdk">
  ...
</Project>
```
To leverage the advantages of the `Microsoft.NET.Sdk` too, you should import the `Microsoft.NET.Sdk`'s **.props** file in 
 `YourSdk`'s **.props** file at the very top just like that:
```xml
<Project Sdk="MySdk">
  ...
  ...  
    <Import Project="Sdk.props" Sdk="Microsoft.NET.Sdk" /> <!-- From here it'll load  the user's build settings-->
</Project>
```

And also in  `YourSdk`'s **.targets** file must import the `Microsoft.NET.Sdk`'s **.targets** file like that:
```xml
<Project Sdk="MySdk">
  ...
  ...  
    <Import Project="Sdk.props" Sdk="Microsoft.NET.Sdk" /> <!-- From here it'll load  the user's build settings-->
</Project>
```
With that approach the user has still full power on your custom Sdk and can set/modify any **build setting**, just like 
before,but he can use your predefined standards(**build settings**) too.  
- To change on this and force your custom **Sdk's** **build settings** over the user's settings import the `Microsoft.
  NET.Sdk`'s **.targets** file at the very top of `YourSdk`'s **.targets** file

### Ejecting a project from `YourSdk`
At some point you might want to eject a project from `YourSdk` because for example, it leaves the organization and 
becomes opensource with complete different maintainers. 
- Just simply retargeting to `Microsoft.NET.Sdk` will lead to lot compile errors and headaches

#### CLI Tool
You can create a .NET  CLI tool that manages this ejecting:
1. It would basically strip the **build setting** from the referenced `YourSdk` version and write them down on the 
   disk as files in a directory named `.YourSdk`.
2. It modifies the `Directory.build.props` and `Directory.build.props`, or just the `.csproj` by importing the 
   previously stripped `YourSdk` files.

Now you already detached your application from `YourSdk` and can calmy read what **build settings** does it set.

### Advanced: keeping your Sdk files in the repository.
The .NET CLI tool could check the referenced `YourSdk` version after parsing your project. Then when launching the tool
it could write out the **build settings** (This is what we basically have in previous section).  
But now we don't want to eject from the `YourSdk`, it's still reference, and you update to a new version.
- The previous version's files still on the disk
- The newer version of the `YourSdk` does nothing because it is smart enough to detect, that you have the files on 
  the disk.
- The stripped out (previous version) `YourSdk` has a version property, and so at build time, the new Sdk will print
an error, that you need to update the version with the cli tool.
In theory, we could do that without using the CLI tool too, but I don't think overwriting the repository content, with
a nuget package update is a good idea, it's a horrible idea for sure...
#### Going back:
There is an option in the .NET CLI tool, to go back to the original state, where the files aren't kept locally in 
the repository, it'll just basically delete the folder...