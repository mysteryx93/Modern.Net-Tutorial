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

### Fody

These extensions will save you a lot of time.

#### [ReactiveUI.Fody](https://github.com/kswoll/ReactiveUI.Fody): Simplify change notification for Reactive classes.

```c#
[Reactive]public string SearchId { get; set; }

// Instead of...
public string SearchId 
{
    get { return _SearchId; }
    set { this.RaiseAndSetIfChanged(ref _SearchId, value); }
}
```

#### [Fody.PropertyChanged](https://github.com/Fody/PropertyChanged): Simplify change notification for non-Reactive classes.

```c#
public string GivenNames { get; set; }

// Instead of...
string name;
public string Name
{
    get => name;
    set
    {
        if (value != name)
        {
           name = value;
           PropertyChanged?.Invoke(nameof(Name));
        }
    }
}
```

#### [Fody.ConfigureAwait](https://github.com/Fody/ConfigureAwait): Configure async code's ConfigureAwait at a global level.

One of the bad designs of C# is the need to call `.ConfigureAwait()` on every Task Await call. I did not cover async programming in this tutorial, but basically, every time you use the `await` keyword, you must follow it with `.ConfigureAwait()` with `true` if you need to get back to the same thread (important for UI-related tasks that can only run on the UI thread!), or `false` if the code can continue in a different thread after waiting. `false` is almost always needed for class libraries, yet the default is `true`.

For a class library where you got no UI thread, you often have to specify `.ConfigureAwait(false)` for every single call in the whole library! Very annoying and error-prone.

```c#
await DoSomeWorkAsync().ConfigureAwait(false);
```

This gets even worse when using `await using` keyword:

```c#
var connection = new SqlConnection(Options.ConnectionString);
await using (connection.ConfigureAwait(false))
{
    var command = new SqlCommand(Query, connection) { CommandTimeout = Options.CommandTimeout };
    await using (command.ConfigureAwait(false))
    {
        await connection.OpenAsync().ConfigureAwait(false);
        var sqlDependency = new SqlDependency(command);
        sqlDependency.OnChange += SqlDependencyOnChange;
        CurrentSqlDependency = sqlDependency;
        await using var reader = (await command.ExecuteReaderAsync().ConfigureAwait(false)).ConfigureAwait(false);
    }
}
```

Solution? Simply add [Fody.ConfigureAwait](https://github.com/Fody/ConfigureAwait) to your project and forget all about that mess!

### HanumanInstitute.MediaPlayerUI

[HanumanInstitute.MediaPlayerUI](https://github.com/mysteryx93/MediaPlayerUI.NET/) is a bit more niche, but if you need a media player, or the UI of a media player for your custom needs, this is it. Both BASS and MPV players are cross-platform compatible.

[> Next: Unit Testing](6_UnitTesting.md)
