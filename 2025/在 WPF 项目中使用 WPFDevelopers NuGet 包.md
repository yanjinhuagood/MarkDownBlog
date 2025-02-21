<span style="display:block;text-align:center;">  **在 WPF 项目中使用 WPFDevelopers NuGet 包**</span> 
> 控件名：Install-Package WPFDevelopers -Version 1.1.0.3-preview 
>
> 作   者：WPFDevelopersOrg - **驚鏵**
>
>[原文链接](https://github.com/WPFDevelopersOrg/WPFDevelopers "原文链接")：https://github.com/WPFDevelopersOrg/WPFDevelopers
>
>[码云链接](https://gitee.com/WPFDevelopersOrg/WPFDevelopers "码云链接")：https://gitee.com/WPFDevelopersOrg/WPFDevelopers

- 框架支持`.NET4 至 .NET8`；
- `Visual Studio 2022`;

 
###### 1. 管理 **Nuget** 程序包 

![image](https://github.com/user-attachments/assets/21a0ce08-7666-477d-b9a0-9c0ee5a1cce2)

 
###### 2.搜索 **WPFDevelopers** ，并勾选**包括预发行版**，并安装最新包
![image](https://github.com/user-attachments/assets/0b979df3-8c55-4912-a62b-e177e1ee9f9e)


###### 3.**App.xaml** 引入 **WPFDevelopers** 
- 先引入命名空间
```
xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers"
```
- **Application.Resources** 新增 **WPFDevelopers** 资源
```xml
<Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <wd:Resources Theme="Light" Color="Fuchsia"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
```
![image](https://github.com/user-attachments/assets/8fc91337-bbdf-4b99-a234-94b1cbcbd493)
![image](https://github.com/user-attachments/assets/549918fd-2ca5-412b-8e68-b23f634a51b5)

##### 使用 **XAML** 示例
- 以 `DateRangePicker` 和 `Button` 为例
~~~xml
<Window
    x:Class="WDSample.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:wd="https://github.com/WPFDevelopersOrg/WPFDevelopers"
    Title="WPFDevelopersSample"
    Width="800"
    Height="450"
    mc:Ignorable="d">
    <Grid wd:PanelHelper.Spacing="2">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition />
        </Grid.RowDefinitions>
        <StackPanel
            HorizontalAlignment="Center"
            VerticalAlignment="Center"
            wd:PanelHelper.Spacing="2"
            Orientation="Horizontal">
            <Button Content="WPFDevelopers" />
            <Button
                wd:ElementHelper.CornerRadius="3"
                Content="WPFDevelopers"
                Style="{StaticResource WD.PrimaryButton}" />
        </StackPanel>
        <wd:DateRangePicker
            Grid.Row="1"
            Width="240"
            HorizontalAlignment="Center"
            VerticalAlignment="Center"
            wd:ElementHelper.CornerRadius="3"
            EndWatermark="结束时间"
            StartWatermark="开始时间" />
    </Grid>
</Window>
~~~
![image](https://github.com/user-attachments/assets/875486b0-3f93-45de-8bc9-a2f87d99a636)

![image](https://github.com/user-attachments/assets/0197ad3b-b23f-4b34-9c29-5692255a3485)

[GitHub 源码地址](https://github.com/WPFDevelopersOrg/WPFDevelopers/blob/dev/src/WPFDevelopers.Samples.Shared/App.xaml "GitHub 源码地址")

[Gitee 源码地址](https://gitee.com/WPFDevelopersOrg/WPFDevelopers/blob/dev/src/WPFDevelopers.Samples.Shared/App.xaml"Gitee "Gitee 源码地址")
