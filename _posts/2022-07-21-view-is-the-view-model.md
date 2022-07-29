# SwiftUI View is also a View Model 

In this post, I will cover how a SwiftUI View is not only a view but also a ViewModel. We will compare it with WPF framework and see how SwiftUI already has built-in support for ViewModel right within the view. This means that in most cases, you don't need to create an extra layer of View Model per screen.    

> Make sure to check out the resources section. I have created several YouTube videos to answer questions related to Networking, Validation, Business Layer etc. 

## Windows Presentation Foundation 

Long time ago, in a galaxy far away when I was a .NET developer, I worked on several WPF (Windows Presentation Foundation) projects. WPF is used to build Windows applications and it is the successor to Windows Forms. 

WPF used XAML (UI Language) to build the user interface for the applications. XAML looked similar to HTML and XML but also had the ability to refer to classes in C# or even VB.NET files. Binding was built into XAML files, this means a TextBox in XAML can be bind to a C# View Model. When user types text in the TextBox, it will automatically populate the properties of the View Model. 

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

``` csharp 
public class CustomerViewModel {
    public String Name { get; set; }
}
```

> Code above is used for demonstration purposes. But the working XAML and C# code is very similar to the one mentioned above.  
 
Now, when the user types in the TextBox declared in XAML, the CustomerViewModel ```Name``` property is automatically populated due to the binding features of XAML. Now, let's implement the same scenario in SwiftUI application. 

In SwiftUI, the implementation is quite simple as shown below: 

``` swift 
struct CustomerAddScreen: View {
    
    @State private var name: String = ""
    
    var body: some View {
        TextField("Name", text: $name)
    }
}
```

We don't need a separate View Model like ```CustomerViewModel``` to bind the value of TextField. SwiftUI view already supports data binding features like ```@State```, ```@Binding``` etc. This means SwiftUI view is not only a View but also a View Model. Although you can add another layer of abstraction in the form of a View Model, it is not needed in most cases. 

Check out the comparison between WPF and SwiftUI below: 

![SwiftUI vs WPF](/images/view-is-view-model.png)

The basic client side validation like required and matching fields can be performed right within your view. For more complicated client side validation, I suggest implementing some sort of ```RulesEngine```. 

Using this architecture the View acts as a View and also a View Model. The Model becomes the business layer (specially for apps with no server component). This means if you create a ```NetworkService```, it can be directly called from the View, which is also a View Model. I personally, create an aggregate root model object and then call the web service layer from there. This allows me to handle business logic in my aggregate root model or the models that it is hosting. Below you can find a screenshot from Apple's [article](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app), which also confirms that the View Model is not needed in most scenarios when building SwiftUI applications. This is because the data binding features are baked right into the View, which is also serving as a View Model. 

![Managing model data in your app](/images/data-model.png)

Even if you read Apple docs or watch WWDC videos, you will never hear the term MVVM or you will never see Apple engineers using another layer of abstraction. They will mostly implement StoreManagers or StoreService and then call it directly from the View (ViewModel) and populate the global state so they can maintain a single source of truth.  

Having said that there are some situations in which a flatter model is needed. A good example is if you are working with MKLocalSearch and it returns ```MKPlacemark```. You probably want to expose a much leaner object to the view instead of ```MKPlacemark``` instance. 

One of the biggest issues I encountered with ViewModel for each screen approach is keeping track of ```EnvironmentObject``` changes. Yes, you can pass ```EnvironmentObject``` to the ViewModel or you can inject it using a dependency injection framework but why make things so complicated. Why not use ```EnvironmentObject``` from within the View (ViewModel) like it is supposed to be used. 

> No one architecture fits all the needs. Next time you are writing your SwiftUI application, think about adding a ViewModel per screen. Is it going to provide you benefits or will it become a hurdle in your application. 

## Resources 

1. [I was wrong! MVVM is NOT a good choice for building SwiftUI applications](https://azamsharp.com/2022/07/17/2022-swiftui-and-mvvm.html)
2. [Stop Using MVVM with SwiftUI](https://youtu.be/LVx93PfGjdo)
3. [Consuming JSON API in SwiftUI Using State Pattern (Not Using MVVM)](https://youtu.be/YOCZuZz4vAw)
4. [Client Server SwiftUI App Using State Pattern](https://youtu.be/j2x7GylAnmE)


<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>




