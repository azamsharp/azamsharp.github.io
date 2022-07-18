# How MVVM Design Pattern Work Against the SwiftUI Framework 

SwiftUI was introduced at WWDC 2019 and it completely changed how we build our apps for Apple platform. SwiftUI provided a declarative framework, which allowed developers to quickly and easily build user interface as compared to its predecessor UIKit or AppKit. Somewhere along the lines we also adopted MVVM (Model View ViewModel) design pattern as the default pattern when building SwitUI applications. In this post, I will cover my experience of using MVVM pattern with SwiftUI framework and how with it resulted in an anti-pattern when building SwiftUI applications. 


## Note

Before I begin, I want to point out that I was the biggest advocate of the MVVM design pattern. I have written books on SwiftUI and MVVM, I created video courses, YouTube videos and even wrote several articles explaining the benefits of using MVVM pattern for SwiftUI applications. If you are a developer and you jumped on the MVVM bandwagon then I was the driver. 

## React and Flutter 

I started rock climbing few months ago. Some climbs need more technical skills rather than brute strength. During the beginning when I was stuck at a particular route, I would ask someone for the Beta. Beta is a fancy term for instructions on how to complete a climb. Once, I understand the Beta, the climbs becomes much easier. 

SwiftUI is not the first declarative framework. React and Flutter existed before SwiftUI. When implementing solutions in React or Flutter, you will rarely hear the term MVVM. Just like Beta in rock climbing, we can learn a great deal from more mature frameworks like React. React mainly uses Context API or Redux for providing data to its components (views in SwiftUI). Flutter uses bloc, provider and also Redux. Even in Apple's own documentation, they never mentioned the word MVVM or even the term View Model. We as a community (myself included) just thought that MVVM might be a good design pattern to use when building SwiftUI applications. It turns out that if you use MVVM with SwiftUI then you will constantly be fighting the SwiftUI framework. Let's first understand how MVVM adds an extra layer to our application.   

## Layers 

> All problems can be solved by adding another layer, expect the problem of having too many layers. 

The main idea behind View Model is to add an extra layer between the model and the user interface. The View Model can trigger the API/Service and fetch all the records and then pass it down to the view, where it can be displayed. Below you can see the diagram of the MVVM flow in a client server application. 

![MVVM in Client Server Application](/images/2022-07-17-mvvm-swiftui-1.png)

> The View Model does not have the code to perform network request. View Model communicates with an APIService/Webservice to make a call, which returns the Model/DTO objects, which are later populated in a ```@Published``` property of the View Model. 

One very important thing to note in the diagram is the use of word Model or DTO but not **domain model**. The client (SwiftUI App) does not have a domain model. The main logic of the application resides on the server, hence the domain logic/layer is on the server not on the client.

Here is a typical code flow for consuming an API in a SwiftUI application using the MVVM principles. 

The ```Webservice``` is responsible for making the actual request to the server and getting the results. 

``` swift 
class Webservice {
    
    func fetchAllProducts() async throws -> [Product] {
        let (data, _) = try await URLSession.shared.data(from: URL(string: "https://fakestoreapi.com/products")!)
        return try JSONDecoder().decode([Product].self, from: data)
    }
    
}
```

The View Model calls the Webservice and populates an array of ProductViewModel. ProductViewModel simply, exposes the data already contained in the model/DTO object. The implementation is shown below: 

``` swift 
@MainActor
class ProductListViewModel: ObservableObject {
    
    @Published var products: [ProductViewModel] = []
    
    func getProducts() async {
        
        do {
            let products = try await Webservice().fetchAllProducts()
            self.products = products.map(ProductViewModel.init)
        } catch {
            print(error.localizedDescription)
        }
        
    }
    
}

struct ProductViewModel: Identifiable {
    private var product: Product
    
    init(product: Product) {
        self.product = product
    }
    
    var id: Int {
        product.id
    }
    
    var title: String {
        product.title
    }
    
    var price: Double {
        product.price
    }
    
    var description: String {
        product.description
    }
} 
```
Finally, ContentView uses the ProductListViewModel and displays the data on the screen. The code is shown below: 

``` swift 
struct ContentView: View {
    
    @StateObject private var vm: ProductListViewModel = ProductListViewModel()
    
    var body: some View {
        List(vm.products) { product in
            VStack(alignment: .leading) {
                Text(product.title)
                Text(product.description)
                    .font(.caption)
                // display price and other information 
            }
        }
        .listStyle(.plain)
        .task {
            await vm.getProducts()
        }
    }
}
```
![Displays Products on the Screen](/images/2022-07-17-mvvm-swiftui-2.png)

Great! We can see the products displayed on the screen. At this point the main question is that what is the purpose of View Model. You might say that View Model is providing an extra layer between the View and the **Domain Model**. That statement is WRONG! The View Model is not providing a layer/separating between the View and the **Domain Model**. The View Model is providing separation between the View and the Model/DTO. But there is no reason for such separation. A DTO/Model (Not Domain Model) can be given directly to the View. 

Let's update our Webservice to support these changes. 

``` swift 
@MainActor
class Webservice: ObservableObject {
    
    @Published var products: [Product] = []
    
    func fetchAllProducts() async throws {
        let (data, _) = try await URLSession.shared.data(from: URL(string: "https://fakestoreapi.com/products")!)
        self.products = try JSONDecoder().decode([Product].self, from: data)
    }
    
}
```

Now, the website is responsible for keeping the state of products. Webservice is also marked with ObservableObject, which means that the changes to the Webservice ```@Published``` properties can be observed. 

We no longer need the View Model. The View has been updated to call Webservice directly. This is shown below: 

``` swift 
struct ContentView: View {
    
    @StateObject private var service = Webservice()
    
    var body: some View {
        List(service.products, id: \.id) { product in
            VStack(alignment: .leading) {
                Text(product.title)
                Text(product.description)
                    .font(.caption)
                // display price and other information 
            }
        }
        .listStyle(.plain)
        .task {
            do {
                try await service.fetchAllProducts()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

The result is exactly the same. All the products will get displayed successfully. 

> The main confusion stems from the idea that View is directly consuming the domain model. The reality is that the data sent from the server to the client is not the domain model but simply a DTO object, which in our case is based on the JSON returned from the server. 

## Global State:  

One of the built in features of SwiftUI is the concept of Global State. Global state contains the data available to all views. When the data changes then those views (who are interested in global state) automatically gets re-rendered. This is similar to the Redux pattern in React. 

Unlike React, which requires a lot of work to setup and use global state, global state is easily accessible in SwiftUI applications using ```@EnvironmentObject``` property wrapper.

The @EnvironmentObject is only available inside the Views. If you want to access @EnvironmentObject inside a View Model then you will have to pass it through the construction (dependency injection). 



As you can see sharing global state when using View Models becomes hard. On the other hand, if we remove View Models from the equation then things get very simple. 

We implement CounterService class, which maintains the counter state. This is implemented below: 

``` swift 
class CounterService: ObservableObject {
    
    @Published var counter: Int = 0
    
    
    func increment() {
        counter += 1 
    }
    
}
```

We inject it into the root view as shown below: 

``` swift 
 ContentView().environmentObject(CounterService())
```

And finally, we can access it from inside the ContentView. 

``` swift 
struct ContentView: View {
    
    @EnvironmentObject private var counterService: CounterService
    
    var body: some View {
        VStack {
            Text("\(counterService.counter)")
            Button("Increment") {
                counterService.increment()
            }
        }
        
    }
} 
```

Pretty simple if you don't fight the SwiftUI framework right! 

In WWDC 2019 talk [Data Flow Through SwiftUI](https://developer.apple.com/videos/play/wwdc2019/226/) Apple talked about single source of truth. Single source of truth basically means that the data should flow from one direction. EnvironmentObject in SwiftUI allow us to make apps with un-directional flow. Let's take the same example of products API and see how we can store list of products in the global state without using the MVVM pattern.    

We will start with the same webservice as shown below: 

``` swift 
class Webservice: ObservableObject {
    
    @Published var products: [Product] = []
    
    func fetchAllProducts() async throws {
        let (data, _) = try await URLSession.shared.data(from: URL(string: "https://fakestoreapi.com/products")!)
        
        Task { @MainActor in
            self.products = try JSONDecoder().decode([Product].self, from: data)
        }
       
    }
    
}
```

The ```Webservice`` will host list of products. Now, we can add Webservice in the environment object. This is shown below: 

``` swift 
@main
struct LearnApp: App {
    
    let webservice = Webservice()
    
    var body: some Scene {
        WindowGroup {
            ContentView().environmentObject(webservice)
        }
    }
}
```

Now finally we can use it in the view. 

``` swift 
struct ContentView: View {
    
    @EnvironmentObject private var webservice: Webservice
    
    var body: some View {
        VStack {
            List(webservice.products, id: \.id) { product in
                Text(product.title)
            }.task {
                try? await webservice.fetchAllProducts()
            }
        }
        
    }
}
```

Much simpler than using View Models and injecting environment objects into the view models. 

When using @EnvironmentObjects, one of the biggest complain is that since it is a global state, if the state gets updated then a lot of views will be needlessly rendered. You can easily avoid that by slicing the global state into smaller pieces. Let's say you want to manage the products in the global state but you also want to put a counter in the global state. You don't want to render the counter view if the products gets updated. The solution is to simply divide the state into multiple slices. This is shown below: 

Below, you can see the code for Webservice which manages the products. 

``` swift 
class Webservice: ObservableObject {
    
    @Published var products: [Product] = []
    
    func fetchAllProducts() async throws {
        let (data, _) = try await URLSession.shared.data(from: URL(string: "https://fakestoreapi.com/products")!)
        
        Task { @MainActor in
            self.products = try JSONDecoder().decode([Product].self, from: data)
        }
       
    }   
}
```

And for the counter, we have created another slice as shown below: 

``` swift 
class CounterService: ObservableObject {
    
    @Published var counter: Int = 0
    
    func increment() {
        counter += 1 
    }   
}
```

Finally, we inject both of them as an environment object to the root view. 

``` swift 
@main
struct LearnApp: App {
    
    let webservice = Webservice()
    let counterService = CounterService()
    
    var body: some Scene {
        WindowGroup {
            
            ContentView()
                .environmentObject(webservice)
                .environmentObject(counterService)
        }
    }
}
```

That's it! Now when the counter gets updated only CounterView will get rendered and when the products get updated then only the ContentView will get rendered. I wrote an article about slicing the global environment state. You can read it here. 

[Slicing Global State in SwiftUI Using Multiple EnvironmentObjects](https://azamsharp.com/2022/07/01/slicing-environment-object.html)

> One thing I have noticed with every SwiftUI project is that sooner or later you always need a global state with a uni-directional flow. This was one of the biggest pain point when using MVVM design pattern with SwiftUI. Managing global state using View Models can get very tricky. 

## Business Logic 

One of the most common reasons developers use the MVVM pattern with SwiftUI is because they don't want their view to directly access the domain layer. But as I mentioned earlier, when using client server application your domain layer will exist on the server side. The client (SwiftUI) will only be responsible for using the server endpoints to perform CRUD operations. 

## Core Data 

SwiftUI consists of @FetchRequest and @SectionedFetchRequest property wrapper for working with Core Data. @FetchRequest property wrapper not only fetches data from the persistent storage but also makes sure that the user interface is up to date by rendering it when the data changes. 

One main complain about using @FetchRequest is that we are exposing our domain objects to the view. The objects you retrieve using Core Data are not domain objects, they are data model objects. 

If your application is not server based then you can introduce local domain services that can act and perform logical operations on those objects. This means the objects will still remain data model objects but the main logic and rules will reside in local domain services. 

Going back to the @FetchRequest property wrapper. If you try to use MVVM pattern with Core Data then you will need to implement @FetchRequest feature by yourself. This means you will end up implement NSFetchedResultsController, which requires a lot of code. 

You can check out my YouTube video in which I demonstrate how to use MVVM pattern and implement NSFetchedResultsController. 

[Core Data MVVM in SwiftUI App Using NSFetchedResultsController](https://youtu.be/gGM_Qn3CUfQ)

> I may write a separate article in which I will discuss how to design apps that don't have a server component but still consists of business logic. 


## Realm 

Recently, I was working on some sponsored videos for Realm. Realm is a framework that allows local persistence and also cloud sync services using object oriented database and provides much better API as compared to Core Data. 

Realm has implemented special property wrappers to work seamlessly with SwiftUI. It allows the user interface to always stay in-sync with the data and even provides easy methods to perform create, update and delete operations. 

If I had used MVVM pattern when working with Realm then I may have not taken advantage of all the available features. In short, I would be fighting the framework every step of the way. 

## Unit Testing 

When writing tests, our main focus should be testing the application domain. This is the heart of the application. Domain layer consists of all the rules and logic that powers the application. In a client/server application, domain layer exists on the server. So this means you will be writing unit tests for the code that is hosted on the server. 

Next, you will be implementing your UI tests and then finally the integration tests. Even if you were using MVVM design pattern, you don't have to write tests for the the view models because view models don't contain any logic. 

Please don't write tests just for the sake if writing tests. Write good tests and make sure your domain is well tested with high code coverage. 

## Conclusion 

SwiftUI is designed in such a way that the view also serves as the model. @State and @Binding allows for two-way binding and @EnvironmentObject provides global state. SwiftUI is a really powerful framework but only if you use it in a way, it was designed to be used. MVVM pattern is great, when used with frameworks that can support it. SwiftUI does not need MVVM pattern, actually MVVM acts as an anti-pattern when building SwiftUI applications. 

Next time when building a SwiftUI app, try to surrender to the framework instead of fighting it. Your productivity will increase and you will not hit brick walls. 





