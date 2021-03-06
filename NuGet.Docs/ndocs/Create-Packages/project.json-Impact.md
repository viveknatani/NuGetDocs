# project.json Impact on Package Authors

The project.json system used in NuGet 3.x affects package authors in several ways as described in the following sections.

## Changes affecting existing packages usage

Traditional NuGet packages support a set of features that are not carried over to the transitive world. 

### Install and Uninstall scripts 

These scripts are not supported and will be ignored. In case they exist in the package a project using transitive restore. 

The main reason we removed support for this is that in the transitive model, is that there is no concept of package install time. A package is either present or not present, but there is no consistent process that occurs when a package is installed. Install scripts only worked in Visual Studio and other IDE had to mock Visual Studio extensibility API in an attempt to support this scenario. But common editors have no support, and similarly command line tools have no support for running these scripts. 


### Content Transforms 

Similarly to install scripts, transforms run on package install and are typically not idempotent. Since there is no install time anymore, XDT Transform and similar features are not supported, and will be ignored if such a package is used in a transitive scenario. 


### Content 

Traditional NuGet packages are shipping content files such as source code and configuration files. There are used typically in two scenarios 

1. Initial files dropped into the project so the user can edit them at a later time. The common example is default configuration files. 

2. Content files used as companions to the assemblies installed in the project. The example here would be a logo image used by an assembly. 

Support for content is currently disabled for similar reasons for scripts and transforms, but we are in the process of designing support for content. 

Content files can still be carried inside the packages, and will be ignored currently, however the end user can still copy them into the right spot. 

You can see one of the proposals for bringing back content files, and follow its progress, here: [https://github.com/NuGet/Home/issues/627](https://github.com/NuGet/Home/issues/627)   

## Impact for Package Authors

Packages using the above features would have to use a different mechanism. The most commonly useful mechanism for this would be the MSBUILD targets/props that continues to get fully supported. The build system can choose to pick up other conventions in the package. This is how msbuild targets are supported as well as Roslyn analyzers. It is possible to build packages that supports targets and analyzers for packages.config and project.json scenarios. 

Packages that attempt to modify the project to ease startup, typically work in a very limited set of scenarios, and will instead provide a readme, or guidance on how to use the package. 

Most existing packages should not need to use the new package format described below. 

The new format enables native content as a first class scenario. This means that managed assemblies depending on close to hardware implementations to ship binary implementations alongside the managed assemblies based on the target platform. For example System.IO.Compression package is utilizing this technology.  [https://www.nuget.org/packages/System.IO.Compression](https://www.nuget.org/packages/System.IO.Compression)  

In summary if the functionality above is not absolutely necessary, we recommend sticking with the existing package format. The new format will be supported only by NuGet 3.0. 

It would be possible to build packages to work for both packages.config and project.json scenarios through shimming, however it is often simpler to just structure the packages the traditional way, without the deprecated features mentioned above. 


## 3.x Package Format  ##

The 3.x package format allows for several new features: 

1. Defining a reference assembly used for compilation and a set of implementation assemblies used for runtime on different platforms/devices. Which allows you to take advantage of platform specific APIs while providing a common surface area for your consumers. Specifically this makes writing intermediate portable libraries easier. 

2. Allows packages to pivot on platforms e.g. operating systems or CPU architecture. 

3. Allows for separation of platform specific implementations to companion packages. 

4. Support Native dependencies as a first class citizen. 