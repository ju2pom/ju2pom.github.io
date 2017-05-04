---
layout: post
title: Pure xaml staring control
date:   2014-09-29 03:00:08
categories: Xaml
comments: true
description: WPF, C#, .NET, Xaml, Rating, Kaxaml
---
One of the best feature in wpf/xaml is ability to completely redesign a built-in control without any line of code.
Most of the time you will just define a custom theme for a selection of useful controls.
But it becomes more interesting when "hacking" an existing control to make something new.

In my day work I had to make a star rating control and I thought it would be interesting to reuse the slider control for that purpose.
I also wanted it to be pure xaml.
I use a shape proposed by [Kiran Kumar](http://kenkumar.blogspot.fr/2011/04/xaml-code-for-displaying-star.html) which I slightly modified.
Here is a visual (nothing really surprising though):

![screenshot](/images/star-ratings.png)

If you want to try, simply copy/paste the following piece of xaml to your favorite xaml editor (I used [Kaxaml](http://www.kaxaml.com/), which is a great tool):

```xml  
<Page
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Background="#2d2d20">
  <Page.Resources>
    <SolidColorBrush x:Key="NoStar" Color="#29ffffff" />
    <SolidColorBrush x:Key="Star" Color="Yellow" />

    <Style
      x:Key="StarRatingShapeStyle"
      TargetType="Polygon">
      <Setter Property="Width" Value="12" />
      <Setter Property="Height" Value="12" />
      <Setter Property="Fill" Value="{StaticResource NoStar}" />
      <Setter Property="Points" Value="9,2 11,7 17,7 12,10 14,15 9,12 4,15 6,10 1,7 7,7" />
      <Setter Property="Stretch" Value="Fill" />
      <Setter Property="Stroke" Value="White" />
      <Setter Property="StrokeLineJoin" Value="Round" />
      <Setter Property="StrokeThickness" Value=".5" />
      <Setter Property="IsHitTestVisible" Value="False" />
    </Style>

    <Style x:Key="RatingSliderRepeatButtonStyle" TargetType="{x:Type RepeatButton}">
      <Setter Property="Padding" Value="0" />
      <Setter Property="Margin" Value="0" />
      <Setter Property="Background" Value="Transparent" />
      <Setter Property="Template">
        <Setter.Value>
          <ControlTemplate>
            <Border
              Background="{TemplateBinding Background}"
              BorderThickness="0"
              />
          </ControlTemplate>
        </Setter.Value>
      </Setter>
    </Style>

    <Style x:Key="RatingSliderThumbStyle" TargetType="{x:Type Thumb}">
      <Setter Property="OverridesDefaultStyle" Value="true" />
      <Setter Property="Focusable" Value="False" />
      <Setter Property="Padding" Value="0" />
      <Setter Property="Margin" Value="0" />
      <Setter Property="Background" Value="Transparent" />
      <Setter Property="Template">
        <Setter.Value>
          <ControlTemplate TargetType="{x:Type Thumb}">
            <Border
              Background="{TemplateBinding Background}"
              BorderThickness="0"
              />
          </ControlTemplate>
        </Setter.Value>
      </Setter>
    </Style>

    <Style
      x:Key="StarRatingSliderStyle"
      TargetType="Slider">
      <Setter Property="Minimum" Value="0" />
      <Setter Property="Maximum" Value="5" />
      <Setter Property="SmallChange" Value="1" />
      <Setter Property="LargeChange" Value="1" />
      <Setter Property="TickFrequency" Value="1" />
      <Setter Property="IsSnapToTickEnabled" Value="True" />
      <Setter Property="IsMoveToPointEnabled" Value="True" />
      <Setter Property="Template">
        <Setter.Value>
          <ControlTemplate
            TargetType="Slider">
            <Grid>
              <Track
                x:Name="PART_Track"
                >
                <Track.DecreaseRepeatButton>
                  <RepeatButton
                    Style="{StaticResource RatingSliderRepeatButtonStyle}"
                    Command="{x:Static Slider.DecreaseLarge}"
                    />
                </Track.DecreaseRepeatButton>
                <Track.IncreaseRepeatButton>
                  <RepeatButton
                    Style="{StaticResource RatingSliderRepeatButtonStyle}"
                    Command="{x:Static Slider.IncreaseLarge}"
                    />
                </Track.IncreaseRepeatButton>
                <Track.Thumb>
                  <Thumb
                    x:Name="PART_Thumb"
                    Style="{StaticResource RatingSliderThumbStyle}"
                    />
                </Track.Thumb>
              </Track>

              <StackPanel
                Orientation="{TemplateBinding Orientation}">
                <Polygon
                  x:Name="S5"
                  Style="{StaticResource StarRatingShapeStyle}"
                  />
                <Polygon
                  x:Name="S4"
                  Style="{StaticResource StarRatingShapeStyle}"
                  />
                <Polygon
                  x:Name="S3"
                  Style="{StaticResource StarRatingShapeStyle}"
                  />
                <Polygon
                  x:Name="S2"
                  Style="{StaticResource StarRatingShapeStyle}"
                  />
                  <Polygon
                  x:Name="S1"
                  Style="{StaticResource StarRatingShapeStyle}"
                  />
              </StackPanel>
            </Grid>

            <ControlTemplate.Triggers>
              <Trigger Property="Value" Value="1">
                <Setter TargetName="S1" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S2" Property="Fill" Value="{StaticResource NoStar}"/>
                <Setter TargetName="S3" Property="Fill" Value="{StaticResource NoStar}"/>
                <Setter TargetName="S4" Property="Fill" Value="{StaticResource NoStar}"/>
                <Setter TargetName="S5" Property="Fill" Value="{StaticResource NoStar}"/>
              </Trigger>
              <Trigger Property="Value" Value="2">
                <Setter TargetName="S1" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S2" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S3" Property="Fill" Value="{StaticResource NoStar}"/>
                <Setter TargetName="S4" Property="Fill" Value="{StaticResource NoStar}"/>
                <Setter TargetName="S5" Property="Fill" Value="{StaticResource NoStar}"/>
              </Trigger>
              <Trigger Property="Value" Value="3">
                <Setter TargetName="S1" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S2" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S3" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S4" Property="Fill" Value="{StaticResource NoStar}"/>
                <Setter TargetName="S5" Property="Fill" Value="{StaticResource NoStar}"/>
              </Trigger>
              <Trigger Property="Value" Value="4">
                <Setter TargetName="S1" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S2" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S3" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S4" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S5" Property="Fill" Value="{StaticResource NoStar}"/>
              </Trigger>
              <Trigger Property="Value" Value="5">
                <Setter TargetName="S1" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S2" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S3" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S4" Property="Fill" Value="{StaticResource Star}"/>
                <Setter TargetName="S5" Property="Fill" Value="{StaticResource Star}"/>
              </Trigger>
            </ControlTemplate.Triggers>
          </ControlTemplate>
        </Setter.Value>
      </Setter>
    </Style>
  </Page.Resources>
  <StackPanel>
    <Slider
      x:Name="Ranking"
      Orientation="Vertical"
      HorizontalAlignment="Center"
      Background="Transparent"
      Style="{StaticResource StarRatingSliderStyle}"
      />
    <Slider
      x:Name="Test"
      Orientation="Horizontal"
      Minimum="0"
      Maximum="5"
      Width="200"
      Height="200"
      IsSnapToTickEnabled="True"
      TickFrequency="1"
      Value="{Binding Value, ElementName=Ranking}"
      />
  </StackPanel>
</Page>
```

This xaml implementation is not completely usable because of two small issues:

Depending on where you click on a star (above or below the middle) you got different behaviors. But it could be easily solved with a converter that would ceil or floor the value.
Horizontal and vertical orientation is handled, but in horizontal orientation the order is reversed. I guess I should not use the Stackpanel orientation but rather a LayoutTransform.
