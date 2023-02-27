## Understanding the Architecture 

The main idea behind the MV Pattern is to allow views directly talk to the model. This eliminate the need for creating unnecessary layer of view models for each view, which simply contribute to the size of the project instead of providing any benefits.  

> In SwiftUI, view is similar to Component in React or Widget in Flutter. This means, it is not only used for displaying data but can also handle binding capabilities. **The View in SwiftUI is also a View Model.**. Keep in mind that if you plan to put all the logic in your views then check out Container Pattern. MV Pattern is not the same as Container Pattern. 

Apple has shown MV pattern in several places with different flavors. This includes Fruta, FoodTruck and ScrumDinger applications. My focus for this article is on client/server applications as they are one of the most common types of iOS applications. 

In WWDC 2020 talk titled "Data Essentials in SwiftUI" Apple presented the following diagram. 

![ObservableObject as the data dependency surface](/images/single-source.png)

The main idea is to provide view access to a single layer that serves as source of truth and allows access to all the entities in the application.

Apple demonstrated this approach in their talk titled
**"Use Xcode for server-side development"**. The screenshot from the talk is shown below. 

![Use Xcode for server-side development](/images/xcode-server.png)

[Use Xcode for server-side development](https://developer.apple.com/videos/play/wwdc2022/110360/)

Apart from the WWDC video, Apple also demonstrated this technique in their Fruta and FoodTruck applications, although in their sample apps they did not use a network layer.  

Here is the updated diagram to supporting networking layer.  

![Aggregate Root](/images/aggregate-model-updated.001.jpeg)

> I know what you are thinking. Are we going to take advice based on Apple's code samples blindly? No! Never take any advice blindly. Always invest time and research the advantages and disadvantages of each approach. I have evaluated many different techniques and patterns and found this to be the best and simplest option when building client/server apps using SwiftUI. **Do your research!**  

Based on Apple's recommendation in their WWDC videos and code samples, I have been implementing a single aggregate model, which holds the entire state of the application. For small and medium sized apps, a single aggregate model might be enough. For complicated apps, you can have multiple aggregate models which will group related entities together. Multiple aggregate models are discussed later in the article. 

> Keep in mind that this article is about client/server apps. If you are using Core Data or anything else then you will have to do your research. For purely Core Data apps, I have been experimenting with Active Record Pattern. You can read about it [here](https://azamsharp.com/2023/01/30/active-record-pattern-swiftui-core-data.html). 

The implementation of my aggregate model is shown below: 

``` swift 
class StoreModel: ObservableObject {
    
    private var storeHTTPClient: StoreHTTPClient
    
    init(storeHTTPClient: StoreHTTPClient) {
        self.storeHTTPClient = storeHTTPClient
    }
    
    @Published var products: [Product] = []
    @Published var categories: [Category] = []
    
    func addProduct(_ product: Product) {
        // storeHTTPClient.addProduct(product)
    }
    
    func populateProducts() async throws {
        self.products = try await storeHTTPClient.loadProducts()
    }
}
```

> In Domain-Driven Design (DDD), an aggregate is a cluster of related objects that are treated as a single unit of work for the purpose of data consistency and transactional boundaries. An aggregate model, then, is a representation of an aggregate in code, typically as a class or group of classes.

```StoreModel``` is an aggregate model that centralizes all the data for the application. Views communicate directly with the StoreModel to perform queries and persistence operations. StoreModel also utilizes ```StoreHTTPClient```, which is used to perform network operations. StoreHTTPClient is a stateless network layer. This means it can be used in other parts of the application that are not SwiftUI, meaning UIKit. 

StoreModel can be used in a variety of different ways. You can use StoreModel as a @StateObject if you only want the data available to a particular view and if you want to tie the life of the object with the life of the view. But quite often I find myself adding StoreModel to @EnvironmentObject so that it can be available in the injected view and all its sub views.   

``` swift 
@main
struct StoreAppApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(StoreModel(client: StoreHTTPClient()))
            
        }
    }
}
```

Now, you can use the ```StoreModel``` as shown in the implementation below. 

``` swift 
struct ContentView: View {

    @EnvironmentObject private var model: StoreModel
    
    var body: some View {
        ProductListView(products: model.products)
            .task {
                do {
                    try await model.populateProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
    }
}
```

> You might be tempted to use @EnvironmentObject inside all views. Although, it will work as expected but for larger applications you need to make presentation views independent of any dependencies. Presentation views are usually child views that are created for the purpose of reusability. 

Apart from fetching and persistence, StoreModel can also provide sorting, filtering, searching and other operations etc. 

A single StoreModel is ideal for small or even medium sized apps. But for larger apps it will be a good idea to introduce multiple aggregate models based on the bounded context of the application. In the next section, we will cover how to introduce multiple aggregate root models and how they can benefit when working in large teams.  

## Multiple Aggregate Models 

As you learned in the previous section, the purpose of an aggregate model is to expose data to your view. As Luca explained in [Data Essentials in SwiftUI WWDC 2020 (11:30)](https://developer.apple.com/videos/play/wwdc2020/10040/) "The aggregate model is an ```ObservableObject```, which acts as your data dependency surface. This allows us to model the data using value type and manage its life cycle and side effects with a reference type."  

As your business grows, a single aggregate model might not be enough to maintain the life cycle and side effects of an entire application. This is where we will introduce multiple aggregate models. These aggregate models are based on the bounded context of the application. Bounded context refers to a specific area of the system that has clear boundaries and is designed to serve a particular business purpose or domain. 

In an e-commerce application, we have have several bounded contexts including checkout process, inventory management system, catalog, fulfillment, shipment, ordering, marketing and customer management module. 

Defining bounded context is important in software development and it helps to break down the application into small manageable pieces. This also allows teams to work on different parts of the system without interfering with each other. 

Developers are usually not good in finding bounded context of software applications. The main reason is that their technical knowledge does not directly map into domain knowledge. Domain knowledge requires different set of skills and a domain expert is a better fit for this kind of role. A domain expert is a person, who may not be tech savvy but understands how the business or a particular domain works. In large projects, you may have multiple domain experts each handling a different business domain. This is why it is extremely important for developers to communicate with domain experts and understand the domain before starting any development.  

Once, you have identified different bounded contexts associated with your application you can represent them in the form of aggregate models. This is shown in the diagram below. 

![Multiple Aggregate Root](/images/aggregate-model-updated.002.jpeg)

The network layer can also be divided into multiple HTTP clients or you can use a single generic network layer for your entire application. This is shown in the diagram below.  

![Multiple Aggregate Root](/images/aggregate-model-updated.003.jpeg)

The Catalog aggregate model will be responsible for providing the view with all the entities associated with Catalog. This can include but not limited to: 

- Product 
- Category 
- Brand 
- Review 

The Ordering aggregate model will be responsible for providing view with all the ordering related entities. This can include but not limited to:  

- Order 
- OrderLineItem 
- OrderStatus
- ShippingMethod 
- Discount 

The ```Catalog``` and ```Ordering``` aggregate models are be reference types conforming to ObservableObject. And all the entities they provide will be value types. 

The outline of ```Catalog``` aggregate model and ```Product``` entity is shown below: 

``` swift 

struct Product: Codable {
    let productId: Int
    let name: String
    let category: Category
    let price: Double
    let description: String
    let reviews: [Review]?
}

@MainActor 
class Catalog: ObservableObject {
    
    // designated or generic HTTP client 
    let storeHTTPClient: StoreHTTPClient
    
    @Published var products: [Product]
    @Published var categories: [Category]
    
    init(storeHTTPClient: StoreHTTPClient) {
        self.storeHTTPClient = storeHTTPClient
    }
    
    func loadProducts() {
         products = storeHTTPClient.loadProducts
    }
    
    func getProductById(_ productId: Int) -> Product? {
        // fetch product by id 
    }
    
    func getProductsByCategory(_ categoryId: Int) -> [Product] {
       // get products by category
    }
    
    func getCategories() -> [Category] {
        categories = storeHTTPClient.loadCategories()
    }
}
```

Catalog and Ordering aggregate models are injected into the application as an environment object. You can inject them directly in your application root view or the root view of each bounded context. The later is shown below: 

``` swift 
@main
struct StoreAppApp: App {
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(Catalog(client: CatalogHTTPClient()))
                .environmentObject(Ordering(client: OrderingHTTPClient()))
            
        }
    }
}
```

Now, inside a view you can access Catalog or Order by accessing it through ```@EnvironmentObject```. This is shown below: 

``` swift 
struct CatalogListScreen: View {
    
    @EnvironmentObject private var catalog: Catalog
    
    var body: some View {
        List(catalog.products) { product in
            Text(product.name)
        }.task {
            do {
                try await catalog.loadProducts()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

If your view needs to access order then it can utilize the Ordering aggregate model. This is shown below: 

``` swift 
struct AdminDashboardScreen: View {
    
    @EnvironmentObject private var catalog: Catalog
    @EnvironmentObject private var ordering: Ordering
    
    var body: some View {
        VStack {
            List(catalog.products) { product in
                Text(product.name)
            }
            List(ordering.allOrders) { order in
                Text(order.status)
            }
        }.task {
            do {
                try await catalog.loadProducts()
                try await ordering.loadOrders()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

There are scenarios when your aggregate model will need to access information from another aggregate model. In those cases, your aggregate model will simply use the network service to fetch the information that is needed. 

> It is important that your caching layer is called from within the network layer and not from aggregate models. This will allow all aggregate models to take advantage of caching through the network layer. 

As mentioned earlier for small or even medium sized apps, you may only need a single aggregate model. For larger apps you can introduce new aggregate models. Make sure to consult with a domain expert before creating application boundaries. 

The concept of domain boundaries can also be applied to user interfaces. This allows us to reuse user interface elements in other applications. 

![Factor out common pieces](/images/user-interface.png)

> You can factor out common interface elements using Swift Package Manager and import those packages into other applications. 

Let's go ahead and zoom out and see how our architecture looks like with all the pieces in place. 

![Architecture](/images/architecture.001.jpeg)

As discussed earlier, each bounded context is represented by its own module. These modules can be represented by a folder or a package dependency.

**CatalogUI:** Represents user interface associated with catalog. This can include all the catalog specific stuff like AddCatalogScreen, UpdateCatalogScreen etc. 

**Catalog:** Represents the models associated with catalog. This will contain the aggregate model and all the entities exposed by the aggregate model.

**CatalogKit**: Represents the HTTP client for accessing catalog services. 

**Foundation Core**: Represents the resources used by all modules. This can include helper classes/structs, reusable views, images, icons and even preview content used for testing. 

> Each module like Shipping, Inventory, Ordering etc can be represented by a folder structure of a package dependency. This really depends on how reusable your module is and if you wish to use it in other projects. 

Using this architecture, future aggregate models and data access services can be added without interfering with existing ones. This also allows more collaborative environment as different teams can work on different modules without interfering with each other. 