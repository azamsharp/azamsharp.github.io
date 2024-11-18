
# Deep Dive into Environment in SwiftUI 

// TODO 

### Observable Object Protocol

Before the introduction of the `Observation` framework in iOS 17, SwiftUI developers primarily used the `ObservableObject` protocol to create new sources of truth for their applications. 

> I will discuss Observation framework later. 

The ObservableObject protocol is a fundamental feature in SwiftUI used to create and manage shared data that can be observed by SwiftUI views. By conforming to ObservableObject, a class becomes a source of truth that SwiftUI views can automatically monitor for changes, allowing the views to update their UI dynamically when the data changes.

One of the benefits of a class conforming to ```ObservableObject``` protocol is that the instance of the class can be placed in the Environment Object, making it available globally in the views. Globally is a loaded term since the availability of the Environment Object depends on where in the hierarchy the object was injected. 

Take a look at the following hierarchy. 

![SwiftUI View Hierarchy](/images/env-1.png)

The root view has two children and each child has further two children. If we inject an environment object at the root of the application then it will be available to all the views, including the root view. This is shown in the following screenshot. 

![Injecting Environment Object](/images/env-2.png)

The environment object is injected at the root of the application, this means it is available in the root view and all the other views that are children of the root view. 

If environment object was injected in one of the child views then it will only be available to that child and all the children of that child view. This is shown in the following screenshot. 

![Injecting Environment Object](/images/env-3.png)

You are also not limited to a single environment object. Based on your needs you can inject multiple environment objects at different or even same points of your application. This is shown in the screenshot below: 

![Multiple Environment Objects](/images/env-4.png)

Why would you choose to inject different environment objects at various points in your SwiftUI view hierarchy? Consider a scenario where you're building a hospital app with multiple tabs such as **Home**, **Patients**, and **Doctors**. 

In this case, each tab can have its own dedicated data store. For instance, the **Patients** tab could use a `PatientStore`, while the **Doctors** tab could use a `DoctorStore`. These stores manage data specific to their respective contexts and can be injected into their corresponding parts of the view hierarchy. 

Both `PatientStore` and `DoctorStore` could rely on a shared service layer to fetch the necessary data, ensuring a clean separation of concerns and efficient data handling tailored to each feature in the app. This approach promotes modularity, encapsulates dependencies, and simplifies state management within a complex SwiftUI application.

However, a common practice among developers is to inject all environment objects at the root of the application. This makes them accessible from any part of the app's view hierarchy. While convenient, this approach can lead to tightly coupled components and potentially unnecessary dependencies being available everywhere, even in parts of the app that don’t require them. This overexposure can make the application harder to maintain and test, especially as it grows in complexity.

![Environment Objects Injected at the Root](/images/env-5.png)

## Re-Evaluation (Diffing) vs Re-Rendering  

One of the confusion regarding using environment objects is that developers believe that when using environment object, it will automatically re-render all the views that are dependent on it. In order to understand this we first must understand the difference between re-evaluation and re-rendering. 

In SwiftUI, when the state changes, the framework triggers a process called **re-evaluation** for the affected views. During this process, SwiftUI determines which parts of the view hierarchy need to be updated. 

Re-evaluation involves **diffing**, where SwiftUI compares the previous and current versions of the views to identify what has changed and what has remained the same. This efficient mechanism ensures that only the necessary views are updated, optimizing performance and minimizing the impact of state changes on the user interface. This declarative approach makes UI updates seamless and reactive to state changes in your application.

Take a look at the following code: 

``` swift 
struct CounterView: View {
    
    @State private var count: Int = 0
    
    var body: some View {
        let _ = Self._printChanges()
        VStack {
            Text("\(count)")
            Button("Increment Counter") {
                count += 1
            }
            
            List(1...20, id: \.self) { index in
                Text("\(index)")
            }
        }
    }
}
```

When you press the button, the counter is incremented, and the view's `body` is re-evaluated. Inside the `body`, the line `let _ = Self._printChanges()` is executed, which logs information about changes whenever the `body` is re-evaluated. 

> It's important to understand that **re-evaluation of the body does not mean all views within it are re-rendered.** and usually the process of diffing is highly optimized and is performed really quickly. 

In this example, only the `Text("\(count)")` view will be re-rendered because it directly depends on the `count` property. When `count` changes, SwiftUI uses its **diffing** mechanism to determine that this specific `Text` view needs updating, while other views in the hierarchy remain unchanged. This targeted update ensures efficient rendering and performance optimization.

### Environment Object and View Rendering 

Let’s explore a simple example of using an **Environment Object** (iOS 16 and earlier) to understand its behavior during view rendering. Below is the implementation of a `Store` class that conforms to the `ObservableObject` protocol. This class has a `count` property, and the `ObservableObject` conformance makes the `Store` class a **source of truth**, enabling SwiftUI views to observe and react to changes in its state.

```swift
@MainActor
class Store: ObservableObject {
    @Published var count: Int = 0
}
```

If we want to use `Store` as a shared global state, we can inject it into a specific view. Here’s how it’s done:

```swift
#Preview {
    ContentScreen()
        .environmentObject(Store())
}
```

This makes the `Store` instance accessible to the `ContentScreen` view and all its child views. By doing so, any view within this hierarchy can use the `Store` instance without needing to pass it explicitly as a parameter.

To use the `Store` within a view, we can retrieve it using the `@EnvironmentObject` property wrapper. Here's an example:

```swift
struct ContentScreen: View {
    
    @EnvironmentObject private var store: Store
    
    var body: some View {
        
        let _ = Self._printChanges()
        
        VStack {
            Text("\(store.count)")
            Button("Increment") {
                store.count += 1
            }
            
        }
    }
}
```

When button is pressed, count property of ```Store``` is updated and displayed in a Text view. 

Now, let's take a look at an example of what will happen if you add subviews to the ```ContentScreen```, which also accesses ```Store``` environment object. 

We have added two new subviews inside the ```ContentScreen```. The implementation is shown below: 

``` swift 
struct NumberListView: View {
    
    @EnvironmentObject private var store: Store
    
    var body: some View {
        let _ = Self._printChanges()
        VStack {
            Text("NumberListView")
        }
    }
}

struct LightBulbView: View {
    
    @EnvironmentObject private var store: Store
    
    var body: some View {
        let _ = Self._printChanges()
        VStack {
            Text("LightBulbView")
        }
    }
}
```

One interesting thing to note about both the views is that even though they have a reference to the environment object, none of them utilize any properties of the ```Store``` class in their body. 

The ```ContentScreen``` is updated to the following implementation: 

``` swift 
struct ContentScreen: View {
    
    @EnvironmentObject private var store: Store
    
    var body: some View {
        
        let _ = Self._printChanges()
        
        VStack {
            Text("\(store.count)")
            Button("Increment") {
                store.count += 1
            }
            
            NumberListView()
            LightBulbView()
            
        }
    }
}
```

Now, if you run the app and tap on the **Increment** button then you will see the following output. 

``` swift 
ContentScreen: _store changed.
NumberListView: _store changed.
LightBulbView: _store changed.
```

This means when you increment the number. All three views are getting re-evaluated and diffed. 

> Once again this does not mean that all three views are getting re-rendered. Only views that are changed will be re-rendered. 

To address the issue, we can improve our implementation by passing only the specific dependencies needed by each subview. This ensures that each subview is independent and only depends on the data it requires. For instance:

**NumberListView** requires the count value.

**LightBulbView** requires the isOn state.

Here’s the updated implementation of these subviews:

```swift
struct NumberListView: View {
    
    let count: Int
    
    var body: some View {
        let _ = Self._printChanges() // Logs view re-evaluation
        VStack {
            Text("\(count)")
        }
    }
}

struct LightBulbView: View {
    
    let isOn: Bool
    
    var body: some View {
        let _ = Self._printChanges() // Logs view re-evaluation
        VStack {
            Text(isOn ? "ON" : "OFF")
        }
    }
}
```

The ```Store``` class is updated to include the ```isOn``` property, which tracks the on/off state:

``` swift 
@MainActor 
class Store: ObservableObject {
    @Published var count: Int = 0
    @Published var isOn: Bool = false
}
```

The ContentScreen now passes only the required data to each subview:

``` swift
struct ContentScreen: View {
    
    @EnvironmentObject private var store: Store
    
    var body: some View {
        let _ = Self._printChanges() // Logs view re-evaluation
        
        VStack {
            Text("\(store.count)")
            
            Button("Add Number") {
                store.count += 1
            }
            
            Toggle(isOn: $store.isOn) {
                EmptyView()
            }
            .fixedSize()
            
            // Passing only required data to subviews
            NumberListView(count: store.count)
            LightBulbView(isOn: store.isOn)
        }
    }
}

```

This updated implementation improves the design by decoupling subviews, ensuring they rely only on the data they require, such as `NumberListView` depending solely on `count` and `LightBulbView` on `isOn`. This makes the subviews more reusable and easier to test. Additionally, the explicit passing of dependencies reduces reliance on `EnvironmentObject`, avoiding tight coupling and improving clarity. 

Performance is also enhanced, as SwiftUI efficiently re-evaluates and updates only the affected parts of the UI, ensuring that changes in `store.count` or `store.isOn` do not trigger unnecessary re-renders elsewhere in the view hierarchy. This approach results in cleaner, more maintainable, and performance-optimized code.

> It’s important to understand that in SwiftUI, the natural flow of data is from parent views to child views. Typically, the parent view is indicated by a name ending with the suffix `Screen`, while child views are represented with the suffix `View`. The key principle is to have the parent view handle data loading and then pass only the necessary data down to its child views. This approach ensures clarity, reduces unnecessary dependencies, and promotes reusable, modular components within the view hierarchy.

### Environment Object and Navigation Stack 

One of the powerful benefits of using `EnvironmentObject` in SwiftUI is that all screens within a `NavigationStack` automatically stay in sync with the shared data. This happens because all screens access the same global state, eliminating the need for manual updates. 

In contrast, a common anti-pattern seen in the past is developers passing refresh flags between screens to ensure data synchronization after adding or updating an item. This approach not only adds unnecessary complexity but also indicates a misunderstanding of SwiftUI’s declarative nature. With `EnvironmentObject`, such workarounds become unnecessary, as SwiftUI handles state updates seamlessly, ensuring a clean and efficient implementation.

This means that if your `EnvironmentObject` contains state used across 20 different screens within a `NavigationStack`, all those screens will automatically stay in sync without any additional effort on your part. SwiftUI ensures that all screens in the stack share and react to the same global state.

The only adjustment you need to make is to inject the `Store` at the `NavigationStack` level, instead of at an individual screen level like `ContentScreen`. Here’s how you can do it:

```swift
NavigationStack {
    ContentScreen()
}
.environmentObject(Store())
```

By injecting the `Store` at the `NavigationStack`, it becomes available to all screens in the navigation hierarchy, ensuring seamless state management and reducing boilerplate code.

Keep in mind that the same diffing rules apply to screens inside the NavigationStack. This means that if you have 20 screens in the stack, and all of them use ```@EnvironmentObject```, each screen will be re-evaluated during a state change, even if they don’t directly depend on or use any properties from the global state. 

After re-evaluation, only the views that depend on the state change will get re-rendered. 

> Developers often worry about screens in a `NavigationStack` being re-evaluated due to `@EnvironmentObject`. However, the more pertinent question to ask is: how many screens are typically in your `NavigationStack` at any given time?

### New @Observable Macro

In iOS 17, Apple introduced the **Observation** framework, offering a cleaner syntax and significant performance improvements. One key enhancement is that views that do not actively use the environment state properties are no longer re-evaluated, further optimizing SwiftUI’s rendering process. This improvement ensures that only views dependent on specific state changes are updated, reducing unnecessary computations and enhancing app performance.

In order to update our current code to use Observation framework, we will start with the ```Store```. 

``` swift 
@MainActor
@Observable
class Store {
    var count: Int = 0
    var isOn: Bool = false
}
```

We don't need to conform to ```ObservableObject``` anymore. We can simply use the ```@Observable``` macro. Also, if you noticed the ```@Published``` keyword has been removed. Both stored properties ```count``` and ```isOn``` are automatically published properties. 

### Environment Usages





