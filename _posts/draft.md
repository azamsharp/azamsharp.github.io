
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

The following implementation demonstrates this approach:

``` swift 

    @Environment(PlatziStore.self) private var store 

    var body: some View {
        ProductListView(products: store.products)
        LocationsView(locations: store.locations)
    }

```

## Multiple Stores 

