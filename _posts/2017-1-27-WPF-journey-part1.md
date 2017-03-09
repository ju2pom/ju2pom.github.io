---
layout: post
title: WPF journey, Part 1
date:   2017-01-27 03:00:08
categories: Wpf
comments: false
description: WPF, C#, .NET, Xaml, MVVM
---
# Context

I started working with WPF/Xaml about 6 years ago with good background in C# and WinForms.
Today I must admit I have mixed feeling about Xaml: I love doing it, but I <i>hate </i>how it's done.
Anyway the goal of this article is to share my experience and talk about the best practices we have adopted in my team.

*This article is not written for total beginner but rather for developers with an average knowledge in WPF/Xaml*

I will share my WPF journey through several post to cover the following topics:

1. MVVM and Xaml formatting
2. Bindings, Converters
3. ControlTemplate, DataTemplate and Style
4. Behaviors and Attached properties

# MVVM
MVVM is a big topic, but here I'll try to focus on its impact on how to write Xaml code. So here are the pros and cons you'll see using this pattern with WPF

| Pros | Cons |
| --- | --- |
| Keep UI code and behavior/logic well separated | Xaml code is more complex/verbose |
| Makes your code testable | Some behaviors are more difficult to implement |
| Increase code reusability |
| Keeps business logic separated from UI |

To help you on this path there are plenty of libraries on the net. Among them [MVVM Light toolkit](http://www.mvvmlight.net/) which we use in my team.
Let's go deeper and start talking about code now. Practically what does it mean to implement MVVM and how those libraries can help you, what are the good practices?

- Don't use this kind of code:

```xml
<Button GotFocus="OnButtonGotFocus" />
```

Using this kind of code means that you are going to implement the logic inside the view code (in code behind). Prefer using *EventToCommand* available in most MVVM libraries:

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

The binding you see above is the how you transfert the logic responsibility from the view to the viewmodel. Then the viewmodel should implements something like this:

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

- As a general rule of thumb, try to keep the code behind (xaml.cs file) as empty as possible. You'll often think there is no way to avoid writing code behind, but give it a second though because it's less often that you think !
- If you need to make the viewmodel communicate with the view, you should do it without referencing the view in anyway (and avoid referencing the viewmodel in the view). Our best practice here is to use the Messenger pattern. Let's say you want to open a window according to some logic. The problem here is that the logic in the viewmodel should not reference a view. Here is how you can do it, in the viewmodel:

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
  Messenger.Default.Register<ColorDialogMessage>(this, this.ShowColorDialog);
}

private void ShowColorDialog(ColorDialogMessage message)
{
  ...
}
```

- Don't forget to cleanup viewmodels, especially if you register to events, that's the good place to unregister (to avoid memory leaks)
- Keep your viewmodels small and prefer aggragating sub-viewmodels
- *ObservableCollection<T>* is good friend when dealing with collections. Bind it to a *CollectionViewSource* in the view and you have updates, filtering and sorting features build-in !

In the viewmodel:
```csharp
public ObservableCollection<User> Users
{
  get { return this.users; }
}
```

In the view:

```xml
<UserControl.Resources>
  <CollectionViewSource x:Key="cvs"
    Source="{Binding Path=Users}"
    />
</UserControl.Resources>

<ListBox
  ItemsSource="{Binding Source={StaticResource cvs}}"
  />
```
      
# XAML Formatting
To me Xaml is a quite complex UI description language that needs a lot of attention because its syntax is quite cumbersome. So if code formatting can raise infinite debates it's very important for xaml to follow a few simple rules.

- Naming convention: this one is a tip I had from a colleague of mine, it will allow you to see a control name even when it's collapsed in the code editor !

Instead of this:

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

- As shown in the samples above it's also nice to keep a single parameter on each line because it's easier to read, makes shorter lines and ease merging task.
- I also like to post-fix resources name with the type of resource (UsersComboboxStyle, AutoHideScrollBarTemplate, DarkColorBrush ....) because you often end up having several resources relative to the same element.
- We often try many settings before finding the expected design which sometimes leads to properties that are set to their default values. Xaml is a verbose language by design, so take the time to remove useless definitions (default margin, alignments ...)
- If like me you never use the WPF designer in Visual Studio set the code editor as the default editor for xaml files:
  - right click on a xaml file
  - select "*Open Width"*
  - select "*Source Code (Text) Editor
  - and click on the "*Set as Default*" button

[Jump to Part2](/WPF-journey-part2)
