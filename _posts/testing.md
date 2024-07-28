
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

React allow developers to perform navigation using a simple ```navigate``` hook. We can apply the same techniques to create relevant hooks using SwiftUI custom Environment Values. In this section, I will introduce a ```navigate``` hook   

### TabView Navigation 

### NavigationSplitView (Coming soon)

### Conclusion 