# FluentWPF

FluentWPF is a library which offers an alternative Xaml.

# Why?
- Reduce verbosity
- Provide more flexibility
- Improve reusability
- Leverage tools and C# langage capabilities
- Ease unit testing

# How
- Provide a fluent API
- Make this library easily extensible
- Reuse existing WPF types
- Provide many sample usages

# What
Let's see what it looks like with some piece of code.

## Simple window
```csharp
new Window()
  .AsFluent<Window>()
  .DataContext(new RootViewModel())
  .NoBorder()
  .NoResize()
```

I hope the code is pretty straitforward but let's write the xaml equivalent:
```xml
<Window
  x:Class="SomeNameSpace.MyWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  xmlns:local="clr-namespace:SomeNameSpace.ViewModels"
  ResizeMode="NoResize"
  WindowStyle="None"
  >
  <Window.DataContext>
   <local:RootViewModel />
  </Window.DataContext>
```

## Alignment is easy

```csharp
new Expander()
  .AsFluent()
  .Top()
  .Size(400, 400)
```

Indeed it's as easy as to align `Left` or `Right` or `Center` or `Bottom`.

## Set a property

```csharp
new TextBlock()
  .AsFluent()
  .Set(FrameworkElement.MarginProperty, new Thickness(4))
  .Set(TextBlock.TextProperty, "Some text"))
```

## Create a grid and populate it

```csharp
new Grid()
  .AsFluent()
  .DefaultCellSize("*", "*")
  .Cell(GridCellExtensions.Create()
    .Contains(new Button()
      .AsFluent()
      .Contains("Button1")))
  .Cell(GridCellExtensions.Create()
    .Column(1)
    .Contains(new Button()
      .AsFluent()
      .Contains("Button2")))
```

The grid API is meant to ease dealing with grids. For exemple you don't need to first declare rows and columns.
And you can define default width and heigh and specify a specific value for any cell if wanted.
The sample above only demonstrate a very small part of the Grid API

## Experiment with bindings

```csharp
new Button()
      .AsFluent()
      .Contains("Button1")
      .Bind(BindingExtensions
        .OneTime(ButtonBase.CommandProperty)
        .With(nameof(RootViewModel.SimpleCommand)))
```


## Simple style creation

```csharp
var style = StyleExtensions.Create()
    .Set(Control.ForegroundProperty, colors.Text.Normal)
    .When(TriggerExtensions.Property(ToggleButton.IsCheckedProperty)
      .Is(true)
      .Then(Control.FontWeightProperty, FontWeights.Bold)
      .Then(Control.ForegroundProperty, colors.Control.Selected))
    .When(TriggerExtensions.Property(UIElement.IsMouseOverProperty)
      .Is(true)
      .Then(Control.BackgroundProperty, colors.Control.Over))
    .When(TriggerExtensions.Property(FrameworkElement.TagProperty)
      .Is("test")
      .Then(ContentControl.ContentProperty, null))
    .Template(template)
    .AsStyle<CheckBox>();
```

Control template creation API
```csharp
var template = TemplateExtensions.Create<Border>()
    .TemplateBinding(Control.BackgroundProperty, Control.BackgroundProperty)
    .TemplateBinding(Control.BorderBrushProperty, Control.BorderThicknessProperty)
    .TemplateBinding(Control.BorderThicknessProperty, Control.BorderThicknessProperty)
    .Contains<StackPanel>()
    .Set(StackPanel.OrientationProperty, Orientation.Horizontal)
    .Contains(TemplateExtensions.Create<Ellipse>()
      .Set(FrameworkElement.WidthProperty, 8d)
      .Set(FrameworkElement.HeightProperty, 8d)
      .TemplateBinding(Shape.FillProperty, Control.ForegroundProperty))
    .Contains(TemplateExtensions.Create<ContentPresenter>()
      .Set(FrameworkElement.HorizontalAlignmentProperty, HorizontalAlignment.Center)
      .Set(FrameworkElement.VerticalAlignmentProperty, VerticalAlignment.Center)
      .TemplateBinding(ContentPresenter.ContentProperty, ContentControl.ContentProperty))
    .AsControlTemplate<CheckBox>();
```
# FluentWPF vs Xaml:

*FluentWPF*

```csharp
var TemplateExtensions.Create<Border>()
    .TemplateBinding(Control.BackgroundProperty, Control.BackgroundProperty)
    .Contains<ContentPresenter>()
    .Set(FrameworkElement.HorizontalAlignmentProperty, HorizontalAlignment.Center)
    .Set(FrameworkElement.VerticalAlignmentProperty, VerticalAlignment.Center)
    .TemplateBinding(ContentPresenter.ContentProperty, ContentControl.ContentProperty)
    .AsControlTemplate<CheckBox>();

var NiceCheckBox = StyleExtensions.Create()
    .When(TriggerExtensions.Property(ToggleButton.IsCheckedProperty)
      .Is(true)
      .Then(Control.FontWeightProperty, FontWeights.Bold)
      .Then(Control.ForegroundProperty, new SolidColorBrush(Colors.DarkBlue)))
    .When(TriggerExtensions.Property(UIElement.IsMouseOverProperty)
      .Is(true)
      .Then(Control.BackgroundProperty, new SolidColorBrush(Colors.Bisque)))
    .Template(template)
    .AsStyle<CheckBox>();
```    
*XAML*

```xml
<ControlTemplate x:Key="NiceCheckBoxTemplate" TargetType="CheckBox">
  <Border x:Name="Border"
    Background="{TemplateBinding Background}">
    <ContentPresenter
      HorizontalAlignment="Center"
      VerticalAlignment="Center"
      />
  </Border>
</ControlTemplate>

<Style x:Key="NiceCheckBox" TargetType="CheckBox">
  <Setter Property="Template" Value="{StaticResource NiceCheckBoxTemplate}">
  <Style.Triggers>
    <Trigger Property="IsChecked" Value="True">
      <Setter Property="FontWeight" Value="Bold" />
      <Setter Property="Foreground" Value="DarkBlue" />
    </Trigger>
    <Trigger Property="IsMouseOver" Value="True">
      <Setter Property="Background" Value="Bisque" />
    </Trigger>
  </Style.Triggers>
</Style>
  ```
  
# Roadmap
- Cover same functionalities as XAML
  - Support animations
  - Support triggers
  - Support behaviors
- Write samples
- Offer a code friendly Theme engine
- More samples
- Provide Visual studio code snippets
- API documention (or more samples)
- Extend the API functionalities beyond XAML
  - Leverage lambdas (for triggers?)
  - Support inheritance/mixins for styles
- Further reduce API verbosity
