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

## 2. Avalonia UI

Avalonia is a modern WPF-like UI that works on all OS: Windows, Linux, MacOS, iOS, Android and even Web Assembly! It is mature enough for commercial use.

First, install the plugin in your IDE. [Instructions here.](https://docs.avaloniaui.net/docs/getting-started/ide-support/jetbrains-rider-setup)

### Resources

- [Main website](https://avaloniaui.net/)
- [GitHub source code](https://github.com/AvaloniaUI/Avalonia)
- [Documentation](https://docs.avaloniaui.net/)
- [FluentAvalonia](https://github.com/amwx/FluentAvalonia) - Better UI
- [AwesomeAvalonia](https://github.com/AvaloniaCommunity/awesome-avalonia) - List of 3rd party tools

The documentation is still limited, but the pages are very well-written. Take the time to read the various sections. It is very similar to WPF, with some differences that are great improvements. It's a lot easier to translate WPF to Avalonia than the other way around. For undocumented classes, you can often refer to WPF and UWP documentation that will be very similar.

For more help:
- [GitHub Avalonia Discussions](https://github.com/AvaloniaUI/Avalonia/discussions)
- [GitHub Avalonia Issues](https://github.com/AvaloniaUI/Avalonia/issues)
- [StackOverflow](https://stackoverflow.com/)
- [Discord DotNetEvolution](https://discord.com/invite/HSuhTyG)

### XAML

Do not pay attention to the code-behind aspect; that will be covered in the MVVM section. What you need to learn is the XAML language and creating styles and resources. Practice creating some great UIs that do nothing.

One limitation is that you get no visual editor and instead only get a visual preview. This forces you to write all the XAML manually. As I found out, this is a good thing. My WPF code was all built with absolute coordinates and large margins to position the controls. That's not how layouts should be done. `StackPanel` is your best friend for layouts, so use that instead.

Also use `Grid` with rows and columns. Margins should be used only to control spacing between controls, not to position the controls in the window.

```xaml
<Grid Margin="10,6,10,10" ColumnDefinitions="150,*" RowDefinitions="*,40">
```

Rewrite your auto-generated WPF code with StackPanel and grids and you'll get much cleaner code.

### FluentAvalonia

[FluentAvalonia](https://github.com/amwx/FluentAvalonia) provides better styles and more controls, so you may want to use that too. There are no build-in MessageBox in Avalonia, and FluentAvalonia provides that.

Download their source code and run their sample app to view all the controls in action. The sample application also allows you to browse all style resources and active colors, and is thus a very useful tool.

### Icons

[Svg.Skia](https://github.com/wieslawsoltes/Svg.Skia) is an option for SVG icons; but it cannot adapt the icon color dynamically.

FluentAvalonia provides `SymbolIcon` with build-in icons for most of your needs. Download their source code and run their sample app to view them.

```xaml
<flu:SymbolIcon Symbol="Add" FontSize="20" />
```

If you need icons that are not in SymbolIcon, there is a way to convert SVG to a Path, but that can get complicated, and Path requires you to set a fixed Brush so it won't adapt to the button Forecolor.

I find it easier to create a font containing the custom icons.

[Here's a simple guide on creating an icons font using FontForge.](https://mohammedraji.github.io/posts/The-Definitive-guide-to-create-an-icon-font/)

My button then looks like this
```c#
<Button Classes="icon" Width="35" Content="I" />
```

With `icon` defined as

```xaml
<Style Selector="Button.icon">
    <Setter Property="FontFamily" Value="avares://Common.Avalonia.App/Styles/HanumanInstituteAppIcons.otf#" />
    <Setter Property="FontSize" Value="17" />
</Style>
```

### Sample Code

[Here are some sample Views created using these principles.](https://github.com/mysteryx93/HanumanInstituteApps/tree/master/Converter432hz/Views)
I've created custom [Styles](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Common.Avalonia.App/Styles/CommonStyles.axaml) and [Resources](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Common.Avalonia.App/Styles/CommonResources.axaml) that are registered in [App.xaml](https://github.com/mysteryx93/HanumanInstituteApps/blob/master/Converter432hz/App.axaml).

The app then works great with Light, Dark and HighContrast themes that can be set on-the-fly. The styles add a background gradient, changes the style of buttons to look like the FluentAvalonia 'accent' style with 35% opacity, alters the ListBox so that double-click covers the whole item area, and adjusts a few details for my needs.

### Visual Debugging

Setting and altering styles can be difficult, as you don't know how controls are internally structured.

First, run your app and press F12 to open the Avalonia DevTools. Put the tools window on the right-side of the screen and your app on the left. Click on the Visual Tree tab, hover over the control that you want to analyze, and press CTRL+SHIFT. It will browse to that element in the visual tree and you can see what attributes are in effect.

Also, you can browse the source styles, [Avalonia.Themes.Fluent](https://github.com/AvaloniaUI/Avalonia/tree/master/src/Avalonia.Themes.Fluent) if using the built-in styles, or [FluentAvalonia](https://github.com/amwx/FluentAvalonia/tree/master/FluentAvalonia/Styling).

I found out that some styles are set in FluentAvalonia as `Height="32"` or `Padding="2"` directly on the controls without using resources; those cannot be oveerriden at all unless you replace the whole template as [explained in this ticket](https://github.com/amwx/FluentAvalonia/issues/176).

### Code-Behind

Finally, here's what my code-behind looks like. I want to keep it that way. You might wonder how I achieve a fully-functional app with no code-behind whatsoever. I'll cover that later.

```c#
public partial class MainView : CommonWindow<MainViewModel>
{
    protected override void Initialize() => AvaloniaXamlLoader.Load(this);
}
```

In the XAML, 'd' prefix allows setting design-time properties. This allows setting a DataContext for the designer. We'll create the ViewModelLocator class in the next section.

```xaml
d:DataContext="{x:Static local:ViewModelLocator.Main}"
```

### Create Your Project

There are two project templates available: desktop-only or cross-platform. Either way, I highly recommend going with Avalonia v11 even if it is in preview.

For desktop-only project, [follow these instructions to create a "Avalonia MVVM Application".](https://docs.avaloniaui.net/tutorials/todo-list-app/creating-a-new-project)

For cross-platform project, [follow these instructions.](https://docs.avaloniaui.net/tutorials/developing-for-mobile/create-a-cross-platform-solution)

Make sure that you target the right framework, that Nullables are enabled, and that you use the latest language features, by having this in your `csproj`.

```
<TargetFramework>net7.0</TargetFramework>
<Nullable>enable</Nullable>
<LangVersion>default</LangVersion>
```

For code styles, if you want to use the most recommended coding practices, it is safe to grab the `.editorconfig` file from the .NET Roslyn Compiler project [here](https://github.com/dotnet/roslyn/blob/main/.editorconfig). Place it in your solution folder to be available for all projects.

Also make sure to create a file `GlobalUsings.cs` that contains namespaces that you commonly use.

```c#
global using System;
global using System.Collections;
global using System.Collections.Generic;
global using System.Threading.Tasks;
global using RxCommandUnit = ReactiveUI.ReactiveCommand<System.Reactive.Unit, System.Reactive.Unit>;
global using static System.FormattableString; 
```

### App.xaml.cs

Put this in App.xaml.cs

As of writing this, there's a bug where XAML namespaces of custom libraries do not get recognized by the designer. You must use `GC.KeepAlive` on any class within such libraries to solve this.

Also, I found out that when using the designer, there is no point in initializing your MainView and application; you can skip app initialization and the preview will show faster.

```c#
public override async void OnFrameworkInitializationCompleted()
{
#if DEBUG
    // Required by Avalonia XAML editor to recognize custom XAML namespaces. Until they fix the problem.
    GC.KeepAlive(typeof(SvgImage));
    GC.KeepAlive(typeof(EventTriggerBehavior));

    if (Design.IsDesignMode)
    {
        base.OnFrameworkInitializationCompleted();
        return;
    }
#endif
```

[> Next: Dependency Injection](3_DependencyInjection.md)
