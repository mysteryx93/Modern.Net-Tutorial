# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)  
[9. Assembly Trimming](9_AssemblyTrimming.md)  
[10. Multiple Environments](10_MultipleEnvironments.md)

## 9. Assembly Trimming

WPF does not support Assembly Trimming, neither does WinUI3.

> Built a toy MAUI app with three textboxes and SQLite, and it is a whopping 300 MB (127 MB zipped) to set a couple values in a database.

Avalonia v0.10.18 supports trimming with "copyused" mode by adding this to your project file. Unfortunately, .NET 7 does not support "copyused".

```xaml
<TrimMode>CopyUsed</TrimMode>
```

Avalonia v11 supports full Assembly Trimming as of `preview6`. This is essential to deploy applications on multiple platforms where you want to embed the .NET framework. Particularly on Android/iOS when you need to embed it and use AOT to improve startup time and increase battery life (at the cost of disk space).

To enable Assembly Trimming (reduce file size) and enable AOT (Ahead of Time compilation to shorten startup time), add this to your `.csproj`

```xaml
<AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
<PublishTrimmed>true</PublishTrimmed>
<PublishReadyToRun>true</PublishReadyToRun>
```

The `432Hz Player` app for `linux-x64` with embedded .NET7 takes 45.4mb with no trimming or AOT, 27.4mb with trimming and AOT, and 17.1mb with trimming but no AOT. That's very decent. That's a compressed AppImage file.

For `win-x64` without compression, it takes 104.5mb with no trimming or AOT, 37.6mb with trimming, and 68.8mb with trimming and AOT.

As you can see, Assembly Trimming makes a significant difference. Also note that Avalonia11 removed any runtime usage of `System.Xml` which resulted in 2mb trimming reduction! A Xaml platform that doesn't use Xml?? `System.Xml` has tons of dependencies so you might want to consider avoiding it for better trimming.

Although Assembly Trimming is a publishing concern, it requires design decisions that should be taken early on to avoid headaches.

Here are some aspects to consider.

## Reflection

Reflection should be avoided as much as possible.

For types that are dynamically accessed, adding this attribute to your Main method works.

    [DynamicDependency(DynamicallyAccessedMemberTypes.All, typeof(MyClass))]

Adding this to the class itself should also work, but I could not get that to work. I do not know why.

    [DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.All)]

## ViewLocator

The default ViewLocator relies on reflection so that's the first thing to change.

`HanumanInstitute.MvvmDialogs` provides a `StrongViewLocator` to manually specify ViewModel-View pairs without the need for reflection.

```c#
var viewLocator = new StrongViewLocator()
    .Register<AboutViewModel, AboutView>()
    .Register<MainViewModel, MainView>()
    .Register<SettingsViewModel, SettingsView>();

container.Register(() => (IDialogService)new DialogService(new DialogManager(
    viewLocator: viewLocator,
    dialogFactory: new DialogFactory().AddFluent()),
    viewModelFactory: t => Locator.Current.GetService(t)));
```

## Bindings

Avalonia used reflection bindings by default, which can result in your ViewModel class members being trimmed. Instead of marking all of your classes as dynamically accessed, you can use compiled bindings.
 
Before attempting to trim your application, add this to your `.csproj` to enable compiled bindings. This can result in a few problems to resolve.

    <AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>

Then, each View will require `DataType` to be set.

    x:DataType="vm:MainViewModel"

Compiled bindings still have a few bugs, which is why it is not enabled by default.

- [CompiledBinding unable to bind to base class in different assembly](https://github.com/AvaloniaUI/Avalonia/issues/10494)
- [CompileBinding are unable to bind to nested property of generic class](https://github.com/AvaloniaUI/Avalonia/issues/10485)

In practice, I had to copy/paste the base classes into the main assembly, and add `x:CompileBindings="False"` on ListBoxes and ComboBoxes that bind to a `CollectionView<T>`. Any further binding within the ListBox, such as click events, then need `x:CompileBindings="True" x:DataType="vm:IPlaylistViewModel"`.

## EventTriggerBehavior

EventTriggerBehavior relies on reflection. Implement your events as RoutedEvents and use RoutedEventTriggerBehavior instead, which relies on a binding.

If you need to bind to an event that is not a RoutedEvent, you will need to use a custom attached behavior or attached property.

## Serialization

Serialization depends on reflection which will have issues with Assembly Trimming. One solution is to set DynamicDependency attributes.

A better solution, if possible, is to use [Json Serialization Source Generator](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation). There is also a [Xml Source Generator](https://learn.microsoft.com/en-us/dotnet/core/additional-tools/xml-serializer-generator) but the Json implementation is a lot better. Plus, using Json allows to trim `System.Xml` out of your app.

Replacing Xml serialization with Json Source Generator provides these benefits

- Much faster initialization (faster app startup time if loading configuration file, don't underestimate the initialization time of reflection-based serializers)
- Better performance
- Avoids reflection
- Allows trimming `System.Xml`
- Enables assembly trimming without issues

Note: Json Source Generator do not work to serialize anonymous types.

## Incompatible libraries

Certain libraries, such as [YouTubeExplode](https://github.com/Tyrrrz/YoutubeExplode), [do not support Assembly Trimming](https://github.com/Tyrrrz/YoutubeExplode/issues/696)

Add this to your `.csproj` to work around that

```xaml
<ItemGroup>
  <TrimmerRootAssembly Include="YoutubeExplode" />
</ItemGroup>
```

This resulted in about 0.2mb larger file size.

## UI Thread

Some operations, such as adding items to a list shown on the UI from a background thread, would work fine until I enable trimming. Make sure you perform those operations from the UI thread.

I'm also still having issues with [Reactive Commands not executing on the UI thread after Assembly Trimming](https://github.com/AvaloniaUI/Avalonia/issues/10711). I will update once the problem is solved.

[> Next: Multiple Environments](10_MultipleEnvironments.md)
