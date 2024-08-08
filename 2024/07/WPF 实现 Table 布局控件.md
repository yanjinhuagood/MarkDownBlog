<span style="display:block;text-align:center;">  **WPF 实现 Table 布局控件**</span> 
> 控件名：Table
>
> 作 者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers
- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;

有开发者提出希望 `WPF` 使用 `HTML` 的 `table` 标签进行显示表格？

`html`示例如下：
~~~html
 <table>
   <th>Header 1</th>  
   <tr>
     <td>Cell 1</td>
   </tr>
 </table>
~~~
因为需要快速实现表格功能时，我选择基于 `Grid` 控件，因为 `Grid` 提供了对跨行和跨列布局的支持。

一、新建 `Td.cs` 控件继承自 `Label` 代码如下：

##### 主要内容：

1.  属性 `RowSpan` 和 `ColumnSpan`：
   - `RowSpan` 和 `ColumnSpan` 分别定义了行跨度和列跨度的依赖属性，设置了默认值为 `1`。
   - 属性可以在 `C#` 或者 `XAML` 中进行设置和获取。




`Td` 控件适用于在 `WPF` 中实现复杂的表格布局，通过行和列的跨度属性，可以灵活地控制表格中单元格的布局。
~~~C#
using System.Windows;
using System.Windows.Controls;

namespace WPFTableGrid
{
    public class Td : Label
    {
        public static readonly DependencyProperty RowSpanProperty =
            DependencyProperty.Register("RowSpan", typeof(int), typeof(Td), new PropertyMetadata(1));

        public int RowSpan
        {
            get { return (int)GetValue(RowSpanProperty); }
            set { SetValue(RowSpanProperty, value); }
        }

        public static readonly DependencyProperty ColumnSpanProperty =
            DependencyProperty.Register("ColumnSpan", typeof(int), typeof(Td), new PropertyMetadata(1));

        public int ColumnSpan
        {
            get { return (int)GetValue(ColumnSpanProperty); }
            set { SetValue(ColumnSpanProperty, value); }
        }
    }
}
~~~
2.  设置 `Td` 控件的 `Style`
~~~xml
<Style TargetType="local:Td">
    <Setter Property="Background" Value="White" />
    <Setter Property="BorderBrush" Value="Black" />
    <Setter Property="Foreground" Value="Black" />
    <Setter Property="HorizontalContentAlignment" Value="Center" />
    <Setter Property="VerticalContentAlignment" Value="Center" />
    <Setter Property="SnapsToDevicePixels" Value="True" />
</Style>
~~~

二、新建 `Tr.cs` 继承自 `Grid`，代表表格中的一行代码如下：
~~~C#
using System.Windows.Controls;

namespace WPFTableGrid
{
    public class Tr : Grid
    {
        
    }
}

~~~

三、新建 `Th.cs` 继承自 `Label`，表示表头的单元格代码如下：
- `BorderBrush` 设置表格线颜色。
- `BorderThickness` 设置 `Table` 表格边框。
~~~C#
using System.Windows.Controls;

namespace WPFTableGrid
{
    public class Th : Label
    {
    }
}
~~~
设置 `Th` 控件的 `Style`
~~~xml
  <Style TargetType="local:Th">
      <Setter Property="Background" Value="LightGray" />
      <Setter Property="FontWeight" Value="Bold" />
      <Setter Property="BorderBrush" Value="Black" />
      <Setter Property="Foreground" Value="Black" />
      <Setter Property="HorizontalContentAlignment" Value="Center" />
      <Setter Property="VerticalContentAlignment" Value="Center" />
      <Setter Property="SnapsToDevicePixels" Value="True" />
  </Style>
~~~

四、新建 `Table.cs` 继承自 ` Grid`  控件,支持子控件行和列的定义。代码如下：
- `_zIndex` 记录控件的 `Z` 顺序。
- 构造函数：注册 `Loaded` 事件处理控件的行列。
- 算出表格的行数：统计内部 `Tr` 控件的数量，加上一行（表头）。
- 算出列数：找到所有 `Tr` 中的 `Td` 单元格，并根据 `GetColumnSpan` 方法确定最大列数。
- 每个表头 `Th` ，设置其在表格中的位置，并调整边框。
- 循环每行 `Tr` 并处理其子控件 `Td`。
- 删除其原始父容器 `Tr` 的引用。
- 将其添加到 `Table` 的子集。
- 设置 `Td` 的行和列。
- 如果 `Td` 跨行或跨列，会更改其 `Z` 顺序。

~~~C#
using System.Windows;
using System.Windows.Controls;

namespace WPFTableGrid
{
    public class Table : Grid
    {
        private int _zIndex = 1;
        public Table()
        {
            Loaded += OnLoaded;
        }

        private void OnLoaded(object sender, RoutedEventArgs e)
        {
            UpdateControl();
            Loaded -= OnLoaded;
        }

        private void UpdateControl()
        {
            RowDefinitions.Clear();
            ColumnDefinitions.Clear();

            if (InternalChildren.Count > 0)
            {
                var rows = InternalChildren.OfType<Tr>().Count() + 1;
                for (int i = 0; i < rows; i++)
                    RowDefinitions.Add(new RowDefinition());

                var columns = InternalChildren.OfType<Tr>().Max(child => child.Children.OfType<Td>().Sum(td => GetColumnSpan(td)));
                for (int i = 0; i < columns; i++)
                    ColumnDefinitions.Add(new ColumnDefinition());

                var index = 0;
                var list = InternalChildren.OfType<Tr>().ToList();

                var ths = InternalChildren.OfType<Th>().ToList();
                var hIndex = 0;
                for (int j = 0; j < ths.Count; j++)
                {
                    Th header = ths[j];
                    if (j == 0)
                        header.BorderThickness = new Thickness(1, 1, 1, 1);
                    else
                        header.BorderThickness = new Thickness(0, 1, 1, 1);
                    SetRow(header, index);
                    SetColumn(header, hIndex);
                    hIndex++;
                }
                for (int j = 0; j < list.Count; j++)
                {
                    index++;
                    Tr row = list[j];
                    var cIndex = 0;
                    var childElements = row.Children.OfType<Td>().ToList();
                    foreach (var cell in childElements)
                    {
                        if (cell.Parent is Panel oldParent)
                        {
                            oldParent.Children.Remove(cell);
                        }
                        Children.Add(cell);
                        if (cIndex == 0)
                            cell.BorderThickness = new Thickness(1, 0, 1, 1);
                        else
                            cell.BorderThickness = new Thickness(0, 0, 1, 1);
                        SetRow(cell, index);
                        SetColumn(cell, cIndex);
                        cell.SetValue(RowSpanProperty, cell.RowSpan);
                        cell.SetValue(ColumnSpanProperty, cell.ColumnSpan);
                        if (cell.RowSpan > 1 || cell.ColumnSpan > 1)
                        {
                            SetZIndex(cell, _zIndex);
                            _zIndex++;
                        }
                        cIndex++;
                    }
                }
            }
        }
    }
}
~~~
四、新建 `TableExample.xaml` 示例代码如下：
~~~xml
<wd:Window
    x:Class="WPFTableGrid.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:local="clr-namespace:WPFTableGrid"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers"
    Title="WPFDevelopers - Table"
    Width="800"
    Height="450"
    mc:Ignorable="d">
    <Window.Resources>
        <Style TargetType="local:Th">
            <Setter Property="Background" Value="LightGray" />
            <Setter Property="FontWeight" Value="Bold" />
            <Setter Property="BorderBrush" Value="Black" />
            <Setter Property="Foreground" Value="Black" />
            <Setter Property="HorizontalContentAlignment" Value="Center" />
            <Setter Property="VerticalContentAlignment" Value="Center" />
            <Setter Property="SnapsToDevicePixels" Value="True" />
        </Style>
        <Style TargetType="local:Td">
            <Setter Property="Background" Value="White" />
            <Setter Property="BorderBrush" Value="Black" />
            <Setter Property="Foreground" Value="Black" />
            <Setter Property="HorizontalContentAlignment" Value="Center" />
            <Setter Property="VerticalContentAlignment" Value="Center" />
            <Setter Property="SnapsToDevicePixels" Value="True" />
            
        </Style>
    </Window.Resources>
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition />
        </Grid.RowDefinitions>
        <Button
            Margin="0,10,0,0"
            HorizontalAlignment="Center"
            Click="BtnHeader_Click"
            Content="Header"
            Style="{StaticResource WD.DangerPrimaryButton}" />
        <local:Table Grid.Row="1" Margin="10">
            <local:Th Content="Header 1" />
            <local:Th Content="Header 2" />
            <local:Th Content="Header 3" />
            <local:Tr>
                <local:Td Content="Cell 1" />
                <local:Td>
                    <TextBlock Margin="5" Text="Cell 2" />
                </local:Td>
                <local:Td>
                    <TextBlock Margin="5" Text="Cell 3" />
                </local:Td>
            </local:Tr>
            <local:Tr>
                <local:Td RowSpan="3">
                    <TextBlock Margin="4" Text="Cell 4" />
                </local:Td>
                <local:Td>
                    <TextBlock Margin="5" Text="Cell 5" />
                </local:Td>
                <local:Td>
                    <TextBlock Margin="5" Text="Cell 6" />
                </local:Td>
            </local:Tr>
            <local:Tr>
                <local:Td>
                    <TextBlock Margin="5" Text="Cell 7" />
                </local:Td>
                <local:Td ColumnSpan="2">
                    <TextBlock Margin="5" Text="Cell 8" />
                </local:Td>
                <local:Td>
                    <TextBlock Margin="5" Text="Cell 9" />
                </local:Td>
            </local:Tr>
        </local:Table>
    </Grid>
</wd:Window>

~~~
![GIF 2024-08-07 23-20-12](https://github.com/user-attachments/assets/9a8142ee-ab17-46a1-95a7-9baae1a31479)

该方案基于 `Grid` 控件进行实现。对于更高级的需求，可以考虑将其修改为基于 `Panel`，自定义列的显示方式以及实现跨行和跨列的布局。

文中 `XAML` 中使用 [WPFDevelopers](https://www.nuget.org/packages/WPFDevelopers/ "WPFDevelopers") 库，如果直接拷贝使用，需要确保将相关的资源和控件进行正确的替换和配置。

[如果你对此有任何更好的想法或建议，我们将非常感激并乐于听取。](https://github.com/WPFDevelopersOrg/WPFDevelopers/issues/new "如果你对此有任何更好的想法或建议，我们将非常感激并乐于听取。")

