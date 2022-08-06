# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)

## 7. Reactive

The majority of Reactive users have been programming 2 to 8 years without it, and then regretted not using it earlier. There is a learning curve to it, and you already got many big topics to learn, but Reactive is apparently worth the investment as its users do not look back.

If you're learning many new topics with this tutorial, Avalonia, MVVM and Unit Tests are really the essentials. Reactive code can wait for later.

I'm no expert at Reactive either. It's a bit of a challenge to teach and learn it.

For most code, you use [Reactive](https://www.reactiveui.net/); or plain Linq. For collections, you use [DynamicData](https://github.com/reactivemarbles/DynamicData).

Where it gets a bit tricky is that Linq, Reactive and DynamicData each have their own set of extension methods that are sometimes incompatible with each other. This makes it less intuitive to use and can introduce confusion.

From my limited experience, Reactive itself is relatively intuitive; but DynamicData can get complicated even for moderate tasks. Do not make the mistake of using Reactive just for the sake of using Reactive. Use it where it makes sense. It can make your life much easier for certain things and much harder for others.

### Common Usage

Here are some simple usage patterns of Reactive to get started.

#### ReactiveUI.Fody

You cannot use Fody/PropertyChanged with Reactive. Instead, use [ReactiveUI.Fody](https://github.com/kswoll/ReactiveUI.Fody).

```c#
[Reactive]
public string Value { get; set; } = string.Empty;
```

#### Commands

You can bind this to a button, as explained in the Avalonia section.

```c#
public ObservableCollection<string> Files { get; } = new();

public ReactiveCommand<Unit, Unit> Clear => _clear ??= ReactiveCommand.Create(ClearImpl,
    Files.ToObservableChangeSet().ToCollection().Select(x => x.Any()));
private ReactiveCommand<Unit, Unit>? _clear;
private void ClearImpl()
{
    Files.Clear();
} 
```

ReactiveCommand has several benefits over traditional ICommand, among which:

- It guarantees that the command will never be called twice in parallel.
- Execute takes a typed parameter, and can return data.
- CanExecute is automatically set to false when the command is running.

#### Automatic Property

WhenAnyValue tracks INotifyPropertyChanged. First argument(s) is the property (or properties) to track (can be multiple arguments), and the last is to compute the expression. In this case, `IsBitRateVisible` will automatically be True if SelectedValue is not Flac or Wav.

```c#
public MyClass() {
    _isBitrateVisible = this.WhenAnyValue(x => 
        x.FormatsList.SelectedValue, value => value != EncodeFormat.Flac && value != EncodeFormat.Wav)
        .ToProperty(this, x => x.IsBitrateVisible);
}
    
public bool IsBitrateVisible => _isBitrateVisible.Value;
private readonly ObservableAsPropertyHelper<bool> _isBitrateVisible;
```

There is a LOT more that can be done, we're barely scratching the surface.

### Ressources

Read the [Reactive documentation](https://www.reactiveui.net/docs/).

For [DynamicData](https://github.com/reactivemarbles/DynamicData), here is a [collection of Snippets](https://github.com/RolandPheasant/DynamicData.Snippets) to achieve various tasks.

For support, you can get help on the [Reactive Slack channel](https://reactivex.slack.com).

[> Next: Deployment](8_Deployment.md)
