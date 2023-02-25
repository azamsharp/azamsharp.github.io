# Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture

In one of my previous articles, I mentioned that how I am using MV Pattern for building my client/server SwiftUI applications. 

## Understanding MV Pattern 

The main idea behind the MV Pattern is to allow views directly talk to the model. This eliminate the need for creating unnecessary layer of view models for each view, which simply contribute to the size of the project instead of providing any benefits.  

> In SwiftUI, view is similar to Component in React or Widget in Flutter. This means, it is not only used for displaying data but can also handle binding capabilities. **The View in SwiftUI is also a View Model.**. If you plan to put logic in your views then check out Container Pattern.    

Apple has shown MV pattern in several places with different flavors. This includes Fruta, FoodTruck and ScrumDinger applications. My focus for this article is on client/server applications as they are one of the most common types of iOS applications. 

In WWDC 2020 talk titled "Data Essentials in SwiftUI" Apple presented the following diagram. 

![Aggregate Root](/images/aggre-root.png)

In a client/server application, the view needs to display data fetched from the server. Data can be in many different formats but mostly JSON is preferred. How can we provide data to the view with the least amount of effort yet still maintain separation of concerns? 

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


In the code below, we used a StoreHTTPClient which is responsible for fetching the data from the API. The data is populated in the local state of ContentView and then passed to the ProductListView for display. 

> This code is an example of Container Pattern. 

``` swift 
struct ContentView: View {
    
    @State private var products: [Product] = []
    let storeHTTPClient: StoreHTTPClient
    
    var body: some View {
        ProductListView(products: products)
            .task {
                do {
                    products = try await storeHTTPClient.loadProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
    }
}
```

If you want to perform any logic then it will go inside the container view. The main disadvantage of this approach is that it becomes hard to perform unit testing on your code that resides inside the view. Once again, if unit testing is not your priority then this is a perfectly valid pattern to use. 





In WWDC 2020 talk titled "Use Xcode for server-side development" Apple 



![Multiple Aggregate Root](/images/mul-aggregate-root.png)



