
# Configuring Global Routing in SwiftUI Using NavigationStack 

NavigationStack in iOS 16 allows developers to setup global routing for your application. Now, just like React and Flutter we can configure routes for our entire application in a central place. In this post, I will cover how you can setup global routing for your SwiftUI application. 

## Setting Up Global Routes 

We will start by creating an enum, which will represent all the routes in our application. The implementation is shown below: 

```swift 
enum Route: Hashable {
    case list
    case create(Animal)
    case detail(Animal)
}
```

Now, we can use NavigationStack to utilize the routes. This is shown below: 

```swift 
@main
struct LearnNavApp: App {
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                ContentView()
                    .navigationDestination(for: Route.self) { route in
                        switch route {
                            case .list:
                                Text("Show all animals")
                            case .create(let animal):
                                Text("Create an animal \(animal.name)")
                            case .detail(let animal):
                                Text("Detail \(animal.name)")
                        }
                    }
            }
                
        }
    }
}
```

We have injected the routes at the root level, so it will be available on all the views. The caller will look like this: 

```swift 
  NavigationLink("Go to Detail", value: Route.detail(Animal(name: "Rabbit")))
```

NavigationStack is a big improvement over NavigationView in SwiftUI. NavigationStack allows you to setup global routes and also allow programmatic navigation. If you want to learn more about NavigationStack then check out the following resurces. 

## Resources 

- [Video - First Look at the NavigationStack in iOS](https://youtu.be/3Ur6kUStKRo)
- [Video - Setting Global Routes Using NavigationStack](https://youtu.be/QZKA4fyJerI)
- [Video - Multi Column Layout Using NavigationStack](https://youtu.be/UPmffv18zP0)
- [Book - Navigation API in SwiftUI](https://azamsharp.gumroad.com/l/navigation-api-swiftui)

<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>


