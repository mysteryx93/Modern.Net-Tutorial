# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)

## 3. Dependency Injection

The idea of Dependency Injection is to turn spaghetti code into ravioli code.

It may require a bit more code than spaghetti code, but...

A class should do one thing and one thing only, and the only reason that class should ever change is if that one thing changes.

It means that you take the time to write your classes correctly, and then you will rarely need to edit them again.

For example, you could create a service to convert currencies by fetching online feeds for exchange rates. It needs to fetch exchange rates, cache with configurable expiration, and perform the currency calculations. This class does NOT need to know about the UI, and you could use it from a web app, a desktop app and a mobile app.

The other great benefit of dependency injection is that it is required to create unit tests!

All inputs and outputs must be done through classes passed via Dependency Injection.

There really are two aspects to the Dependency Injection model: consuming services and writing services.

### Consuming Services

It's easier to start by consuming existing services. Honestly, it's easier to learn Dependency Injection with ASP.NET because it's designed around it. For desktop applications, you need to create your own classes and register everything manually. Create some basic ASP.NET application and study how they register and design their classes.

It's called Dependency Injection because your class will take all its services and dependencies in the constructor.

```c#
private readonly IPitchDetector _pitchDetector;
private readonly IFileSystemService _fileSystem;
private readonly ISettingsProvider<AppSettingsData> _settings;

public MyClass(IPitchDetector pitchDetector, IFileSystemService fileSystem, ISettingsProvider<AppSettingsData> settings)
{
    _pitchDetector = pitchDetector;
    _fileSystem = fileSystem;
    _settings = settings;
}
```

Here, my class needs access to the file system ([System.IO.Abstractions](https://github.com/TestableIO/System.IO.Abstractions)), a service to calculate audio pitch of a music file, and access to my application settings. These are custom classes that I created. IFileSystemService inherits IFileSystem from System.IO.Abstractions and add custom IO methods that my application uses, such as any common tasks. [Here's my IFileSystemService class.](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Common.Services/Services/FileSystemService.cs)

What if you need to use a service that provides no such interface? You may need to create your own class that encapsulates it. Here's a sample of [creating an interface around YouTubeDownloader](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Downloads/YouTubeDownloader.cs), because that definitely needs to be mocked for unit testing.

### Interfaces

You'll note that most of my classes use an interface. In most cases, you will register interfaces in your Dependency Injection Container.

Why? Because for unit tests, you must be able to swap a class with a fake one (mock). For unit tests, all inputs and outputs must be done through dependencies that can be swapped, allowing you to test your class in a controlled environment with virtual inputs and outputs.

Some series of classes may run together for the tests, at which point it is questionable whether to use an interface or not. Truth be told...

- It's not much more work to create the extra interface
- It allows me to quickly visualize the public API of the class
- It allows separating the comments from the code. Once you have the code documentation in your interface, you can add `/// <inheritdoc />` in your implementation to copy those comments.
- Whether to run the classes together or with mocks is a question I'll ask myself later as I write the test. Some tests may run them together while other tests use mocks to simplify the tests.

This is a personal preference, but the only place where I do not use interfaces is for end-user ViewModel classes that are not used within other ViewModels. 

Why? Interfaces define class public APIs that should be relatively stable. ViewModels plug pieces together to show them on the UI. Whenever you add a label or a button, you need to add properties to the ViewModel. It's a very volatile API, and having to edit multiple places just to add a label gets annoying for no real benefit.

One situation where an interface can be useful is to create a design-time ViewModel. Instead of creating an interface, I'll create a derived class `ViewModelDesign` that inherits the ViewModel, has an empty contructor and fills the base ViewModel's dependencies manually, and disables anything that shouldn't run at design-time. A derived class is much less maintenance than implementing a full interface.

### Registering Services

There are many Dependency Injection Container available. Many of them are designed for web applications. [Windsor Castle](https://www.castleproject.org/projects/windsor/) is among the most popular ones for web applications. It take long to initialize, takes considerable memory, but then can respond to thousands of requests per second. It's not ideal for desktop applications, even less for mobile apps.

For desktop and mobile apps, you need something lightweight. You'll often initialize your container to initialize a single instance of your class.

[Avalonia](https://avaloniaui.net/) comes with [Reactive](https://www.reactiveui.net/) built-in, and Reactive comes with [Splat](https://github.com/reactiveui/splat), so I gave Splat a try as a Dependency Injection container.

Splat is the most lightweight and fastest solution. By default, you need to pass your dependencies manually so it doesn't do any automatic dependency injection.

For that, you need [Splat.DependencyInjection.SourceGenerator](https://github.com/reactiveui/Splat.DI.SourceGenerator). It's fastest because it does the dependency injection resolution at compile-time!

To register design-time classes, I use this syntax. Whether we want to show the design-time or run-time class, we must generate the run-time dependency resolution at compile-time. Here, I register the class with "Init" contract, which means that the class will only get returned if we specify `GetService("Init")`. Then, I return either instances as desired. `Locator.Current.GetService<MainViewModel>()` will then return either design-time or runtime classes.

```c#
SplatRegistrations.Register<MainViewModel>("Init");
container.Register(() => 
    Design.IsDesignMode ? new MainViewModelDesign() : Locator.Current.GetService<MainViewModel>("Init"));
```

I tried to turn this into a generic utility method and it doesn't work. The Splat IL generator doesn't handle generics.

I initialize the Dependency Injection Container in a ViewModelLocator class that gets initialized on first use.

```c#
public static class ViewModelLocator
{
    static ViewModelLocator()
    {
        var container = Locator.CurrentMutable;
            
        // Services
        container.Register(() => (IDialogService)new DialogService(new DialogManager(
            viewLocator: new ViewLocator(),
            dialogFactory: new DialogFactory().AddMessageBox()),
            viewModelFactory: t => Locator.Current.GetService(t)));
            
        // ViewModels
        SplatRegistrations.Register<MainViewModel>();

        // Business
        SplatRegistrations.RegisterLazySingleton<ISettingsProvider<AppSettingsData>, AppSettingsProvider>("Init");
        container.Register(() => 
            Design.IsDesignMode ? new AppSettingsProviderDesign() : Locator.Current.GetService<ISettingsProvider<AppSettingsData>>("Init"));
            
        SplatRegistrations.SetupIOC();
    }

    public static MainViewModel Main => Locator.Current.GetService<MainViewModel>()!;
    public static ISettingsProvider<AppSettingsData> SettingsProvider => Locator.Current.GetService<ISettingsProvider<AppSettingsData>>()!;
}
```

### Designing Services

To design a new service, consider this.

1. Your class should do one thing and one thing only. The only reason that class should change is if that one thing changes.
2. You should not care at all about what the UI is going to look like.
3. All inputs and outputs must be done through interface dependencies injected via the constructor so that they can be replaced with fakes.

Sometimes the work is split into various layers. Here I have [AudioEncoder](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/BassAudio/AudioEncoder.cs) to encode audio files via BASS library. It is then used by [EncoderService](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Converter432hz/Business/EncoderService.cs) to do multi-threaded batch-processing for a list of files. EncoderService doesn't care about the UI. I then use it in [MainViewModel](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Converter432hz/ViewModels/MainViewModel.cs) that binds the encoder settings to the UI. MainViewModel glues various components together to provide an UI but really doesn't do much in itself.

### Programming Guidelines

When programming, I always avoid complex code. If something is complex, I write it once in a service with a clean API, or as an extension method, and then call it in a simple way.

For example, processing 100 items, 8 simultaneously, to produce a result in the same order, can be complicated multi-threading code. [Not anymore with this extension method.](https://github.com/mysteryx93/HanumanInstitute.Validators/blob/master/Validators/ListExtensions.cs#L182)

```c#
var result = await list.ForEachOrderedAsync(x => DoSomeWorkAsync(x), 8);
```

Ideally, your code should do nothing complex, and no hacks. Anything hacky is a red flag. Write it once and write it well.

It's easy to write complex code. It's a lot harder to write simple and polished code. You often need to rewrite it a few times to trim the complexity out of it.

If you find yourself having more than 8 dependencies, then your class is probably doing more than one thing and could be split into two separate classes.

Avoid static methods and classes. They cannot be mocked for unit testing. Instead, use Singleton classes (single instance for the whole application) or extension methods. You can make rare exceptions only if the class does something very specific and predictable with no side-effect.

For example, I created a [Cloning](https://github.com/mysteryx93/HanumanInstitute.Validators/blob/master/Validators/Cloning.cs) class to facilitate cloning objects. Actually, it could have been an extension method, but I avoid creating extension methods on type Object to avoid spamming the intellisense.

Another rule of thumb is to avoid `new` keyword in your code. New classes should generally be injected in the constructor, or if you need to create multiple instances, you will inject a Factory in your constructor that is responsible for creating instances. [Here's a sample Factory.](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Player432hz/ViewModels/PlaylistViewModelFactory.cs) The class to create has dependencies, and the factory is responsible for filling those dependencies. The only occasion where it is OK to use the `new` keyword is for well-encapsulated objects that have no side-effects.

### Options

For configurable settings, you'll generally want to use the standard [Options pattern](https://docs.microsoft.com/en-us/dotnet/core/extensions/options). That document is a bit scary. Basically your class takes `IOptions<MySettings>` as a constructor parameter, which, in the simplest case, can be set with `Options.Create(new MySettings())`. IOptions exposes a single method: Value. You need to respect this pattern particularly if you code can ever be used from an ASP.NET project. To use this, you need a reference to `Microsoft.Extensions.Options`.

For a desktop application that saves its settings into a XML configuration file, I also created a [SettingsProvider](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Common.Services/Utilities/SettingsProvider.cs) that handles the data, along with a [design-time version](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Common.Services/Utilities/SettingsProviderDesign.cs) that doesn't load nor save settings to the hard drive. I then use it in my project [like this](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Player432hz/Business/AppSettingsProvider.cs), alongside a [design-time version](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Player432hz/Business/AppSettingsProviderDesign.cs). I then [register](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Player432hz/ViewModelLocator.cs#L44) both the runtime and design-time SettingsProvider into my Dependency Injection Container.


[> Next: MVVM Design](4_MVVM.md)
