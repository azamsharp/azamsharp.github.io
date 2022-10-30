# Evolving SwiftUI Client/Server Architecture 

TODO


## Consuming JSON in React 

React was introduced in 2013, it means React had a head start of around 6 years over SwiftUI. React, SwiftUI and Flutter are all declarative frameworks and they are extremely similar in nature. It is always a good idea to learn patterns an practices from more mature frameworks. We may not use React or Flutter patterns in our SwiftUI apps but it is always a good idea to know different ways and techniques of building apps. 

In the code below you can see the App component (Comparable to SwiftUI View), fetching all the products from an API and then displaying it on the screen. 

```swift 
import React, { useState, useEffect } from 'react'

function App() {

  const [products, setProducts] = useState([]) 

  useEffect(() => {
    fetchProducts()
  }, [])

  const fetchProducts = async () => { 
    const response = await fetch('https://api.escuelajs.co/api/v1/products?offset=0&limit=10')
    const products = response.json() 
    setProducts(products)
  }

  const productItems = products.map(product => {
    return `<li>{product.title}</li>`
  })

  return (
    <div>
      {productItems}
    </div>
  );
}
```

For iOS developers, here is the equivalent code in SwiftUI. 

```swift 
struct ContentView: View {
    
    @State private var products: [Product] = []
    
    var body: some View {
        List(products) { product in
            Text(product.title)
        }.task {
            do {
                products = try await fetchProducts()
            } catch {
                print(error)
            }
        }
    }
    
    private func fetchProducts() async throws -> [Product] {
        let (data, _) = try await URLSession.shared.data(from: Constants.Urls.allProducts)
        return try JSONDecoder().decode([Product].self, from: data)
    }
}
```

We used the private/local @State to hold on to the products and made the call to the API right within our View (View Model). 

>> SwiftUI views are basically equivalent to components in React or widgets in Flutter. SwiftUI views acts as a view model. Having said that, you should not put business logic in your view. Business logic belongs either in model or on the server in a client/server app.  

The main issue with the above approach is that, it will be hard to reuse ```fetchProducts``` in any other view, since it is part of the view (view model).  

But what if I don't plan to display products in any other view. Can I implement ```fetchProducts``` inside the view (view model)?  

Even though you may not call ```fetchProducts``` in any other view, it is recommended to move it out into a designated class. This way you will always have the flexibility to call it from anywhere and also add features like additional request headers, authorization, caching and even testing.  

## Implementing the NetworkModel 

As mentioned in the previous section, it is a good idea to move network calls in a separate dedicated class. This will allow you to easily reuse the network calls in separate views. 

The implementation of ```NetworkModel``` is shown below: 

```swift 
@MainActor
class NetworkModel: ObservableObject {
    
    @Published var products: [Product] = []
    
    func fetchProducts(url: URL) async throws {
        let (data, _) = try await URLSession.shared.data(from: url)
        products = try JSONDecoder().decode([Product].self, from: data)
    }
}
```

The NetworkModel class can be called directly from the ContentView. 

```swift 
struct ContentView: View {
    
   @StateObject private var networkModel = NetworkModel()
    
    var body: some View {
        List(networkModel.products) { product in
            Text(product.title)
        }.task {
            do {
                try await networkModel.fetchProducts(url: Constants.Urls.allProducts)
            } catch {
                print(error)
            }
        }
    }
}
```

>> In the above code, ContentView serves as a container view and the List can be extracted into a reusable presentation view. The primary purpose of container view is not to provide reusability but host views that are reusable. 



## Resources 

- [SwiftUI Architecture - A Complete Guide to MV Pattern Approach](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html)
- [Embracing Core Data in SwiftUI](https://azamsharp.com/2022/10/11/embracing-core-data-in-swiftui.html)
- [SwiftUI View is also a View Model](https://azamsharp.com/2022/07/21/view-is-the-view-model.html)

## Conclusion 
