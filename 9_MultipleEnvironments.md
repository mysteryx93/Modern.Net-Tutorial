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

## 9. Setting Up Multiple Environments

You can develop an Avalonia application and run it on Windows, MacOS and Linux without issues. 
You'll most likely only experience minor problems with file paths, permissions and such.

If you also want to develop for Android and iOS, then things get a bit more complicated to test.

Did you know that you can setup all environments to run on your machine?

### Using Linux

You'll need KVM+QEMU, with Virtual Machine Manager. Do NOT use VirtualBox (slow) nor the Android Emulator (very slow). 
QEMU will provide near-native performance but can be a bit complex to setup (once).

It is also possible to pass a physical GPU to your virtual machines, allowing you to game at near-native performance, 
but that's even more complex to setup and requires two graphic cards. If you have a dual-graphics laptop, [here's a tutorial to set it up](https://github.com/mysteryx93/GPU-Passthrough-with-Optimus-Manager-Guide),
requiring you to close your session when switching OS. The other option is to have two physical cards in your computer, one for Linux and one for Windows/Mac.
For the sake of .NET programming, you won't need GPU passthrough.

You can easily install Windows 10 in KVM. There are plenty of guides online, just google it, I'm not sure what tutorial is the best.

For Android, install [Android-x86](https://www.android-x86.org/). You'll need at least 5GB of space to debug and run your apps.

For MacOS, [Install MacOS on KVM](https://github.com/kholia/OSX-KVM). XCode alone takes about 8GB.

For iOS... there are no KVM emulators for iPhone, but MacOS does have an emulator.

You can setup network shares in Linux using SAMBA, then access your file system with shared project files from Windows.

WARNING: Editing the same projects from Windows and Linux can cause conflicts with line endings (CR vs CRLF). Git standardizes line endings anyway.

I personally got JetBrains Rider installed in Linux and Visual Studio installed in Windows (for the cases where I need Visual Studio). I was told that Rider has 
issues running in MacOS without GPU; that remains to be tested.

To debug Android applications, you'll need Android Connect `android-tools` package.

Get the Android IP address. Pass your QEMU instance name to the 2nd command to get the IP. If you get no results, try with `sudo`.

    virsh list
    virsh domifaddr android-x86

Then, type this command with the IP address. It's that simple!

    adb connect 192.0.0.0

In Rider, the QEMU instance automatically shows up as a physical device. Hit run, and it runs in Android. Easier than expected!

For iOS, that remains to be tested. There is an iOS emulator in MacOS. Remains to be seem whether it works with decent performance without GPU.

### Using Windows

QEMU also works on Windows but does not have the same performance. You can try [improving QEMU performance by enabling HAXM](https://www.qemu.org/2017/11/22/haxm-usage-windows/).

Android Emulator in Windows should have decent performance with [HAXM enabled](https://learn.microsoft.com/en-us/xamarin/android/get-started/installation/android-emulator/hardware-acceleration?pivots=windows).

If you get your environment setup in Windows, you can open up a discussion to let us know what works or not for you.

### Using MacOS

If you get your environment setup in MacOS, you can open up a discussion to let us know what works or not for you.
