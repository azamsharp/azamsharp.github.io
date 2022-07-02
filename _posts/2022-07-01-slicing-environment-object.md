
# Slicing Global State in SwiftUI Using Multiple EnvironmentObjects 

@EnvironmentObject in SwiftUI provides a way to configure global state for your application. Updating the global state, allows the views to re-render/refresh. Sometimes we are only interested to update a view when a small part of the global state changes. In this post, I will cover how you can create segments of your global state so your view only updates when that slice is changes. 

## Scenario 

Let's consider a scenario where we have to keep track of a global counter and also global isAuthenticated state. We can start with creating our global state as shown below: 

```swift 
class AppState: ObservableObject {
    @Published var counter: Int = 0
    @Published var isAuthenticated: Bool = false 
}
```

The ```AppState``` class conforms to the ```ObservableObject``` and consists of ```counter``` and ```isAuthenticated``` properties. Both properties are decorated with ```@Published``` property wrapper, which means when the property is set, it will notify the view so the view can be re-evaluated and re-rendered. 

Next, we will inject our AppState as an environment object. This is mostly performed on the root view, so that the environment object is available to the root view and all the child views of the root view. 

```swift
@main
struct EnvObjectsLearnApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView().environmentObject(AppState())
        }
    }
}
```

Now, we can use environment object in our views as shown below: 

```swift 
struct CounterView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            Text("\(appState.counter)")
            Button("Increment") {
                appState.counter += 1
            }
        }
    }
}
```

When you tap the **Increment** button, it will update the global state and render the view again. 

This works as expected! 

Things get interesting when we add another view, which also read and change values from the global state. The implementation of ```AuthenticationView``` is shown below: 

```swift 
struct AuthenticationView: View {
    
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            Text(appState.isAuthenticated ? "AUTHENTICATED": "NOT AUTHENTICATED")
            Button("Toggle Authentication") {
                appState.isAuthenticated.toggle()
            }
        }
    }
}
```

The **AuthenticationView** works as expected. When you press the **Toggle Authentication** button, it toggles the isAuthenticated global state. 

Since **CounterView** and **AuthenticationView** are both reading the values from the global state, any time the counter is getting updated, it will render the CounterView but it will also render the AuthenticationView. You can easily check which views are getting re-rendered, if you place ```Self._printChanges()``` inside the body property. 

After placing the ```Self._printChanges``` function in the body of both CounterView and AuthenticationView, go ahead and tap on the **Increment** button. You will notice that when you update the counter state, both CounterView and AuthenticationView are rendered again. Here is the output of the printChanges function.  

```
AuthenticationView: _appState changed.
CounterView: _appState changed.
```

Most probably you don't want this to happen. You don't want to re-render the AuthenticationView, when the counter global state is updated. AuthenticationView renders again because it is also reading the values from the same @EnvironmentObject. It there were more views using @EnvironmentObject to display some values then all of them will render again, when any value in global state changes. 

Let's see how we can avoid this by dividing global state into multiple slices. 

## Slicing Global State 

We will start by creating different classes, which will represent different slices of the global state. This is shown below where we have divided our ```AppState``` into ```CounterState``` and ```AuthenticationState```.  

```swift
class AppState: ObservableObject {
    var counterState = CounterState()
    var authenticationState = AuthenticationState()
}

class CounterState: ObservableObject {
    @Published var counter: Int = 0
}

class AuthenticationState: ObservableObject {
    @Published var isAuthenticated: Bool = false
}
```

The ```CounterState``` only manages the counter state and the ```AuthenticationState``` only manages the isAuthenticated state. 

Next, we will update our views so they can read state from the slices they are interested in. This is shown below: 

```swift 
struct AuthenticationView: View {
    
    @EnvironmentObject var authenticationState: AuthenticationState
    
    var body: some View {
        let _ = Self._printChanges()
        VStack {
            Text(authenticationState.isAuthenticated ? "AUTHENTICATED": "NOT AUTHENTICATED")
            Button("Toggle Authentication") {
                authenticationState.isAuthenticated.toggle()
            }
        }
    }
}

struct CounterView: View {
    
    @EnvironmentObject var counterState: CounterState
    
    var body: some View {
        let _ = Self._printChanges()
        VStack {
            Text("\(counterState.counter)")
            Button("Increment") {
                counterState.counter += 1
            }
        }
    }
}

```

Finally, we will inject the slices to our app as shown below: 

```swift 
@main
struct EnvObjectsLearnApp: App {
    
    let appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
                .environmentObject(appState.counterState)
                .environmentObject(appState.authenticationState)
        }
    }
}
```

Now, if you run the app and update the counter you will notice that only CounterView is re-rendered again and not AuthenticationView. This is great, because that is exactly what we wanted. You can do the same for AuthenticationView. If you toggle isAuthentication state, you will notice that only AuthenticationView is rendered again and not the CounterView. This will prevent unnecesary renders of the views. 

## Conclusion

@EnvironmentObject provides an excellent way to setup global state for your application. Having said that you need to be extra careful when using global state, since it can effect a lot of views. Slicing the global state gives you more control of global state and it allows to update only the views where necessary. 

<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>