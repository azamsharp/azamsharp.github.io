# View is the View Model 

Introduction 


## Windows Presentation Foundation 

Long time ago, in a galaxy far away when I used to work as a .NET developer I worked on several WPF (Windows Presentation Foundation) projects. WPF is used to build Windows applications and it is the successor to Windows Forms. 

WPF used XAML (UI Language) to build the user interface for the applications. XAML looked similar to HTML and XML but also had the ability to refer to classes based in your C# or even VB.NET files. Binding was build into XAML files, this means a TextBox in XAML can be bound to a C# View Model. When user types text in the TextBox, it will automatically populate the properties in the View Model. 

The implementation of XAML user interface code is shown below: 

``` html 
<TextBox Text="{Binding Name}">
        <TextBox.BindingContext>
            <local:CustomerViewModel />
        </TextBox.BindingContext>
    </TextBox>
```

The TextBox contains the ```BindingContext```, which defines the context of the binding. TextBox binding is set to ```CustomerViewModel```. The TextBox Text property binds to the Name property of the ```CustomerViewModel```. 

The ```CustomerViewModel``` is shown below: 






