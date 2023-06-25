# The Ultimate Guide to Building SwiftData Applications  

SwiftData was introduced at WWDC 2023 as a replacement for Core Data framework. SwiftData serves as a wrapper on top of Core Data and allows on-device persistence as well as syncing to the cloud. 

The main advantage of SwiftData is its seamless integration with SwiftUI framework. This post is divided into multiple sections. The first part of this post discusses the basics of SwiftData framework and then later on we dive into the architectural topics as well as current limitations of SwiftData framework.  

> SwiftData is part of iOS 17 and at the time of this writing Xcode 15 is still in beta stage. This means content discussed in this article is subject to change. I will try my best to keep the article updated. 

### Enable Core Data Debugging

As I mentioned earlier SwiftData uses Core Data behind the scenes. This means all the debugging techniques for Core Data should also work for SwiftData applications. One of the most common and easy to use debugging technique is through the use of flags in launch arguments. There are several different launch arguments available, but for starting out you can use the following: 

``` swift
-com.apple.CoreData.SQLDebug 1
```

This flag will output the path of the database as well as the SQL queries executed against the database. This can prove to be extremely valuable, when you want to make sure you are not performing excessive actions against the database and it allows you to refactor your code and make it better. 

If you want to learn more then check out this detailed [article](https://useyourloaf.com/blog/debugging-core-data/). 

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

### Relationships 

There is a saying in object oriented programming, "No object is an island". Relations is an integral part of a relational database. From database perspective, these relationships are handled by joining multiple tables together. Luckily, with SwiftData we don't have to worry about joins etc. We define our model relationships in Swift using macros provided by SwiftData. 

There are two relationships we have to define. 

1. A single budget can have many transactions. 
2. Each transaction can belong to a budget.  

We have modified the Budget class to support a list of transactions. This means one budget can have many transactions. The relationship is created using the ```@Relationship``` macro. The cascade option indicates that when the budget is deleted then all the transactions associated with that budget will also be deleted. 

``` swift 
@Model
final class Budget {
    
    var name: String
    var limit: Double
    
    @Relationship(.cascade)
    var transactions: [Transaction] = []
    
    init(name: String, limit: Double) {
        self.name = name
        self.limit = limit
    }
```

The other side of the relationship is from the transaction point of view. a transaction belongs to a budget. This relationship is shown below: 

``` swift 
@Model
final class Transaction {
    var note: String
    var amount: Double
    var date: Date
    var hasReceipt: Bool = false
    
    @Relationship(inverse: \Budget.transactions)
    var budget: Budget?
    
    init(note: String, amount: Double, date: Date, hasReceipt: Bool = false) {
        self.note = note
        self.amount = amount
        self.date = date
        self.hasReceipt = hasReceipt
    }
    
}
```

> I have experienced that even if you remove the @Relationship macro from Transaction class, it will be implicitly discovered by SwiftData. Also make sure to set  budget property as optional or else it will give you runtime error. 

Another thing to keep in mind is that the relationships created in SwiftData only exists in the object graph and are not the same as relationships between database tables. This means if you open the database using applications like Base or BeeKeeper, you will find the relationships section is completely empty. **SwiftData is a framework, which can persist an object graph but it is NOT an ORM**.   

It is important to note that you don't have to pass all the models used in your app to the model container. Depending on the relationships between the models you only need to pass the parent model. 

In the example below, we are only passing Budget.self to the modelContainer modifier. This is because Budget contains references to Transaction class and modelContainer can infer those relationships based on the budget class. 

``` swift 
@main
struct SpendSmartARPApp: App {
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                BudgetListScreen()
            }
        }
        // passing Transaction.self is not needed in the modelContainer below
        .modelContainer(for: Budget.self)
        
    }
}
```

There are several different ways of adding a transaction to a budget. It all depends on your user interface needs. Below you can find the implementation where we create a brand new transaction and then assign its budget property to the budget that was passed to the view.  

``` swift 
struct BudgetDetailScreen: View {
    
    @Environment(\.modelContext) private var context
    
    let budget: Budget
    
    private func saveTransaction() {
        // this function is fired after the validation of the form is successful 
        let transaction = Transaction(note: note, amount: amount!, date: date, hasReceipt: hasReceipt)
        transaction.budget = budget 
    }

    var body: some View {
 TransactionListView(transactions: budget.transactions)
    }
```

You don't need to call save or even insert since the model budget is already part of the context. **This will automatically update both sides of the relationship. It means transaction.budget will have a budget and a new transaction will be added to budget.transactions automatically.** 

Unfortunately, this does not re-render ```TransactionListView```. Even though the transactions in budget instance are updated, it still does not trigger an update on the view. I am not sure about the reason but I believe it may be because the update was caused internally and not through the mechanism that invoke the observation. 

The correct way of adding a transaction to an existing budget which also re-renders the view is by adding it through the ```budget.transactions``` property as shown below: 

``` swift 
  private func saveTransaction() {
        let transaction = Transaction(note: note, amount: amount!, date: date, hasReceipt: hasReceipt)
        // This will also re-render the TransactionListView 
        budget.transactions.append(transaction)
    }

      var body: some View {
 TransactionListView(transactions: budget.transactions)
    }
```

You can also add designated methods on budget class to perform add transaction or remove transaction operations. This allows you to run business logic prior to adding or removing transactions to a budget.    

``` swift 
 func addTransaction(_ transaction: Transaction) {
        // add business domain rules here
        self.transactions.append(transaction)
    }
```

If you plan to use the above approach, you can also make your transactions property ```private(set)``` so that it cannot be altered from outside the budget class. 

``` swift 
 @Relationship(.cascade)
    private(set) var transactions: [Transaction] = []
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

The ```@Query``` property wrapper also supports other arguments like filter, sort, order and animation. Here is the ```@Query``` implementation which supports sort and order. 

``` swift 
 @Query(sort: \.name, order: .forward) private var budgets: [Budget]
```

The budgets array will be sorted based on name property of the Budget type and organized in ascending order (.forward) parameter. 

You can also provide the filter option using predicates. Predicates are implemented using the freestanding macros in Swift. Here is a simple ```Query``` using the a predicate to only return the budgets having limits over $100. 

``` swift 
 @Query(filter: #Predicate { $0.limit > 100 }) private var budgets: [Budget]
```

Depending on your criteria, you can add multiple conditions in the predicate. One example is shown below: 

``` swift 
    @Query(filter: #Predicate { $0.limit > 100 && $0.name.contains("Vac") }) private var budgets: [Budget]
```

Predicate parameters are not always static/fixed. You can also make dynamic predicates. This means predicate will be based on a parameter passed to it. 

The following code snippet demonstrates the initialization of the view with the note parameter. This parameter plays a crucial role in initializing the ```Query``` object, enabling the creation of dynamic queries.   

``` swift 
  @Query private var transactions: [Transaction]
    
    init(note: String) {
        _transactions = Query(filter: #Predicate { $0.note.contains(note) }) 
    }
```

Unfortunately, the dynamic queries does not work in all scenarios. Maybe it is just the current limitation of SwiftData framework, which will be fixed in the future release. Here is an example, which will cause compile time errors. 

In the code below, we are trying to get all the transactions based on the budget's name. Unfortunately, this will result in an error.  

``` swift 
struct BudgetDetailScreen: View {
    
    @Environment(\.modelContext) private var context
    
    let budget: Budget
    
    @Query private var transactions: [Transaction]
    
    init(budget: Budget) {
        self.budget = budget
        // Missing argument label 'lhs:' in call
        // Initializer 'init(_:)' requires that 'Budget' conform to 'Decodable'
        // Initializer 'init(_:)' requires that 'Budget' conform to 'Encodable'
        _transactions = Query(filter: #Predicate { $0.budget!.name.contains(budget.name) })
    }
```

### Xcode Previews 

### Migrations 

### SwiftData with UIKit 

### Syncing with iCloud 

### Resources





