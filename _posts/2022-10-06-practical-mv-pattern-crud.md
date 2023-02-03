# SwiftUI Architecture - A Complete Guide to MV Pattern Approach 

>> Update 02/03/2023 - I recently published few more articles about SwiftUI Architecture. This includes exploring the [Container Pattern](https://azamsharp.com/2023/01/24/introduction-to-container-pattern.html), which is a common pattern when building ReactJS applications and [Active Record Pattern](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). For me personally, Active Record Pattern felt more natural when building **Core Data** applications with SwiftUI. This was primarily due to the similarities between Core Data and a traditional ORM. My recommendation is that you try out all the different architectural patterns and see which one fits your needs. For client/server applications, I am comfortable with using the MV pattern discussed in this article.     

I was listening to an amazing talk by Matias Villaverde and Rens Breur at NSSpain about "Lessons learnt rewriting SoundCloud in SwiftUI". You can watch the complete talk [here](https://vimeo.com/751534042/f1ae29434e). 

This talk really resonated with me because I did similar mistakes when building SwiftUI applications. Instead of embracing the simplicity of the framework, I added unnecessary complexity to please the design pattern Gods. This included creating view models for each view, ignoring @FetchRequest and @SectionFetchRequest property wrappers, passing @EnvironmentObject to the view model instead of accessing it directly in the view and much more.  

After almost two years of driving in the wrong direction, I decided to slam on the brakes and think about my decisions. In this post, I will discuss the SwiftUI architecture that I am using for my apps. 

>> There is no official name for this architecture but in the iOS community it is known as the MV pattern. This pattern is inspired from Apple's WWDC videos and sample applications. You can find the links to the sample apps in the resources section below.

Let's get started!

## Architecture

The architecture for our app will revolve around few different components. This includes an aggregate model, webservice and the server. The primary purpose of an aggregate root model is to provide other model objects to the view. Depending on your app, you may have model objects for Order, Coffee, Category etc. The view will communicate with the aggregate root model to fetch, persist, sort different models. An aggregate model can be used as a @StateObject or injected as a singleton into the @EnvironmentObject so it can be accessed in any view. Since, this is a client server app, an aggregate model will invoke the webservice which will return model objects to the view. 

For small apps you just need a single aggregate model. This is shown in the figure below. 

![MV Architecture Client/Server App](/images/mv-pattern-json-1.jpeg)

For larger apps, you can introduce several aggregate model objects based on the [bounded context](https://martinfowler.com/bliki/BoundedContext.html) of the application.

![MV Architecture Multiple Aggregate Root Models](/images/mv-pattern-json-2.jpeg)

>> You don't need an aggregate model per view. The aggregate models are based on the bounded context of the application and not on the number of screens in an app. 

This pattern was discussed in Apple's WWDC video [Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/). 

The main purpose of an aggregate root model is to allow access to the entities/models under a certain bounded context. The bounded context is determine by the business domain. In the figure below, you can see different aggregate root models for an e-commerce application.  

![MV Architecture Client/Server App](/images/mv-pattern-json-3.jpeg)

Each aggregate root model will be responsible for performing actions on the models that comes under their context. The Inventory model will allow you to access products, product categories, adding new products, removing products, updating products, searching etc.

>> Aggregate root models can also communicate with each other to access entities that are not under their context. For example: A Shipping root model can access the Customer Management root model to find the information about a particular customer. 

For our small application, we will only be working with a single aggregate root model, larger apps can have multiple root models.   

## Implementing the Aggregate Root Model

In Apple's documentation they have used different names for their models. This includes **Model**, **FoodTruckModel**, **FrutaModel** etc. I believe the model name should be based on the bounded context. For an e-commerce app your aggregate root model might be Catalog, Payment, Shipping, Inventory etc. Each aggregate model will allow you to access the entities controlled by the root. 

For the sake of simplicity, I am calling my model **Model**. The basic implementation is shown below: 

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

>> An aggregate root model may look like a view model but it is not. A root model is not responsible for formatting data to be presented on the screen. Unlike view model, a separate root model does not need to be created for each screen. For smaller apps, you can use a single model for your entire app. If your model is getting larger then it would be a good idea to think about separating it and distributing the responsibilities among different aggregate models. You break a root model into smaller models depending on the bounded context of the application domain. You don't create root models based on the number of screen of the app. 

You can read more about bounded context [here](https://martinfowler.com/bliki/BoundedContext.html). 

>> If you were following the MVVM pattern then you would have ended up with several view models including OrderListViewModel, OrderViewModel, AddOrderViewModel, OrderDetailViewModel and more. We have completely removed the view models from the picture and the view is directly consuming the models, which are supplied by the root model. Keep in mind ```View is the View Model``` in SwiftUI. 

>> View is the view model does not mean that you should start putting networking code in the View. As shown in this post, it is a good idea to create a separate networking layer so the same network requests can be invoked in other views.  

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

The model calls the OrderService to get all the orders and performs different actions related to orders like inserting, updating, sorting and filtering etc.   

>> You might be wondering that why can't the view directly call the OrderService and store all the orders in a local/private state using the @State property wrapper. You can definitely do that but since, we are allowing users to edit the orders on a separate screen and then refreshing on the original screen it would make more sense for the orders to be available globally. Also, if you store data in view's local state and the same data is needed by other views then you will have to pass the state to the child views using @Binding. Again, there is nothing wrong with that but it can end up being more work specially when you are passing state too deep into the view hierarchy.  

Another reason is that root model can provide sorting, filtering capabilities, which does not fit well in the OrderService. The root model can also invoke multiple services to aggregate and return data to the view and even providing caching support (through a caching layer) to the app.  

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

>> If needed you can also create a generic webservice, which has all the basic operations like getAll, getById, delete etc. 

Next, let's take a look at the implementation of the SwiftUI views. 

## Implementing Views

SwiftUI views are not just views but they are also view models. But that does not mean that you should put networking code right inside the view. 

>> By networking code I mean URLSession.shared.xxx. Although it will work but it will be harder to reuse the same networking calls from other views. This is the main reason we created the OrderService. In React apps, developers usually call the networking code using libraries like fetch or axios from directly inside the components. This is perfectly fine until, you need to make the same call in some other component. 

In our app all SwiftUI views need to access the root model object so they can persist and fetch orders. @EnvironmentObject will be a good fit for this scenario. The root model object is injected as an @EnvironmentObject in the root view. This is shown in the implementation below: 

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

>> In the above code I am injecting a hard-coded base url. In your  application, the url can be based on the environment (dev, test, qa, production). This will allow you to easy switch environments for testing purposes.  

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

Let's take a look at **AddNewCoffeeOrderView**. The AddNewCoffeeOrderView view is responsible for allowing the user to place a brand new order. The same view is also used to update an existing order. AddNewCoffeeOrderView also performs UI validation and displays error messages to the user. 

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

If you have a large form to validate then you can also introduce a view model, which will be populated through the TextFields. The view model can also provide validation support. One possible implementation is shown below: 

```swift 
struct AddCoffeeViewState {
    
    var name: String = ""
    var coffeeName: String = ""
    var total: String = ""
    var size: CoffeeSize = .medium
    var errors: AddNewCoffeeOrderError = AddNewCoffeeOrderError()
    
    mutating func isValid() -> Bool {
       
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
}
```

The same techniques for validation are used in React apps. If the field validation is unsuccessful then a flag is set, which displays the error message to the user. 

>> UI validation is NOT business logic. UI validation is just checking if the user has entered valid information. The business rules (if any) will be executed on the valid data. Let's say you are building a website, where the user can enter their credit score and get an APR rate (Interest Rate). The UI validation is going to check that the credit score TextField is not left empty. It will also check that the score only consists of numbers between a certain range. All of this will be UI validation. Once the user successfully submits the credit score, the system will run business rules to find out the appropriate APR for the user.  

Even HTML provides attributes to validate the user interface. Take a look at the code below: 

```html 
<input type = 'text' required />
```

The above code will validate the input field (TextBox), when the form is submitted. If the input field is blank then the form will not be submitted and the user will get an error message. This is an example of UI validation and NOT business rules.

>> In most client/server apps, business rules lives on the server and the client just provides the UI. 

One of the benefits of using the @EnvironmentObject is that when the orders are altered, views are automatically reevaluated. Reevaluation is not the same as rerendering. Revaluation means that SwiftUI will look at the diff tree and then decide, which views needs to be rerendered. Revaluation is a very fast process, so although your view body might be getting fired constantly it does not mean that all views in the body are getting rerendered. 

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

If you type in the TextField then you will notice that the body is fired each time. The main reason is that the @State variable **name** is getting new values from the TextField and it is causing the body to be reevaluated. But that does not mean that all views inside the body are getting rerendered. Only, the views that are changed are getting rerendered. 

In the same way when using the @EnvironmentObject, several views may get revaluated but only those who needs to be rerendered are rendered again. If you are getting unwanted rerendering then you can always split your @EnvironmentObject into multiple objects. This is shown in my article [Slicing Global State in SwiftUI Using Multiple EnvironmentObjects](https://azamsharp.com/2022/07/01/slicing-environment-object.html). 

# Testing
One of the arguments of using MVVM with SwiftUI is that it allows developers to easily perform unit testing for their views. This is a valid argument, because having a separate layer of view model does allow easy testing. You can invoke actions on the view model and witness changes on the view model properties. This kind of in-memory UI testing may not possible without an extra layer of view model but you can still write UI Tests for your SwiftUI applications. You can either use built-in Xcode UI Test Project or a framework called ViewInspector.

### Side Note
Kent Beck said it best “I get paid for code that works, not for tests, so my philosophy is to test as little as possible to reach a given level of confidence”.

Nowadays, I see developers religiously testing every single line of their code and aiming for that 100% code coverage. Developers are paid to write features/code, not unit tests. But I always witness in projects that test code is almost 3 times more as compared to the actual codebase.

Testing is definitely very important but only if you are writing meaningful tests. Consider a case, where we have to write a unit test for a following scenario.

>> A user should be able to add transaction to their existing budget

If this operation is part of your model then your test should create a user, add a budget for that user in the database. Then add a new transaction to that budget and then check if the transaction was added successfully or not.

Unfortunately, most developers will ignore the database part and run their test against a mocked object. In the end their test run fast and they are happy to see the test pass but what exactly did they test. They simply tested that their mock object work as expected. In this scenario a real test would hit the database and check if all the rules were met or not.

>> For the above scenario we are considering on device database like Sqlite being managed by Core Data or Realm.

I have worked with companies that have more than 2000+ tests. But if you looked closely you will find out the tests were not testing anything related to the business domain. They were actually testing the programming language. This is why it is extremely important to test the behaviors of your application instead of the implementation. When writing a test, ask yourself what business logic is being tested. If you cannot answer that question then stop writing the test.

>> Testing is very important that is why I give more precedence to domain layer unit tests and full system end-to-end functional tests. Functional system tests will ensure that the system works with all the other layers of the application. You don’t have to write tests for your controller or view models. All of those layers will be tested during the end to end functional tests.

I cover end-to-end testing in my [video](https://www.udemy.com/course/mv-design-pattern-in-ios-for-swiftui/?referralCode=4627986F77F533DEF0C7) course. 

## Conclusion 

SwiftUI already has MVVM built-in. This means you don't need an extra layer of view models for your applications. A lot of other developers are coming to the same [conclusion](https://vimeo.com/751534042/f1ae29434e) and rewriting their apps using features provided by SwiftUI. SwiftUI adds some magic to their property wrappers, which makes everything simple and performance efficient at the same time. For your next app, try creating views without view models and allow the views to directly talk to the model objects. 

Instead of fighting the framework try to embrace it. 

## Source Code

You can download the source code using the link below: 

[https://github.com/azamsharp/CoffeeAppMV]()

## Video Course

If you are interested in learning more about the MV Pattern in iOS then check out my brand new course

 [MV Design Pattern in iOS - Build SwiftUI Apps Apple's Way](https://www.udemy.com/course/mv-design-pattern-in-ios-for-swiftui/?referralCode=4627986F77F533DEF0C7)

## Resources 

- [SwiftUI View is also a View Model](https://azamsharp.com/2022/07/21/view-is-the-view-model.html)
- [WWDC - Data Flow Though SwiftUI](https://developer.apple.com/videos/play/wwdc2019/226/)
- [Udemy Course: MV Design Pattern in iOS - Build SwiftUI Apps Apple's Way](https://www.udemy.com/course/mv-design-pattern-in-ios-for-swiftui/?referralCode=4627986F77F533DEF0C7)
- [Lessons learned rewriting SoundCloud in SwiftUI](https://vimeo.com/751534042/f1ae29434e)
- [Fruta: Building a Feature-Rich App with SwiftUI](https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui)
- [Food Truck: Building a SwiftUI Multiplatform App](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app)
- [Meme Creator](https://developer.apple.com/tutorials/sample-apps/memecreator)





