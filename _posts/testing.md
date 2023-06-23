# Architecturing SwiftData Applications 


SwiftData was introduced at WWDC 2023 as a replacement for Core Data framework. SwiftData serves as a wrapper on top of Core Data and allows on-device persistence as well as syncing to the cloud. 

The main advantage of SwiftData is its seamless integration with SwiftUI framework. This post is divided into multiple sections. The first part of this post discusses the basics of SwiftData framework and then later on we dive into the architectural topics as well as current limitations of SwiftData framework.  

> SwiftData is part of iOS 17 and at the time of this writing Xcode 15 is still in beta stage. This means content discussed in this article is subject to change. I will try my best to keep the article updated. 

### Getting Started with SwiftData

SwiftData allows you to declare the schema in code. This is different from Core Data, where you had to define a separate mapping file to create your schema. SwiftData uses the ```@Model``` macro, which dictates that this model is the source of truth and allows persistence. 

> Maybe in the future Apple can introduce a tool, which can allow developers to view the relationships between different SwiftData models in a more visual way. 

``` swift 
import SwiftData

@Model
final class Budget {
    
    var name: String
    var limit: Double
        
    init(name: String, limit: Double) {
        self.name = name
        self.limit = limit
    }
}
```

In the above code we have created a Budget class and decorated with ```@Model``` macro. This means Budget is now a source of truth and it can be persisted to the database. By default SwiftData uses SQLite database but it can be configured to persist data in XML, binary and even in-memory databases. 

> SwiftData automatically handles the identification aspect of a model, eliminating the need for explicitly defining an `id` property or conforming to the `Identifiable` protocol.

Keep in mind that the ```@Model``` macro can only be applied to classes. If you apply it to a type ```struct``` then you will be greeted with errors. 

To persist Budget to the database you will need to configure the model container. This is done in the App file as shown below: 

``` swift 
@main
struct SpendSmartARPApp: App {
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                BudgetListScreen()
            }
        }
        .modelContainer(for: Budget.self)
    }
}
```

Now you can persist the budget to the database by using the modelContext through the Environment as shown below: 

``` swift 
struct BudgetListScreen: View {
    
    @Environment(\.modelContext) private var context

    @State private var name: String = ""
    @State private var limit: Double?
    
    func saveBudget() {
        // saveBudget is fired only after the data has been successfully validated
        let budget = Budget(name: name, limit: limit!)
        context.insert(budget)
    }
```

One thing to notice is that we are not explicitly calling the save function on context. The insert function will add the model to the context and then internally call the save function. SwiftData autosaves the model context. The autosave events are triggered based on the UI related events. 

Not all apps require autosave feature. If you are not interested in autosave then you can turn it off at the model container level as shown below: 

``` swift 
 var body: some Scene {
        WindowGroup {
            NavigationStack {
                BudgetListScreen()
            }
        }
        .modelContainer(for: Budget.self, isAutosaveEnabled: false)
        
    }
```

### Querying Data 

Once the data has been persisted, the next step is to display it on the screen. SwiftData uses the ```@Query``` property wrapper for fetching data from the database. In the code below we fetch all the budgets and then display it in the list. 

``` swift 
struct BudgetListScreen: View {
    
    @Environment(\.modelContext) private var context
    @Query private var budgets: [Budget]
    
    var body: some View {
        Form {
            
            Section("Budgets") {
                List(budgets) { budget in
                        HStack {
                            Text(budget.name)
                            Spacer()
                            Text(budget.limit, format: .currency(code: "USD"))
                        }
                }
            }
        }
    }
}
```

> It is important to point out that you don't  have to pass all the models used in your app to the model container. Depending on the relationships between the models you only need to pass the parent model.   



### Resources





