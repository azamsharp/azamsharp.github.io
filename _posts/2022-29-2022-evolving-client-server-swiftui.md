# Evolving SwiftUI Client/Server Architecture 

In the last architecture, we discussed in detail about [SwiftUI Architecture using the MV Pattern](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). It is highly recommended that you read the original post. In this post, we will cover how to create SwiftUI client/server applications using ReactJS architectural patterns. 

>> There are no silver bullets, use the architecture that fit your needs. 

## Consuming JSON in React 

React was introduced in 2013, it means React had a head start of around 6 years over SwiftUI. React, SwiftUI and Flutter are all declarative frameworks and they are extremely similar in nature. You may not use React or Flutter to build iOS applications but it is always a good idea to know different ways and techniques of building apps. 

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

We used the private/local @State to hold on to the products and made the call to the API right within our view (view model). 

>> SwiftUI views are basically equivalent to components in React or widgets in Flutter. SwiftUI views acts as a view model. Having said that, you should not put business logic in your view. Business logic belongs either in model, domain service or on the server. 

The main issue with the above approach is that, it will be hard to reuse ```fetchProducts``` in any other view, since it is part of a particular view (view model).  

But what if you don't plan to display products in any other view. Can you implement ```fetchProducts``` inside the view (view model)?  

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

The NetworkModel class can be called directly from the ```ContentView``` as shown below:  

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

We can even refactor the above code to create ```ProductListView``` and ```ProduceCellView```. This is shown below: 

```swift 
struct ProductCellView: View {
    
    let product: Product
    
    var body: some View {
        VStack(alignment: .leading) {
            Text(product.title)
                .bold()
                .padding([.bottom], 10)
            Text(product.description)
        }
    }
}

struct ProductListView: View {
    
    let products: [Product]
    
    var body: some View {
        List(products) { product in
            ProductCellView(product: product)
        }
    }
}

struct ContentView: View {
    
   @StateObject private var networkModel = NetworkModel()
    
    var body: some View {
        ProductListView(products: networkModel.products)
            .task {
            do {
                try await networkModel.fetchProducts(url: Constants.Urls.allProducts)
            } catch {
                print(error)
            }
        }
    }
}
```


>> In the above code, ContentView serves as a container view and the ProductListView and ProductCellView are presentation views. Container views serves the purpose of containing other views (reusable views). Container views might download data from an API and then pass it down to the child views.  


## Searching

Most of the time when you are displaying a list of data, you also want to perform different actions on the data like searching and sorting. Let's see how we can accomplish the task of searching. 

We are going to use the ```searchable``` modifier, which will allow us to create a search bar. We are also going to use the ```onChange``` modifier, which will trigger each time a user will enter something in the TextField. The ```onChange``` modifier calls the performSearch function, which searches the products returned from networkModel and assigns it to the filteredProducts array. 
 
The new private property ```products``` returns filteredProduct if not empty, otherwise it returns list of all products.  

```swift 
struct ContentView: View {
    
    @StateObject private var networkModel = NetworkModel()
    @State private var search: String = ""
    @State private var filteredProducts: [Product] = []
    
    private func performSearch(keyword: String) {
        filteredProducts = networkModel.products.filter { product in
            product.title.lowercased().contains(keyword.lowercased())
        }
    }
    
    private var products: [Product] {
        filteredProducts.isEmpty ? networkModel.products: filteredProducts
    }
    
    var body: some View {
        NavigationStack {
            ProductListView(products: products)
                .searchable(text: $search)
                .onChange(of: search, perform: performSearch)
                .task {
                    do {
                        try await networkModel.fetchProducts(url: Constants.Urls.allProducts)
                    } catch {
                        print(error)
                    }
                }
        }
    }
}
```

>> The ContentView in SwiftUI, which is also a view model is not performing any business logic operations. All the business rules are run on the server side and only the result is sent back to the client (iOS App) for display. 

Now, let's take a look at sorting and how we can implement sorting in our application. 

## Sorting 

We are going to allow the user to sort products in ascending or descending order, based on the title of the product. The ```SortDirection``` is implemented below: 

```swift 
enum SortDirection {
    case asc
    case desc
}
```

The sorting will be performed by a button press. We also need to toggle the text of the button. This is performed by a local @State property as shown below: 

```swift
 var sortButtonText: String {
        sortDirection == .asc ? "Sort Descending": "Sort Ascending"
    }

      VStack {
                Button(sortButtonText) {
                    sortDirection = sortDirection == .asc ? .desc: .asc
                }
                ProductListView(products: products)
                    .searchable(text: $search)
                    .onChange(of: search, perform: performSearch)
                    .onChange(of: sortDirection, perform: performSort)
                    .task {
                        do {
                            try await networkModel.fetchProducts(url: Constants.Urls.allProducts)
                        } catch {
                            print(error)
                        }
                }
            }
```

The performSort function is fired, whenever the sortDirection changes. The performSort is responsible for sorting the list in ascending or descending order. 

```swift 
 private func performSort(direction: SortDirection) {
        
        switch direction {
            case .asc:
                networkModel.products = networkModel.products.sorted(by: \.title)
            case .desc:
                networkModel.products = networkModel.products.sorted(by: \.title).reversed()
        }
    }
```

>> The heart of sorting functionality is the [```sorted extension```](https://www.swiftbysundell.com/articles/the-power-of-key-paths-in-swift/), which allows sorting based on KeyPath. This means, it is a reusable function. 

If you find some functionality, that you will need to use in other views then it is always a good idea to move it aside in a separate class. This was the main reason we moved our networking code into a NetworkModel. 

>> You don't have to settle for just a single NetworkModel class for your complete app. A single NetworkModel may work for small sized applications but for larger apps you can create multiple NetworkModels based on the domain. This can include UserNetworkModel, AccountNetworkModel, ProductNetworkModel, CatalogNetworkModel etc. 

## Caching 

Caching allows your client (iOS App) to get responses from memory or filesystem instead of going all the way to the server. This can drastically improve performance of your apps. Even caching for just 5-6 seconds (Micro Caching) can help you save tons of requests and improve the performance of your app. 

The caching layer can be implemented in the NetworkModel. Caching will be covered in the future post, but you check out the comments below to get an idea. 

```swift 
@MainActor
class NetworkModel: ObservableObject {
    
    @Published var products: [Product] = []
    
    func fetchProducts(url: URL) async throws {
        
        // use a CacheService to check the data in the cache
        // if the data is already in cache then return it from cache
        // otherwise fetch a new copy of the data
        
        let (data, _) = try await URLSession.shared.data(from: url)
        products = try JSONDecoder().decode([Product].self, from: data)
    }
}
```

>> Sometimes developers find comfort in adding an intermediatory layer between the view and the network. We like to call that layer an aggregate root model and then your NetworkModel becomes Webservice. That approach was discussed in the last article [here](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). Depending on your app, you can evaluate which architecture works better for your app. 

## Resources 

- [SwiftUI Architecture - A Complete Guide to MV Pattern Approach](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html)
- [Embracing Core Data in SwiftUI](https://azamsharp.com/2022/10/11/embracing-core-data-in-swiftui.html)
- [SwiftUI View is also a View Model](https://azamsharp.com/2022/07/21/view-is-the-view-model.html)
- [SwiftUI List|Searching & Sorting|JSON API](https://youtu.be/hMZdRduyA_4)

## Conclusion 

In this post, you learned about SwiftUI architecture for client/server applications. This architecture is inspired from React applications. As mentioned before React, Flutter and SwiftUI share a lot of similarities and as developers, we should always try to learn from more mature frameworks. 

