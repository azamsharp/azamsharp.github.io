# Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture

Software architecture is always a topic for hot debate, specially when there are so many different choices. For the last 8-12 months, I have been experimenting with MV pattern to build client/server apps and wrote about it in my original article [SwiftUI Architecture - A Complete Guide to MV Pattern Approach](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). In this article, I will discuss how MV pattern can be applied to build large scale client/server applications. 

> Architecture and patterns depends on the type of application you are building. No single architecture will work in all scenarios. Choose the best architecture suitable for your application needs. 

The outline of this article is shown below: 

- [Modular Architecture](#modular-architecture) 
- [Understanding the MV Pattern](#understanding-the-mv-pattern)    
- [Screens vs Views](#screens-vs-views) 
- [Multiple Aggregate Models](#multiple-aggregate-models) 
- [View Specific Logic](#view-specific-logic)
- [Validation](#validation) 
- [Navigation](#navigation) 
- [Grouping View Events](#grouping-view-events) 
- [Testing](#testing) 

## Modular Architecture 

Modular architecture in software refers to the design and organization of software systems into small, self contained modules or components. These modules can be tested and maintained independently of one another. Each module serves a specific purpose and solve a specific business requirement. 

Modular architecture also provides advantages when working on large projects consisting of multiple teams. Each team can work on a particular module, without interfering with each other. 

> If you are working on a module that will be consumed or used by other teams then make sure that you are communicating with them and not creating the module in complete isolation. A lot of problems in software development exists solely because of lack of communication between teams.   

Modularity can be achieved in several different ways. You can expose each module as a package (SPM), which can be imported into different applications. Modularity can also be achieved by structuring your app based on specific grouping or folder structure. Keep in mind that when using folders for modularity you have to pay special attention to separation of concerns and single responsibility principles. 

> The focus of this article is not Swift Package Manager, but how to achieve modularity by breaking the app based on the bounded context of the application. **Swift Package Manager can be used to package those dependencies into reusable modules.** 

## MVVM in SwiftUI (Updated) 

In the traditional MVVM pattern, when you create a new view, you also create a corresponding view model (though not all views require one). The view model's role is to manage bindings, handle network operations (using a network layer), manage validations, among other tasks. Let's consider an example of an app that displays a movie list, allows you to add a new movie, and shows movie details. To structure such an app using the MVVM pattern, you can follow the approach outlined below.

| View | View Model |
| --- | ----------- |
| MovieListView | MovieListViewModel |
| AddMovieView | AddMovieViewModel |
| MovieDetailView | MovieDetailViewModel | 

A typical view model can consists of networking code (through a networking layer), UI validation and data transformation code. Each of these view models is conforming to ObservableObject protocol. The main purpose of ObservableObject protocol is used to define a new source of truth. Source of truth is one of the most important concepts in SwiftUI as it is responsible for keeping the data and the view sync. In a client/server application, source of truth is the server. So, if the source of truth is the same, why are we creating new source of truths by conforming to ObservableObject protocol. 

> This doesn't imply that you ought to restrict yourself to just one ObservableObject for your entire application. While you can certainly incorporate multiple ObservableObjects into your application, their introduction should not be solely driven by the inclusion of a new view. The primary motivation behind introducing a new ObservableObject should stem from the integration of a new source of truth. Furthermore, you'll come to understand that there are instances where you'll introduce ObservableObjects based on the bounded context of the application.

Another concern arises from the fact that each of these view models relies on the networking layer. This implies that a networking service must be provided as a dependency for every individual view model. In certain scenarios, you might even find yourself injecting a view model into a view that also relies on a network service. This can introduce significant complexity and become unwieldy, especially in larger applications featuring numerous view models.

This scenario is shown in the code below: 

```swift
protocol HTTPClientProtocol {
    // contract
}

struct HTTPClient: HTTPClientProtocol {
    // conform to the contract
}

struct Movie: Codable {
    let name: String
}

class MovieListViewModel: ObservableObject {
    
    @Published var movies: [Movie] = []
    let httpClient: HTTPClientProtocol
    
    init(httpClient: HTTPClientProtocol) {
        self.httpClient = httpClient
    }
    
    func loadMovies() {
        // movies = httpClient.load(resource...)
    }
}

struct MovieListView: View {
    
    let vm: MovieListViewModel
    
    var body: some View {
        Text("Display Movies")
    }
}

struct MoviesApp: App {
    var body: some Scene {
        MovieListView(vm: MovieListViewModel(httpClient: HTTPClient()))
    }
}

```

Lastly, the functionality offered by a view model is already available in the view. The primary responsibility of a view model is to support binding. This is already possible inside the view through the use of ```@State```, ```@Binding```, ```@EnvironmentObject``` property wrappers. I strongly believe that Apple did a disservice to the iOS community by calling it a view. They should have called it a component or something similar. This has led to a lot of confusion in the iOS community. The word view entails that it is a visual only component like HTML in web development or XAML in WPF. But that is not the case. The view in SwiftUI is the return from the ```body``` property. Even that is not an actual view but just the declaration of the view. 

SwiftUI views are mapped to the UIView counter parts and then rendered on the screen. SwiftUI views are just basic value type structs. They have no knowledge on how to paint pixels on the screen. SwiftUI uses the declaration of the views to render actual UIViews. The mapping for some of the common SwiftUI to UIView is shown below:

| SwiftUI View | UIView |
| --- | ----------- |
| List | UICollectionView |
| Text | UILabel |
| TextField | UITextField | 

All this mapping is hidden from the developers but you can always look at the inner details through the use of Instruments. 

TODO: screenshot from instruments

The views in SwiftUI, reminds me of ReactJS JSX syntax. Let's take a look at a very small example. 

``` swift 
function App() {
    return (
        <div>
            <h1>Hello World</h1>
            <button>Save</button>
        </div>
    )
}
```

In the above ReactJS code, we have created a functional component called ```App```. The App component returns a ```<div>``` element containing a ```<h1>``` and ```<button>```. The thing to notice here is that those are not actual HTML elements. Those are virtual DOM (Document Object Model) elements managed by React framework. The main reason is that React needs to track changes to those elements so it can only render, what has changed. Once React finds out the changed elements using the diffing process, those virtual DOM elements are used to render real HTML elements on the screen.  

I believe SwiftUI uses the same concepts internally. The views in the body property are not actual views but the declaration of views. Eventually, those views gets converted to real views and then displayed on the screen. John Sundell also talked about it in his article [SwiftUI views versus modifiers](https://www.swiftbysundell.com/articles/swiftui-views-versus-modifiers/). 

If you are interested in learning more about the concept of virtual DOM then check out this talk title [Tom Occhino and Jordan Walke: JS Apps at Facebook](https://youtu.be/GW0rj4sNH2w?t=301). This is the talk, where Facebook introduced ReactJS to the public. 

## Understanding the MV Pattern 

The main idea behind the MV pattern is to allow views directly talk to the model. In MV pattern, **views are the view model**. This eliminates the need for creating unnecessary view models for each view, which simply contributes to the size of the project but does not provide any additional benefits. 

> MV pattern does not advocate putting all the business logic inside a view. That particular pattern is known as [Container Pattern](https://www.patterns.dev/posts/presentational-container-pattern/). I have also talked about it on my blog. You can read about it [here](https://azamsharp.com/2023/01/24/introduction-to-container-pattern.html).   

MV pattern can take different forms depending on the type of app you are writing. This article is mainly focused on a client/server apps, where a SwiftUI app serves as a client.

In WWDC 2020 talk titled [Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/) Apple presented the following diagram. 

![ObservableObject as the data dependency surface](/images/single-source.png)

The central concept revolves around granting views access to a unified layer or interface, which functions as the definitive source of information and grants entry to all components within the application. On occasions, this unified layer might correspond to the network layer. This approach is advisable in situations where your views autonomously consume the data, and making alterations to the data doesn't necessitate synchronization with other segments of your application. A straightforward example could involve a third-party service that furnishes a list of up-to-date news articles. In this case, your view can directly interact with the network layer to showcase the news articles. This concept is illustrated below:

```swift
struct NewsListScreen: View {
    
    @Environment(\.httpClient) private var httpClient
    @State private var articles: [Article] = []
    
    private var sortedArticles: [Article] {
        articles.sorted { lhs, rhs in
            // sorting logic
        }
    }
    
    private func loadArticles() async {
        let resource = Resource(Constants.Urls.articles)
        do {
            try articles = httpClient.load(resource)
        } catch {
            // handle error
        }
            
    }
    
    var body: some View {
        List(sortedArticles) { article in
            ArticleView(article)
        }.task {
            await loadArticles()
        }
        
    }
}
```

The ```NewsListScreen``` serves as a container view. This means it is responsible for making the network call and fetching the data. Once the data is fetched, it can be passed down to the presentation view. At present the only reusable view we have in the above code is ```ArticleView```. Depending on your app and requirements, you can also extract out List into a separate ```ArticleListView``` component. 

Another thing to notice in the above code is the use of ```sortedArticles``` private property. As I mentioned earlier that in MV pattern, views are the view models. This is no need to create a view model associated with ```NewsListScreen```. If your view is getting large then use view decomposition to break it into smaller pieces. Keep in mind that views in SwiftUI are value types. Value types are cheap to create. This gives you the flexibility to break your view into multiple reusable pieces.  

> If you are wondering how would you test the sortedArticles property then keep reading. Testing will be covered in the later section of this article. 

Apple sample projects, which includes [Fruta](https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui) and [FoodTruck](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app) applications demonstrated how to use this pattern against a hard-coded data source. But in WWDC video title **"[Use Xcode for server-side development](https://developer.apple.com/videos/play/wwdc2022/110360/)"** Apple showed how to update the existing FoodTruck app and consume the data from an API response. 

The screenshot below shows ```FoodTruckModel``` using the ```DonutsServerClient``` to retrieve list of donuts. DonutsServerClient is responsible for making an actual request to the server and downloading the donuts. Once the donuts are downloaded they are assigned to the serverDonuts property maintained by the FoodTruckModel.  

![Use Xcode for server-side development](/images/xcode-server.png)

[Use Xcode for server-side development](https://developer.apple.com/videos/play/wwdc2022/110360/)

Here is the updated diagram to support the networking layer.  

![Aggregate Root](/images/aggregate-model-updated.001.jpeg)

> I know what you are thinking. Are we going to take advice based on Apple's code samples blindly? No! Never take any advice blindly. Always invest time and research and weigh the advantages and disadvantages of each approach. I have evaluated many different techniques and patterns and found this to be the best and simplest option when building client/server apps using SwiftUI. **Do your research!**.   

Based on Apple's recommendation in their WWDC videos and code samples and my own personal experience, I have been implementing a single aggregate model, which holds the entire state of the application. For small and medium sized apps, a single aggregate model might be enough. For complicated apps, you can have multiple aggregate models which will group related entities together. Multiple aggregate models are discussed later in this article. 

> Once again keep in mind that this article is about client/server apps. If you are using Core Data or anything else then you will have to do your research. For purely Core Data apps, I have been experimenting with Active Record Pattern. You can read about it [here](https://azamsharp.com/2023/01/30/active-record-pattern-swiftui-core-data.html). 

Following the pattern discussed in [Use Xcode for server-side development](https://developer.apple.com/videos/play/wwdc2022/110360/) talk, here is the StoreModel I have implemented for my application.  

``` swift 
class StoreModel: ObservableObject {
    
    private var storeHTTPClient: StoreHTTPClient
    
    init(storeHTTPClient: StoreHTTPClient) {
        self.storeHTTPClient = storeHTTPClient
    }
    
    @Published var products: [Product] = []
    @Published var categories: [Category] = []
    
    func addProduct(_ product: Product) async throws {
         try await storeHTTPClient.addProduct(product)
    }
    
    func populateProducts() async throws {
        self.products = try await storeHTTPClient.loadProducts()
    }
}
```

```StoreModel``` is an aggregate model that centralizes all the data for the application. Views communicate directly with the StoreModel to perform queries and persistence operations. StoreModel also utilizes ```StoreHTTPClient```, which is used to perform network operations. StoreHTTPClient is a stateless network layer. This means it can be used in other parts of the application that are not SwiftUI, meaning UIKit or even on a different platform (macOS).  

> In Domain-Driven Design (DDD), an aggregate is a cluster of related objects that are treated as a single unit of work for the purpose of data consistency and transactional boundaries. An aggregate model, then, is a representation of an aggregate in code, typically as a class or group of classes.

StoreModel can be used in a variety of different ways. You can use StoreModel as a @StateObject if you only want the data available to a particular view and if you want to tie the object with the lifetime of the view. But quite often I find myself adding StoreModel to @EnvironmentObject so that it can be available in the injected view and all of its sub views.   

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

After the StoreModel is injected through the @EnvironmentObject, you can access the ```StoreModel``` as shown in the implementation below. 

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

> You might be tempted to use ```@EnvironmentObject``` inside all the views. Although, it will work as expected but for larger applications you need to make presentation views free of any dependencies. Presentation views are usually child views that are created for the purpose of reusability. It you try to access ```@EnvironmentObject``` inside the child views then it effects their reusability status and they become less useful. The main reason is that now they are dependent on the ```@EnvironmentObject``` to provide data to them. Instead we should follow the top-down approach, where the data is passed from the parent view to the child view. This is also known as the [Container/Presentation pattern](https://www.patterns.dev/posts/presentational-container-pattern/).    

Apart from fetching and persistence, StoreModel can also provide sorting, filtering, searching and other operations directly to the view. 

> If I was using the traditional MVVM pattern then I would create several view models to accommodate each screen. This can include ```ProductListViewModel```, ```ProductViewModel```, ```AddProductViewModel```, ```ProductDetailViewModel``` and many more. Most of the time, these view models end up with one or two functions and maintaining a single source of truth can become very hard. In MV pattern, the view itself is the view model so we don't need to create unnecessary view models for most of the time. The view, which is also a view model is simply going to ask model (Aggregate Model) for the data. 

**The source of truth in a client/server application is the server.**. This means you should not be adding view models conforming to ObservableObject (new source of truth) protocol just because you added a new view. The source of truth for that view has not changed, it is still the server. 

A single StoreModel is ideal for small or even medium sized apps. But for larger apps it will be a good idea to introduce multiple aggregate models based on the bounded context of the application. In the next section, we will cover multiple aggregate models and how they benefit when working in large teams.  

The MV pattern and MVVM pattern may appear similar at first glance, but they exhibit significant differences. In a client/server app using MVVM, a separate view model is created for each screen. For instance, in a movies management app, you might end up with multiple view models like MovieListViewModel, AddMovieViewModel, MovieDetailViewModel, and possibly MovieViewModel. 

However, MV takes a different approach altogether, omitting the creation of view models. Instead, it directly binds the DTO objects (models from the server) to the view. In some cases, the view can directly utilize the network layer to fetch the DTO objects, while in others, a single Data Store or Aggregate Model (ObservableObject) aids in accessing the movies. Any required UI validation or data transformation is implemented within the view itself.

To ensure reusability of child views, the container and presenter pattern come into play. One view is responsible for requesting and obtaining the data, while another view, known as the presenter, takes charge of displaying the data. This separation of responsibilities ensures a more modular and maintainable design.

With MV, the overall structure differs from MVVM, allowing for a more direct handling of data and views without the need for multiple view models. By leveraging the container and presenter pattern, your app can achieve better organization and reusability of views.

## Multiple Aggregate Models 

As you learned in the previous section, the purpose of an aggregate model is to expose data to your view. As Luca explained in [Data Essentials in SwiftUI WWDC 2020 (11:30)](https://developer.apple.com/videos/play/wwdc2020/10040/) "The aggregate model is an ```ObservableObject```, which acts as your data dependency surface. This allows us to model the data using value type and manage its life cycle and side effects with a reference type."  

As your business grows, a single aggregate model might not be enough to maintain the life cycle and side effects of an entire application. This is where we will introduce multiple aggregate models. These aggregate models are based on the bounded context of the application. Bounded context refers to a specific area of the system that has clear boundaries and is designed to serve a particular business purpose. 

In an e-commerce application, we can have several bounded contexts including checkout process, inventory management system, catalog, fulfillment, shipment, ordering, marketing and customer management modules. 

Defining bounded context is important in software development and it helps to break down the application into small manageable pieces. This also allows teams to work on different parts of the system without interfering with each other. 

Developers are usually not good in finding bounded context for software applications. The main reason is that their technical knowledge does not directly map to domain knowledge. Domain knowledge requires different set of skills and a domain expert is a better suited for this kind of role. A domain expert is a person, who may not be tech savvy but understands how the business or a particular domain works. In large projects, you may have multiple domain experts, each handling a different business domain. This is why it is extremely important for developers to communicate with domain experts and understand the domain before starting any development.  

Once, you have identified different bounded contexts associated with your application you can represent them in the form of aggregate models. This is shown in the diagram below. 

![Multiple Aggregate Root](/images/aggregate-model-updated.002.jpeg)

The network layer can also be divided into multiple HTTP clients or you can use a single generic network layer for your entire application. This is shown in the following diagram. 

![Multiple Aggregate Root](/images/aggregate-model-updated.003.jpeg)

The Catalog aggregate model will be responsible for providing views with all the entities associated with Catalog. This can include but not limited to: 

- Product 
- Category 
- Brand 
- Review 

The Ordering aggregate model will be responsible for providing views with all the ordering related entities. This can include but not limited to:  

- Order 
- OrderLineItem 
- OrderStatus
- ShippingMethod 
- Discount 

The ```Catalog``` and ```Ordering``` aggregate models will be reference types conforming to ```ObservableObject``` protocol. And all the entities they provide will be value types. 

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

Catalog and Ordering aggregate models are injected into the application as an environment object. You can inject them directly in your application root view or the root view of each section of the application. The later is shown below: 

``` swift 
@main
struct StoreApp: App {
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(Catalog(client: CatalogHTTPClient()))
                .environmentObject(Ordering(client: OrderingHTTPClient()))
            
        }
    }
}
```

Now, inside a view you can use Catalog or Ordering models by accessing it through ```@EnvironmentObject```. The implementation is shown below: 

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

If your view needs to access ordering information then it can utilize the Ordering aggregate model too.

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

There are scenarios when your aggregate model will need to access information from another aggregate model. In those cases, your aggregate model will simply use the network service to fetch the information that is needs. 

> It is important that your caching layer is called from within the network layer and not from aggregate models. This will allow aggregate models to take advantage of caching through the network layer, instead of implementing it on their own. By accessing caching layer from inside the network layer, all your aggregate models can benefit from faster response through the use of cached resources. 

> As mentioned earlier for small or even medium sized apps, you may only need a single aggregate model. For larger apps you can introduce new aggregate models. Make sure to consult with a domain expert before creating application boundaries. 

The concept of domain boundaries can also be applied to user interfaces. This allows us to reuse user interface elements in other applications. 

![Factor out common pieces](/images/user-interface.png)
* Permission has been granted from the original author of the image to use it in this article.


> You can factor out common interface elements using Swift Package Manager and import those packages into other applications. 

Let's go ahead and zoom out and see how our architecture looks like with all the pieces in place. 

![Architecture](/images/architecture-model-updated.jpeg)
* This image has been updated and the permission has been granted from the original author of the image to use it in this article. 

As discussed earlier, each bounded context is represented by its own module. These modules can be represented by a folder or a package dependency.

**CatalogUI:** Represents user interface associated with catalog. This can include all the catalog specific stuff like AddCatalogScreen, UpdateCatalogScreen etc. 

**Catalog:** Represents the models associated with catalog. This will contain the aggregate model and all the entities exposed by the aggregate model.

**MyStoreKit**: Represents the HTTP client for performing network calls. 

**Foundation Core**: Represents resources used by all modules. This can include helper classes/structs, reusable views, images, icons and even preview content used for testing. 

> Each module like Shipping, Inventory, Ordering etc can be represented by a folder structure or a package dependency. This really depends on your needs and if you wish to reuse your modules in other projects. 

Using this architecture, future business requirements and data access services can be added without interfering with existing ones. This also allows more collaborative environment as different teams can work on different modules without interfering with each other. 

## View Specific Logic 

In this last section, I talked about how aggregate models can serve as a single source of truth and provide required data to the views. But what about view specific logic? Where should that logic be placed and what options do we have to perform testing on that logic. 

In the code below, we want to filter the products based on the minimum and maximum price. The implementation is shown below: 

``` swift 
struct ContentView: View {
    
    let httpClient: HTTPClientProtocol
    @State private var products: [Product] = []
    @State private var min: Double?
    @State private var max: Double?
    @State private var filteredProducts: [Product] = []
    
    private func filterProducts() {
        
        guard let min = min,
              let max = max else { return }
        
        filteredProducts = products.filter {
            $0.price >= min && $0.price <= max
        }
    }
    
    private var isFormValid: Bool {
        
        guard let min = min,
              let max = max else { return false }
        
        return min < max
    }
    
    var body: some View {
        VStack {
            HStack {
                TextField("Min", value: $min, format: .number)
                    .textFieldStyle(.roundedBorder)
                TextField("Max", value: $max, format: .number)
                    .textFieldStyle(.roundedBorder)
            }
            Text("Max must be larger than min.")
                .frame(maxWidth: .infinity, alignment: .leading)
                .font(.caption)
                .padding([.bottom], 20)
            
            Button("Apply") {
                filterProducts()
            }
            
            .disabled(!isFormValid)
          
            List(filteredProducts.isEmpty ? products: filteredProducts) { product in
                HStack {
                    Text(product.title)
                    Spacer()
                    Text(product.price, format: .currency(code: "USD"))
                }
            }
            .task {
                do {
                    products = try await httpClient.loadProducts()
                } catch {
                    print(error)
                }
        }
        }.padding()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView(httpClient: HTTPClientStub())
    }
}
```

> If filterProducts or similar functions will be involved in any model logic then you can also put it inside the aggregate root model, instead of the view.  

Please note that instead of invoking the real service, we are using a stubbed version of the HTTPClient that returns pre-configured response. Another good option would be to create separate JSON files for each response and read data from those files, when using Xcode previews. I covered that in one of my YouTube video, [Building SwiftUI Xcode Previews Using JSON File](https://youtu.be/EycwLxTU-EA). 

> Keep in mind that in the above scenario, if no results are found during filtering then the original products array is returned. 

We have two pieces of code in the view that constitute as logic, ```isFormValid``` and ```filterProducts```. If we want to test that code we have number of ways. 

Use Xcode previews! I know this does not sound fancy but I encourage you to use Xcode previews to test your view based logic. Xcode previews is extremely fast (depending on the machine you are using) and it gives you the same feeling as Red/Green/Refactor cycle. For this particular scenario, Xcode previews will be my first choice. 

> Xcode previews is not the answer to everything. If you are dealing with complicated view logic then it will be a good idea to move out all the logic into a separate struct and then write unit tests for that piece of code. Remember, one of the important aspects of why we test is to [gain confidence about our code](https://azamsharp.com/2023/02/15/testing-is-about-confidence.html). 

Another option is to extract the logic from the view and then write unit tests against it. This is shown in the implementation below: 

``` swift 
struct ProductFilterForm {
    
    var min: Double?
    var max: Double?
    
    func filterProducts(_ products: [Product]) -> [Product] {
        
        guard let min = min,
              let max = max else { return [] }
        
        return products.filter {
            $0.price >= min && $0.price <= max
        }
    }
}
```

```ProductFilterForm``` can now be unit tested in isolation. The unit test is shown below: 

``` swift 

func test_user_can_filter_products_by_price() throws {
        
        self.continueAfterFailure = false
      
        let products = [
            Product(id: 1, title: "Product 1", price: 10),
            Product(id: 2, title: "Product 2", price: 100),
            Product(id: 3, title: "Product 3", price: 200),
            Product(id: 4, title: "Product 4", price: 500)
        ]
        
        let expectedFilteredProducts = [
            Product(id: 2, title: "Product 2", price: 100),
            Product(id: 3, title: "Product 3", price: 200),
            Product(id: 4, title: "Product 4", price: 500)
        ]
        
        let productFilterForm = ProductFilterForm(min: 100, max: 500)
        let filteredProducts = productFilterForm.filterProducts(products)
        
        for expectedProduct in expectedFilteredProducts {
            
            let product = filteredProducts.first { $0.id == expectedProduct.id }
            
            XCTAssertNotNil(product)
            XCTAssertEqual(product!.title, expectedProduct.title)
            XCTAssertEqual(product!.price, expectedProduct.price)
        }
        
    }

```

> Unit testing view's logic in isolation as shown above can be beneficial for complicated user interfaces. Keep in mind that just because your unit test passes, does not mean that your user interface is working as expected. 

And the final kind of test you can write is an end-to-end test. E2E tests are great because they test the app from user's point of view and they are best against regression. The downside is that E2E tests are slower then running unit tests. The main reason they are slower is because they are testing the complete application instead of small units. Most of the issues in software exists because the application was tested at unit level and not at system level. I encourage you to spend some time writing meaningful E2E tests. 

Here is an implementation of an E2E test for the above scenario. 

``` swift 
  func test_user_can_filter_products_based_on_price() {
        
        let app = XCUIApplication()
        app.launchEnvironment = ["ENV": "TEST"]
        app.launch()
        
        app.textFields["minTextField"].tap()
        app.textFields["minTextField"].typeText("100")
        
        app.textFields["maxTextField"].tap()
        app.textFields["maxTextField"].typeText("500")
        
        app.buttons["applyButton"].tap()
        
        // assert that the count is correct
        XCTAssertEqual(3, app.collectionViews["productList"].cells.count)
        // assert that the items are correct
        XCTAssertEqual("Product 2", app.collectionViews["productList"].staticTexts["Product 2"].label)
        XCTAssertEqual("Product 3", app.collectionViews["productList"].staticTexts["Product 3"].label)
        XCTAssertEqual("Product 4", app.collectionViews["productList"].staticTexts["Product 4"].label)
    }
```

In the end you will have to decide where in [testing pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) you want to invest your time to get the best return on your investment.  

> If you want to learn more about testing then you can check out my course [Test Driven Development in iOS Using Swift](https://www.udemy.com/course/test-driven-development-in-ios-using-swift/?referralCode=07649C41E6E184CE86B3). 

## Screens vs Views 

When I was working with Flutter, I observed a common pattern for organizing the widgets. Flutter developers were separating the widgets based on whether the widgets represents an entire screen of just a reusable control. Since React, Flutter and SwiftUI are extremely similar in nature we can apply the same principles when building SwiftUI applications. 

For example when displaying details of a movie, instead of calling that view MovieDetailView, you can call it MovieDetailScreen. This will make it clear that the detail view is an actual screen and not some reusable child view. Here are few more examples. 

**Screens** 
- MovieDetailScreen
- HomeScreen 
- LoginScreen
- RegisterScreen 
- SettingsScreen 

**Views** 
- RatingsView 
- MessageView 
- ReminderListView
- ReminderCellView 

I find that it is always a good idea to keep a close eye on our friendly neighbors React and Flutter. You never know what ideas you will bring from other declarative frameworks into SwiftUI. 

## Validation 

There is a famous saying in software development, garbage in, garbage out. This means if you allow users to enter incorrect information (garbage) through the user interface then that garbage will eventually end up in your database. And usually when this happens, it becomes extremely difficult and time consuming to clean the database. 

You must take necessary steps to prevent users from submitting incorrect information in the first place. 

Consider a simple ```LoginScreen``` view with username and password TextFields. If we want to enable the login Button only when the view is validated correctly, we can use the implementation below: 

``` swift 
struct LoginScreen: View {
    
    @State private var username: String = ""
    @State private var password: String = ""
    
    private var isFormValid: Bool {
        !username.isEmptyOrWhiteSpace && !password.isEmptyOrWhiteSpace
    }
    
    var body: some View {
        Form {
            TextField("Username", text: $username)
            TextField("Password", text: $password)
            Button("Login") {
                
            }.disabled(!isFormValid)
        }
    }
}
```

For such trivial logic, you can use Xcode Previews to quickly perform manual testing and validate the outcome. 

If you are working on a more complicated form, then it is advised to extract it into its own struct. This concept is shown in the implementation below. 

``` swift 
struct LoginFormConfig {
    
    var username: String = ""
    var password: String = ""
    
    var isFormValid: Bool {
        !username.isEmptyOrWhiteSpace && !password.isEmptyOrWhiteSpace
    }
}

struct LoginScreen: View {
    
    @State private var loginFormConfig: LoginFormConfig = LoginFormConfig()
    
    var body: some View {
        Form {
            TextField("Username", text: $loginFormConfig.username)
            TextField("Password", text: $loginFormConfig.password)
            Button("Login") {
                
            }.disabled(!loginFormConfig.isFormValid)
        }
    }
}
```

```LoginFormConfig``` encapsulates the form validation. This also allows us to write unit tests against the LoginFormConfig. Few unit tests are shown below: 

``` swift 
final class LearnTests: XCTestCase {

    func test_login_form_validates_successfully() {
        
        let expectedOutputs: [[String: Any]] = [
            ["username": "johndoe", "password": "password", "isFormValid": true],
            ["username": "", "password": "password", "isFormValid": false],
            ["username": "johndoe", "password": " ", "isFormValid": false],
            ["username": "", "password": " ", "isFormValid": false],
            ["username": "   ", "password": "password", "isFormValid": false],
            ["username": " johndoe", "password": " password", "isFormValid": true]
        ]
        
        for expectedOutput in expectedOutputs {
            let username = expectedOutput["username"] as! String
            let password = expectedOutput["password"] as! String
            let isFormValid = expectedOutput["isFormValid"] as! Bool
            
            let loginFormConfig = LoginFormConfig(username: username, password: password)
            XCTAssertEqual(loginFormConfig.isFormValid, isFormValid)
        }
    }
}
```

In the end extracting the form validation into a separate struct and writing unit tests for it depends on your level of confidence. Simple forms can be tested easily through Xcode Previews and do not require additional structure or even unit tests.   

> Validation helper functions like isEmptyOrWhiteSpace, isNumeric, isEmail, isLessThan can be moved into a separate Swift package. This will promote reusability and other projects can also benefit from using it.

I covered few different ways of handling and displaying validation errors in one of my previous articles that you can read [here](https://azamsharp.com/2022/08/09/intro-to-mv-state-pattern.html). 

## Displaying Errors 

Displaying errors is an integral part of any application. 

In SwiftUI, we can centralize displaying errors to a single place. This will prevent us from writing repetitive code and also provide a single point in codebase to change the layout and appearance.  

We can start by creating an ErrorWrapper, which will be responsible for wrapping the actual error and also providing guidance to the user on the next steps. 

``` swift 
struct ErrorWrapper: Identifiable {
    let id = UUID()
    let error: Error
    let guidance: String
}
```

ErrorWrapper will be used by ErrorView. ErrorView will be responsible for displaying the details of the error in a visual format. You can find basic implementation of an ErrorView below.

``` swift 
struct ErrorView: View {
    
    let errorWrapper: ErrorWrapper
    
    var body: some View {
        VStack {
            Text("Error has occured in the application.")
                .font(.headline)
                .padding([.bottom], 10)
            Text(errorWrapper.error.localizedDescription)
            Text(errorWrapper.guidance)
                .font(.caption)
        }.padding()
    }
}
```

> ErrorView is simply a view and you can customize it as much as you want. 

In order to set the error wrapper from any part of our application, we will add an ErrorState as an ObservableObject and inject it in an EnvironmentObject. 

``` swift 
class ErrorState: ObservableObject {
    @Published var errorWrapper: ErrorWrapper?
}

@main
struct StoreApp: App {
    
    @StateObject private var errorState = ErrorState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(errorState)
                .sheet(item: $errorState.errorWrapper) { errorWrapper in
                    ErrorView(errorWrapper: errorWrapper)
                }
        }
    }
}

```

Whenever the errorState changes, a sheet will be displayed with the latest error. Once again, you are free to use a different method of displaying the error instead of a sheet. 

This technique allows you to have a single point in your codebase, which is responsible for displaying errors. 

## Grouping View Events 

One way to create reusable views in SwiftUI is to delegate the events to the parent view. This allows views to be used in different scenarios and without tying them to a particular logic. One way to accomplish this is to use closures. 

Consider a ```ReminderCellView```, which allows the user to perform check/uncheck and delete operations. The implementation is shown below: 

``` swift 
struct ReminderCellView: View {
    
    let index: Int
    let onChecked: (Int) -> Void
    let onDelete: (Int) -> Void

    var body: some View {
        HStack {
            Image(systemName: "square")
                .onTapGesture {
                    onChecked(index)
                }
            Text("ReminderCellView \(index)")
            Spacer()
            Image(systemName: "trash")
                .onTapGesture {
                    onDelete(index)
                }
        }
    }
}
```

```ReminderCellView``` exposes ```onChecked``` and ```onDelete``` closures. The caller can use these closures to perform a particular task. The calling side is shown below: 

``` swift 
struct ContentView: View {

    var body: some View {
        List(1...20, id: \.self) { index in
            ReminderCellView(index: index, onChecked: { index in
                // do something
            }, onDelete: { index in
                // do something
            })
        }
    }
}
```

As the complexity of ```ReminderCellView``` increases and it exposes more events then the calling side will become more complicated.

We can fix this issue by grouping all the events into a simple enum. This is shown below: 

``` swift 
enum ReminderCellEvents {
    case onChecked(Int)
    case onDelete(Int)
}
```

The ```ReminderCellView``` can be updated to use ```ReminderCellEvents```. This is shown below: 

``` swift 
struct ReminderCellView: View {
    
    let index: Int
    let onEvent: (ReminderCellEvents) -> Void
    
    var body: some View {
        HStack {
            Image(systemName: "square")
                .onTapGesture {
                    onEvent(.onChecked(index))
                }
            Text("ReminderCellView \(index)")
            Spacer()
            Image(systemName: "trash")
                .onTapGesture {
                    onEvent(.onDelete(index))
                }
        }
    }
}
```

Now, instead of dealing with multiple closures we are only handling a single enum based event structure. The calling site also looks much cleaner. 

``` swift 
struct ContentView: View {

    var body: some View {
        List(1...20, id: \.self) { index in
            ReminderCellView(index: index) { event in
                switch event {
                    case .onChecked(let index):
                        print(index)
                    case .onDelete(let index):
                        print(index)
                }
            }
        }
    }
}
```

In the end you will have to decide when you want to group events into an enum and when you want to use them individually (multiple closures). I tend to prefer using enum events if I have more than two closures exposed by the view. 

## Navigation 

SwiftUI introduced NavigationStack in iOS 16, which allowed developers to configure global routes for their application. 

There are several different ways to handle routing in SwiftUI. One possible approach is to handle all the routes in the root view of the application. We will start by creating the Routes enum, which will setup the routes for entire application. The implementation is shown below: 

``` swift 

enum Routes: Hashable {
    case catalog(CatalogRoutes)
    case inventory(InventoryRoutes)
    
    enum CatalogRoutes: Hashable {
        case home
        case detail(Category)
                
    }
    
    enum InventoryRoutes: Hashable {
        case home
        case detail(Product)
    }
}
```

The ```Routes``` enum represents the root routes for each section of the screen. This includes ```catalog```, ```inventory```, ```shipping``` etc. The routes for each section are declared inside their corresponding route enums (```CatalogRoutes```, ```InventoryRoutes```).  

For programmatic navigation, we added ```NavigationState```. NavigationState keep tracks of all the routes and will be injected into the app through the ```@EnvironmentObject```. 

``` swift 
class NavigationState: ObservableObject {
    @Published var routes: [Routes] = []
}
```

Finally, we handle all the routes in our root view of the application. This is shown below: 

```swift 
@main
struct LearnApp: App {
    
    @StateObject private var navigationState = NavigationState()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $navigationState.routes) {
                ContentView()
                    .navigationDestination(for: Routes.self) { route in
                        switch route {
                            case .catalog(let catalogRoutes):
                                switch catalogRoutes {
                                    case .home:
                                        Text("Home View")
                                    case .detail(let category):
                                        // show category detail view
                                        Text(category.name)
                                }
                            case .inventory(let inventoryRoutes):
                                switch inventoryRoutes {
                                    case .home:
                                        // show home view
                                        Text("Home View")
                                    case .detail(let product):
                                        // show product details View
                                        Text(product.name)
                                }
                        }
                    }
            }.environmentObject(navigationState)
        }
    }
}
```

The above approach works but, it can quickly get messy because we are putting all the routes of our application in a single place. One way to control this problem is to create separate routers for each section of the screen and allow routers to handle specific routing behavior. The implementation of ```CatalogRouter``` is shown below. CatalogRouter is responsible for handling all the routes related to the catalog screens.  

``` swift 
struct CatalogRouter {
    
    let routes: CatalogRoutes

    @ViewBuilder
    func configure() -> some View {
        switch routes {
            case .detail(let category):
                return Text(category.name)
        }
    }
}
```

Similarly, we can add router for Inventory. 

``` swift 
struct InventoryRouter {
    let routes: InventoryRoutes
    
    // view builder
    @ViewBuilder
    func configure() -> some View {
        switch routes {
            case .detail(let product):
                Text(product.name)
            case .home:
                Text("Home")
        }
    }
}
```

Now, our navigationDestination looks much cleaner. 

``` swift 
@main
struct LearnApp: App {
    
    @StateObject private var navigationState = NavigationState()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $navigationState.routes) {
                ContentView()
                    .navigationDestination(for: Routes.self) { route in
                        switch route {
                            case .catalog(let routes):
                                CatalogRouter(routes: routes).configure()
                            case .inventory(let routes):
                                InventoryRouter(routes: routes).configure()
                        }
                    }
            }.environmentObject(navigationState)
        }
    }
}
```

If you need to call a certain route, you can append the route in ```NavigationState```. This is shown in the implementation below: 

``` swift 
struct ContentView: View {
    
    @EnvironmentObject private var navigationState: NavigationState
    
    var body: some View {
       
        VStack {
            
            Button("Go to Catalog") {
                navigationState.routes.append(.catalog(.detail(Category(name: "Category 1"))))
            }
            
            Button("Go to Inventory") {
                navigationState.routes.append(.inventory(.detail(Product(name: "Product 1"))))
            }
           
        } .padding()
    }
}
```

> I am sure there are other ways of handling navigation in SwiftUI. Send me a [Gist](https://gist.github.com/) of your suggestion on [Twitter](https://twitter.com/azamsharp) and I will be more than happy to review it. 

> I also wrote a book on Navigation API in SwiftUI. If you are interested, you can download it free of charge from [here](https://azamsharp.com/books). 


## Testing 

> This section of the article is taken from my post [Pragmatic Testing and Avoiding Common Pitfalls](https://azamsharp.com/2012/12/23/pragmatic-unit-testing.html)

The main purpose of writing tests is to make sure that the software works as expected. Tests also gives you confidence that a change you make in one module is not going to break stuff in the same or other modules. 

Not all applications requires writing tests. If you are building a basic application with a straight forward domain then you can test the complete app using manual testing. Having said that in most professional environments, you are working with a complicated domain with business rules. These business rules form the basis on which company operates and generates revenue. 

In this article, I will discuss different techniques of writing tests and how a developer can write good tests to get the most return on their investment. 

## Not all tests are created equal 

Consider a scenario that you are writing an application for a bank. One of the business rules is to charge overdraft fees in case of insufficient funds. Banks generate [billions of dollars income by just fees](https://www.depositaccounts.com/blog/banks-income-fees.html) alone. As a developer, you must write good quality tests to make sure that overdraft fee calculation works as expected.

In the same bank app, you may have features like rendering templates for emails or logging certain interactions. These features are important but may not produce the same return on investment as compared to charging overdraft fees. This means if the email template is not in the correct format then the banks are not going to loose millions of dollars and you will not receive a call in the middle of the night. If the logging is meant for developers then in most cases you don't even need to write tests for it. It is just an implementation detail. 

>> If you are building a logging framework then it is essential that you thoroughly test the public API exposed by your framework.  

Next time you are writing a test, ask yourself how important this feature is for the business. If it is an integral part of the business then make sure to test it thoroughly and go for high code coverage.  

## Test behavior not implementation 

One of the biggest mistakes developers make is to focus on writing tests against the implementation details instead of the behavior of the application.

>> A trigger to add a new test is the requirement, not a class or a function. 

Just because you added a new class or a function does not mean that you will start writing tests. That is just an implementation detail which can change overtime. Your tests should target the business requirements and not the implementation details. 

Here are few examples of behaviors, derived from business requirements: 

1. When a customer withdraw amount and has insufficient funds then charge an overdraft fee.  
2. The number of stocks specified by customer are submitted for trade at a specified price, once the limit has reached. 

The behavior stems from the requirement of the project. Tests that checks the implementation details instead of the behavior tends to be very brittle and can easily break when the implementation changes even though the behavior remains the same.  

Let's consider a scenario, where you are building an application to display a list of products on the screen. The products are fetched from a JSON API and rendered using SwiftUI framework, following the principles of MVVM design pattern.

First we will look at a common way of testing the above scenario that is adopted by most developers and then later we will implement tests in a more **pragmatic** way. 

The complete app might look like the implementation below: 

```swift 
class Webservice {
    
    func fetchProducts() async throws -> [Product] {
        // ignore the hard-coded URL. We can inject the URL from using test configuration. 
        let url = URL(string: "https://test.store.com/api/v1/products")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Product].self, from: data)
    }
    
}

class ProductListViewModel: ObservableObject {
    
    @Published var products: [ProductViewModel] = []
    
    func populateProducts() async {
        do {
            let products = try await Webservice().fetchProducts()
            self.products = products.map(ProductViewModel.init)
        } catch {
            print(error)
        }
    }
    
}

struct ProductViewModel: Identifiable {
    
    private let product: Product
    
    init(product: Product) {
        self.product = product
    }
    
    var id: Int {
        product.id
    }
    
    var title: String {
        product.title
    }
}


struct ProductListScreen: View {
    
    @StateObject private var vm = ProductListViewModel()
    
    var body: some View {
        List(vm.products) { product in
            Text(product.title)
        }.task {
            await vm.populateProducts()
        }
    }
}
```

The above application works as expected and produces the expected result. Instead of testing the concrete implementation of the ```Webservice```, we will introduce an interface/contract/protocol just so that we can inject a mock. The sole purpose of creating the protocol is to satisfy the tests, even though there is only one concrete implementation that conforms to that protocol/interface.  

>> This is called [**Test Induced Damage**](https://dhh.dk/2014/test-induced-design-damage.html). The tests are dictating that we should add dependencies so you can mock out the service. The only purpose of introducing a protocol/contract/interface is so you can eventually mock it. Keep in mind there is nothing wrong with using protocols/contracts in your application. They do serve a very important purpose to hide the implementation details from the user and providing abstraction, but just to add contracts to satisfy testing goals in not a good practice as it complicates the implementation and your tests are directed away from testing the actual behavior of the app. 

In the code below we have introduced a WebserviceProtocol. Both Webservice and the newly created MockedWebservice conforms to the WebserviceProtocol as shown below: 

```swift 
protocol WebserviceProtocol {
    func fetchProducts() async throws -> [Product]
}

class Webservice: WebserviceProtocol {
    
    func fetchProducts() async throws -> [Product] {
        
        let url = URL(string: "https://test.store.com/api/v1/products")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Product].self, from: data)
    }
}

class MockedWebService: WebserviceProtocol {
    func fetchProducts() async throws -> [Product] {
        return [Product(id: 1, title: "Product 1"), Product(id: 2, title: "Product 2")]
    }
}
```

>> You should probably use a better name, instead of calling it WebserviceProtocol. The main reason, I am calling it WebserviceProtocol is just for the sake of simplicity and convenience.   

The webservice is now injected as a dependency to our ProductListViewModel. This is shown below: 

```swift 
class ProductListViewModel: ObservableObject {
    
    private let webservice: WebserviceProtocol
    @Published var products: [ProductViewModel] = []
    
    init(webservice: WebserviceProtocol) {
        self.webservice = webservice
    }
    
    func populateProducts() async {
        do {
            let products = try await Webservice().fetchProducts()
            self.products = products.map(ProductViewModel.init)
        } catch {
            print(error)
        }
    }
    
}
```

The view, ProductListScreen is also updated to reflect the change. 

```swift 
struct ProductListScreen: View {
    
    @StateObject private var vm = ProductListViewModel(webservice: WebserviceFactory.create())
    
    var body: some View {
        List(vm.products) { product in
            Text(product.title)
        }.task {
            await vm.populateProducts()
        }
    }
}
```

>> WebserviceFactory is responsible for either returning the Webservice or MockedWebservice, depending on the application environment. 

Now, let's go ahead and check out the test. 

```swift 
final class ProductsTests: XCTestCase {
    
    func test_populate_products() async throws {
        
        let mockedWebService = MockedWebService()
        let productListVM = ProductListViewModel(webservice: mockedWebService)
        
        await productListVM.populateProducts()
        
        // This line is verifying the implementation detail.
        // Implementation details can change
        // fetchProducts can change to getProducts and the test will fail. 
        verify(mockedWebService.fetchProducts()).wasCalled()
        
        XCTAssertEqual(2, productListVM.products.count)
    }
}
```

We created an instance of ```MockedWebservice``` inside our test and pass it to the ```ProductListViewModel```. Next, we invoke the ```populateProducts``` function on the view model and then check to make sure that the ```fetchProducts``` on the mockedWebservice instance was called. Finally, the test checks the products property of the ````ProductListViewModel``` instance to make sure that is is populated correctly. 
 
The problem with the above test is that it is not testing the behavior but the implementation. The following line of code is an implementation detail. 

```swift
verify(mockedWebService.fetchProducts()).wasCalled()
```

This means if you decide to refactor your code and rename the function ```fetchProducts``` to ```getProducts``` then your test will fail. These kind of tests are often known as brittle tests as they break when the internal implementation changes even though the functionality/behavior provided by the API remains the same. This is also the main reason that your test should validate the behavior instead of the implementation.  

> The code that you write is a liability, including tests. When writing tests, focus on the quality of the tests instead of the quantity. Remember, you are not only responsible for writing tests but also maintaining them. 

> If you are using MVVM pattern then your VM may have logic. It is perfectly fine to write unit tests against the logic contained in the view model.   

## End to End Testing 

In the previous section, you learned that and mocking in most scenarios does not provide the return on your investment. Tests written that use mocking usually end up being too brittle and can fail because of refactoring, breaking all the dependent tests even though the behavior remained the same. 

Human psychology also plays an important role when writing tests. As software developers we want fast feedback with small amounts of dopamine hit along the way. There is nothing wrong with receiving fast feedback. Fast feedback is one of the important characteristics of a unit test. Unfortunately, sometimes we are going too fast to realize that we were on the wrong path. We start behaving like a test addict, who wants to see green checkmarks alongside the tests instantly.  

As explained earlier adding tests that test the implementation details instead of behavior does not provide any benefit to your project. It may even work against you in the long run since now you will be responsible for maintaining those test cases and anytime the implementation detail changes, all your test will break even though the functionality remained the same. 

> I am not proposing that you should not write unit tests. Unit tests are great when you are testing small units of code. I am proposing that you must make sure that you are testing the behavior of the code and not implementation details. This means if you want to write unit tests for your view models, you can. 

Apart from unit tests and integration tests, end to end tests are best against regression. A good end to end will test one complete story/behavior. Below you can find the implementation of an end to end test. 

```swift 
final class ProductTests: XCTestCase {
    
    private var webservice: Webservice!
    // products
    let products = [Product(id: 1, title: "Handmade Fresh Table"),Product(id: 2, title: "Durable Water Bottle")]
    
    override func setUp() {
        // make sure the Webservice is using the TEST server endpoints and not PRODUCTION
        webservice = Webservice()
        
        // add few products // seeding the database
        for product in products {
            await webservice.addProduct(product: product)
        }
    }
    
    func test_display_list_of_all_products() async {
        
        let app = XCUIApplication()
        app.launch()
        
        let productList = app.tables["productList"]
        
        // check if the item numbers is correct
        XCTAssertEqual(productList.tables.cells.count, 2)
        
        // check if the correct items are displayed
        for(index, product) in products.enumerated() {
            let cell = productList.cells.element(boundBy: index)
            XCTAssertEqual(cell.staticTexts["productTitle"].label, product.title)
        }
        
    }
    
    override func tearDown() async throws {
        // make sure to delete ALL records from the database so future test results are not influenced
        await webservice.deleteProductById(productId: 1)
        await webservice.deleteProductById(productId: 2)
    }
    
}
```
  
>> Developers can run E2E tests locally on their development machine. This will require initial setup such as testing framework, test environment, dependencies (database, services). E2E tests can be time-consuming, as a result developers may choose to run E2E tests less frequently than unit tests or other types of tests.  

E2E tests are slower than the previous tests discussed earlier in the section but the main reason they are slower is because they tests all layers of the application. E2E tests are complete test and targets a particular behavior of the app. 

End to end tests also requires some initial setup that will allow your test to run database migrations, insert seed data, simulate user interface events and then rolling back changes after the tests are completed. 

End to end tests are NOT replacement of your domain model tests. You MUST write tests against your domain models, specially if your app is domain heavy and consists of lots of business rules. 

>> You will have to find the right balance as to how often to run end to end tests. If you run it with each code check-in then your continuous integration server will always be running 100% of the time. If you run it once every couple of days then you will be notified of failures much later than expected. Keep in mind that you can run E2E tests locally on your machine. Once you get the desired outcome, the CI server can run all the tests during the code check-in process.   

## What about Integration Testing 

Integration tests are performed to make sure that two different systems can work together. These systems can be external dependencies like database or API but it can also be different modules within the same system. 

Dependencies can be classified as managed and unmanaged dependencies. A managed dependency includes database, file systems etc. For managed dependencies, it is important that you use real instance and not a mock. Unmanaged dependencies include SMTP server, payment gateway etc. For unmanaged dependencies use mocks to verify their behavior. 

Let's check out a sample integration test for a network service for user login operation. 

```swift 
// This test is generated by ChatGPT AI 
import XCTest

class IntegrationTests: XCTestCase {
    func testLogin() {
        // Set up the URL for the login endpoint
        let url = URL(string: "https://api.example.com/login")!

        // Create a URL request
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")

        // Set the body of the request to a JSON object with the login credentials
        let body = ["username": "user123", "password": "password"]
        request.httpBody = try! JSONSerialization.data(withJSONObject: body)

        // Create a URLSession and send the request
        let session = URLSession.shared
        let task = session.dataTask(with: request) { data, response, error in
            // Make sure there is no error
            XCTAssertNil(error)

            // Check the response status code
            let httpResponse = response as! HTTPURLResponse
            XCTAssertEqual(httpResponse.statusCode, 200)

            // Check the response data
            XCTAssertNotNil(data)
            let responseBody = try! JSONSerialization.jsonObject(with: data!, options: []) as! [String: Any]
            XCTAssertEqual(responseBody["status"], "success")
        }
        task.resume()
    }
}

```

The above integration test makes sure that the HTTP client layer is working as expected. The integration is between the network client and the server. The client is making sure that the response is correct and valid for a successful login operation.  

Unmanaged dependencies like payment gateway, SMTP clients etc can be mocked out during integration tests. For managed dependencies, use the concrete implementations. 

## Code Coverage 

Code coverage is a metric that calculates how much of your code is covered under test. Let's take a very simple example. In the code below we have a ```BankAccount``` class, which consists of ```deposit``` and ```withdraw``` functions. 

>> Keep in mind that in real world scenario, a bank account is not implemented as a calculator. A bank account is recorded in a ledger, where all financial transactions are persisted. 

```swift 
class BankAccount {
    
    private(set) var balance: Double
    
    init(balance: Double) {
        self.balance = balance
    }
    
    func deposit(_ amount: Double) {
        self.balance += amount
    }
    
    func withdraw(_ amount: Double) {
        self.balance -= amount
    }
    
}
```

One possible test for the BankAccount may check if the account is successfully deposited. 

```swift 
final class BankAccountTests: XCTestCase {
    
    func test_deposit_amount() {
        
        let bankAccount = BankAccount(balance: 0)
        bankAccount.deposit(100)
        XCTAssertEqual(100, bankAccount.balance)
        
    }
}
```

If this is the only test we have in our test suite then our code coverage is not 100%. This means not all paths/functions are under test. This is true because we never implemented the test for ```withraw``` function. 

You may be wondering that should you always have 100% code coverage. The simple answer is NO. But it also depends on the apps that you are working on. If you are writing code for NASA, where it will be responsible for landing rover on Mars then you better make sure that every single line is tested and your code coverage is 100%. 

If you are implementing an app for a pace maker device that helps to regulate the heartbeat then you better make sure that your code coverage is 100%. One line of missed and untested code can result in someones life... literally. 

So, what is the ideal code coverage number. It really depends on the app but any number above 70% is considered a decent code coverage. 

>> When calculating code coverage make sure to ignore the third party libraries/frameworks as their code coverage is not your responsibility. 

## Unit Testing, Data Access and File Access 

Most developers that I have talked to believe that a unit test cannot access a database or a file system. **That is incorrect and plain wrong**. A unit test CAN access a database or a file system. 

It is very important to understand that ```Unit test is the isolation, not the thing under test```. This is so important that I am going to repeat it again. 

>> Unit test is the isolation, not the thing under test 

One of the valid reasons of not accessing a database or a file system during unit tests is that a test may leave data behind which may cause other tests to behave in unexpected manner. The solution is to make sure that the database is always reverted to an initial state after each test is completed so that future tests gets a clean database without any side effects.  

Some frameworks also allows you to construct in-memory databases. Core Data for instance uses SQLite by default but it can be configured to use an in-memory database as shown below: 

```swift 
storeDescription.type = NSInMemoryStoreType
```

In-memory database provides several benefits including: 

- No removal of test data 
- Run faster 
- Can be initialized before each test run 

Even thought these benefits looks appealing, I personally do not recommend using in-memory database for testing purposes. The main reason is that in-memory databases does not represent an actual production environment. This means you may not encounter the same issues during tests, which you may witness when using an actual database. 

>> It is always a good idea to to make sure that your test environment and production environment are nearly identical in nature. 

## Testing View Model Does NOT Validate the User Interface  

Couple of weeks ago, I was having a discussion with another developer, who was mentioning that they test their **user interface** through View Models in SwiftUI. I was not sure what he meant so I checked the source code and found that they had lot of unit tests for their View Models and they were just assuming that if the View Model tests are passing then the user interface will automatically work.

> Please keep in mind that I am not suggesting that you should not write unit tests for your View Models. I am simply saying that your View Model unit tests does not validate that the user interface is working as expected. 

Let's take a very simple example of building a counter application. 

``` swift 
class CounterViewModel: ObservableObject {
    
    @Published var count: Int = 0
    
    func increment() {
        count += 1
    }
}

struct ContentView: View {
    
    @StateObject private var counterVM = CounterViewModel()
    
    var body: some View {
        VStack {
            Text("\(counterVM.count)")
            Button("Increment") {
                counterVM.increment()
            }
        }
    }
}
```

When the increment button is pressed, we call the increment function on the CounterViewModel instance and increment the count. Since count property is decorated with @Published property wrapper, it notifies the view to reevaluate and eventually rerender. 

In order to test that the count is incremented and displayed on the screen, the following unit test was written. 

``` swift 
import XCTest
@testable import Learn

final class LearnTests: XCTestCase {

    func test_user_updated_count() {
        let vm = CounterViewModel()
        vm.increment()
        XCTAssertEqual(1, vm.count)
    }

}
```

This is a perfectly **valid** unit test but it does not verify that the count has been updated and displayed on the screen. Let me repeat it again. **A View Model unit test does not verify that the count is successfully displayed on the screen. This is a unit test not a UI test.**

To prove that a View Model unit test does not verify user interface elements, simply remove the Button view or even the Text view from the ContentView. The unit test will still pass. This can give you false confidence that your interface is working. 

A better way to verify that a user interface is working as expected is to implement a UI test. Take a look at the following implementation. 

``` swift 
final class LearnUITests: XCTestCase {

    func testExample() throws {
        // UI tests must launch the application that they test.
        let app = XCUIApplication()
        app.launch()

        app.buttons["incrementButton"].tap()
        XCTAssertEqual("1", app.staticTexts["countLabel"].label)
    }
}
```

This test will launch the app in a simulator and verify that when the button is pressed, label is updated correctly. 

> Depending on the complexity of the behavior you are testing, you may not even need to write a user interface test. I have found that most of the user interfaces can be tested quickly using Xcode Previews. 

So what is the right balance? How many unit tests should you have for your View Model as compared to UI tests. 

The answer is **it depends**. If you have complicated logic in your View Model then unit test can help. UITest (E2E) tests provide the best defense against regression. For each story, you can write couple of long happy path user interface tests and couple of edge cases. Once again, this really depends on the story and the complexities associated with the story. 

In the end [testing is all about **confidence**](https://azamsharp.com/2023/02/15/testing-is-about-confidence.html). Sometimes you can gain confidence by writing fewer or no tests and other times you have to write more tests to achieve the level of confidence. 

## The Ideal test 

We talked about several different types of tests. You may be wondering what is the best kind of test to write. What is the ideal test? 

Unfortunately, there is no ideal test. It all depends on your project and requirements. If your project is domain heavy then you should have more domain level tests. If your project is UI heavy then your should have end to end tests. Finally, if your project integrates with managed and unmanaged dependencies then integration tests will be more suitable in those scenarios.

Remember to test the public API exposed by the module and not the implementation details. This way you can write useful quality tests, which will also help you to catch errors.  

Don't create protocols/interfaces/contracts with the sole purpose of mocking. If a protocol consists of a single concrete implementation then use the concrete implementation and remove the interface/contract. Your architecture should be based on current business needs and not on what if scenarios that may never happen. Remember YAGNI (You aren't going to need it). Less code is better than more code. 

## Conclusion 

Application architecture is a complicated subject and in the end the best architecture for a project depends on many factors. These factors can include the size and complexity of the project, the team's skills and experience, the project's goals and requirements. 

Ultimately, the key to a successful application architecture is to choose a pattern that fits the project's unique needs, and to constantly evaluate and adjust the architecture as the project evolves.

By investing time and resources into designing a thoughtful and effective application architecture, teams can ensure that their codebase is maintainable, scalable, and flexible enough to adapt to changing requirements and technology trends.