<span style="display:block;text-align:center;">  **如何优雅的为 DataGrid 设置圆角**</span> 
> 控件名：WDBorder
>
> 作 者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers
- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;

如何优雅的为 DataGrid 设置圆角？

一般情况都会通过重新样式，最外层嵌套 `Border` 设置 `CornerRadius` 但是这样设置后会发现 `DataGrid` 还是无法显示四周的圆角。
~~~xml
<DataGrid x:Name="MyDataGrid" Margin="10">
    <DataGrid.Resources>
        <Style TargetType="{x:Type DataGrid}">
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="{x:Type DataGrid}">
                        <Border
                            Background="{TemplateBinding Background}"
                            BorderBrush="{TemplateBinding BorderBrush}"
                            BorderThickness="{TemplateBinding BorderThickness}"
                            CornerRadius="10" ClipToBounds="True">
                            <ScrollViewer x:Name="DG_ScrollViewer" Focusable="false">
                                <ScrollViewer.Template>
                                    <ControlTemplate TargetType="{x:Type ScrollViewer}">
                                        <ScrollContentPresenter />
                                    </ControlTemplate>
                                </ScrollViewer.Template>
                                <ItemsPresenter SnapsToDevicePixels="{TemplateBinding SnapsToDevicePixels}" />
                            </ScrollViewer>
                        </Border>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </DataGrid.Resources>
</DataGrid>
~~~
![17fc97597781fdb5b840805bcb79a14](https://github.com/user-attachments/assets/e40fe24d-d565-41d1-af00-63c907de200f)

方式一、那如何最简单的设置圆角？可以通过 `Border` 设置 `Padding` 属性，进行显示圆角。
  ~~~xml
<Border
    Background="{TemplateBinding Background}"
    BorderBrush="{TemplateBinding BorderBrush}"
    BorderThickness="{TemplateBinding BorderThickness}"
    CornerRadius="10" ClipToBounds="True"
    Padding="10">
  <!--省略ScrollViewer-->
</Border>
  ~~~
![99dc2734b74b07ef0f6ac3853e3a65a](https://github.com/user-attachments/assets/d4165aa9-aa07-401a-bd65-f269400ebe10)

方式二、通过设置内部容器的 `Clip` 进行裁剪。


1）`WDBorder.cs` 代码如下：
- `ContentClip` ：类型为 `Geometry`，用于定义控件内容的裁剪区域。
- `CalculateContentClip` 方法：
  - 计算返回一个 `Geometry` 对象，作为内容裁剪的区域。
  - 获取边框 `BorderThickness` 和圆角 `CornerRadius`，计算区域的宽和高。
  - 如果宽和高大于 `0`，则创建一个 `Rect` 显示区域。
  - 使用 `GeometryHelper` 类的静态方法生成一个 `Geometry` ，根据 `Rect` 和 `CornerRadius` 来填充 `StreamGeometry`，最后将 `StreamGeometry` 冻结，并返回 `StreamGeometry`。
~~~c#
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using WPFDevelopers.Helpers;

namespace WPFDevelopers.Controls
{
    public class WDBorder : Border
    {
        public Geometry ContentClip
        {
            get
            {
                return (Geometry)GetValue(ContentClipProperty);
            }
            set
            {
                SetValue(ContentClipProperty, value);
            }
        }

        private Geometry CalculateContentClip()
        {
            var borderThickness = BorderThickness;
            var cornerRadius = CornerRadius;
            var renderSize = RenderSize;
            var width = renderSize.Width - borderThickness.Left - borderThickness.Right;
            var height = renderSize.Height - borderThickness.Top - borderThickness.Bottom;
            if (width > 0.0 && height > 0.0)
            {
                var rect = new Rect(0.0, 0.0, width, height);
                var radii = new GeometryHelper.Radii(cornerRadius, borderThickness, false);
                var streamGeometry = new StreamGeometry();
                using (StreamGeometryContext streamGeometryContext = streamGeometry.Open())
                {
                    GeometryHelper.GenerateGeometry(streamGeometryContext, rect, radii);
                    streamGeometry.Freeze();
                    return streamGeometry;
                }
            }
            return null;
        }

        protected override void OnRender(DrawingContext dc)
        {
            SetValue(ContentClipPropertyKey, CalculateContentClip());
            base.OnRender(dc);
        }

        public static readonly DependencyPropertyKey ContentClipPropertyKey = 
            DependencyProperty.RegisterReadOnly("ContentClip", typeof(Geometry), typeof(WDBorder), new PropertyMetadata(null));

        public static readonly DependencyProperty ContentClipProperty = ContentClipPropertyKey.DependencyProperty;
        
    }
}

~~~~
2）`GeometryHelper.cs` 代码如下：
- 以下代码来源于官方 [Border](https://referencesource.microsoft.com/#PresentationFramework/src/Framework/System/Windows/Controls/Border.cs,c3528d1bcccc5443 "Border") 的源码
~~~C#
using System.Windows.Media;
using System.Windows;
using WPFDevelopers.Utilities;
using System;

namespace WPFDevelopers.Helpers
{
    public static class GeometryHelper
    {
        public static void GenerateGeometry(StreamGeometryContext ctx, Rect rect, Radii radii)
        {
            var point = new Point(radii.LeftTop, 0.0);
            var point2 = new Point(rect.Width - radii.RightTop, 0.0);
            var point3 = new Point(rect.Width, radii.TopRight);
            var point4 = new Point(rect.Width, rect.Height - radii.BottomRight);
            var point5 = new Point(rect.Width - radii.RightBottom, rect.Height);
            var point6 = new Point(radii.LeftBottom, rect.Height);
            var point7 = new Point(0.0, rect.Height - radii.BottomLeft);
            var point8 = new Point(0.0, radii.TopLeft);
            if (point.X > point2.X)
            {
                var x = radii.LeftTop / (radii.LeftTop + radii.RightTop) * rect.Width;
                point.X = x;
                point2.X = x;
            }
            if (point3.Y > point4.Y)
            {
                var y = radii.TopRight / (radii.TopRight + radii.BottomRight) * rect.Height;
                point3.Y = y;
                point4.Y = y;
            }
            if (point5.X < point6.X)
            {
                var x2 = radii.LeftBottom / (radii.LeftBottom + radii.RightBottom) * rect.Width;
                point5.X = x2;
                point6.X = x2;
            }
            if (point7.Y < point8.Y)
            {
                var y2 = radii.TopLeft / (radii.TopLeft + radii.BottomLeft) * rect.Height;
                point7.Y = y2;
                point8.Y = y2;
            }
            var vector = new Vector(rect.TopLeft.X, rect.TopLeft.Y);
            point += vector;
            point2 += vector;
            point3 += vector;
            point4 += vector;
            point5 += vector;
            point6 += vector;
            point7 += vector;
            point8 += vector;
            ctx.BeginFigure(point, true, true);
            ctx.LineTo(point2, true, false);
            var width = rect.TopRight.X - point2.X;
            var height = point3.Y - rect.TopRight.Y;
            if (!DoubleUtil.IsZero(width) || !DoubleUtil.IsZero(height))
            {
                ctx.ArcTo(point3, new Size(width, height), 0.0, false, SweepDirection.Clockwise, true, false);
            }
            ctx.LineTo(point4, true, false);
            width = rect.BottomRight.X - point5.X;
            height = rect.BottomRight.Y - point4.Y;
            if (!DoubleUtil.IsZero(width) || !DoubleUtil.IsZero(height))
            {
                ctx.ArcTo(point5, new Size(width, height), 0.0, false, SweepDirection.Clockwise, true, false);
            }
            ctx.LineTo(point6, true, false);
            width = point6.X - rect.BottomLeft.X;
            height = rect.BottomLeft.Y - point7.Y;
            if (!DoubleUtil.IsZero(width) || !DoubleUtil.IsZero(height))
            {
                ctx.ArcTo(point7, new Size(width, height), 0.0, false, SweepDirection.Clockwise, true, false);
            }
            ctx.LineTo(point8, true, false);
            width = point.X - rect.TopLeft.X;
            height = point8.Y - rect.TopLeft.Y;
            if (!DoubleUtil.IsZero(width) || !DoubleUtil.IsZero(height))
            {
                ctx.ArcTo(point, new Size(width, height), 0.0, false, SweepDirection.Clockwise, true, false);
            }
        }
        public struct Radii
        {
            internal Radii(CornerRadius radii, Thickness borders, bool outer)
            {
                var left = 0.5 * borders.Left;
                var top = 0.5 * borders.Top;
                var right = 0.5 * borders.Right;
                var bottom = 0.5 * borders.Bottom;
                if (!outer)
                {
                    LeftTop = Math.Max(0.0, radii.TopLeft - left);
                    TopLeft = Math.Max(0.0, radii.TopLeft - top);
                    TopRight = Math.Max(0.0, radii.TopRight - top);
                    RightTop = Math.Max(0.0, radii.TopRight - right);
                    RightBottom = Math.Max(0.0, radii.BottomRight - right);
                    BottomRight = Math.Max(0.0, radii.BottomRight - bottom);
                    BottomLeft = Math.Max(0.0, radii.BottomLeft - bottom);
                    LeftBottom = Math.Max(0.0, radii.BottomLeft - left);
                    return;
                }
                if (DoubleUtil.IsZero(radii.TopLeft))
                {
                    LeftTop = (TopLeft = 0.0);
                }
                else
                {
                    LeftTop = radii.TopLeft + left;
                    TopLeft = radii.TopLeft + top;
                }
                if (DoubleUtil.IsZero(radii.TopRight))
                {
                    TopRight = (RightTop = 0.0);
                }
                else
                {
                    TopRight = radii.TopRight + top;
                    RightTop = radii.TopRight + right;
                }
                if (DoubleUtil.IsZero(radii.BottomRight))
                {
                    RightBottom = (BottomRight = 0.0);
                }
                else
                {
                    RightBottom = radii.BottomRight + right;
                    BottomRight = radii.BottomRight + bottom;
                }
                if (DoubleUtil.IsZero(radii.BottomLeft))
                {
                    BottomLeft = (LeftBottom = 0.0);
                    return;
                }
                BottomLeft = radii.BottomLeft + bottom;
                LeftBottom = radii.BottomLeft + left;
            }

            internal double LeftTop;

            internal double TopLeft;

            internal double TopRight;

            internal double RightTop;

            internal double RightBottom;

            internal double BottomRight;

            internal double BottomLeft;

            internal double LeftBottom;
        }
    }
}
~~~
3）`DataGrid.xaml` 代码如下：
~~~xml
 <Style TargetType="{x:Type DataGrid}">
     <Setter Property="Template">
         <Setter.Value>
             <ControlTemplate TargetType="{x:Type DataGrid}">
                 <WD:Border
                     Background="{TemplateBinding Background}"
                     BorderBrush="{TemplateBinding BorderBrush}"
                     BorderThickness="{TemplateBinding BorderThickness}"
                     CornerRadius="10" ClipToBounds="True">
                     <ScrollViewer x:Name="DG_ScrollViewer" Focusable="false" Clip="{Binding RelativeSource={RelativeSource AncestorType=WD:WDBorder}, Path=ContentClip}">
                         <ScrollViewer.Template>
                             <ControlTemplate TargetType="{x:Type ScrollViewer}">
                                 <ScrollContentPresenter />
                             </ControlTemplate>
                         </ScrollViewer.Template>
                         <ItemsPresenter SnapsToDevicePixels="{TemplateBinding SnapsToDevicePixels}" />
                     </ScrollViewer>
                 </Border>
             </ControlTemplate>
         </Setter.Value>
     </Setter>
 </Style>
~~~
![04f60e2434d39385b758051bae80ef2](https://github.com/user-attachments/assets/bc23210d-ab53-411a-9a7f-724cecde0d91)

当已经成功实现了 `DataGrid` 控件后，实现 `TreeView`、`TabControl` 等其他控件也可以按照类似的方法。
文中 `XAML` 中使用 [WPFDevelopers](https://www.nuget.org/packages/WPFDevelopers/ "WPFDevelopers") 库，如果直接拷贝使用，需要确保将相关的资源和控件进行正确的替换和配置。

[如果你对此有任何更好的想法或建议，我们将非常感激并乐于听取。](https://github.com/WPFDevelopersOrg/WPFDevelopers/issues/new "如果你对此有任何更好的想法或建议，我们将非常感激并乐于听取。")

