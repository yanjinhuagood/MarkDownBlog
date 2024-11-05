<span style="display:block;text-align:center;">  **Drawer 抽屉控件的实现**</span> 
> 控件名：Drawer 
>
> 作   者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers

- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;



##### 抽屉控件的逻辑实现
  定义了一个名为 `Drawer` 的自定义控件，继承自 `HeaderedContentControl`，允许用户在应用程序中创建可展开和收起的抽屉。抽屉的显示和隐藏动画通过 `Storyboard` 实现，支持从不同方向（`左`、`上`、`右`、`下`）展开和收起。
  
##### 关键解释
###### 1.定义模板
- 使用 `TemplatePart` 特性定义了两个模板：`BorderHeaderTemplateName` 和 `BorderMarkTemplateName`，分别代表抽屉的`头`和`蒙板`部分。
~~~C#
[TemplatePart(Name = BorderHeaderTemplateName, Type = typeof(Border))]
[TemplatePart(Name = BorderMarkTemplateName, Type = typeof(Border))]
public class Drawer : HeaderedContentControl

~~~
###### 2.定义依赖属性
- `Position` 抽屉展开的方向（`左`、`上`、`右`、`下`）。
- `IsOpen` 抽屉是否打开，状态变化时调用 `OnIsOpenChanged` 方法。
~~~C#
public static readonly DependencyProperty EdgePositionProperty =
    DependencyProperty.Register("Position", typeof(Position), typeof(Drawer),
        new PropertyMetadata(Position.Left));

public static readonly DependencyProperty IsOpenProperty =
    DependencyProperty.Register("IsOpen", typeof(bool), typeof(Drawer),
        new PropertyMetadata(false, OnIsOpenChanged));
~~~

###### 3.属性实现
~~~C#
public Position Position
{
    get => (Position) GetValue(EdgePositionProperty);
    set => SetValue(EdgePositionProperty, value);
}

public bool IsOpen
{
    get => (bool) GetValue(IsOpenProperty);
    set => SetValue(IsOpenProperty, value);
}

~~~
###### 4.状态变化处理
- `OnIsOpenChanged` 方法根据 `IsOpen` 属性的值，控制抽屉的显示或隐藏和动画效果。
~~~C#
private static void OnIsOpenChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
{
    var ctrl = d as Drawer;
    if (ctrl != null)
    {
        if (ctrl.IsOpen)
        {
            ctrl._headerBorder.Visibility = Visibility.Visible;
            ctrl._enterStoryboard.Begin();
        }
        else
        {
            ctrl._exitStoryboard.Begin();
        }
    }
}

~~~~
###### 5.模板应用
- `OnApplyTemplate` 方法在控件模板应用时调用，获取模板中的 `Border` 部件并注册 `Loaded` 和 `MouseDown` 事件。
~~~C#
public override void OnApplyTemplate()
{
    base.OnApplyTemplate();
    _headerBorder = GetTemplateChild(BorderHeaderTemplateName) as Border;
    if (_headerBorder != null)
        _headerBorder.Loaded += HeaderBorder_Loaded;
    _markBorder = GetTemplateChild(BorderMarkTemplateName) as Border;
    if (_markBorder != null)
        _markBorder.MouseDown += MarkBorder_MouseDown;
}

~~~
###### 6.动画实现
- `HeaderBorder_Loaded` 方法根据 `Position` 的不同，设置不同的动画，并根据属性值控制动画的执行。
~~~C#
        private void HeaderBorder_Loaded(object sender, RoutedEventArgs e)
        {
            TranslateTransform translateTransform;
            DoubleAnimation animation, exitAnimation;
            switch (Position)
            {
                case Position.Left:
                case Position.Right:
                    _headerWidth = _headerBorder.ActualWidth;
                    if (Position == Position.Left)
                        translateTransform = new TranslateTransform(-_headerWidth, 0);
                    else
                        translateTransform = new TranslateTransform(_headerWidth, 0);
                    _headerBorder.RenderTransform = new TransformGroup
                    {
                        Children = new TransformCollection {translateTransform}
                    };
                    animation = new DoubleAnimation
                    {
                        From = Position == Position.Left ? -_headerWidth : _headerWidth,
                        To = 0,
                        Duration = TimeSpan.FromMilliseconds(300)
                    };

                    Storyboard.SetTarget(animation, _headerBorder);
                    Storyboard.SetTargetProperty(animation, new PropertyPath("RenderTransform.Children[0].X"));
                    _enterStoryboard = new Storyboard();
                    _enterStoryboard.Children.Add(animation);

                    exitAnimation = new DoubleAnimation
                    {
                        From = 0,
                        To = Position == Position.Left ? -_headerWidth : _headerWidth,
                        Duration = TimeSpan.FromMilliseconds(300)
                    };

                    Storyboard.SetTarget(exitAnimation, _headerBorder);
                    Storyboard.SetTargetProperty(exitAnimation, new PropertyPath("RenderTransform.Children[0].X"));
                    _exitStoryboard = new Storyboard();
                    _exitStoryboard.Completed += delegate
                    {
                        if (!IsOpen)
                            _headerBorder.Visibility = Visibility.Collapsed;
                    };
                    _exitStoryboard.Children.Add(exitAnimation);
                    break;
                case Position.Top:
                case Position.Bottom:
                    _headerHeight = _headerBorder.ActualHeight;
                    if (Position == Position.Top)
                        translateTransform = new TranslateTransform(0, -_headerHeight);
                    else
                        translateTransform = new TranslateTransform(0, _headerHeight);
                    _headerBorder.RenderTransform = new TransformGroup
                    {
                        Children = new TransformCollection {translateTransform}
                    };
                    animation = new DoubleAnimation
                    {
                        From = Position == Position.Top ? -_headerHeight : _headerHeight,
                        To = 0,
                        Duration = TimeSpan.FromMilliseconds(300)
                    };

                    Storyboard.SetTarget(animation, _headerBorder);
                    Storyboard.SetTargetProperty(animation, new PropertyPath("RenderTransform.Children[0].Y"));
                    _enterStoryboard = new Storyboard();
                    _enterStoryboard.Children.Add(animation);

                    exitAnimation = new DoubleAnimation
                    {
                        From = 0,
                        To = Position == Position.Top ? -_headerHeight : _headerHeight,
                        Duration = TimeSpan.FromMilliseconds(300)
                    };

                    Storyboard.SetTarget(exitAnimation, _headerBorder);
                    Storyboard.SetTargetProperty(exitAnimation, new PropertyPath("RenderTransform.Children[0].Y"));
                    _exitStoryboard = new Storyboard();
                    _exitStoryboard.Completed += delegate
                    {
                        if (!IsOpen)
                            _headerBorder.Visibility = Visibility.Collapsed;
                    };
                    _exitStoryboard.Children.Add(exitAnimation);
                    break;
            }

            _headerBorder.Visibility = Visibility.Collapsed;
            _headerBorder.Loaded -= HeaderBorder_Loaded;
        }
~~~
##### 抽屉控件的样式实现
- `Position` 属性的触发器：根据 `Position` 属性的值（`Top`、`Right`、`Bottom`），调整 `PART_Header` 的对齐方式。
- `IsOpen` 属性的触发器：如果 `IsOpen` 为 `False`，将 `PART_Mark` 的可见性设置为 `Collapsed`，在抽屉关闭时隐藏。
~~~xml
<Style
    x:Key="WD.Drawer"
    BasedOn="{StaticResource WD.ControlBasicStyle}"
    TargetType="{x:Type controls:Drawer}">
    <Setter Property="Background" Value="{DynamicResource WD.BackgroundSolidColorBrush}" />
    <Setter Property="BorderBrush" Value="{DynamicResource WD.BaseSolidColorBrush}" />
    <Setter Property="BorderThickness" Value="1" />
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="{x:Type controls:Drawer}">
                <controls:WDBorder
                    x:Name="Border"
                    Margin="{TemplateBinding Margin}"
                    Background="{TemplateBinding Background}"
                    BorderBrush="{TemplateBinding BorderBrush}"
                    BorderThickness="{TemplateBinding BorderThickness}"
                    CornerRadius="{Binding Path=(helpers:ElementHelper.CornerRadius), RelativeSource={RelativeSource TemplatedParent}}"
                    SnapsToDevicePixels="True">
                    <controls:SmallPanel Clip="{Binding RelativeSource={RelativeSource AncestorType=controls:WDBorder}, Path=ContentClip}">
                        <Border x:Name="PART_Content">
                            <ContentPresenter Content="{TemplateBinding Content}" />
                        </Border>
                        <Border
                            x:Name="PART_Mark"
                            Background="{DynamicResource WD.PrimaryTextSolidColorBrush}"
                            Opacity=".5" />
                        <Border
                            x:Name="PART_Header"
                            HorizontalAlignment="Left"
                            Background="{TemplateBinding Background}"
                            Effect="{StaticResource WD.NormalShadowDepth}">
                            <ContentPresenter Content="{TemplateBinding Header}" />
                        </Border>
                    </controls:SmallPanel>
                </controls:WDBorder>
                <ControlTemplate.Triggers>
                    <Trigger Property="Position" Value="Top">
                        <Setter TargetName="PART_Header" Property="HorizontalAlignment" Value="Stretch" />
                        <Setter TargetName="PART_Header" Property="VerticalAlignment" Value="Top" />
                    </Trigger>
                    <Trigger Property="Position" Value="Right">
                        <Setter TargetName="PART_Header" Property="HorizontalAlignment" Value="Right" />
                        <Setter TargetName="PART_Header" Property="VerticalAlignment" Value="Stretch" />
                    </Trigger>
                    <Trigger Property="Position" Value="Bottom">
                        <Setter TargetName="PART_Header" Property="HorizontalAlignment" Value="Stretch" />
                        <Setter TargetName="PART_Header" Property="VerticalAlignment" Value="Bottom" />
                    </Trigger>
                    <Trigger Property="IsOpen" Value="False">
                        <Setter TargetName="PART_Mark" Property="Visibility" Value="Collapsed" />
                    </Trigger>
                </ControlTemplate.Triggers>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
~~~
##### 示例
示例引入 `WPFDevelopers` 的 `Nuget`包 
~~~xml
<UserControl
    x:Class="WPFDevelopers.Samples.ExampleViews.DrawerExample"
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
    <controls:CodeViewer>
        <Grid>
            <Grid.ColumnDefinitions>
                <ColumnDefinition />
                <ColumnDefinition />
            </Grid.ColumnDefinitions>
            <Grid.RowDefinitions>
                <RowDefinition />
                <RowDefinition />
            </Grid.RowDefinitions>
            <wd:Drawer
                x:Name="MyDrawerTop"
                Margin="2"
                Position="Top">
                <wd:Drawer.Header>
                    <Grid Height="120">
                        <StackPanel HorizontalAlignment="Center">
                            <TextBlock FontSize="18" Text="Drawer" />
                            <Button HorizontalAlignment="Center" Content="contents..." />
                        </StackPanel>
                    </Grid>
                </wd:Drawer.Header>
                <wd:Drawer.Content>
                    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Bottom">
                        <TextBlock Text="抽屉从顶部置滑出，点击遮罩区关闭。" />
                        <Button
                            Margin="0,10"
                            HorizontalAlignment="Center"
                            Click="ButtonTop_Click"
                            Content="Open"
                            Style="{StaticResource WD.PrimaryButton}" />
                    </StackPanel>
                </wd:Drawer.Content>
            </wd:Drawer>
            <wd:Drawer
                x:Name="MyDrawerBottom"
                Grid.Column="1"
                Margin="2"
                wd:ElementHelper.CornerRadius="3"
                Position="Bottom">
                <wd:Drawer.Header>
                    <Grid Height="130">
                        <StackPanel HorizontalAlignment="Center">
                            <TextBlock FontSize="18" Text="Drawer" />
                            <Button
                                HorizontalAlignment="Center"
                                wd:ElementHelper.CornerRadius="3"
                                Content="contents..."
                                Style="{StaticResource WD.DangerDefaultButton}" />
                        </StackPanel>
                    </Grid>
                </wd:Drawer.Header>
                <wd:Drawer.Content>
                    <StackPanel HorizontalAlignment="Center">
                        <TextBlock Text="抽屉从底部滑出，点击遮罩区关闭。" />
                        <Button
                            Margin="0,10"
                            HorizontalAlignment="Center"
                            wd:ElementHelper.CornerRadius="3"
                            Click="ButtonBottom_Click"
                            Content="Open"
                            Style="{StaticResource WD.DangerPrimaryButton}" />
                    </StackPanel>
                </wd:Drawer.Content>
            </wd:Drawer>

            <wd:Drawer
                x:Name="MyDrawerLeft"
                Grid.Row="1"
                Margin="2"
                wd:ElementHelper.CornerRadius="3"
                Position="Left">
                <wd:Drawer.Header>
                    <Grid Width="140" Background="HotPink">
                        <StackPanel>
                            <TextBlock
                                HorizontalAlignment="Center"
                                FontSize="18"
                                Text="Drawer" />
                            <Button
                                HorizontalAlignment="Center"
                                wd:ElementHelper.CornerRadius="3"
                                Content="contents..."
                                Style="{StaticResource WD.SuccessDefaultButton}" />
                        </StackPanel>
                    </Grid>
                </wd:Drawer.Header>
                <wd:Drawer.Content>
                    <StackPanel HorizontalAlignment="Center">
                        <TextBlock Text="抽屉从左侧滑出，点击遮罩区关闭。" />
                        <Button
                            Margin="0,10"
                            HorizontalAlignment="Center"
                            wd:ElementHelper.CornerRadius="3"
                            Click="ButtonLeft_Click"
                            Content="Open"
                            Style="{StaticResource WD.SuccessPrimaryButton}" />
                    </StackPanel>
                </wd:Drawer.Content>
            </wd:Drawer>
            <wd:Drawer
                x:Name="MyDrawerRight"
                Grid.Row="1"
                Grid.Column="1"
                Margin="2"
                Position="Right">
                <wd:Drawer.Header>
                    <Grid Width="100">
                        <StackPanel HorizontalAlignment="Center">
                            <TextBlock
                                HorizontalAlignment="Center"
                                FontSize="18"
                                Text="Drawer" />
                            <Button
                                HorizontalAlignment="Center"
                                Content="contents..."
                                Style="{StaticResource WD.WarningDefaultButton}" />
                        </StackPanel>
                    </Grid>
                </wd:Drawer.Header>
                <wd:Drawer.Content>
                    <StackPanel HorizontalAlignment="Center">
                        <TextBlock Text="抽屉从右侧滑出，点击遮罩区关闭。" />
                        <Button
                            Margin="0,10"
                            HorizontalAlignment="Center"
                            Click="ButtonRight_Click"
                            Content="Open"
                            Style="{StaticResource WD.WarningPrimaryButton}" />
                    </StackPanel>
                </wd:Drawer.Content>
            </wd:Drawer>
        </Grid>
        <controls:CodeViewer.SourceCodes>
            <controls:SourceCodeModel CodeSource="/WPFDevelopers.SamplesCode;component/ExampleViews/DrawerExample.xaml" CodeType="Xaml" />
            <controls:SourceCodeModel CodeSource="/WPFDevelopers.SamplesCode;component/ExampleViews/DrawerExample.xaml.cs" CodeType="CSharp" />
        </controls:CodeViewer.SourceCodes>
    </controls:CodeViewer>
</UserControl>
~~~
![Drawer](https://github.com/user-attachments/assets/566d280b-bad3-4f03-8474-b96303e83248)

[GitHub 源码地址](https://github.com/WPFDevelopersOrg/WPFDevelopers/tree/dev/src/WPFDevelopers.Shared/Controls/Drawer/Drawer.cs "GitHub 源码地址")

[Gitee 源码地址](https://gitee.com/WPFDevelopersOrg/WPFDevelopers/tree/dev/src/WPFDevelopers.Shared/Controls/Drawer/Drawer.cs "Gitee 源码地址")
