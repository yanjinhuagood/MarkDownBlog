<span style="display:block;text-align:center;">  **一招搞定！轻松优雅地关闭 TabControl 的 Tab 页**</span> 
> 控件名：TabControl 
>
> 作   者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers

- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;



##### 逻辑实现
  在本篇将介绍如何在 `TabControl` 中为每个 `TabItem` 添加一个关闭按钮。将使用一个附加属性来控制关闭按钮的显示和隐藏。通过自定义 `ControlTemplate`，可以为 `Tab` 页提供关闭操作。
###### **TabItem** 逻辑如下
- 在每个 `TabItem` 的右侧添加一个`关闭`按钮。
- 使用附加属性来控制关闭按钮的`显示`和`隐藏`。

###### 1. 定义 **TabItem** 样式
  - 通过 `XAML` 中的样式为 `TabItem` 设置外观，并添加一个`关闭按钮`。
~~~xml
<Style x:Key="WD.BaseTAndBTabItem"
       BasedOn="{StaticResource WD.ControlBasicStyle}"
       TargetType="{x:Type TabItem}">
    <Setter Property="HorizontalContentAlignment" Value="Stretch" />
    <Setter Property="VerticalContentAlignment" Value="Stretch" />
    <Setter Property="Background" Value="Transparent" />
    <Setter Property="Padding" Value="{StaticResource WD.DefaultPadding}" />
    <Setter Property="BorderThickness" Value="{StaticResource WD.TabItemBorderThickness}" />
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="{x:Type TabItem}">
                <controls:SmallPanel Width="{TemplateBinding Width}"
                                     Height="{TemplateBinding Height}"
                                     Background="{TemplateBinding Background}">
                    <Border x:Name="PART_Border" BorderThickness="{TemplateBinding BorderThickness}" />
                    <Grid>
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition />
                            <ColumnDefinition Width="Auto" />
                        </Grid.ColumnDefinitions>
                        <ContentPresenter x:Name="PART_ContentPresenter"
                                          Margin="{TemplateBinding Padding}"
                                          HorizontalAlignment="{Binding Path=HorizontalContentAlignment, RelativeSource={RelativeSource AncestorType={x:Type ItemsControl}}}"
                                          VerticalAlignment="{Binding Path=VerticalContentAlignment, RelativeSource={RelativeSource AncestorType={x:Type ItemsControl}}}"
                                          ContentSource="Header"
                                          SnapsToDevicePixels="{TemplateBinding SnapsToDevicePixels}" />
                        <!-- 关闭按钮 -->
                        <Button x:Name="PART_CloseButton"
                                Grid.Column="1"
                                Width="20"
                                Height="20"
                                Margin="0,0,4,0"
                                helpers:ElementHelper.CornerRadius="4"
                                helpers:ElementHelper.IsClear="{Binding Path=(helpers:ElementHelper.IsClear), RelativeSource={RelativeSource TemplatedParent}}"
                                Style="{StaticResource WD.NormalButton}"
                                ToolTip="{Binding [Close], Source={x:Static resx:LanguageManager.Instance}}"
                                Visibility="Collapsed">
                            <controls:PathIcon Width="10"
                                               Height="10"
                                               Foreground="{DynamicResource WD.SecondaryTextSolidColorBrush}"
                                               Kind="WindowClose" />
                        </Button>
                    </Grid>
                </controls:SmallPanel>
                <ControlTemplate.Triggers>
                    <!-- 选中状态触发器 -->
                    <Trigger Property="IsSelected" Value="True">
                        <Setter TargetName="PART_Border" Property="BorderBrush" Value="{DynamicResource WD.PrimaryNormalSolidColorBrush}" />
                        <Setter Property="Background" Value="{DynamicResource WD.DefaultBackgroundSolidColorBrush}" />
                        <Setter TargetName="PART_ContentPresenter" Property="TextElement.Foreground" Value="{DynamicResource WD.PrimaryNormalSolidColorBrush}" />
                    </Trigger>
                    <!-- 鼠标悬停状态触发器 -->
                    <Trigger Property="IsMouseOver" Value="True">
                        <Setter TargetName="PART_ContentPresenter" Property="TextElement.Foreground" Value="{DynamicResource WD.PrimaryNormalSolidColorBrush}" />
                    </Trigger>
                    <!-- 控制是否显示关闭按钮的触发器 -->
                    <Trigger Property="helpers:ElementHelper.IsClear" Value="True">
                        <Setter TargetName="PART_CloseButton" Property="Visibility" Value="Visible" />
                    </Trigger>
                </ControlTemplate.Triggers>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>

~~~

###### 2. 使用附加属性来控制关闭按钮
- 1. 定义附加属性 `IsClear`
```C#
public static readonly DependencyProperty IsClearProperty =
   DependencyProperty.RegisterAttached("IsClear", typeof(bool), typeof(ElementHelper),
       new PropertyMetadata(false, OnIsClearChanged));
```

- 2. `OnIsClearChanged` 方法
- 判断 `IsClear` 的新值 `e.NewValue` 是否为 `true`（按钮是否应当具有关闭功能）。

- 如果 `true`，则为 `Button` 添加 `Click` 事件处理器 `ButtonClear_Click`，即点击该按钮时将触发关闭功能。
- 如果 `false`，则移除 `Click` 事件处理器，按钮不再响应点击事件。
```C#
private static void OnIsClearChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
{
    var button = d as Button;
    if (button != null)
    {
        if ((bool)e.NewValue)
            button.Click += ButtonClear_Click;
        else
            button.Click -= ButtonClear_Click;
    }
}
 
```
- 3. `ButtonClear_Click` 事件处理器
- 首先，检查 `sender` 是否是 `Button` 类型。
- 然后，通过 `button.TemplatedParent` 获取按钮的模板父元素，通常在这里是 `TabItem`。
- 接下来，获取 `TabItem` 的父控件，应该是 `TabControl`。如果 `tabControl` 不为 `null`，表示按钮点击事件有效。
- 最后，通过 `tabControl.Items.Remove(tabItem)` 从 `TabControl` 中移除当前的 `TabItem`，即实现了关闭 `TabItem` 页的功能。
```C#
private static void ButtonClear_Click(object sender, RoutedEventArgs e)
{
    if (sender is Button button)
    {
        if (button.TemplatedParent is TabItem tabItem)
        {
            var tabControl = tabItem.Parent as TabControl;
            if (tabControl != null)
                tabControl.Items.Remove(tabItem); 
        }
    }
}

```



##### **XAML** 示例
示例引入 `WPFDevelopers` 的 `Nuget` 正式包 
- 可以使用附加属性 `helpers:ElementHelper.IsClear` 来绑定或控制 `TabItem` 是否显示关闭按钮。
~~~xml
<UserControl
    x:Class="WPFDevelopers.Samples.ExampleViews.DateRangePickerExample"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:controls="clr-namespace:WPFDevelopers.Samples.Controls"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:local="clr-namespace:WPFDevelopers.Samples.ExampleViews"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers"
    d:DesignHeight="450"
    d:DesignWidth="800"
    mc:Ignorable="d">
       <UniformGrid
    Margin="0,10"
    Columns="2"
    Rows="2">
    <TabControl Margin="4" wd:ElementHelper.CornerRadius="3">
        <TabItem Padding="10" Header="TabItem1">
            <Rectangle Fill="{DynamicResource WD.DangerSolidColorBrush}" />
        </TabItem>
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem2">
            <Rectangle Fill="{DynamicResource WD.InfoSolidColorBrush}" />
        </TabItem>
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem3">
            <Rectangle Fill="{DynamicResource WD.WarningSolidColorBrush}" />
        </TabItem>
    </TabControl>
    <TabControl Margin="4" TabStripPlacement="Bottom">
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="1">
            <Rectangle Fill="{DynamicResource WD.InfoSolidColorBrush}" />
        </TabItem>
        <TabItem
            x:Name="MyTabItem"
            MaxWidth="120"
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem2LongLongLongLongLongLongLongLong"
            ToolTip="{Binding ElementName=MyTabItem, Path=Header}">
            <Rectangle Fill="{DynamicResource WD.DangerSolidColorBrush}" />
        </TabItem>
        <TabItem
            MaxWidth="120"
            Padding="10"
            wd:ElementHelper.IsClear="True">
            <TabItem.Header>
                <TextBlock
                    VerticalAlignment="Center"
                    Text="TabItem3LongLongLongLongLongLongLongLong"
                    TextTrimming="CharacterEllipsis"
                    TextWrapping="NoWrap"
                    ToolTip="{Binding Text, RelativeSource={RelativeSource Self}}" />
            </TabItem.Header>
            <Rectangle Fill="{DynamicResource WD.WarningSolidColorBrush}" />
        </TabItem>
    </TabControl>
    <TabControl Margin="4" TabStripPlacement="Left">
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem1">
            <Rectangle Fill="{DynamicResource WD.WarningSolidColorBrush}" />
        </TabItem>
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem2">
            <Rectangle Fill="{DynamicResource WD.InfoSolidColorBrush}" />
        </TabItem>
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem3">
            <Rectangle Fill="{DynamicResource WD.DangerSolidColorBrush}" />
        </TabItem>
    </TabControl>
    <TabControl
        Margin="4"
        IsEnabled="False"
        TabStripPlacement="Right">
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem1">
            <Rectangle Fill="{DynamicResource WD.SuccessSolidColorBrush}" />
        </TabItem>
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem2">
            <Rectangle Fill="{DynamicResource WD.InfoSolidColorBrush}" />
        </TabItem>
        <TabItem
            Padding="10"
            wd:ElementHelper.IsClear="True"
            Header="TabItem3">
            <Rectangle Fill="{DynamicResource WD.WarningSolidColorBrush}" />
        </TabItem>
    </TabControl>
</UniformGrid>
</UserControl>
~~~


[GitHub 源码地址](https://github.com/WPFDevelopersOrg/WPFDevelopers/blob/master/src/WPFDevelopers.Shared/Styles/Styles.TabControl.xaml "GitHub 源码地址")

[Gitee 源码地址](https://gitee.com/WPFDevelopersOrg/WPFDevelopers/blob/master/src/WPFDevelopers.Shared/Styles/Styles.TabControl.xaml "Gitee 源码地址")

![TabControlClear](https://github.com/user-attachments/assets/8311da42-b998-4011-b75e-fdaa738bd10a)

