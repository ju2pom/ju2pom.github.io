---
layout: post
title: 6 years of WPF, Part 1
date:   2017-01-27 03:00:08
categories: Wpf
comments: false
---
# Context

I started working with WPF/Xaml about 6 years ago with good background in C# and WinForms.
Today I must admit I have mixed feeling about Xaml: I love doing it, but I <i>hate </i>how it's done.
Anyway the goal of this article is to share my experience and talk about the best practices we have adopted in my team.

*This article is not written for total beginner but rather for developers with an average knowledge in WPF/Xaml*

I can't share 6 years of WPF in a single post so this one is the first of a serie. Here I'll stick with MVVM and upcoming posts will cover:
1. Bindings, Converters
2. ControlTemplate, DataTemplate and Style
3. Behaviors and Attached properties

# MVVM
MVVM is a big topic, but here I'll try to focus on its impact on how Xaml code. So here are the pros and cons you'll see using this pattern with WPF

| Pros | Cons |
| --- | --- |
| Keep UI code and behavior/logic well separated | Xaml code is more complex |
| Makes your code testable | Some behaviors are more difficult to implement |
| Increase code reusability |  |

To help you on this path there are plenty of libraries on the net. Among them *MVVM Light toolkit* which we use in my team.
Let's go deeper and start talking about code now. Practically what does it mean to implement MVVM and how those library can help you?

1. Don't use this kind of code:

```xml
<Button GotFocus="OnButtonGotFocus" />
```

Using this kind of code means that you are going to implement the logic inside the view code (in code behind). Prefer using a great addition from your preferred MVVM toolkit:

```xml
<Button>
  <i:Interaction.Triggers>
    <i:EventTrigger EventName="GotFocus">
      <mvvlight:EventToCommand
        PassEventArgsToCommand="False"
        Command="{Binding DoSomethingCommand}"
        />
    </i:EventTrigger>
  </i:Interaction.Triggers>
</Button>
```

The binding you see here is the how you transfert the logic responsibility to the viewmodel which implements something like this:

```csharp
protected MyViewModel()
{
  this.DoSomethingCommand = new RelayCommand(this.DoSomething);
}

public ICommand DoSomethingCommand { get; private set; }

private void DoSomething(object parameter)
{
  ...
}
```

2. As a general rule of thumb, try to keep the code behind (xaml.cs file) as empty as possible. You'll often think there is no way to avoid writing code behind, but give it a second though because it's less often that you think !
3. If you need to make the viewmodel communicate with the view, you should do it without referencing the view in anyway (and avoid referencing the viewmodel in the view). Our best practice here is to use the Messenger pattern. Let's say you want to open a window according to some logic. The problem here is that the logic in the viewmodel should not reference a view. Here is how you can do it:

```csharp
public class ColorBoxViewModel
{
  ...

  private void ShowColorDialog(object obj)
  {
    ColorDialogMessage dialogMessage = new ColorDialogMessage(this, this.OnColorSelected);
    dialogMessage.Dialog.AllowFullOpen = true;
    dialogMessage.Dialog.FullOpen = true;
    dialogMessage.Dialog.AnyColor = true;
    dialogMessage.Dialog.Color = this.toolColor.Color.ToDrawingColor();
    dialogMessage.Dialog.CustomColors = this.GetCustomColors();

    Messenger.Default.Send(dialogMessage, this);
  }
}
```

And in the view:

```csharp
public ColorBox()
{
  Messenger.Default.Unregister<ColorDialogMessage>(this, this.ShowColorDialog);
}

private void ShowColorDialog(ColorDialogMessage message)
{
  ...
}
```

# XAML Formatting
To me Xaml is a quite complex UI description language that needs a lot of attention because its syntax is quite cumbersome. So if code formatting can raise infinite debates it's very important for xaml to follow a few simple rules.
1. This one is a tip I had from a colleague of mine. Instead of this:

```xml
<!-- Overall progress bar control -->
<ProgressBar 
  x:Name="OverallProgress"
  Grid.Row="1"
  Height="20"
  Value="{Binding Path=GlobalProgression, Mode=OneWay}"
  />
```
  
  Prefer this:
  
```xml
  <ProgressBar x:Name="OverallProgress"
             Grid.Row="1"
             Height="20"
             Value="{Binding Path=GlobalProgression, Mode=OneWay}"
             />
```

This way you can read the name of the control even when the element is collapsed which makes the comment useless !
2. As shown in the samples above it's also nice to keep a single parameter on each line because it's easier to read, makes shorter lines and ease merging task.
3. I also like to post-fix resources name with the type of resource (UsersComboboxStyle, AutoHideScrollBarTemplate, DarkColorBrush ....) because you often end up having several resources relative to the same element.
4. We often try many settings before finding the expected design which sometimes leads to properties that are set to their default values. Xaml is a verbose language by design, so take the time to remove useless definitions (default margin, alignments ...)
5. If you want to change the default style of a particular control type create a resource file with the following style:

```xml
<Style TargetType="{x:Type Button}">
  <Setter Property="FontSize" Value="{StaticResource H2FontSize}" />
  <Setter Property="FontWeight" Value="{StaticResource H2FontWeight}" />
  <Setter Property="Template" Value="{StaticResource StandardButtonTemplate}" />
</Style>
```

Now add the following piece of code in your App.xaml file:

```xml
<Application.Resources>
  <ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
      <ResourceDictionary Source="pack://application:,,,/Assembly-name;component/your-resource-file.xaml" />
        </ResourceDictionary.MergedDictionaries>
  </ResourceDictionary>
</Application.Resources>
```

And all your application's button that don't specify any style will use your customized style !
6. Xaml resources management is a nightmare if not done properly. What we have found very practical is to create an assembly dedicated to xaml resources. See the screenshot below:

![theme.jpg](/images/theme.jpg)

And in this same assembly you should put an additional resource file (let's call it theme.xaml) with the following content (this is an exemple):

```xml
<ResourceDictionary
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  >

  <!--
    Dummy style to prevent dictionary optimization.
    This flags the dictionary as containing default styles.
  -->
  <Style TargetType="{x:Type Viewport3D}" />

  <ResourceDictionary.MergedDictionaries>
    <ResourceDictionary Source="Colors/Colors.xaml" />
    <ResourceDictionary Source="Controls/Rating.xaml" />
    <ResourceDictionary Source="Controls/Button.xaml" />
    <ResourceDictionary Source="Controls/TextBlock.xaml" />
    <ResourceDictionary Source="Controls/TextBox.xaml" />
    <ResourceDictionary Source="Controls/CheckBox.xaml" />
    <ResourceDictionary Source="Controls/ComboBox.xaml" />
    <ResourceDictionary Source="Controls/Expander.xaml" />
    <ResourceDictionary Source="Controls/ProgressBar.xaml" />
    <ResourceDictionary Source="Controls/RadioButton.xaml" />
    <ResourceDictionary Source="Controls/ScrollBar.xaml" />
    <ResourceDictionary Source="Controls/Slider.xaml" />
    <ResourceDictionary Source="Controls/TabItem.xaml" />
    <ResourceDictionary Source="Controls/ToggleButton.xaml" />
  </ResourceDictionary.MergedDictionaries>

</ResourceDictionary>
```

Then in the WPF application merge this "theme.xaml" with the application's resource dictionary (in App.xaml). This will allow you to use any resource declared in your theme assembly as static resource. Sadly it will work at runtime but not in Visual Studio Xaml designer !

```xml
<Application.Resources>
  <ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
      <ResourceDictionary Source="pack://application:,,,/Theme-assembly-name;component/Main.xaml" />
    </ResourceDictionary.MergedDictionaries>
  </ResourceDictionary>
</Application.Resources>
```

:point_right: [Jump to Part2](https://ju2pom.github.io/6-years-of-WPF-part2)
