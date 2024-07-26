
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

You can start by declaring your routes. This can be done in several different ways but enum is considered a better c  


Dynamic routing also allows you to create unwind segues. Unwind segues is a technique where you can jump from View C to View A instantly instead of going through View B. 

### Enum Based Routing 

### Environment Values 

### TabView Navigation 


### Conclusion 