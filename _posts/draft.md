
# Understanding Communication Patterns Between Observable Stores in SwiftUI 

// TODO 

## What is a Store? 

A **Store** is a reference type that either conforms to the `ObservableObject` protocol or uses the `@Observable` macro. Unlike view models, you don’t create a separate store for each screen. Instead, stores are organized around the **bounded contexts** of your application. In other words, a store represents a logical slice of your app’s data and business rules, not just a single view.

For example, a `ProductStore` could power multiple views such as `ProductListScreen`, `AddProductScreen`, and `ProductDetailScreen`. The same store instance ensures consistency across these screens.

Stores primarily manage **business logic**, not presentation logic. Any presentation-specific behavior can live inside SwiftUI views themselves or, if complex, be extracted into dedicated value types (structs).

Stores can also have **dependencies**, such as a networking layer. They use these dependencies to fetch or persist data, while holding only the minimal state required by the view.

> For instance, even if the server holds 100,000 records, your store might fetch only 50 at a time using pagination and expose that slice to the view.

Over the years, I’ve come across many different implementations of what we now call *stores*. Depending on the team or the codebase, they’ve been labeled as *services*, *controllers*, or even *view models*.

## A Single Store

Depending on your app and its requirements, you may start with a **single store** that manages the entire application state. This approach works well for small to medium-sized apps. Personally, I prefer beginning with a single store and introducing additional stores only when the complexity of the app demands it.

Here’s a snippet from my `PlatziStore` implementation:


``` swift 
@MainActor
@Observable
class PlatziStore {
    
    let httpClient: HTTPClient
    var categories: [Category] = []
    var locations: [Location] = []
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func loadCategories() async throws {
        let resource = Resource(url: Constants.Urls.categories, modelType: [Category].self)
        categories = try await httpClient.load(resource)
    }
    
    func createCategory(name: String) async throws {
       // code ...
    }
    
    func fetchProductsBy(_ categoryId: Int) async throws -> [Product] {
       // code ...
    }
    
    func createProduct(title: String, price: Double, description: String, categoryId: Int, images: [URL]) async throws -> Product {
        // code ...
    }
    
    func deleteProduct(_ productId: Int) async throws -> Bool {
       // code ...
    }
    
    func loadLocations() async throws {
       // code ...
    }
    
}
```

Stores can be injected either at the root of your application or at any specific node in the SwiftUI view hierarchy. Once injected, they become accessible through the `@Environment` property wrapper.

A good practice is to limit direct store access to **container views**. Child views should instead receive only the data they need via their initializers. This keeps child views lightweight, reusable, and free from unnecessary dependencies. 
This also improves SwiftUI’s performance, since views are only re-evaluated and re-rendered when their dependencies actually change.

The following implementation demonstrates this approach:

``` swift 

    @Environment(PlatziStore.self) private var store 

    var body: some View {
        ProductListView(products: store.products)
        LocationsView(locations: store.locations)
    }

```


A single store may be sufficient for small to medium-sized applications, but as your app grows larger, it becomes important to split responsibilities across multiple stores.

## Multiple Stores 

I prefer to design and divide stores based on the bounded context of the application. A bounded context defines a clear boundary around a specific domain or responsibility, ensuring each store manages only the data and logic relevant to that area. For example, in an e-commerce app:

* **`InsuranceStore`** manages insurance rates, insurance plans etc. 
* **`CatalogStore`** handles categories, filters, and sorting options.
* **`InventoryStore`** tracks stock levels and availability.
* **`UserStore`** manages user profiles, and order history.
* **`FulfillmentStore`** oversees shipping, delivery, and returns.

By aligning stores with bounded contexts rather than screens, the codebase remains modular, scalable, and easier to reason about.

A store does not operate in isolation—it often needs to communicate with other stores to retrieve or share information. This becomes especially important when an action in one store triggers a sequence of events that other stores must handle. In the next section, we’ll explore different strategies for propagating and handling domain events across stores.

## Handling Store Events in the View 

Consider a scenario in a healthcare or insurance app: whenever a user adds a new dependent, the insurance rates must be recalculated. In this setup, a `UserStore` is responsible for managing user information (including dependents), while an `InsuranceStore` handles insurance options, premiums, and rates for each user.

The question is: **how should the insurance rates be updated when a new dependent is added?**

One straightforward approach is to give the view access to both `UserStore` and `InsuranceStore`. After successfully adding a dependent through `UserStore`, the view can directly trigger a recalculation of insurance rates by calling the appropriate function on `InsuranceStore`.

The following implementation demonstrates this approach:

``` swift 
struct ContentView: View {
    
    @Environment(UserStore.self) private var userStore
    @Environment(InsuranceStore.self) private var insuranceStore
    @State private var insuranceRate: InsuranceRate?
    
    let userId = UUID()
    
    private func addDependent() async {
        let dependent = Dependent(id: UUID(), name: "Nancy", dob: Date())
        do {
            // add dependent to user management store
            try await userStore.addDependent(dependent, to: userId)
            insuranceRate = try await insuranceStore.calculateInsuranceRates(userId: userId)
        } catch {
            print(error.localizedDescription)
        }
    }
}
```

One major drawback of this approach is that any additional side effects must also be triggered directly from the view. For example, just as we manually called `insuranceStore.calculateInsuranceRates` after adding a dependent, we would need to add more calls in the view as new side effects are introduced. Over time, this makes the view more complex, tightly coupled to business logic, and harder to test.

## Handling Store Events Through Delegate 

Although I don’t typically use the delegate pattern in SwiftUI, it can still be applied if you want to forward an event to a single store. The first step is to define a delegate protocol, as shown in the implementation below:

``` swift 
@MainActor
protocol UserStoreDelegate: AnyObject {
    func dependentAdded(dependent: Dependent, userId: UUID) async
}
```

The `UserStore` is then updated to include a `delegate` property. At composition time, this property can be assigned to another store, which will act as the delegate and handle events emitted by `UserStore`.


``` swift 
@MainActor
@Observable
class UserStore {
    
    let httpClient: HTTPClient
    weak var delegate: (any UserStoreDelegate)?
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    // add a dependent to the user
    func addDependent(_ dependent: Dependent, to userId: UUID) async throws {
        
        guard var user = try await getUser(by: userId) else {
            throw UserError.userNotFound
        }
        
        user.dependents.append(dependent)
        await delegate?.dependentAdded(dependent: dependent, userId: userId)
    }
    
}
```

The `Insurance` becomes conforms to the delegate and implements the `dependentAdded` function. 

``` swift 
@MainActor
@Observable
class InsuranceStore: UserStoreDelegate {
    
    let httpClient: HTTPClient
    var insuranceRate: InsuranceRate?
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func dependentAdded(dependent: Dependent, userId: UUID) {
        // calculate rates
        calculateInsuranceRates(dependent: dependent, userId: userId)
        print("dependentAdded InsuranceStore")
    }
    
    private func calculateInsuranceRates(dependent: Dependent, userId: UUID) {
        insuranceRate = InsuranceRate(
            monthlyPremium: Decimal(Double.random(in: 300...600)),
            deductible: Decimal(Double.random(in: 500...2000)),
            coverageAmount: Decimal(Double.random(in: 10_000...50_000))
        )
    }
}
```

Make sure to hook up the delegate in the App file. This is shown below: 

``` swift 
@main
struct LearnApp: App {
    
    @State private var userStore: UserStore
    @State private var insuranceStore: InsuranceStore
    
    init() {
        let http = HTTPClient()
        let userStore = UserStore(httpClient: http)
        let insurance = InsuranceStore(httpClient: http)
        userStore.delegate = insurance   // wire them here
        _userStore = State(initialValue: userStore)
        _insuranceStore = State(initialValue: insurance)
    }
```


Now, whenever the `UserStore` adds a new dependent, the delegate’s method—`InsuranceStore.dependentAdded`—is invoked automatically.

However, a key limitation of this approach is that the delegate pattern only supports a **single listener**. In other words, if another store—such as `DocumentStore`—also needs to react when a new dependent is added, this setup won’t work.

In the following sections, we’ll explore alternative techniques that make it possible to notify **multiple listeners** when such events occur.



