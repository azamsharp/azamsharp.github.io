# SwiftUI Architecture - Embracing the Framework

I was listening to an amazing talk by Matias Villaverde and Rens Breur at NSSpain about "Lessons learnt rewriting SoundCloud in SwiftUI". You can watch the complete talk [here](https://vimeo.com/751534042/f1ae29434e). 

This talk really resonated with me because I did similar mistakes when building SwiftUI applications. Instead of embracing the framework, I added unnecessary complexity to fight against the framework. This included creating View Models for each view, ignoring @FetchRequest, passing @EnvironmentObject to the ViewModel and getting into the dependency injection nightmare.  

After almost two years of driving in the wrong direction, I decided to press the brake and go back to the drawing board. In this post, I will discuss the SwiftUI architecture I am using for my apps. The architecture might look extremely simple but it does utilize the true power of SwiftUI framework. 

>> This architecture mentioned in this post works for my apps. When adopting an architecture make sure it fits your scenario. There is no architecture, fit all solution. 

## Architecture

The architecture for our app will revolve around few different components. This includes an Aggregate Model, Webservice and the server. The primary purpose of an Aggregate Model is to provide model objects to the view. Since, this is a client server app, the aggregate model will invoke a webservice which will return model objects to the view. 

For small apps you need just a single aggregate model. This is shown in the figure below. 

![MV Architecture Client/Server App](/images/mv-pattern-json-1.jpeg)

For larger apps, you can introduce several aggregate model objects based on the bounded context of the application. 

![MV Architecture Multiple Aggregate Root Models](/images/mv-pattern-json-2.jpeg)

Let's take a look at what I mean by Aggregate Root.

The main purpose of an Aggregate Root Model is to allow you access to the entities/models under a certain bounded context. The bounded context is determine by business domain. In the figure below, you can see different Aggregate Root models for an e-commerce application.  

![MV Architecture Client/Server App](/images/mv-pattern-json-3.jpeg)

Each Aggregate Root will be responsible for performing actions on the models that comes under their umbrella. The Inventory model will allow you to access products, product categories, adding new products, removing products etc. 

For our small application, we will only be working with a single Aggregate Root Model. 

## Implementing the Aggregate Root Model

In Apple's documentation they have used different names for their model. This includes Model, FoodTruckModel, FrutaModel etc. I believe the model name should be based on the bounded context. For an e-commerce app your aggregate model might be Catalog, Payment, Shipping, Inventory etc. 

For the sake of simplicity, I am calling my model "Model". The basic implementation is shown below: 


```swift 
@MainActor
class Model: ObservableObject {
    
    @Published var orders: [Order] = []
    let orderService: OrderService
    
    init(orderService: OrderService) {
        self.orderService = orderService
    }
}
```

>> A Model may look like View Model but it is not. A model is not responsible for formatting data to be presented on the screen. Unlike View Model, a separate model does need to be created for each screen. For smaller apps, you can use a single model for your entire app. If your model is getting larger then it would be a good idea to think about separating it and distributing the responsibilities among different aggregate models. You break a model into small models depending on the application domain, which is usually based on the bounded context of the application domain.  

The complete implementation of the model is shown below: 

```swift 
@MainActor
class Model: ObservableObject {
    
    @Published var orders: [Order] = []
    let orderService: OrderService
    
    init(orderService: OrderService) {
        self.orderService = orderService
    }
    
    func sortOrders() {
        // code to sort orders
    }
    
    func filterOrders() {
        // code to filter orders
    }
    
    func getAllOrders() async throws {
        // you can call a Cache layer to see if the orders are already in the cache
        // if the orders are in the cache then return from the cache
        // otherwise fetch orders by making a network call
        self.orders = try await orderService.getAllOrders()
    }
    
    func placeOrder(_ order: Order) async throws {
        
        guard let newOrder = try await orderService.placeOrder(order) else {
            throw OrderError.custom("Unable to place an order.")
        }
       
        orders.append(newOrder)
    }
    
    func updateOrder(_ order: Order) async throws {
        guard let updatedOrder = try await orderService.updateOrder(order) else {
            throw OrderError.custom("Unable to update an order.")
        }
        
        guard let index = orders.firstIndex(where: { $0.id == updatedOrder.id }) else {
            return
        }
        
        orders[index] = updatedOrder
    }
}
```

The model calls the OrderService to get all the orders and performs different actions related to orders like inserting and updating. 

>> You might be wondering that why can't the View directly call the OrderService. The main reason is that Model provides sorting, filtering capabilities, which does not fit well in the OrderService. Another reason is that the model can invoke multiple services to aggregate and return data to the view. 

Next, let's take a look at the OrderService. 

## Implementing OrderService

The main purpose of the OrderService is to fetch the data from the server and then hand it to the aggregate model. The complete implementation of the OrderService is shown below: 

```swift 
class OrderService {
    
    let baseURL: URL
    
    init(baseURL: URL) {
        self.baseURL = baseURL
    }
    
    func getAllOrders() async throws -> [Order] {
        
        guard let url = URL(string: "/orders", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Order].self, from: data)
    }
    
    func placeOrder(_ order: Order) async throws -> Order? {
        
        guard let url = URL(string: "/newOrder", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(order)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        let savedOrder = try JSONDecoder().decode(Order.self, from: data)
        
        return savedOrder
    }
    
    func updateOrder(_ order: Order) async throws -> Order? {
        
        guard let url = URL(string: "/orders/\(order.id!)", relativeTo: baseURL) else {
            throw OrderServiceError.badURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "PUT"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(order)
        
        let (data, _) = try await URLSession.shared.data(for: request)
        let updatedOrder = try JSONDecoder().decode(Order.self, from: data)
        
        return  updatedOrder 
    }
    
}
```

>> You can create a generic service, which has all the basic operations like getAll, getById etc. 

## Implementing Views

SwiftUI views are not just views but they are also view models. But that does not mean that you should put networking code right inside the view. 

>> By networking code I mean URLSession.shared.xxx. Although it will work as expected but it will make it harder to use the same networking calls from other views. This is the main reason we created the OrderService. In React apps, developers usually call the networking code (fetch or axios) from directly inside the Component. This is perfectly fine until, you need to make the same call somewhere else. 

In our app, SwiftUI views need to access the Model object so it can make a call and get the list of orders. We also want a global access of the list of orders. @EnvironmentObject will be a good fit for this scenario. The model object is injected as an @EnvironmentObject in the root view. This is shown in the implementation below: 

```swift 
@main
struct LearnApp: App {
    
    private var service = OrderService(baseURL: URL(string: "https://island-bramble.glitch.me/orders")!)
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(Model(orderService: service))
        }
    }
}
```

>> In the above code I am injecting a hard-coded base url. In your  application, the url can be based on the environment (dev, test, qa, production). 

The implementation for the ContentView is shown below. The ContentView is responsible for displaying all the orders to the user. 

```swift 
struct ContentView: View {
    
    @EnvironmentObject private var model: Model
        
    var body: some View {
        NavigationStack {
            
            List {
                
                ForEach(model.orders) { order in
                    HStack {
                        VStack(alignment: .leading) {
                            Text(order.name)
                            Text(order.coffeeName)
                                .opacity(0.5)
                        }
                        Spacer()
                        Text(order.total as NSNumber, formatter: NumberFormatter.currency)
                    }
                    .contentShape(Rectangle())
                    .onLongPressGesture {
                        activeSheet = .update(order)
                    }
                }
            }.task {
                 do {
                    try await model.getAllOrders()
                } catch {
                    print(error.localizedDescription)
                }
}
```

Let's take a look at **AddNewCoffeeOrderView**. The AddNewCoffeeOrderView view is responsible for allowing the user to place a brand new order. This view also performs UI validation and displays error messages to the user. 

```swift 
struct AddNewCoffeeOrderError {
    var name: String = ""
    var coffeeName: String = ""
    var total: String = ""
}

struct AddNewCoffeeOrderView: View {
    
    @State private var name: String = ""
    @State private var coffeeName: String = ""
    @State private var total: String = ""
    @State private var size: CoffeeSize = .medium
    @State private var errors: AddNewCoffeeOrderError = AddNewCoffeeOrderError()
    
    let order: Order? 
    
    @Environment(\.dismiss) private var dismiss
    @EnvironmentObject private var model: Model
    
    init(order: Order? = nil) {
        self.order = order
    }
    
    private func placeOrder(_ order: Order) async {
        try? await model.placeOrder(order)
        dismiss()
    }
    
    private func updateOrder(_ order: Order) async {
        try? await model.updateOrder(order)
        dismiss() 
    }
    
    private func updateOrPlaceOrder() async {
        if let order {
            // update the order
            let order = Order(id: order.id, name: name, coffeeName: coffeeName, total: Double(total)!, size: size)
            await updateOrder(order)
            
        } else {
            // place the order
            let order = Order(name: name, coffeeName: coffeeName, total: Double(total)!, size: size)
            await placeOrder(order)
        }
    }
    
    var isFormValid: Bool {
        
        if name.isEmpty {
            errors.name = "Name cannot be empty!"
        }
        
        if coffeeName.isEmpty {
            errors.coffeeName = "Coffee name cannot be empty"
        }
        
        if total.isEmpty {
            errors.total = "Total cannot be empty"
        } else if !total.isNumeric {
            errors.total = "Total can only be numbers"
        } else if total.isLessThan(0) {
            errors.total = "Total cannot be less than 0"
        }
        
        return errors.name.isEmpty && errors.coffeeName.isEmpty && errors.total.isEmpty
        
    }
    
    var body: some View {
        
        NavigationStack {
            Form {
                TextField("Name", text: $name)
                !errors.name.isEmpty ? Text(errors.name) : nil
                
                TextField("Coffee name", text: $coffeeName)
                !errors.coffeeName.isEmpty ? Text(errors.coffeeName) : nil
                
                TextField("Total", text: $total)
                !errors.total.isEmpty ? Text(errors.total) : nil
                
                Picker("Coffee Size", selection: $size) {
                    ForEach(CoffeeSize.allCases) { coffeeSize in
                        Text(coffeeSize.rawValue).tag(coffeeSize)
                    }
                }.pickerStyle(.segmented)
            }
            .onAppear {
                if let order {
                    // editing mode
                    name = order.name
                    coffeeName = order.coffeeName
                    total = String(order.total)
                    size = order.size
                }
            }
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button(order != nil ? "Update Order": "Place Order") {
                        Task {
                            if isFormValid {
                                await updateOrPlaceOrder()
                            }
                        }
                    }
                }
            }
        }
        
    }
}
```

We have used the same techniques for validation as used for React framework. If the field validation is unsuccessful then a flag is set, which displays the error messages to the user. 

>> UI validation is NOT business logic. UI validation is just checking if the user has input valid information. The business rules (if any) will be executed on the valid data. Let's say you are building a website, where the user can enter their credit score and get an APR rate (Interest Rate). The UI validation is going to check that the credit score TextField is not left empty. It will also check that the score only consists of numbers between a certain range. All of this will be UI validation. Once the user successfully submits the credit score, we will run business rules to find out the appropriate APR for the user.  

Even HTML provides attributes to validate the UI. Take a look at the code below: 

```html 
<input type = 'text' required />
```

The above code will validate the input field (TextBox), when the form is submitted. If the input field is blank then the form will not be submitted and the user will get an error message. This is an example of UI validation and NOT business rules.

>> In most client/server apps, business rules lives on the server and the client just provides the UI. 

One of the benefits of using the @EnvironmentObject is that when the orders are altered, views are automatically reevaluated. Reevaluation is not the same as re-rendering. Revaluation means that SwiftUI will look at the diff tree and then decide, which views needs to be re-rendered. Revaluation is a very fast process, so although your view body might be getting fired constantly it does not mean that all views in the body are getting re-rendered. 

 Take a look at the code below: 

 ```swift 
 struct DemoView: View {
    
    @State private var name: String = ""
    
    var body: some View {
        VStack {
            let _ = Self._printChanges()
            
            List(1...20, id: \.self) { index in
                Text("\(index)")
            }
            
            TextField("Name", text: $name)
            
            Button("Greet") {
                
            }
        }
    }
}
 ```

If you type in the TextField then you will notice that the body is fired each time. The main reason is that the @State variable name is getting new values from the TextField. But that does not mean that the all the views inside the body are getting re-rendered. Only, the views that are changed are getting re-rendered. So in the above code, only TextField is getting re-rendered. 

In the same way when using the @EnvironmentObject, several views may get revaluated but only those who needs to be re-rendered are rendered again. If you are getting unwanted re-rendering then you can always split your @EnvironmentObject into multiple objects. This is shown in my article [Slicing Global State in SwiftUI Using Multiple EnvironmentObjects](https://azamsharp.com/2022/07/01/slicing-environment-object.html). 

## Resources 

- [SwiftUI View is also a View Model](https://azamsharp.com/2022/07/21/view-is-the-view-model.html)
- [WWDC - Data Flow Though SwiftUI](https://developer.apple.com/videos/play/wwdc2019/226/)
- [Udemy Course: MV Design Pattern in iOS - Build SwiftUI Apps Apple's Way](https://www.udemy.com/course/mv-design-pattern-in-ios-for-swiftui/?referralCode=4627986F77F533DEF0C7)
- [Lessons learned rewriting SoundCloud in SwiftUI](https://vimeo.com/751534042/f1ae29434e)


## Conclusion 

SwiftUI does not need MVVM, since it already has MVVM built in. Although you can use MVVM with SwiftUI but it needlessly complicate things. A lot of other developers are coming to the same conclusion and rewriting their apps using features provided by SwiftUI. SwiftUI adds some magic to their property wrappers, which makes everything simple. For your next app, try creating views without View Models and allow the views to directly talk to the model. 


