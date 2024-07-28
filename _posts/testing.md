
# Navigation Patterns in SwiftUI  

Navigation has always been a pain point in building SwiftUI applications. When SwiftUI was introduced, it came with NavigationView, which was later replaced by NavigationStack in iOS 16.

NavigationStack allowed developers to perform dynamic/programmatic routes and even provided ways to centralize routes for the whole application. In this article, I will cover common navigation patterns that you can use when building SwiftUI applications.

### Basic List Navigation  

One of the most common patterns for navigation involves basic List navigation. This is where a user taps on an item in the list, which takes the user to the destination screen. There are multiple ways to perform list navigation. 

In the following implementation we have used ```NavigationLink``` to represent the destination and the label. Once the user taps on the customer name, they are taken to the CustomerDetailScreen. 

``` swift 
struct ContentView: View {
    
    let customers = [Customer(id: 1, name: "John Doe"), Customer(id: 2, name: "Mary Doe")]
    
    var body: some View {
        List(customers) { customer in
            NavigationLink {
                CustomerDetailScreen(customer: customer)
            } label: {
                Text(customer.name)
            }
        }
    }
}
```

Keep in mind that the initializer of CustomerDetailScreen's initializer is called repeatedly for every single item in the list. It is recommended that you do not perform resource intensive operation in the initializer of the view. 

> If you like to perform a network call inside a view then use the task view modifier. The task modifier is async in nature and the task is automatically cancelled when the view no longer exists.  

Another option is to use the ```navigationDestination```. The implementation is shown below: 

``` swift 
struct ContentView: View {
    
    let customers = [Customer(id: 1, name: "John Doe"), Customer(id: 2, name: "Mary Doe")]
    
    var body: some View {
        List(customers) { customer in
            NavigationLink(customer.name, value: customer)
        }.navigationDestination(for: Customer.self) { customer in
            CustomerDetailScreen(customer: customer)
        }
    }
}
```

> When using NavigationLink with value parameter, we also need to make sure that Customer is Hashable. 

The ```navigationDestination``` approach relies on the type of the value that triggered the navigation. The NavigationLink value property is set to the customer. The navigationDestination is also setup for the Customer type using the for: Customer.self argument. This allows the navigationDestination view modifier to trigger whenever a customer value is set. 

Both options work and either one can be used in this scenario. I would personally use the first approach and not the navigationDestination because it is more clearer and simple to understand. 

Later in this article, I will demonstrate how to use ```navigationDestination``` when constructing dynamic routing in SwiftUI. 

### Dynamic/Programmatic Routing 

NavigationStack also supports dynamic or programmatic routing. This means you can programmatically add or remove routes from the NavigationStack. This can be useful in scenarios where you have performed an asynchronous action and, based on the result of that action, you want to direct the user to a particular destination. A very common example is a user signing in and then navigating to the dashboard screen after the successful sign in. 

Dynamic routes can be achieved in many different ways. A common approach is to use an enum to represent all the available dynamic routes. The implementation is shown below: 

``` swift 
enum Route: Hashable {
    case dashboard
    case detail(Customer)
}
```

Next, we can use the NavigationStack ```path``` parameter to initialize our route binding. This is shown in the implementation below: 

``` swift 
struct ContentView: View {
    
    @State private var routes: [Route] = []
    
    var body: some View {
        NavigationStack(path: $routes) {
            VStack {
                Button("Login") {
                    Task {
                        // perform async call
                        try! await Task.sleep(for: .seconds(2))
                        // take user to dashboard screen
                        routes.append(.dashboard)
                    }
                }
            }.navigationDestination(for: Route.self) { route in
                switch route {
                    case .dashboard:
                        DashboardScreen() 
                    case .detail(let customer):
                        CustomerDetailScreen(customer: customer)
                }
            }
        }
    }
}
```

We start by creating the routes array as a private state. Once the Login button is tapped, we fake a call by waiting for 2 seconds and then add the dashboard route to the routes array. This causes the ```navigationDestination``` to trigger. Inside the ```navigationDestination``` we get access to the route enum and based on the selected case we direct the user to the correct destination screen.  

Based on the complexity of your app, you can also organize your routes as nested enums. This can be helpful if your routing is based on different sections of the app. The implementation is shown below: 

``` swift 
enum Route: Hashable {
    
    case patient(PatientRoute)
    case doctor(DoctorRoute)
    
    enum PatientRoute: Hashable {
        case list
        case create
        case detail(Patient)
    }
    
    enum DoctorRoute: Hashable {
        case list
        case create
        case detail(Doctor)
    }
}
```

As you can see the ```Route``` enum serves as the parent enum and it contains other nested enums ```PatientRoute``` and ```DoctorRoute``` respectively. The nested enums specifies different sections of the application. 

Next, we can attach ```navigationDestination``` to the root view and handle the routes. This is shown below: 

``` swift 

@State private var routes: [Route] = []

var body: some View {
        NavigationStack(path: $routes) {
            VStack {
                Button("Go to Patient List Screen") {
                    routes.append(.patient(.list))
                }
            }.navigationDestination(for: Route.self) { route in
                switch route {
                    case .patient(let patient):
                        handlePatientRoutes(patient)
                    case .doctor(let doctor):
                        handleDoctorRoutes(doctor)
                }
            }
        }
    }
```

> If you prefer you can create a Router to handle all the routes and then return the appropriate view based on the case. I am implementing it right inside the view to keep things simple. 

All patient routes are handled inside the ```handlePatientRoutes``` function and all doctor routes are handled inside ```handleDoctorRoutes``` function. The implementation of ```handlePatientRoutes``` is shown below: 

``` swift 
 @ViewBuilder
    private func handlePatientRoutes(_ patient: Route.PatientRoute) -> some View {
        switch patient {
            case .list:
                Text("List")
            case .create:
                Text("Create")
            case .detail(let patient):
                Text(patient.name)
        }
    }
```

The ```handlePatientRoutes``` function is a ```@ViewBuilder``` and it returns the destination view. The usage is shown below: 

``` swift 
 Button("Go to Patient List Screen") {
    routes.append(.patient(.list))
}
```

Now, when you tap on the button you will be taken to the patient list screen. 

> When structuring your nested enums, make sure to talk a domain expert and get insights of the business aspects of your application. Knowledge of domain will help you craft better and intuitive routes, which will provide seamless experience to the developers. 

### Global Routing in SwiftUI

In the last section, we implemented routing based on an enum. We also looked at nested enums and how nesting routes can help you organize and structure complicated scenarios in your application.

We implemented the ```routes``` array as a local private state for the ```ContentView```. This means if you want to perform navigation from other screens, you won't be able to access the ```routes``` array. 

One solution is to introduce global state through the use of ```@Observable``` macro. The implementation below shows ```Router```, which contains the routing behavior of the entire application. 

``` swift 
@Observable
class Router {
    
    var routes: [Route] = []
    
    @ViewBuilder
    func destination(for route: Route) -> some View {
        switch route {
            case .patient(let patient):
                handlePatientRoutes(patient)
            case .doctor(let doctor):
                handleDoctorRoutes(doctor)
        }
    }
    
    @ViewBuilder
    private func handleDoctorRoutes(_ doctor: Route.DoctorRoute) -> some View {
        switch doctor {
            case .list:
                Text("List")
            case .create:
                Text("Create")
            case .detail(let doctor):
                Text(doctor.name)
        }
    }
    
    @ViewBuilder
    private func handlePatientRoutes(_ patient: Route.PatientRoute) -> some View {
        switch patient {
            case .list:
                Text("List")
            case .create:
                Text("Create")
            case .detail(let patient):
                Text(patient.name)
        }
    }
}
```

The ```Router``` class exposes the destination function and returns the appropriate view based on the selected route. Nested routes are controlled by functions ```handleDoctorRoutes``` and ```handlePatientRoutes``` appropriately. These functions are marked private, since they are not meant to be available to be called from outside. 

In order to use ```Router``` as a global state, it needs to be injected in the environment. This is shown below: 

``` swift 
#Preview {
    ContentView()
        .environment(Router())
}
```

Now, we can access ```Router``` globally from the ```Environment``` inside the view. This is shown below: 

``` swift 
struct ContentView: View {
    
    @Environment(Router.self) private var router

    var body: some View {
        
        @Bindable var router = router
        
        NavigationStack(path: $router.routes) {
            VStack {
                Button("Go to Patient List Screen") {
                    router.routes.append(.patient(.list))
                }
            }.navigationDestination(for: Route.self) { route in
                router.destination(for: route)
            }
        }
    }
}
```

Much simpler right! 

The line ```@Bindable var router = router``` is required in order to bind routes array to the ```NavigationStack```. If you directly bind the ```router.routes``` from the Environment then it will cause errors. 

In the next section I will talk about exposing navigation through Environment Values. This technique is similar to navigate hook in React. 

### Environment Values 

React allow developers to perform navigation using a hook called ```useNavigate```. Once imported you can use it inside your React components as shown below: 

``` swift 
navigate("/session-timed-out");
```

We can apply the same techniques to create relevant hooks using SwiftUI custom environment values. In this section, I will introduce a ```navigate``` custom environment value that allows users to perform programmatic navigation in SwiftUI.  

We will start by implementing custom Environment Key as shown below: 

``` swift 
struct NavigateEnvironmentKey: EnvironmentKey {
    static var defaultValue: NavigateAction = NavigateAction(action: { _ in })
}
```

The NavigateAction is a struct containing action that represents the route closure. 

``` swift 
struct NavigateAction {
    typealias Action = (Route) -> ()
    let action: Action
    func callAsFunction(_ route: Route) {
        action(route)
    }
}
```

Next we can extend EnvironmentValues to add a new custom environment. This is shown below: 

``` swift 
extension EnvironmentValues {
    var navigate: (NavigateAction) {
        get { self[NavigateEnvironmentKey.self] }
        set { self[NavigateEnvironmentKey.self] = newValue }
    }
}
```

Once the custom environment key has been added, we are ready to use it in our application. The implementation is shown below: 

``` swift 
struct ContentContainerView: View {
    
    @State private var router = Router()
    
    var body: some View {
        NavigationStack(path: $router.routes) {
            ContentView() // This is the root view 
                .navigationDestination(for: Route.self) { route in
                    router.destination(for: route)
                }
        }
        .environment(\.navigate, NavigateAction(action: { route in
            router.routes.append(route)
        }))
       
    }
}
```

The ```navigate``` environment value on the NavigationStack and once the action is fired the received route is appended to the router routes collection. This results in navigationDestination being fired where the route is sent to the ```router.destination``` function and the appropriate view is returned.  

The usage is shown below: 

``` swift 
struct ContentView: View {
    
    @Environment(\.navigate) private var navigate
    
    var body: some View {
            VStack {
                Button("Login") {
                    // use task modifier if you want to
                    Task {
                        try! await Task.sleep(for: .seconds(2.0))
                        navigate(.patient(.list))
                    }
                }
            }
    }
}
```

The usage of ```navigate``` custom environment value provides an easy and intuitive way to create programmatic routes in SwiftUI. 

> I have always advocated the idea of learning from other frameworks and platforms. React is a much mature framework as compared to SwiftUI and we can take ideas from it and see how they behave in SwiftUI world. The implementation and usage of ```navigate``` custom environment value is a clear example of something that already exists in React world but can also be utilized in SwiftUI applications. 

When using navigation in custom reusable views, we have to be very careful about the usability narrative. This means we cannot tie the navigation to the implementation of custom views. Because this makes the views not reusable and they will always navigate to a hard-coded destination. In the next section, we will cover how to separate the navigation responsibility from view, making it more reusable. 

### Removing Navigation Dependency from Custom Views 

Consider a scenario that you have a view called ```ProductView```. In ```ProductView```, you have a button that can add the product as a favorite. After the product has been marked as favorite, the user is navigated back to the ```ProductListScreen```. This implementation is shown below: 

``` swift 
struct ProductView: View {
    
    @Environment(\.navigate) private var navigate
    
    var body: some View {
        VStack {
            Button("Add to Favorite") {
                Task {  // use task modifier if you want to
                    // perform a POST request and add to favorite
                    try! await Task.sleep(for: .seconds(2.0))
                    // navigate to ProductListScreen
                    navigate(.product(.list))
                }
            }
        }
    }
}
```

The code above works as expected. The problem arises when you have to use ```ProductView``` in a context, where it has to navigate to a different destination. The reason is that the navigation is tied up in the implementation of the ```ProductView```. ```ProductView``` will always navigate to the product list screen after successfully marking the product as favorite. 

There are several ways to solve this problem. One approach is to pass the destination route in the ```ProductView``` initializer. This is implemented below: 

``` swift 
struct ProductView: View {
    
    @Environment(\.navigate) private var navigate
    let destinationRoute: Route
    
    var body: some View {
        VStack {
            Button("Add to Favorite") {
                Task {  // use task modifier if you want to
                    // perform a POST request and add to favorite
                    try! await Task.sleep(for: .seconds(2.0))
                    // navigate to ProductListScreen
                    navigate(destinationRoute)
                }
            }
        }
    }
}
```

The ```ProductView``` is no longer hard-coded to a particular route. The destination route is passed as an argument, which is later utilized by the ```navigate``` function to perform the actual navigation.

Even though this technique works, it does come with a limitation. You cannot perform any custom task when "Add to Favorite" button is tapped. For example, maybe you wanted to issue another POST request or log the behavior for the action. With this technique you are limited to only performing the navigation once the button is tapped. 

The way to resolve this issue is to allow the parent to handle the "Add to Favorite" tap event. This will allow the parent view to add more behavior to the event if needed. The implementation is shown below: 

``` swift 
struct ProductView: View {
    
    let addToFavorite: (Product) -> Void
    
    var body: some View {
        VStack {
            Button("Add to Favorite") {
                Task {  // use task modifier if you want to
                    // perform a POST request and add to favorite
                    try! await Task.sleep(for: .seconds(2.0))
                    // call the addToFavorite closure
                    addToFavorite(Product(name: "Shirt"))
                }
            }
        }
    }
}
```

The ```ProductView``` now exposes a closure called ```addToFavorite```. When the button is tapped, the selected product is passed to the caller using the ```addToFavorite``` closure. This transfers the responsibility from the ```ProductView``` to the parent screen/view. 

The parent can handle the callback as shown below: 

``` swift 
struct ContentView: View {
    
    @Environment(\.navigate) private var navigate
    
    var body: some View {
            VStack {
                Button("Login") {
                    // use task modifier if you want to
                    Task {
                        try! await Task.sleep(for: .seconds(2.0))
                        navigate(.patient(.list))
                    }
                }
                
                ProductView { product in
                    // log the product
                    // do other actions with the product
                    navigate(.product(.detail(product)))
                }
            }
    }
}
```

The ```ContentView``` uses the ```ProductView``` and then handles the ```addToFavorite``` closure. This allows the ```ContentView``` to perform additional actions based on the result of ```addToFavorite``` closure. 

The ```ProductView``` is now completely free from any navigation code, making it more reusable in different parts of the application. 

> The big question is should we always implement our reusable views using the techniques discussed above (closures/passing routes). The answer is it depends. I personally start with a simple implementation of my views and when and if the time comes, I will refactor to include closures etc. There are scenarios, where you will keep the navigation inside the reusable views and reuse them in multiple places of the application. Those views will also perform the same navigation, so there is no point of extracting out the navigation and making it more complicated. 

### TabView Navigation 

### What About Modals?

Modals or sheets are not part of the NavigationStack period! They are displayed on top of the view and are never added to the navigation history. You can implement a similar approach for modals and it is recommended to keep it separate from all the approaches discussed above. 

### Conclusion    

In this article, we discussed several different patterns for navigation in SwiftUI. Each pattern has its own set of advantages and disadvantages. It is important that you select the pattern, which fits perfectly for your requirements. When starting out, don't try to complicate things by introducing generics, protocols etc. These things can be added when and if they are needed in the future. Keep it simple! 