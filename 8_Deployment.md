# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)  
[9. Multiple Environments](9_MultipleEnvironments.md)

## 8. Deployment

Your Avalonia app is working! Now what? You need to deploy it for Windows, Linux, MacOS, iOS, Android, Blazor...

The best approach is to create yourself some script files that automate the builds. Create a [Publish folder](https://github.com/mysteryx93/HanumanInstituteApps/tree/master/Publish) to place your scripts.

I create a script for each target OS that takes 2 parameters: project folder and target runtime. Ex: `eg: publish-win Player432hz win-x86`

I then have another script that compiles all versions for a specific OS (x64, x86).

These scripts will parse your project files for project metadata and version, create a folder with the project version, and dump the release file in it.

To publish, I open the terminal and type

```
# on Windows
publish-win-all.bat

# on Linux
publish-linux-all
publish-osx-all
```

[The releases then come with a long list of assets!](https://github.com/mysteryx93/HanumanInstituteApps/releases)

### Windows (x64, x86)

For Windows, you *must* build it in Windows (or in A Windows VM) because [msbuild has a bug](https://github.com/dotnet/runtime/issues/3828) where icons and attributes do not get set when published on other platforms.

Here's my script for Windows:
- [publish-win.bat](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-win.bat)
- [publish-win-all.bat](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-win-all.bat)

If they fix msbuild on Linux/MacOS, I got a Bash version of the script too
- [publish-win](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-win)
- [publish-win-all](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-win-all)

This will create a ZIP file with your binaries. One file for x86, and one file for x64.

TODO: You could use [Inno Setup](https://jrsoftware.org/isinfo.php) to create an installer. (You can post the instructions if you do so)

### Linux (x64, arm64)

For Linux, the best option is to publish an [AppImage](https://appimage.org/). It creates a single file that runs on every Linux distro. You can also post your project on [AppImageHub](https://www.appimagehub.com/).

You *must* build it in Linux; or in WSL. [Publish-AppImage](https://github.com/kuiperzone/Publish-AppImage) is a script that will do the job for you. You'll need [publish-appimage](https://github.com/kuiperzone/Publish-AppImage/blob/main/publish-appimage) in your `Publish` folder and [publish-appimage.conf](https://github.com/kuiperzone/Publish-AppImage/blob/main/publish-appimage.conf) in your project folder.

I then got these scripts:
- [publish-linux](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-linux)
- [publish-linux-all](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-linux-all)

It will create an AppImage file. One file for x64, and one file for arm64. You can click on it and your application will run. Your users can install [AppImageLauncher](https://github.com/TheAssassin/AppImageLauncher) for a better experience.

#### Linux Arch AUR

For Arch users, you can also publish your application in the [AUR](https://aur.archlinux.org/). It's easy to publish from the AppImage content, and your package should end with `-appimage` sufix.

- [AUR Submission Guidelines](https://wiki.archlinux.org/title/AUR_submission_guidelines)
- [Arch Package Guidelines](https://wiki.archlinux.org/title/Arch_package_guidelines)
- [makepkg](https://wiki.archlinux.org/title/Makepkg)

After having my account and credentials configured...

**How to create a new repo in the AUR**

This will fetch the latest Release from your GitHub repo.

First, create the new Git repo. This will create folder `yangdownloader-appimage.git` in the current directory.

    git clone ssh://aur@aur.archlinux.org/yangdownloader-appimage.git
    cd yangdownloader-appimage.git

Second, add a `PKGBUILD` file. [Here we add a package that requires ffmpeg.](#)

Third, you must generate the security checksum. This will generate/update `sha256sums_x86_64` in PKGBUILD.

    updpkgsums

Fourth, validate the package by building it

    makepkg

If everything is good, publish it

    makepkg --printsrcinfo > .SRCINFO
    git add PKGBUILD .SRCINFO
    git commit -m "useful commit message"
    git push

Voilà!

**How to update an AUR repo**

First, publish a new Release on GitHub.

Second, update `pkgver` in PKGBUILD.

Third, generate checksum with `updpkgsums`.

Fourth, validate package with `makepkg`.

Fifth, publish with the same commands as above.

### OSX (x64, arm64)

To run your application on MacOS, it must be contained within an [Application Bundle](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html) or it won't run at all. Furthermore, the latest arm64 M1 chipsets will refuse to run any code that is not signed! Signing your code requires an [Apple Developper](https://developer.apple.com/) account at $99/year.

This [tutorial explains how to publish an application to the OSX App Store](https://docs.avaloniaui.net/docs/distribution-publishing/macos). The [Microsoft tutorial](https://docs.microsoft.com/en-us/xamarin/mac/deploy-test/publishing-to-the-app-store/) can also be useful.

Overall, it's a $99 per year investment and a LOT of work. Not suitable for free software. Oh, getting that Developer Account also requires a Mac device because your account will be bound to it.

Wait! There *IS* a work-around. To remove code-signing security warnings, run this command in the terminal:

    /Applications % xattr -d com.apple.quarantine Converter432hz.app

Building for OSX requires Linux or MacOS to set the executable flag on the file.

My script will create the application bundle folder structure and zip it:
- [publish-osx](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-osx)
- [publish-osx-all](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/publish-osx-all)

It also requires [App.entitlements](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/App.entitlements), [Info.plist](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Publish/Info.plist), and an icon file in ICNS format.

This will create a ZIP file with your bundle. One file for x64, and optionally one file for arm64. Note that x64 should run fine on arm64 anyway.

### Native Binaries

If you got native binaries that need to be deployed for all these platforms, I came up with this solution. Again, this is not a solution I've seen anyone recommend but that's how I do it.

Create a new project. For BASS audio libraries, I created BassDlls.csproj with this.

Add `or '$(RuntimeIdentifier)'==''"` to the binaries you want at compile-time for development.

Add your binaries in each folder for the respective runtimes. [Sample here.](https://github.com/mysteryx93/HanumanInstituteApps/tree/master/DLL/Bass)

By adding a reference to this project, your will automatically get all needed DLLs directly in your output folder!

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <RuntimeIdentifiers>win-x64;win-x86;linux-x64;osx-x64;osx-arm64</RuntimeIdentifiers>
  </PropertyGroup>

  <ItemGroup Condition="'$(RuntimeIdentifier)'=='win-x64' or '$(RuntimeIdentifier)'==''">
    <Content Include="win-x64\**" >
      <Link>%(Filename)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)'=='win-x86'">
    <Content Include="win-x86\**" >
      <Link>%(Filename)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)'=='linux-x64' or '$(RuntimeIdentifier)'==''">
    <Content Include="linux-x64\**" >
      <Link>%(Filename)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)'=='linux-arm64'">
    <Content Include="linux-arm64\**" >
      <Link>%(Filename)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)'=='osx-x64'">
    <Content Include="osx-x64\**" >
      <Link>%(Filename)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
  <ItemGroup Condition="'$(RuntimeIdentifier)'=='osx-arm64'">
    <Content Include="osx-x64\**" > <!-- Same binaries for osx-x64 and osx-arm64 -->
      <Link>%(Filename)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
</Project>
```

### Automations

You can probably automate these builds with [GitHub Actions](https://github.com/features/actions).

If you come up with a good solution, you can post your instructions! You could also post how to automate the tests at the same time.
