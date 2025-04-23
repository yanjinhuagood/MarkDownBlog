<span style="display:block;text-align:center;">  **WPF 实现描点导航**</span> 
> 控件名：NavScrollPanel 
>
> 作   者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers

- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;

有一位开发者需要实现类似「左侧导航栏 + 右侧滚动内容」的控件，需要支持数据绑定、内容模板、同步滚动定位等功能。
##### 1. 新增 **NavScrollPanel.cs**  
- 左侧导航栏`ListBox`：显示导航，支持点击定位；
- 右侧滚动内容区 `ScrollViewer` 和 `StackPanel`：展示对应内容模板，支持滚动自动选中导航项。
- 通过 `ItemsSource` 绑定内容集合；
- 自定义 `ItemTemplate` 显示内容；
- 通过 `TranslatePoint` 方法，可以获取元素相对于容器的坐标位置，并确保该位置不受 `Margin` 的影响。通过调用 `ScrollToVerticalOffset` 来滚动右侧容器;
~~~C#
public class NavScrollPanel : Control
{
    static NavScrollPanel()
    {
        DefaultStyleKeyProperty.OverrideMetadata(typeof(NavScrollPanel),
            new FrameworkPropertyMetadata(typeof(NavScrollPanel)));
    }

    public IEnumerable ItemsSource
    {
        get => (IEnumerable)GetValue(ItemsSourceProperty);
        set => SetValue(ItemsSourceProperty, value);
    }

    public static readonly DependencyProperty ItemsSourceProperty =
        DependencyProperty.Register(nameof(ItemsSource), typeof(IEnumerable), typeof(NavScrollPanel), new PropertyMetadata(null, OnItemsSourceChanged));

    public DataTemplate ItemTemplate
    {
        get => (DataTemplate)GetValue(ItemTemplateProperty);
        set => SetValue(ItemTemplateProperty, value);
    }

    public static readonly DependencyProperty ItemTemplateProperty =
        DependencyProperty.Register(nameof(ItemTemplate), typeof(DataTemplate), typeof(NavScrollPanel), new PropertyMetadata(null));

    public int SelectedIndex
    {
        get => (int)GetValue(SelectedIndexProperty);
        set => SetValue(SelectedIndexProperty, value);
    }

    public static readonly DependencyProperty SelectedIndexProperty =
        DependencyProperty.Register(nameof(SelectedIndex), typeof(int), typeof(NavScrollPanel), new PropertyMetadata(-1, OnSelectedIndexChanged));

    private ListBox _navListBox;
    private ScrollViewer _scrollViewer;
    private StackPanel _contentPanel;

    public override void OnApplyTemplate()
    {
        base.OnApplyTemplate();

        _navListBox = GetTemplateChild("PART_ListBox") as ListBox;
        _scrollViewer = GetTemplateChild("PART_ScrollViewer") as ScrollViewer;
        _contentPanel = GetTemplateChild("PART_ContentPanel") as StackPanel;

        if (_navListBox != null)
        {
            _navListBox.DisplayMemberPath = "Title";
            _navListBox.SelectionChanged -= NavListBox_SelectionChanged;
            _navListBox.SelectionChanged += NavListBox_SelectionChanged;
        }
        if (_scrollViewer != null)
        {
            _scrollViewer.ScrollChanged += ScrollViewer_ScrollChanged;
        }
        RenderContent();
    }

    private void NavListBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        SelectedIndex = _navListBox.SelectedIndex;
    }

    private void ScrollViewer_ScrollChanged(object sender, ScrollChangedEventArgs e)
    {
        double currentOffset = _scrollViewer.VerticalOffset;                                    
        double viewportHeight = _scrollViewer.ViewportHeight;                                  

        for (int i = 0; i < _contentPanel.Children.Count; i++)
        {
            var element = _contentPanel.Children[i] as FrameworkElement;
            if (element == null) continue;

            Point relativePoint = element.TranslatePoint(new Point(0, 0), _contentPanel);

            if (relativePoint.Y >= currentOffset && relativePoint.Y < currentOffset + viewportHeight)
            {
                _navListBox.SelectionChanged -= NavListBox_SelectionChanged;
                SelectedIndex = i;
                _navListBox.SelectionChanged += NavListBox_SelectionChanged;
                break;
            }
        }
    }


    private static void OnItemsSourceChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is NavScrollPanel control)
        {
            control.RenderContent();
        }
    }

    private static void OnSelectedIndexChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        if (d is NavScrollPanel control)
        {
            int index = (int)e.NewValue;

            if (control._contentPanel != null &&
                index >= 0 && index < control._contentPanel.Children.Count)
            {
                var target = control._contentPanel.Children[index] as FrameworkElement;
                if (target != null)
                {
                    var virtualPoint = target.TranslatePoint(new Point(0, 0), control._contentPanel);
                    control._scrollViewer.ScrollToVerticalOffset(virtualPoint.Y);
                }
            }
        }
    }

    private void RenderContent()
    {
        if (_contentPanel == null || ItemsSource == null || ItemTemplate == null)
            return;
        _contentPanel.Children.Clear();
        foreach (var item in ItemsSource)
        {
            var content = new ContentControl
            {
                Content = item,
                ContentTemplate = ItemTemplate,
                Margin = new Thickness(10,50,10,50)
            };
            _contentPanel.Children.Add(content);
        }
    }
}
~~~
##### 2. 新增 **NavScrollPanel.Xaml**   
- 控件模板通过 `ListBox` 和 `ScrollViewer` 实现。
~~~xml
<Style TargetType="local:NavScrollPanel">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="local:NavScrollPanel">
                <Grid>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="120" />
                        <ColumnDefinition Width="*" />
                    </Grid.ColumnDefinitions>
                    <ListBox
                        x:Name="PART_ListBox"
                        ItemsSource="{TemplateBinding ItemsSource}"
                        SelectedIndex="{TemplateBinding SelectedIndex}" />
                    <ScrollViewer
                        x:Name="PART_ScrollViewer"
                        Grid.Column="1"
                        VerticalScrollBarVisibility="Auto">
                        <StackPanel x:Name="PART_ContentPanel" />
                    </ScrollViewer>
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>

~~~
##### 3. 使用示例 
###### 1. 定义数据结构
~~~C#
public class SectionItem {
    public string Title { get; set; }
    public object Content { get; set; }
}
~~~
###### 2. 初始化数据并绑定
~~~C#
Sections = new ObservableCollection<SectionItem>
{
    new SectionItem{ Title = "播放相关", Content = new PlaybackSettings()},
    new SectionItem{ Title = "桌面歌词", Content = new DesktopLyrics()},
    new SectionItem{ Title = "快捷键", Content = new ShortcutKeys()},
    new SectionItem{ Title = "隐私设置", Content = new PrivacySettings()},
    new SectionItem{ Title = "关于我们", Content = new About()},
};
DataContext = this;

~~~
###### 3. 模板定义
~~~xml
<DataTemplate x:Key="SectionTemplate">
    <StackPanel>
        <TextBlock Text="{Binding Title}" FontSize="20" Margin="0,10"/>
        <Border Background="#F0F0F0" Padding="20" CornerRadius="10">
            <ContentPresenter Content="{Binding Content}" FontSize="14"/>
        </Border>
    </StackPanel>
</DataTemplate>

~~~

###### 4. 使用控件
~~~xml
<local:NavScrollPanel
    ItemTemplate="{StaticResource SectionTemplate}"
    ItemsSource="{Binding Sections}" />

~~~
###### 5. 新增**NavScrollPanelExample.xaml** 

~~~xml
<wd:Window
    x:Class="WpfNavPanel.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:local="clr-namespace:WpfNavPanel"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers"
    Title="NavScrollPanel - 锚点导航"
    Width="800"
    Height="450"
    mc:Ignorable="d">
    <wd:Window.Resources>
        <DataTemplate x:Key="SectionTemplate">
            <StackPanel>
                <TextBlock
                    Margin="0,10"
                    FontSize="20"
                    Text="{Binding Title}" />
                <Border
                    Padding="20"
                    Background="#F0F0F0"
                    CornerRadius="10">
                    <ContentPresenter Content="{Binding Content}" TextElement.FontSize="14" />
                </Border>
            </StackPanel>
        </DataTemplate>
    </wd:Window.Resources>
    <Grid Margin="4">
        <local:NavScrollPanel ItemTemplate="{StaticResource SectionTemplate}" ItemsSource="{Binding Sections}" />
    </Grid>
</wd:Window>
~~~


[GitHub 源码地址](https://github.com/WPFDevelopersOrg/WPFDevelopers "GitHub 源码地址")

[Gitee 源码地址](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "Gitee 源码地址")

![NavScrollPanel](https://github.com/user-attachments/assets/5e7606cf-2ac2-4821-891d-b56892a16493)

