# Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture

In one of my previous articles, I mentioned that how I am using MV Pattern for building my client/server SwiftUI applications. 

## Understanding MV Pattern 

The main idea behind the MV Pattern is to allow views directly talk to the model. This eliminate the need for creating unnecessary layer of view models for each view, which simply contribute to the size of the project instead of providing any benefits.  

> In SwiftUI, view is similar to Component in React or Widget in Flutter. This means, it is not only used for displaying data but can also handle binding capabilities. **The View in SwiftUI is also a View Model.**. If you plan to put logic in your views then check out Container Pattern.    

Apple has shown MV pattern in several places with different flavors. This includes Fruta, FoodTruck and ScrumDinger applications. My focus for this article is on client/server applications as they are one of the most common types of iOS applications. 

In WWDC 2020 talk titled "Data Essentials in SwiftUI" Apple presented a very similar diagram. 

![Aggregate Root](/images/aggre-root.png)


The main idea is to provide view access to a single layer that serves as source of truth and allows access to all the entities in the application.

Apple demonstrated the approach in their talk titled
**"Use Xcode for server-side development"**. The screenshot from the talk is shown below. 

![Use Xcode for server-side development](/images/xcode-server.png)

[Use Xcode for server-side development](https://developer.apple.com/videos/play/wwdc2022/110360/)

Apart from the WWDC video, Apple also demonstrated this technique in their Fruta and FoodTruck applications. 

> I know what you are thinking. Are we going to take advice based on Apple's code samples blindly? No! Never take any advice blindly. Put in the time and research the advantages and disadvantages of each approach. I have evaluated many different techniques and patterns and found this to be the best and simplest option when building client/server apps using SwiftUI. **Do your research!**  

Based on Apple's recommendation in their WWDC videos and code samples, I have been creating a single aggregate model, which holds the state for the entire application. The implementation is shown below: 

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

```StoreModel``` 


![Multiple Aggregate Root](/images/mul-aggregate-root.png)



