
# Navigation Patterns in SwiftUI  

- Updated (07/31/2024) Added onChange modifier technique in section [Removing Navigation Dependency from Custom Views](#removing-navigation-dependency-from-custom-views).
- Updated (08/23/2024) Added a new section on TabView Navigation using Environment Values (navigate) syntax. 


Navigation has often been a challenge in SwiftUI applications. Initially, SwiftUI introduced `NavigationView`, which was later replaced by `NavigationStack` in iOS 16.

`NavigationStack` enhanced navigation by enabling dynamic and programmatic routing, and it also offered ways to centralize routes for the entire application. In this article, I'll explore common navigation patterns that can be employed when building SwiftUI applications.

The outline of the article is shown below: 

- [Basic List Navigation](#basic-list-navigation)
- [Dynamic Programmatic Routing](#dynamicprogrammatic-routing)
- [Global Routing in SwiftUI](#global-routing-in-swiftui)
- [Implementing navigation Hook Using Environment Values](#implementing-navigation-hook-using-environment-values)
- [Removing Navigation Dependency from Custom Views](#removing-navigation-dependency-from-custom-views)
- [TabView Navigation](#tabview-navigation)
- [TabView Navigation Using Hooks](#tabview-navigation-using-hooks-environment-values)
- NavigationPath (Coming soon)
- NavigationSplitView (Coming soon)
- Navigation Interoperability Between UIKit & SwiftUI Apps (Coming soon)
- [What About Modals?](#what-about-modals)
- [Conclusion](#conclusion)

> If you are interested in learning iOS development then check out my courses on https://azamsharp.school. 

### Basic List Navigation  

One of the most common patterns for navigation involves basic list navigation. This is where a user taps on an item in the list, which takes the user to the destination screen. There are multiple ways to perform list navigation.

In the following implementation, we have used `NavigationLink` to represent the destination and the label. Once the user taps on the customer name, they are taken to the CustomerDetailScreen.

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

Keep in mind that the initializer of `CustomerDetailScreen` is called repeatedly for every single item in the list. It is recommended that you do not perform resource-intensive operations in the initializer of the view.

> If you need to perform a network call inside a view, use the `task` view modifier. The `task` modifier is asynchronous in nature, and the task is automatically cancelled when the view no longer exists.

Another option is to use the `navigationDestination`. The implementation is shown below:

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

The `navigationDestination` approach relies on the type of the value that triggered the navigation. The `NavigationLink` value property is set to the customer. The `navigationDestination` is also set up for the `Customer` type using the `for: Customer.self` argument. This allows the `navigationDestination` view modifier to trigger whenever a customer value is set.

Both options work, and either one can be used in this scenario. Personally, I would use the first approach and not the `navigationDestination` because it is clearer and simpler to understand.

Later in this article, I will demonstrate how to use `navigationDestination` when constructing dynamic routing in SwiftUI.

### Dynamic/Programmatic Routing 

`NavigationStack` also supports dynamic or programmatic routing. This means you can programmatically add or remove routes from the `NavigationStack`. This can be useful in scenarios where you have performed an asynchronous action and, based on the result of that action, you want to direct the user to a particular destination. A very common example is a user signing in and then navigating to the dashboard screen after a successful sign-in.

Dynamic routes can be achieved in many different ways. A common approach is to use an enum to represent all the available dynamic routes. The implementation is shown below:
``` swift 
enum Route: Hashable {
    case dashboard
    case detail(Customer)
}
```

Next, we can use the `NavigationStack` `path` parameter to initialize our route binding. This is shown in the implementation below:

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

We start by creating the routes array as a private state. Once the Login button is tapped, we fake a call by waiting for 2 seconds and then add the dashboard route to the routes array. This causes the `navigationDestination` to trigger. Inside the `navigationDestination`, we get access to the route enum, and based on the selected case, we direct the user to the correct destination screen.

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

As you can see, the `Route` enum serves as the parent enum, and it contains other nested enums, `PatientRoute` and `DoctorRoute`, respectively. The nested enums specify different sections of the application.

Next, we can attach `navigationDestination` to the root view and handle the routes. This is shown below:

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

> If you prefer, you can create a Router to handle all the routes and then return the appropriate view based on the case. I am implementing it right inside the view to keep things simple.

All patient routes are handled inside the `handlePatientRoutes` function, and all doctor routes are handled inside the `handleDoctorRoutes` function. The implementation of `handlePatientRoutes` is shown below:

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

Now, when you tap on the button, you will be taken to the patient list screen.

> When structuring your nested enums, make sure to talk to a domain expert and gain insights into the business aspects of your application. Knowledge of the domain will help you craft better and more intuitive routes, providing a seamless experience for developers.

### Global Routing in SwiftUI

In the last section, we implemented routing based on an enum. We also looked at nested enums and how nesting routes can help you organize and structure complicated scenarios in your application.

We implemented the `routes` array as a local private state for the `ContentView`. This means that if you want to perform navigation from other screens, you won't be able to access the `routes` array.

One solution is to introduce global state through the use of the `@Observable` macro. The implementation below shows `Router`, which contains the routing behavior of the entire application.

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

The `Router` class exposes the `destination` function and returns the appropriate view based on the selected route. Nested routes are controlled by the `handleDoctorRoutes` and `handlePatientRoutes` functions appropriately. These functions are marked private, as they are not intended to be called from outside.

To use `Router` as a global state, it needs to be injected into the environment. This is shown below:

``` swift 
#Preview {
    ContentView()
        .environment(Router())
}
```

Now, we can access `Router` globally from the `Environment` inside the view. This is shown below:

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

Much simpler, right!

The line `@Bindable var router = router` is required to bind the `routes` array to the `NavigationStack`. If you directly bind the `router.routes` from the Environment, it will cause errors.

In the next section, I will talk about exposing navigation through Environment Values. This technique is similar to the `navigate` hook in React.

### Implementing navigation Hook Using Environment Values 

React allows developers to perform navigation using a hook called `useNavigate`. Once imported, you can use it inside your React components as shown below:

``` swift 
navigate("/session-timed-out");
```

We can apply the same techniques to create relevant hooks using SwiftUI custom environment values. In this section, I will introduce a `navigate` custom environment value that allows users to perform programmatic navigation in SwiftUI.

We will start by implementing a custom `EnvironmentKey` as shown below:

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

The `navigate` environment value is added to the `NavigationStack`, and once the action is fired, the received route is appended to the router's routes collection. This results in the `navigationDestination` being triggered, where the route is sent to the `router.destination` function, and the appropriate view is returned.

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

The usage of the `navigate` custom environment value provides an easy and intuitive way to create programmatic routes in SwiftUI.

> I have always advocated the idea of learning from other frameworks and platforms. React is a much more mature framework compared to SwiftUI, and we can take ideas from it and see how they behave in the SwiftUI world. The implementation and usage of the `navigate` custom environment value is a clear example of something that already exists in React but can also be utilized in SwiftUI applications.

When using navigation in custom reusable views, we must be very careful about the usability narrative. This means we cannot tie the navigation to the implementation of custom views, as this would make the views not reusable and always navigate to a hard-coded destination. In the next section, we will cover how to separate the navigation responsibility from the view, making it more reusable.

### Removing Navigation Dependency from Custom Views 

Consider a scenario where you have a view called `ProductView`. In `ProductView`, you have a button that can add the product as a favorite. After the product has been marked as a favorite, the user is navigated back to the `ProductListScreen`. This implementation is shown below:

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

The code above works as expected. The problem arises when you have to use `ProductView` in a context where it needs to navigate to a different destination. The reason is that the navigation is tied up in the implementation of `ProductView`. `ProductView` will always navigate to the `ProductListScreen` after successfully marking the product as a favorite.

There are several ways to solve this problem. One approach is to pass the destination route in the `ProductView` initializer. This is implemented below:

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

The `ProductView` is no longer hard-coded to a particular route. The destination route is passed as an argument, which is later utilized by the `navigate` function to perform the actual navigation.

Even though this technique works, it does come with a limitation. You cannot perform any custom tasks when the "Add to Favorite" button is tapped. For example, if you want to issue another POST request or log the behavior for the action, this technique only supports navigation once the button is tapped.

To resolve this issue, you can allow the parent view to handle the "Add to Favorite" tap event. This approach enables the parent view to add more behavior to the event if needed. The implementation is shown below:

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

The `ProductView` now exposes a closure called `addToFavorite`. When the button is tapped, the selected product is passed to the caller using the `addToFavorite` closure. This transfers the responsibility from `ProductView` to the parent screen/view.

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

The `ContentView` uses the `ProductView` and then handles the `addToFavorite` closure. This allows `ContentView` to perform additional actions based on the result of the `addToFavorite` closure.

The `ProductView` is now completely free from any navigation code, making it more reusable in different parts of the application.

Another technique is to use SwiftUI ```@Binding``` feature to get a callback in the parent view. This will allow you to pass product as binding, change the ```isFavorite``` property to true and then capture the callback using the ```onChange``` modifier. This is shown below: 

The ```ProductView``` implementation is shown below: 

``` swift 
struct ProductView: View {
    
    @Binding var product: Product
    
    var body: some View {
        VStack {
            Button("Add to Favorite") {
                Task {  // use task modifier if you want to
                    // perform a POST request and add to favorite
                    try! await Task.sleep(for: .seconds(2.0))
                    product.isFavorite = true
                }
            }
        }
    }
}
```

The ```ContentView``` is updated to handle the ```onChange``` modifier. 

``` swift 
struct ContentView: View {
    
    @Environment(\.navigate) private var navigate
    @State private var product = Product(name: "Shirts")
    
    var body: some View {
            VStack {
                Button("Login") {
                    // use task modifier if you want to
                    Task {
                        try! await Task.sleep(for: .seconds(2.0))
                        navigate(.patient(.list))
                    }
                }
                
               ProductView(product: $product)
            }
            .onChange(of: product) {
                // do other stuff
               // navigate(.product(.list))
            }
    }
}
```

Both techniques discussed above work correctly and remove the dependency on navigation. This makes your views reusable and allows the parent to make decisions about the target destination.

> The big question is: should we always implement our reusable views using the techniques discussed above (closures/passing routes)? The answer is, it depends. I personally start with a simple implementation of my views and, when and if the time comes, I will refactor to include closures, etc. There are scenarios where keeping the navigation inside the reusable views and reusing them in multiple places in the application can be simpler and more practical. In those cases, there is no point in extracting out the navigation and making it more complicated.

### TabView Navigation 

`TabView` presents some interesting challenges, as each tab needs to maintain its own `NavigationStack`. This means that instead of maintaining a single array with routes, we need to maintain multiple arrays, one for each tab item.

> I am sure you could use a single array to maintain history for all tabs, but I find it simpler to use separate arrays for each tab item. If you use a single array then you will have to manage the filtering of routes based on the selected tab. 

The first step is to create tabs based on the enum options (Patients, Doctors). The implementation is shown below:

``` swift 
enum AppScreen: Hashable, Identifiable, CaseIterable {
    
    case patients
    case doctors
    
    var id: AppScreen { self }
}

extension AppScreen {
    
    @ViewBuilder
    var label: some View {
        switch self {
            case .patients:
                Label("Patients", systemImage: "heart")
            case .doctors:
                Label("Doctors", systemImage: "star")
        }
    }
    
    @ViewBuilder
    var destination: some View {
        switch self {
            case .patients:
                PatientNavigationStack()
            case .doctors:
                DoctorNavigationStack()
        }
    }
}
```

`AppScreen` is an enum that contains options for each `TabItem`. The `label` property returns the visual label for the tab item, while the `destination` property returns the root view for each tab item.

The next step is to iterate through all the cases and render the tabs. This is implemented below:

``` swift 
struct ContentContainerView: View {
    
    @State private var router = Router()
    @State var selection: AppScreen?
    
    var body: some View {
        ContentView(selection: $selection)
            .environment(router)
    }
}

struct ContentView: View {
    
    @Binding var selection: AppScreen?
    
    var body: some View {
        TabView(selection: $selection) {
            ForEach(AppScreen.allCases) { screen in
                screen.destination
                    .tag(screen as AppScreen?)
                    .tabItem { screen.label }
            }
        }
    }
}
```

The `ContentContainerView` is used to render the previews successfully. In your real application, you will call `ContentView` from your `App` struct, and all the code in `ContentContainerView` will be implemented in the `App` struct.

The `PatientNavigationStack` and `DoctorNavigationStack` are the root views for each tab item. They are also responsible for maintaining the navigation history, as they contain the `NavigationStack`. The implementation of `PatientNavigationStack` is shown below:

``` swift 
struct PatientNavigationStack: View {
    
    @Environment(Router.self) private var router
    
    var body: some View {
        
        @Bindable var router = router
        
        NavigationStack(path: $router.patientRoutes) {
            Button("Go to PATIENT Detail") {
                router.patientRoutes.append(.detail(Patient(name: "Mary Doe")))
            }.navigationDestination(for: PatientRoute.self) { route in
                route.destination
            }
        }
    }
}
```

As you can see, the `NavigationStack` is using `router.patientRoutes` as the path. This is because all the routes related to patients are maintained in the `patientRoutes` array. The `navigationDestination` only listens for routes associated with `PatientRoute`. Once the route is found, we use the destination property of the route to return the view.

The implementation of `PatientRoute` is shown below:

``` swift 
enum PatientRoute: Hashable {
    
    case list
    case create
    case detail(Patient)
    
    @ViewBuilder
    var destination: some View {
        switch self {
            case .list:
                Text("PatientRoute List")
            case .detail(_):
                PatientDetailScreen()
            case .create:
                Text("PatientRoute Create")
            
        }
    }
}
```

> You can also add methods like `destinationForPatientRoute` in your router, but the approach of adding the `destination` property to each route felt more natural.

Now, each tab will maintain its own history. You can use either tab to navigate to a deeper screen, and it will not disturb any other tabs.

The `Router` implementation is simplified to only hold the corresponding arrays for each tab item. This is shown below:

``` swift 
@Observable
class Router {
    var patientRoutes: [PatientRoute] = []
    var doctorRoutes: [DoctorRoute] = []
}
```

If you have more tab items, you will create additional arrays for each. Keep in mind that Apple recommends having between 3-5 tab items on iPhone. You can have more tab items on iPad apps, but always aim for simplicity.

You can download the code for TabView Navigation [here](https://gist.github.com/azamsharpschool/06322343914a5791ae0bc4833664b053).

### TabView Navigation Using Hooks (Environment Values) 

This section was added on 08/23/2024.

While it's possible to use a `Router` class as an `Observable`, I’ve never been fond of that approach. The main reason is that I reserve `@Observable` for data that will be displayed or used directly in the body of a view. Navigation, on the other hand, feels more like a service, similar to an `HTTPClient`, that should be accessible through Environment Values rather than through Environment. Another reason is my preference for using structs over classes.

If you follow the approach discussed in [Implementing Navigation Hook Using Environment Values](#implementing-navigation-hook-using-environment-values), it won’t work seamlessly with `TabViews`. The primary issue is that each tab in a `TabView` requires a separate `NavigationStack` to maintain its navigation history. However, this problem can be easily resolved while still using the intuitive syntax like `navigate(.patient(.list))`.

The key is to update your `PatientNavigationStack` to track only patient-related routes. The `navigationDestination` view modifier should focus on the `PatientRoute` type, and the environment value for `NavigationAction` should only append `PatientRoute` types to the routes array. This allows each tab item to maintain its own route array. The implementation is shown below:

``` swift 
struct PatientNavigationStack: View {
    
    @State private var routes: [PatientRoute] = []
    
    var body: some View {
        NavigationStack(path: $routes) {
            PatientDashboardScreen()
            .navigationDestination(for: PatientRoute.self) { route in
                route.destination
            }
        }.environment(\.navigate, NavigateAction(action: { route in
            if case let .patient(patientRoute) = route {
                routes.append(patientRoute)
            }
        }))
    }
}
```

A similar implementation can be used for ```DoctorNavigationStack```. This is shown below: 

``` swift 
struct DoctorNavigationStack: View {
    
    @State private var routes: [DoctorRoute] = []
    
    var body: some View {
        NavigationStack(path: $routes) {
            DoctorDashboardScreen()
            .navigationDestination(for: DoctorRoute.self) { route in
                route.destination
            }
        }.environment(\.navigate, NavigateAction(action: { route in
            if case let .doctor(doctorRoute) = route {
                routes.append(doctorRoute)
            }
        }))
    }
}
```

Now, you can start using your new ```navigate``` environment value as shown below: 

``` swift 
struct PatientDashboardScreen: View {
    
    @Environment(\.navigate) private var navigate
    
    var body: some View {
        Button("Patient List") {
            navigate(.patient(.list))
        }
    }
}
```

I like this approach much better and intuitive as compared to using ```Router```. 

### What About Modals?

Modals or sheets are not part of the `NavigationStack`; they are displayed on top of the view and are never added to the navigation history. It’s recommended to implement a separate approach for managing modals and sheets, keeping them distinct from the navigation techniques discussed above.

### Conclusion    

In this article, we explored various navigation patterns in SwiftUI, each with its own advantages and limitations. It's crucial to choose the pattern that best fits your specific needs. When starting out, avoid overcomplicating your implementation with generics, protocols, or other advanced concepts. Focus on simplicity and add complexity only as necessary when your requirements evolve.