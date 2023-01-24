# Container Pattern in SwiftUI 

Container pattern is a common pattern used in React community. Since React and SwiftUI are quite similar, this pattern can also be used for building SwiftUI applications. 


>> I personally prefer [MV Pattern](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html) instead of the Container Pattern, but depending on your requirements you may find Container Pattern easy and simple to use. 

## What is Container Pattern? 

The main idea behind container pattern revolves around two different kind of views namely container and presenter views. The container view is responsible for fetching the data, sorting, filtering and other operations and then passing it down to presentation view for display. 

In other words, container is a smart view and presenter is a dumb view. The only job of the presenter view is to display data on the screen. Data is always passed down from the container view to the presenter view. 

Here is a small example, where the container view uses network layer to fetch the data and then passes it down to the presenter view for display. 

``` swift 
// Container
struct ContentView: View {
    
    @State private var products: [Product] = []
    
    var body: some View {
        ProductListView(products: products)
            .task {
                do {
                    products = try await Webservice().getProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
    }
}

// Presenter
struct ProductListView: View {
    
    let products: [Product]
    
    var body: some View {
        List(products) { product in
            Text(product.title)
        }
    }
}
```



## Testing 


## Conclusion: 




