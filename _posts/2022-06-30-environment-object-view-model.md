
# @EnvironmentObject in Views May Not be a Good Idea But Avoiding Them is Probably Much Worse 

In SwiftUI, ```@EnvironmentObject``` allows you to create global state that can be shared and manipulated from any view in your application. We tend to put @EnvironmentObject in our views and directly access the global state. This creates a tight coupling between the view and the ```@EnvironmentObject```, but avoiding this approach opens up a whole new cans of worms. In this article, I will discuss how @EnvironmentObject can be used in a SwiftUI view and how avoiding it can create dependency nightmare. 

## EnvironmentObject Counter Example: 

Let's create a simple counter example. Our app will consists of three views. The **ContentView** will be the parent view, which will host **IncrementCounterView** and **DisplayCounterView**. IncrementCounterView will increment the global state counter value and DisplayCounterView will display the value from the global state. 

The global state class ```AppState``` is shown below: 

```swift 
class AppState: ObservableObject {
    @Published var value: Int = 0
}
```

The AppState class conforms to the ```ObservableObject``` protocol and it consists of a single property called value. The value property is responsible for keeping track of the counter. The ```@Published``` property wrapper allows to publish the changes to the value property, which can be used by the view to render themselves.  

The ContentView acts as the root of the application and host our other child views. 

```swift 
struct ContentView: View {
    
    var body: some View {
        VStack {
            DisplayCounterView()
            IncrementCounterView()
        }
    }
}
// inject AppState from the App file
ContentView().environmentObject(AppState())
```

Each child view uses ```@EnvironmentObject``` to access the global state. 

```swift 
struct DisplayCounterView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        Text("\(appState.value)")
    }
}

struct IncrementCounterView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            Button("Increment") {
                appState.value += 1
            }
        }
    }
}
```

The IncrementCounterView increments the counter and DisplayCounterView displays the counter. 

If you run the app and taps the "Increment" button then the counter will be updated and displayed on the screen. 

Everything works as expected! So what is the problem? 

The main problem is that we have introduced a tight coupling between the view and the ```@EnvironmentObject```. Now the view must always use the EnvironmentObject to access the values. We also cannot use the view in other scenarios. Let's see how we can solve this problem by accessing the EnvironmentObject behind a View Model. 

## Implementing View Models

We will start by creating two view models. One for the ```IncrementCounterView``` and one for the ```DisplayCounterView```. This is shown below: 

```swift 
class IncrementCounterViewModel {
    
    var appState: AppState
    
    init(appState: AppState) {
        self.appState = appState
    }
    
    func increment() {
        appState.value += 1
    }
}

class DisplayCounterViewModel {
    
   private var appState: AppState
    
    init(appState: AppState) {
        self.appState = appState
    }
    
    var counter: Int {
        appState.value
    }
}
```

The ```IncrementCounterViewModel``` and ```DisplayCounterViewModel``` both takes AppState as their argument. This means if you want to create either of the view models then you must pass in the AppState. This is shown below: 

```swift 
struct ContentView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            DisplayCounterView(vm: DisplayCounterViewModel(appState: appState))
            IncrementCounterView(vm: IncrementCounterViewModel(appState: appState))
        }
    }
}
```

Inside the ContentView, we get access to the global state using @EnvironmentObject. After that we pass the global object to the DisplayCounterViewModel and IncrementCounterViewModel. 

If you run the app, it will work as expected and the counter value will be updated. Keep in mind that whenever the counter value is changed the ContentView is rendered again. This also allows the DisplayCounterView and IncrementCounterView to be rendered again. If you have another view inside the ContentView, which is not using the appState then it will never be rendered twice. 

You can also introduce protocols for your view models, this will unable your views to not depend on a concrete implementation, making them less coupled. 

```swift 
protocol IncrementCounterViewModelProtocol {
    func increment()
}

protocol DisplayCounterViewModelProtocol {
    var counter: Int { get }
}
```

Update your view models to conform to the protocols. 

```swift 
class IncrementCounterViewModel: IncrementCounterViewModelProtocol {
    
    // ... code here 
    
}

class DisplayCounterViewModel: DisplayCounterViewModelProtocol {
    
   // ... code here 
    
}
```

The IncrementCounterView and DisplayCounterView will also needs to be updated to make use of the new protocols. 

```swift 
struct DisplayCounterView: View {
    
    let vm: DisplayCounterViewModelProtocol
    
    init(vm: any DisplayCounterViewModelProtocol) {
        self.vm = vm
    }
    
    // code here 
}

struct IncrementCounterView: View {
    
    let vm: IncrementCounterViewModelProtocol
    
    init(vm: any IncrementCounterViewModelProtocol) {
        self.vm = vm
    }
    
    // code here 
}
```

The scenario that I painted above is quite simple. In real world, you will have deeply nested views and some great grand child view will need access to the value stored in the global state. 

Let's take a look at another example. 

```swift 
struct FancyCounterView: View {
    var body: some View {
        Text("Display the counter value here")
    }
}

struct FancyView: View {
    var body: some View {
        FancyCounterView()
    }
}


struct ContentView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            FancyView()
        }
    }
}
```

The ContentView contains a FancyView. The FancyView contains a FancyCounterView. We need to access and display the global state counter in the FancyCounterView. We can create view models and pass the state, just like we did before. 

```swift 
struct FancyCounterView: View {
    
    let counter: Int
    
    var body: some View {
        Text("\(counter)")
    }
}

struct FancyView: View {
    
    let vm: FancyViewModel
    
    var body: some View {
        FancyCounterView(counter: vm.counter)
    }
}

class FancyViewModel {
    
    private var appState: AppState
     
    init(appState: AppState) {
        self.appState = appState
    }
    
    var counter: Int {
        self.appState.value
    }
}

struct ContentView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        
        VStack {
            FancyView(vm: FancyViewModel(appState: appState))
        }
    }
}
```

This works as expected but as you can see it quickly becomes a dependency injection nightmare. What if the view was 5-8 levels deep? Will we end up passing the value from the parent to child to grand child to great grand child and so on? 

```swift 
struct GrandChildView: View {
    var body: some View {
        // This view needs the counter value
        Text("GrandChildView")
    }
}

struct ChildView: View {
    var body: some View {
        GrandChildView()
    }
}

struct ParentView: View {
    var body: some View {
        ChildView()
    }
}
```

We can all agree the easiest approach would be to allow GrandChildView to directly read values from the @EnvironmentObject like shown below: 

```swift 
struct GrandChildView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        Text("\(appState.value)")
    }
}
```


This allowed our GrandChildView to easily access the global state through the use of @EnvironmentObject. We did ended up adding a dependency on @EnvironmentObject inside our view but the alternate solution of adding multiple view models and then passing values down from parent to child and then grand child was much worse.  

[Source Code](https://gist.github.com/azamsharp/6b02a5ee5247979cf24efadadbfb8692)

## Conclusion

In this post, you learned that using @EnvironmentObject in a view is a much better solution than passing data down to the child views through the parent. Hopefully, in the future SwiftUI will allow us to use @EnvironmentObject directly from the view models. 

Happy coding! 



























<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>