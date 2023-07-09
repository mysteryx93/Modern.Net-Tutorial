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

## 4. MVVM Design

MVVM stands for Model - View - ViewModel.

If you look at my [project structure](https://github.com/mysteryx93/HanumanInstituteApps/tree/master/Src/App.Converter432Hz/Converter432Hz), I have: Models, Views, ViewModels, Business and Assets. [Services](https://github.com/mysteryx93/HanumanInstituteApps/tree/master/Src/Services) have been moved to a separate project but could definitely be in a folder.

**Models** are data structures to hold data. Ideally, it shouldn't contain any logic at all. Such logic should instead be moved to your services and ViewModels.

**Views** are Avalonia windows with empty code-behind. Other people put certain tasks in the code-behind, such as closing the window when requested. I don't. Call me a purist.

```c#
public partial class MainView : CommonWindow<MainViewModel>
{
    protected override void Initialize() => AvaloniaXamlLoader.Load(this);
}
```

**ViewModels** are logic classes that will be attached as the View's DataContext. All interactions between the View and ViewModel will be done with data binding and commands. You can really accomplish everything that way. You may need [Attached Properties](https://docs.avaloniaui.net/docs/next/concepts/attached-property) and [Binding Converters](https://docs.avaloniaui.net/docs/next/guides/data-binding/how-to-create-a-custom-data-binding-converter) to achieve certain tasks.

**Services** are classes that provide functionalities around a specific topic.

**Business** are classes containing business logic that are not as specific as services and not ViewModels. Business logic is more application-specific whereas services are more general.

**Assets** contain application resources such as images and icons.

If you come from WinForms, it can be a mind-twist to separate the UI from the code; just as if you're used to compact spaghetti code, it can be a mind-twist to split your comptact class into a series of individual services. The hardest part is shifting your mindset.

### Simplifying with Utility Classes

With this [AppStarter](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Apps/Application/AppStarter.cs), `Program.cs` contains only this

```c#
public class Program
{
    [DynamicDependency(DynamicallyAccessedMemberTypes.All, typeof(SettingsPlaylistItem))]
    [STAThread]
    public static void Main(string[] args) => AppStarter.Start<App>(args,
        () => ViewModelLocator.SettingsProvider.Value,
        () => ViewModelLocator.AppPathService.UnhandledExceptionLogPath);

    public static AppBuilder BuildAvaloniaApp() => AppStarter.BuildAvaloniaApp<App>();
}
```

With this [CommonApplication](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Apps/Application/CommonApplication.cs), `App.axaml.cs` contains only this

```c#
public class App : CommonApplication<MainView>
{
    public override void Initialize() => AvaloniaXamlLoader.Load(this);

    protected override INotifyPropertyChanged InitViewModel() => ViewModelLocator.Main;

    protected override void BackgroundInit()
    {
        BassDevice.Instance.InitPlugins();
        BassDevice.Instance.VerifyPlugins();
    }
}
```

With [MvvmDialogs](https://github.com/mysteryx93/HanumanInstitute.MvvmDialogs/), `ViewLocator` contains only this

```c#
public class ViewLocator : ViewLocatorBase
{
    protected override string GetViewName(object viewModel) => viewModel.GetType().FullName!.Replace("ViewModel", "View");
}
```

With [CommonWindow](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Avalonia/Controls/CommonWindow.cs), `MainView.cs` contains only this

```c#
public partial class MainView : CommonWindow<MainViewModel>
{
    protected override void Initialize() => AvaloniaXamlLoader.Load(this);
}
```

I've also kept my `App.axaml.cs` file clean by moving [Styles](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Apps/Styles/CommonStyles.axaml) and [Resources](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Apps/Styles/CommonResources.axaml) into an external file.

```xaml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="using:HanumanInstitute.Converter432Hz"
             xmlns:ui="clr-namespace:FluentAvalonia.Styling;assembly=FluentAvalonia"
             x:Class="HanumanInstitute.Converter432Hz.App">
    <Application.DataTemplates>
        <local:ViewLocator />
    </Application.DataTemplates>
    <Application.Styles>
        <ui:FluentAvaloniaTheme PreferSystemTheme="False" CustomAccentColor="#0a9648" />
        <StyleInclude Source="avares://HanumanInstitute.Apps/Styles/CommonStyles.axaml" />
    </Application.Styles>
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceInclude Source="avares://HanumanInstitute.Apps/Styles/CommonResources.axaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

### Binding to UI

[Binding documentation is here.](https://docs.avaloniaui.net/docs/next/basics/data/data-binding/data-binding-syntax)

To bind ListBox and ComboBox, you may find this [CollectionView](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Avalonia/Controls/CollectionView.cs) and [ListItemCollectionView](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Src/Avalonia/Controls/ListItemCollectionView.cs) very useful to bind both the list and the selection.

```c#
public ListItemCollectionView<EncodeFormat> FormatsList { get; } = new()
{
    { EncodeFormat.Mp3, "MP3" },
    { EncodeFormat.Flac, "FLAC" },
    { EncodeFormat.Opus, "OPUS" }
};
```

```xaml
<ComboBox ItemsSource="{Binding FormatsList}" SelectedItem="{Binding FormatsList.CurrentItem}">
```

To bind ICommand, the author of Reactive recommended me to expose ReactiveCommand instead of ICommand.

```c#
public RxCommandUnit AddPlaylist => _addPlaylist ??= ReactiveCommand.Create(AddPlaylistImpl);
private RxCommandUnit? _addPlaylist;
private void AddPlaylistImpl()
{
    var newPlaylist = _playlistFactory.Create();
    Playlists.Source.Add(newPlaylist);
    Playlists.MoveCurrentToLast();
}
```

```xaml
<Button Width="35" Content="Add" Command="{Binding AddPlaylist}" />
```

OK I'm cheating. `RxCommandUnit` is a shortcut for `ReactiveCommand<Unit, Unit>` defined in `Usings.cs`

```c#
global using RxCommandUnit = ReactiveUI.ReactiveCommand<System.Reactive.Unit, System.Reactive.Unit>;
```

### Handling Events

As simple as it sounds, that's a bit of a challenge.

Option 1 is to use [Avalonia.Xaml.Behaviors](https://github.com/wieslawsoltes/AvaloniaBehaviors).

```xaml
<Interaction.Behaviors>
    <RoutedEventTriggerBehavior RoutedEvent="{x:Static InputElement.DoubleTappedEvent}">
        <InvokeCommandAction Command="{Binding Play}" />
    </RoutedEventTriggerBehavior>
</Interaction.Behaviors>
```

Option 2 is to use [Attached Properties](https://docs.avaloniaui.net/docs/next/concepts/attached-property) to handle events and create a behavior, particularly if it's a pure visual behavior.

Option 3... I tried porting [Singulink.WPF.Data.MethodBinding](https://github.com/Singulink/Singulink.WPF.Data.MethodBinding) over to [MethodBinding.Avalonia](https://github.com/mysteryx93/MethodBinding.Avalonia) but did not manage to get it working. If you can figure it out, that will become an option. JetBrains Rider also gives warnings on such bindings that cannot be disabled.

Using MvvmDialogs, in the ViewModel, you can automatically handle the View Loaded, Closing and Closed events by implementing `IViewLoaded`, `IViewClosing` and `IViewClosed`.

### Mobile Navigation

MvvmDialogs natively supports mobile devices by turning window dialogs into navigation between views. It even automatically supports the back button to come back to a preview view.

The only difference to use navigation mode is that your ViewLocator needs to return a UserControl instead of a Window. You thus need to implement both `MainView` as UserControl and `MainWindow` as Window, and have your ViewLocator return the correct one. The default ViewLocator replaces ViewModel by View on mobile and ViewModel by Window on desktop.

[> Next: Dialogs and Tools](5_DialogsTools.md)
