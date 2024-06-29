<span style="display:block;text-align:center;">  **WPF 授权码显示密文并支持换行**</span> 
> WPF 授权码显示密文并支持换行
>
> 作 者：WPFDevelopersOrg - **驚鏵**

- 框架使用`.NET8`；
- `Visual Studio 2022`;


有开发者需要制作一个授权码输入框输入内容后显示密文并且能 `Enter` 进行换行输入。
由于无法使用 `PasswordBox` 控件本身不支持换行，因为它设计为单行输入控件。

所以最简单的方法是使用 `TextBox` 并通过自定义逻辑代码来掩盖输入的文本。这样可以实现多行输入，处理文本显示和密码掩码的逻辑。

接下来我们自定义一个控件 `MultiLinePasswordBox` 继承 `TextBox` 输入字符会以密码字符（如默认的 `●` ）显示。

##### 私有字段如下：
  - passwordBuilder 用于存储密码。
  - previousText 用于存放上一次的文本。
  - isUpdating 用于记录是否正在编辑。
  
##### 依赖属性如下：
  - PasswordChar：用于显示密码字符的字符，如果未显示则默认为`●`。
  - PlainText：控件中原始的未掩盖的文本。
  
##### 构造函数：
  - 设置 TextWrapping = TextWrapping.Wrap; 支持多行显示。
  - 设置 AcceptsReturn = true 支持按下 `Enter` 换行。
  - 订阅 TextChanged 事件，文本更改时将输入的文本转换为密码掩码，如果 isUpdating 的标志为 true ，则直接返回，避免重复更新文本。
  - 计算当前输入文本和上一次文本之间的长度差异，如果长度是正数则插入，反之长度差异是负数则删除。
  - 调用 CreateMaskedTextWithLineBreaks 方法创建带有掩码和换行符的文本。
  - 更新控件的文本，保持光标位置不变，并更新PlainText。

1）`MultiLinePasswordBox.cs` 代码如下：
~~~c#
using System.Text;
using System.Windows;
using System.Windows.Controls;

namespace WpfTextOrPasswordBox
{
    public class MultiLinePasswordBox : TextBox
    {
        private StringBuilder passwordBuilder = new StringBuilder();
        private string previousText = string.Empty;
        private bool isUpdating = false;

        public char PasswordChar
        {
            get { return (char)GetValue(PasswordCharProperty); }
            set { SetValue(PasswordCharProperty, value); }
        }

        public static readonly DependencyProperty PasswordCharProperty =
            DependencyProperty.Register("PasswordChar", typeof(char), typeof(MultiLinePasswordBox), new PropertyMetadata('●'));


        public string PlainText
        {
            get { return (string)GetValue(PlainTextProperty); }
            set { SetValue(PlainTextProperty, value); }
        }

        public static readonly DependencyProperty PlainTextProperty =
            DependencyProperty.Register("PlainText", typeof(string), typeof(MultiLinePasswordBox), new PropertyMetadata(string.Empty));


        public MultiLinePasswordBox()
        {
            AcceptsReturn = true;
            TextWrapping = TextWrapping.Wrap;
            VerticalScrollBarVisibility = ScrollBarVisibility.Auto;
            TextChanged += PasswordTextBox_TextChanged;
        }

        private void PasswordTextBox_TextChanged(object sender, TextChangedEventArgs e)
        {
            if (isUpdating)
                return;

            isUpdating = true;
            var caretIndex = CaretIndex;
            var input = Text;
            if (string.IsNullOrWhiteSpace(Text))
            {
                passwordBuilder.Clear();
            }
            else
            {
                int lengthDifference = input.Length - previousText.Length;
                if (lengthDifference > 0)
                {
                    var newText = input.Substring(caretIndex - lengthDifference, lengthDifference);
                    passwordBuilder.Insert(caretIndex - lengthDifference, newText);
                }
                else if (lengthDifference < 0)
                {
                    passwordBuilder.Remove(caretIndex, Math.Abs(lengthDifference));
                }
                
                var maskedText = CreateMaskedTextWithLineBreaks(passwordBuilder.ToString());
                TextChanged -= PasswordTextBox_TextChanged;
                Text = maskedText;
                TextChanged += PasswordTextBox_TextChanged;
            }
            previousText = Text;
            CaretIndex = caretIndex;
            PlainText = passwordBuilder.ToString();
            isUpdating = false;
        }

        private string CreateMaskedTextWithLineBreaks(string text)
        {
            var maskedText = new StringBuilder();
            foreach (char c in text)
            {
                if (c == '\r' || c == '\n')
                    maskedText.Append(c);
                else
					maskedText.Append(PasswordChar.ToString());
            }
            return maskedText.ToString();
        }
    }
}
~~~~
2）`MultiLinePasswordBoxSample.xaml` 代码如下：
~~~xml
<wd:Window
    x:Class="WpfTextOrPasswordBox.MultiLinePasswordBoxSample"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:local="clr-namespace:WpfTextOrPasswordBox"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers"
    Title="MultiLinePasswordBoxSample - WPF开发者"
    Width="800"
    Height="450"
    WindowStartupLocation="CenterScreen"
    mc:Ignorable="d">
    <Grid>
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
            <TextBlock Margin="0,20">
                <Run Text="明文：" />
                <Run Text="{Binding ElementName=myMultiLinePasswordBox, Path=PlainText}" />
            </TextBlock>
            <local:MultiLinePasswordBox
                x:Name="myMultiLinePasswordBox"
                Width="200"
                Height="60"
                wd:ElementHelper.CornerRadius="3"
                wd:ElementHelper.Watermark="请输入授权码" />
        </StackPanel>
    </Grid>
</wd:Window>

~~~

![MultiLinePasswordBoxNew](https://github.com/yanjinhuagood/MarkDown/assets/23089734/fa71e176-c3de-4e96-af2a-b3665dc27dc5)


[如果你对此有任何更好的想法或建议，我们将非常感激并乐于听取。](https://github.com/WPFDevelopersOrg/WPFDevelopers)

