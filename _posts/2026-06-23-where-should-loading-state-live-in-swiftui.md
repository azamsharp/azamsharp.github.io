# Where Should Loading State Live in SwiftUI? 

Handling loading states is something every SwiftUI application has to deal with.

You fetch data from an API. While the request is running, you show a progress indicator. When the request succeeds, you display the data. If something goes wrong, you show an error message.

That part is usually straightforward.

The more interesting question is where the loading state should live. Should it be managed by the view? Should it be part of your store? Or should it be combined with the data itself using a generic loading state enum?

There are many ways to solve this problem, and each approach comes with tradeoffs.

In this article, I will walk through a simple loading state implementation and discuss why I prefer keeping loading state separate from the data maintained by my store.

<div class="azam-books-callout">
    <h3>Ready to Dive Deeper?</h3>
    <p>
        Interested in learning more about SwiftUI Architecture or SwiftData?
    </p>
    <p>
        I have written comprehensive books covering both topics, including many of the
        patterns and techniques I use in production applications.
    </p>
    <a
        href="https://azamsharp.school/books.html"
        class="azam-books-callout__button"
        target="_blank"
        rel="noopener"
    >
        Explore My Books
    </a>
</div>

<style>
.azam-books-callout {
    margin: 2rem 0;
    padding: 1.75rem;
    border-radius: 14px;
    border: 1px solid #dbe4f0;
    background: linear-gradient(
        135deg,
        #f8fbff 0%,
        #eef5ff 100%
    );
    text-align: center;
}

.azam-books-callout h3 {
    margin-top: 0;
    margin-bottom: 0.75rem;
    font-size: 1.4rem;
    color: #1f2937;
}

.azam-books-callout p {
    margin: 0.75rem 0;
    color: #4b5563;
    line-height: 1.6;
}

.azam-books-callout__button {
    display: inline-block;
    margin-top: 1rem;
    padding: 0.75rem 1.25rem;
    border-radius: 10px;
    background: #2563eb;
    color: white !important;
    text-decoration: none;
    font-weight: 600;
    transition: background 0.2s ease;
}
.azam-books-callout__button:hover {
    background: #1d4ed8;
}
</style>

### Implementation 

Let's start with ProductStore.

The ProductStore is responsible for maintaining the source of truth for products on the client side. In a real application, this store would probably use an HTTPClient to fetch products from an API.

But for this example, I want to keep the focus on loading states. So instead of implementing a full networking layer, we will add a small delay inside ProductStore to simulate a network request.

The implementation is shown below:

``` swift 
@Observable
class ProductStore {
    private(set) var products: [Product] = []
    
    func loadProducts() async throws {
        
        // use HTTPClient layer but I am going to skip that...
        try await Task.sleep(for: .seconds(5.0))
        
        products = [
            Product(name: "Wireless Keyboard", description: "Compact keyboard with backlit keys."),
            Product(name: "Desk Lamp", description: "Adjustable LED lamp for focused work."),
            Product(name: "Travel Mug", description: "Insulated mug that keeps drinks hot or cold."),
            Product(name: "Notebook", description: "Hardcover notebook with dotted pages."),
            Product(name: "USB-C Hub", description: "Multiport adapter with HDMI, USB, and SD card support.")
        ]
        
    }
    
    func addProduct(_ product: Product) async throws {
        try await Task.sleep(for: .seconds(2.0))
        products.append(product)
    }
}
```

The view needs access to ProductStore.

There are a few ways to pass the store to the view. You can pass it through the initializer, or you can place it in the SwiftUI environment.

For this example, I am going to use the environment approach. This keeps the code cleaner and allows child views to access the store without manually passing it from screen to screen.

Below you can see the code:

``` swift 
@main
struct LoadingDemoApp: App {
    
    @State private var productStore = ProductStore()
    
    var body: some Scene {
        WindowGroup {
            ProductListScreen()
                .environment(productStore)
        }
    }
}
```

Now, we can focus our attention the loading phase/state. I have defined an enum to manage different loading phases. 

``` swift 
private enum LoadingPhase {
    case loading
    case success
    case failure(Error)
}
```

One thing you will notice is that the success case does not carry the loaded products.

A common approach is to define the enum like this:

```case success(T)```

There is nothing wrong with that approach. In many cases, it works really well. But for this example, I am intentionally keeping the products inside ProductStore.

The reason is simple. I want ProductStore to remain the single source of truth for products on the client side. The loading phase should only describe the current state of the request. I will discuss more in detail of issues I faced when using `success(T)` approach. 

Let's take a look at ProductListScreen, which uses our ProductStore.

``` swift 
 private func loadProducts() async {
        
        if productStore.products.isEmpty {
            loadingPhase = .loading
        }
        
        do {
            try await productStore.loadProducts()
            loadingPhase = .success
        } catch is CancellationError {
            // don't show this error on the screen
        }
        catch {
            print(error.localizedDescription)
            loadingPhase = .failure(error)
        }
    }

Group {
            switch loadingPhase {
            case .loading:
                ProgressView("Loading products...")
                    .task {
                        await loadProducts()
                    }
            case .success:
                if !productStore.products.isEmpty {
                    ProductListView(products: productStore.products)
                        .refreshable {
                            await loadProducts()
                        }
                } else {
                    
                    ContentUnavailableView {
                        Label("No Products", systemImage: "shippingbox")
                    } description: {
                        Text("Products will appear here after they are loaded.")
                    } actions: {
                        Button("Reload") {
                            Task {
                                await loadProducts()
                            }
                        }
                    }
                }
                
            case .failure(let error):
                Text(error.localizedDescription)
            }
        }
```

Inside the body, we switch over the current loadingPhase.

When the phase is .loading, we display a ProgressView and use the task modifier to start loading products.

Once the products are loaded successfully, loadProducts updates the phase to .success. This causes the view to re-evaluate, and the success case displays the products using ProductListView.

If something goes wrong, we update the phase to .failure and show the error message to the user.

The important detail is inside the loadProducts function.

We only set the phase to .loading when there are no products already available. This works well for the initial load because the screen should show a progress indicator.

But when the user pulls to refresh, we do not want the list to disappear and be replaced by a loading screen. The existing products should remain visible while the refresh operation is running.

In the next section, let's cover what hurdles I faced when implementing the `success(T)` approach. 

### Why Not success(T)? 

The main problem I have faced with passing products into the success case is that it can create another source of truth.

The ProductStore already knows about the products. If the same products are also stored inside LoadingPhase.success, then now we have product state in two places.

That does not mean this approach is wrong. It can work fine in many applications. But in this example, it made the code more complicated than it needed to be.

Let's look at a few scenarios.

First, we update LoadingPhase to the following:

``` swift 
private enum LoadingPhase<T> {
    case loading
    case success(T)
    case failure(Error)
}
```

This means ProductListScreen now needs to change to accomodate the generic type. 

``` swift 
struct ProductListScreen: View {
    
    @Environment(ProductStore.self) private var productStore
    @State private var loadingPhase: LoadingPhase<[Product]> = .loading
}
```

Inside loadProducts, we now need to pass the products to the .success case.

This creates a small problem.

The products are already maintained by ProductStore, but the loadProducts function on the store does not return the products. It only updates the store.

So after calling loadProducts, what should we pass to .success?

``` swift 
do {
            try await productStore.loadProducts()
            loadingPhase = .success // ? 
        } catch is CancellationError {
            // don't show this error on the screen
        }
```

We can fix this by passing the products from the store into the .success case.

``` swift 
loadingPhase = .success(productStore.products)
```

But I don't like this approach, as we are making a separate source of truth in the form of arguments passed to the .success case. 

**But what if loadingPhase lives inside ProductStore?**

That is also an option.

Instead of keeping loadingPhase as a @State property in the view, we can move it into the store. This allows the store to manage both the products and the loading state.

The implementation is shown below:

``` swift 
@Observable
class ProductStore {
    
    private(set) var products: [Product] = []
    var loadingState: LoadingPhase<[Product]> = .idle
    
    func loadProducts() async throws {
        // use HTTPClient layer but I am going to skip that...
        
        loadingState = .loading
        
        try await Task.sleep(for: .seconds(5.0))
        
        products = [
            Product(name: "Wireless Keyboard", description: "Compact keyboard with backlit keys."),
            Product(name: "Desk Lamp", description: "Adjustable LED lamp for focused work."),
            Product(name: "Travel Mug", description: "Insulated mug that keeps drinks hot or cold."),
            Product(name: "Notebook", description: "Hardcover notebook with dotted pages."),
            Product(name: "USB-C Hub", description: "Multiport adapter with HDMI, USB, and SD card support.")
        ]
        
        loadingState = .success(products)
    }
}
```

Once you update ProductListScreen to use productStore.loadingPhase instead of the local loadingPhase property, everything works as expected. The view can switch over the different loading states and render the appropriate UI.

The problem I have with this approach is that we are now maintaining product state in multiple places.

The products property already contains the current list of products. Since ProductStore is observable, any changes to the products array will automatically trigger view updates. In other words, products is already acting as the source of truth for the data.

But now we also have a loadingPhase property whose success case contains the same products. This means product state is being maintained both in the products property and inside loadingPhase. For simple examples this may not seem like a big deal, but as the application grows it can make the code harder to reason about.

One option is to remove the products property altogether and use loadingPhase as the only source of truth. The implementation is shown below:

``` swift 
func loadProducts() async throws {
        // use HTTPClient layer but I am going to skip that...
        
        loadingState = .loading
        
        try await Task.sleep(for: .seconds(5.0))
        
        let products = [
            Product(name: "Wireless Keyboard", description: "Compact keyboard with backlit keys."),
            Product(name: "Desk Lamp", description: "Adjustable LED lamp for focused work."),
            Product(name: "Travel Mug", description: "Insulated mug that keeps drinks hot or cold."),
            Product(name: "Notebook", description: "Hardcover notebook with dotted pages."),
            Product(name: "USB-C Hub", description: "Multiport adapter with HDMI, USB, and SD card support.")
        ]
        
        loadingState = .success(products)
    }
```

This will work for loadProducts but what about addProduct. Our current implementation of addProduct is shown below: 

``` swift 
func addProduct(_ product: Product) async throws {
        try await Task.sleep(for: .seconds(2.0))
        products.append(product)
}
```

But since now we are focusing on using loadingPhase instead of products, we have to implement a new solution. This is shown below: 

``` swift 
func addProduct(_ product: Product) async throws {
        try await Task.sleep(for: .seconds(2.0))
        
        if case .success(var products) = loadingState {
            products.append(product)
            loadingState = .success(products)
        }
    }
```

This works, but it feels more complicated than it needs to be. Instead of simply appending a product to an array, we now have to inspect the current loading state, extract the products, mutate the array, and then assign a new success state.

The bigger issue is that this complexity does not stay inside the store. It starts to affect the user interface code as well. The view now has to understand more about the shape of the loading state and how the products are stored inside it.

To be clear, I am not saying that keeping loadingPhase inside ProductStore is wrong. In some applications, that might be a perfectly reasonable approach. But for the example discussed in this article, it introduces more complexity than our original solution.

For this particular scenario, I prefer keeping the products in ProductStore and keeping the loading phase in the view. The store remains responsible for the data, and the view remains responsible for deciding which UI should be displayed.

In the end, it depends on the application you are building. The important thing is to avoid duplicating state unless there is a good reason for it. If you have found a better way to manage loading states in SwiftUI, I would love to hear about it.
### Source Code 

You can download the complete source code from this [Gist](https://gist.githubusercontent.com/azamsharpschool/2327e98b2e549344a2ee03580c8a9bd3/raw/231a502b8664a0789e5c3295642f2427d3a01e53/ProductListScreen.swift). 

### Conclusion 

Managing loading states in SwiftUI is not difficult, but the implementation details can have a big impact on the overall complexity of your application.

In this article, we explored a simple approach where the view is responsible for managing the loading phase while the store remains focused on maintaining application data. This keeps a single source of truth for products and avoids duplicating state in multiple places.

Could you move the loading state into the store? Absolutely. In some applications that might even be the right choice. But as we saw, once you start adding operations like creating, updating, and deleting records, the implementation can become more complicated than expected.

There is no perfect solution that works for every application. The goal is to choose an approach that keeps your code easy to understand and easy to maintain. For this particular scenario, keeping the loading phase in the view and the data in the store provides a clean separation of responsibilities without introducing unnecessary complexity.