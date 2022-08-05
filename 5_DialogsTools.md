# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)

## 5. Dialogs and Tools

Avalonia being quite new, I had to create myself some tools.

### HanumanInstitute.MvvmDialogs

One theme that is generally a complex issue with MVVM is handling dialogs. This is made very simple using [HanumanInstitute.MvvmDialogs.Avalonia](https://github.com/mysteryx93/HanumanInstitute.MvvmDialogs/).

I register `IDialogService` in my Dependency Injection Container.

```c#
container.Register<IDialogService>(() => new DialogService(
      viewModelFactory: x => Locator.Current.GetService(x), 
      dialogManager: new DialogManager(viewLocator: new ViewLocator(),
          dialogFactory: new DialogFactory().AddMessageBox())));
```

Create `DialogExtensions.cs`

```c#
public static class DialogServiceExtensions
{
    public static async Task ShowAdvancedSettingsAsync(this IDialogService service, INotifyPropertyChanged ownerViewModel,
        EncodeSettings settings)
    {
        var vm = service.CreateViewModel<AdvancedSettingsViewModel>();
        Cloning.CopyAllFields(settings, vm.Settings);

        if (await service.ShowDialogAsync(ownerViewModel, vm).ConfigureAwait(false) == true)
        {
            Cloning.CopyAllFields(vm.Settings, settings);
        }
    }
}
```

And here's a Command to show the dialog!

```c#
public RxCommandUnit ShowAdvancedSettings => _showAdvancedSettings ??= ReactiveCommand.CreateFromTask(ShowAdvancedSettingsImpl);
private RxCommandUnit? _showAdvancedSettings;
private Task ShowAdvancedSettingsImpl() =>
    _dialogService.ShowAdvancedSettingsAsync(this, Encoder.Settings);
```

[HanumanInstitute.MvvmDialogs](https://github.com/mysteryx93/HanumanInstitute.MvvmDialogs/) is very well documented so take the time to read the documentation.

MvvmDialogs is what allows me to keep my View code-behind empty while others put some code.

### HanumanInstitute.Validators

[HanumanInstitute.Validators](https://github.com/mysteryx93/HanumanInstitute.Validators) comes with very handy tools.

Instead of writing

```c#
if (value == null) { throw new NullReferenceException(nameof(value)); }
_field = value;
```

You can now write
```c#
_field = value.CheckNotNull(nameof(value));
```

It has other super-useful functions
```c#
value.CheckRange(nameof(value), min: 0);
var msg = "Value {0} is invalid.".FormatInvariant(value);
var result = await list.ForEachOrderedAsync(x => DoSomeWorkAsync(x));
```

This library is optional, but I use these functions all the time so I don't know how you can program without it.

Go through [HanumanInstitute.Validators](https://github.com/mysteryx93/HanumanInstitute.Validators) documentation to check the list of methods available.

### HanumanInstitute.MediaPlayerUI

[HanumanInstitute.MediaPlayerUI](https://github.com/mysteryx93/MediaPlayerUI.NET/) is a bit more niche, but if you need a media player, or the UI of a media player for your custom needs, this is it. For Avalonia, it currently only supports audio playback via [BASS](https://www.un4seen.com/) as of writing this. [MPV](https://mpv.io/) support will come later. Both of these players are cross-platform compatible.

[> Next: Unit Testing](6_UnitTesting.md)
