# Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture

In one of my previous articles, I mentioned that how I am using MV Pattern for building my client/server SwiftUI applications. 

## Understanding MV Pattern 

The main idea behind the MV Pattern is to allow views directly talk to the model. This eliminate the need for creating unnecessary layer of view models for each view, which simply contribute to the size of the project instead of providing any benefits.  

> In SwiftUI, view is similar to Component in React or Widget in Flutter. This means, it is not only used for displaying data but can also handle binding capabilities. **The View in SwiftUI is also a View Model.**. If you plan to put all the logic in your views then check out Container Pattern. MV Pattern is not the same as Container Pattern. 

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

![Multiple Aggregate Root](/images/mul-aggregate-root.png)



