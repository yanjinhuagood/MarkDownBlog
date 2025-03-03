<span style="display:block;text-align:center;">  **基于 WPFDevelopers 库实现主题切换功能**</span> 
> 控件名：Theme 
>
> 作   者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers

- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;

###### 1. 新增 **Theme.cs**  
- `Theme` 继承 `ListBox`，用于显示和颜色主题切换。
- 预定义了`六`种颜色（可自行添加其他色），来源于 `WD` [早期的主题色](https://mp.weixin.qq.com/s/PjnfMnv3v213Vr5Uhg6-9g)。
~~~C#
private readonly List<Brush> themes = new List<Brush>
{
    new SolidColorBrush(Color.FromRgb(64, 158, 255)),  // #409EFF
    new SolidColorBrush(Color.FromRgb(255, 3, 62)),    // #FF033E
    new SolidColorBrush(Color.FromRgb(162, 27, 252)),  // #A21BFC
    new SolidColorBrush(Color.FromRgb(254, 148, 38)),  // #FE9426
    new SolidColorBrush(Color.FromRgb(0, 176, 80)),    // #00B050
    new SolidColorBrush(Color.FromRgb(255, 0, 127))    // #FF007F
};
~~~
- `Orientation` 列表方向`水平`或`垂直`。
- 监听 `OnSelectionChanged` 选择主题色时触发 `ThemeManager`。
~~~C#
protected override void OnSelectionChanged(SelectionChangedEventArgs e)
{
    base.OnSelectionChanged(e);
    if (SelectedItem == null) return;

    var item = SelectedItem as ThemeItem;
    if (item == null) return;

    if (item.Background is SolidColorBrush solidColorBrush)
        ThemeManager.Instance.SetColor(solidColorBrush.Color);
}
~~~
###### 2. 新增 **Theme.xaml** 
- `controls:ThemeItem`主题色项。
- `wd:PathIcon` 作为一个选中图标，当 `IsSelected=True` 则显示，反之则不显示。
- `Orientation` 绑定 `controls:Theme` 控件的 `Orientation` 属性，支持`水平`或`垂直`。
~~~xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:controls="clr-namespace:WPFDevelopers.Samples.Controls"
    xmlns:sys="clr-namespace:System;assembly=mscorlib"
    xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers">

    <Style
        x:Key="WD.ThemeItem"
        BasedOn="{StaticResource WD.ControlBasicStyle}"
        TargetType="{x:Type controls:ThemeItem}">
        <Setter Property="Width" Value="40" />
        <Setter Property="Height" Value="40" />
        <Setter Property="VerticalAlignment" Value="Center" />
        <Setter Property="HorizontalAlignment" Value="Stretch" />
        <Setter Property="VerticalContentAlignment" Value="Bottom" />
        <Setter Property="HorizontalContentAlignment" Value="Right" />
        <Setter Property="Padding" Value="0,0,4,4" />
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="{x:Type controls:ThemeItem}">
                    <Border
                        x:Name="PART_Border"
                        Padding="2"
                        Background="Transparent"
                        BorderBrush="{TemplateBinding Background}"
                        BorderThickness="0"
                        CornerRadius="{Binding Path=(wd:ElementHelper.CornerRadius), RelativeSource={RelativeSource AncestorType=controls:Theme}}">
                        <wd:SmallPanel>
                            <Border
                                x:Name="PART_BorderBackground"
                                Background="{TemplateBinding Background}"
                                CornerRadius="{Binding Path=(wd:ElementHelper.CornerRadius), RelativeSource={RelativeSource AncestorType=controls:Theme}}" />
                            <wd:PathIcon
                                Width="12"
                                Height="10"
                                Margin="{TemplateBinding Padding}"
                                HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                                VerticalAlignment="{TemplateBinding VerticalContentAlignment}"
                                Data="{StaticResource WD.CheckMarkGeometry}"
                                Foreground="{DynamicResource WD.BackgroundBrush}"
                                Kind="CheckMark"
                                Visibility="{TemplateBinding IsSelected,
                                                             Converter={StaticResource WD.Bool2VisibilityConverter}}" />
                        </wd:SmallPanel>
                    </Border>
                    <ControlTemplate.Triggers>
                        <Trigger Property="IsMouseOver" Value="True">
                            <Setter TargetName="PART_BorderBackground" Property="Opacity" Value=".8" />
                            <Setter TargetName="PART_Border" Property="BorderThickness" Value="1" />
                        </Trigger>
                        <Trigger Property="IsSelected" Value="True">
                            <Setter TargetName="PART_Border" Property="BorderThickness" Value="1" />
                        </Trigger>
                    </ControlTemplate.Triggers>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>

    <Style
        x:Key="WD.Theme"
        BasedOn="{StaticResource WD.ControlBasicStyle}"
        TargetType="{x:Type controls:Theme}">
        <Setter Property="VerticalAlignment" Value="Center" />
        <Setter Property="HorizontalAlignment" Value="Left" />
        <Setter Property="Foreground" Value="{DynamicResource WD.PrimaryTextBrush}" />
        <Setter Property="ItemContainerStyle" Value="{StaticResource WD.ThemeItem}" />
        <Setter Property="Padding" Value="{StaticResource WD.Padding}" />
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="{x:Type controls:Theme}">
                    <wd:WDBorder
                        Background="{TemplateBinding Background}"
                        BorderBrush="{TemplateBinding BorderBrush}"
                        BorderThickness="{TemplateBinding BorderThickness}"
                        CornerRadius="{Binding Path=(wd:ElementHelper.CornerRadius), RelativeSource={RelativeSource TemplatedParent}}">
                        <ItemsPresenter
                            Margin="{TemplateBinding Padding}"
                            HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                            VerticalAlignment="{TemplateBinding VerticalContentAlignment}"
                            Clip="{Binding RelativeSource={RelativeSource AncestorType=wd:WDBorder}, Path=ContentClip}" />
                    </wd:WDBorder>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
        <Setter Property="ItemsPanel">
            <Setter.Value>
                <ItemsPanelTemplate>
                    <StackPanel wd:PanelHelper.Spacing="4" Orientation="{Binding Path=Orientation, RelativeSource={RelativeSource AncestorType=controls:Theme}}" />
                </ItemsPanelTemplate>
            </Setter.Value>
        </Setter>
    </Style>

    <Style BasedOn="{StaticResource WD.Theme}" TargetType="{x:Type controls:Theme}" />
</ResourceDictionary>
~~~
##### **XAML** 示例
- 示例引入 `WPFDevelopers 1.1.0.3-preview3 ` 的 `Nuget` 正式包 
- 引入命名空间 `xmlns:controls="clr-namespace:WPFDevelopers.Samples.Controls"`
~~~xml
<controls:Theme Margin="0,10" />
~~~
![theme](https://github.com/user-attachments/assets/e9bb8a64-ca46-4c9c-9fdd-25e8ef6e63be)

[GitHub 源码地址](https://github.com/WPFDevelopersOrg/WPFDevelopers/tree/dev/src/WPFDevelopers.Samples.Shared/Controls/Theme "GitHub 源码地址")

[Gitee 源码地址](https://gitee.com/WPFDevelopersOrg/WPFDevelopers/tree/dev/src/WPFDevelopers.Samples.Shared/Controls/Theme "Gitee 源码地址")
