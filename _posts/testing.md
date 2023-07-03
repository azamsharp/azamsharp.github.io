# The Ultimate Guide to Building SwiftData Applications  

SwiftData was introduced at WWDC 2023 as a replacement for Core Data framework. SwiftData serves as a wrapper on top of Core Data and allows on-device persistence as well as syncing to the cloud. 

The main advantage of using SwiftData is its seamless integration with SwiftUI framework. This post is divided into multiple sections. The first part of this post discusses the basics of SwiftData framework and then later on we dive into the architectural topics as well as migrations and testing using SwiftData framework. 

> SwiftData is part of iOS 17 and at the time of this writing Xcode 15 is still in beta stage. This means content discussed in this article is subject to change. I will try my best to keep the article updated. 

The outline of this article is shown below: 

- [Enable Core Data Debugging](#enable-core-data-debugging) 
- [Getting Started with SwiftData](#getting-started-with-swiftdata)
- [Migrations](#migrations)
- [Architecture](#architecture)
- [Testing](#testing)

### Enable Core Data Debugging

As I mentioned earlier SwiftData uses Core Data behind the scenes. This means all the debugging techniques for Core Data should also work for SwiftData applications. One of the most common and easy to use debugging technique is through the use of flags in launch arguments. There are several different launch arguments available, but for starting out you can use the following: 

``` swift
-com.apple.CoreData.SQLDebug 1
```

This flag will output the path of the database as well as the SQL queries executed against the database. This can prove to be extremely valuable, when you want to make sure that you are not performing excessive queries against the database and it allows you to refactor your code and make it better. 

If you want to learn more then check out this detailed [article](https://useyourloaf.com/blog/debugging-core-data/). 

### Getting Started with SwiftData

SwiftData allows you to declare the schema in code. This is different from Core Data, where you had to define a separate mapping file to create your schema. SwiftData uses the ```@Model``` macro, which dictates that this model is the source of truth and allows persistence. 

> At this time there is no tool to view the relationships between different SwiftData models visually. Hopefully, in the future Apple can introduce a tool to provide this visualization.

Below you can see the implementation of the ```Budget``` model:

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

The ```Budget``` class looks like a normal class, except that it is decorated with ```@Model``` macro. The ```@Model``` macro will generate the required code, which will allow the ```Budget``` class to be persisted to the persisted store. By default SwiftData uses SQLite database but it can be configured to persist data in XML, binary and even in-memory databases. 

Keep in mind that the ```@Model``` macro can only be applied to classes. If you apply it to a type ```struct``` then you will be greeted with errors. This means your models must be a reference type and not value types.  

> SwiftData automatically handles the identification aspect of a model, eliminating the need for explicitly defining an `id` property or conforming to the `Identifiable` protocol.

To persist Budget to the database you will need to configure the model container. This can be done in the App file as shown below: 

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

Now you can persist the budget to the database by using the ```modelContext``` through the Environment as shown below: 

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

One thing to notice is that we are not explicitly calling the save function on the context. The insert function will add the model to the context and then internally call save. SwiftData autosaves the model context. The autosave events are triggered based on the UI related events. 

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

There is a saying in object oriented programming, "No object is an island". Relations is an integral part of a relational database. From database perspective, these relationships are handled by joining multiple tables together. Luckily, with SwiftData we don't have to worry about joins etc. We define our model relationships in Swift using the ```@Model``` macro provided by SwiftData. 

There are two relationships we have to define. 

1. A single budget can have many transactions. 
2. Each transaction can belong to a single budget.  

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

> If you are planning to use iCloud and CloudKit to sync your SwiftData records then you need to make sure that the relationships you define in SwiftData are optional with a default value. This means the above relationship will be written as ```@Relationship(.cascade)
    var transactions: [Transaction]? = []```

The other side of the relationship is from the transaction point of view. A transaction can belong to a budget. This relationship is shown below: 

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

> I have experienced that even if you remove the @Relationship macro from Transaction class, it will be implicitly discovered by SwiftData. Also make sure to set budget property as optional or else it will give you runtime error. 

Another thing to keep in mind is that the relationships created in SwiftData only exists in the object graph and they are not the same as relationships between database tables. This means if you open the database using applications like [Base](https://menial.co.uk/base/) or [BeeKeeper](https://www.beekeeperstudio.io/), you will not find any foreign key constraints listed under the relationships section. **SwiftData is a framework, which can persist an object graph to different stores, but it is NOT an ORM**.   

It is also important to note that you don't have to pass all the models used in your app to the model container in the ```SpendSmartARPApp``` struct. Depending on the relationships between the models you only need to pass the parent model and all the child relationships are automatically inferred.  

In the example below, we are only passing ```Budget``` type to the modelContainer modifier. This is because Budget contains reference to ```Transaction``` class and modelContainer can infer those relationships based on the budget class. 

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

Now that we have setup the relationship between budget and transaction, the next step is to add a transaction to a particular budget. There are several different ways of adding a transaction to a budget. It all depends on your user interface needs. Below you can find the implementation where we create a brand new transaction and then assign it to the budget property. The budget was passed to the view through the constructor. 

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

Unfortunately, this does not re-render ```TransactionListView```. Even though the transactions in budget instance are updated, it still does not trigger an update on the view. I am not sure about the reason but I believe it may be because the update was caused internally and not through the mechanism that invoke the observation behavior. 

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

SwiftData relations are not limited to one-to-one or one-to-many but you can also construct many-to-many relationships, also known as siblings relationship. In the code below, we have defined two models ```Recipe``` and ```Ingredient```. The relationship between Recipe and Ingredient is many to many.

- One recipe can have many ingredients 
- One ingredient can belong to many recipes 

``` swift 
@Model
final class Recipe {
    let name: String
    
    init(name: String) {
        self.name = name
    }
    
    @Relationship
    var ingredients: [Ingredient] = []
}

@Model
final class Ingredient {
    
    let name: String
    
    @Relationship(inverse: \Recipe.ingredients)
    var recipes: [Recipe] = []
    
    init(name: String) {
        self.name = name
    }
}
```

When adding a new recipe with ingredients you must make sure that recipe has already been persisted. Take a look at the following code, which will result in an error: 

``` swift 
   Button("Save") {
                
                let recipe = Recipe(name: recipeName)
                let ingredients = ingredientNames.components(separatedBy: ",")
                    .map { $0.trimmingCharacters(in: .whitespaces) }
                
                ingredients.forEach { name in
                    let ingredient = Ingredient(name: name)
                    recipe.ingredients.append(ingredient)
                }
                context.insert(recipe)
            }
```

After creating the recipe instance, we loop though the ingredient names and append them to the recipe through the ingredients property. But since recipe is not yet saved to the context, this will cause an exception. The fix is to add the recipe to the context right after the recipe instance is created. This is shown in the implementation below: 

``` swift 
  Button("Save") {
                
                let recipe = Recipe(name: recipeName)
                
                context.insert(recipe)

                let ingredients = ingredientNames.components(separatedBy: ",")
                    .map { $0.trimmingCharacters(in: .whitespaces) }
                
                
                ingredients.forEach { name in
                    let ingredient = Ingredient(name: name)
                    recipe.ingredients.append(ingredient)
                }
            }
```

Now, when you run the code it will not have any exceptions. Behind the scenes, this operation is performed by at least 3 tables. One table for recipes, one for ingredients and one will be a pivot table used for the relationships between recipes and ingredients. 

We have not provided the ```.cascade``` option for the relationships, because we don't want to delete all the ingredients when a recipe is deleted and vice versa. If you delete an ingredient from ```recipe.ingredients``` array then it will be simply be removed from the array. Same goes for removing a recipe from ```ingredients.recipes``` array.  

### Querying Data 

Once the data has been persisted, the next step is to display it on the screen. SwiftData uses the ```@Query``` property wrapper for fetching data from the database. In the code below we fetch all the budgets and then display it in the list. 

> The ```@Query``` property wrapper may remind you of ```@FetchRequest``` property wrapper in Core Data. They do share a lot of common characteristics.  

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

The ```@Query``` property wrapper also supports other arguments like filter, sort, order and animation. Here is the ```@Query``` implementation which supports sorting and ordering. 

``` swift 
 @Query(sort: \.name, order: .forward) private var budgets: [Budget]
```

The budgets array will be sorted based on ```name``` property of the Budget type and organized in ascending order (.forward) parameter. 

You can also provide the filter option using predicates. Predicates are implemented using the [freestanding macros](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/macros/) in Swift. Here is a simple ```Query``` using a predicate to only return the budgets having limits over $100. 

``` swift 
@Query(filter: #Predicate { $0.limit > 100 }) private var budgets: [Budget]
```

Depending on your criteria, you can add multiple conditions in the predicate. One example is shown below: 

``` swift 
@Query(filter: #Predicate { $0.limit > 100 && $0.name.contains("Vac") }) private var budgets: [Budget]
```

Predicates are not always implemented using static/fixed values. You can also make dynamic predicates. This means predicate will be based on a parameter passed to it. 

The following code snippet demonstrates the initialization of a view with the note parameter. This parameter plays a crucial role in initializing the ```Query``` object and enabling the creation of dynamic queries.   

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

> This might be related to a bug in SwiftData. Keep in mind that at the time of this writing SwiftData is still not released. 

As you learned earlier that queries are implemented using the ```@Query``` property wrapper. The ```@Query``` property wrapper is only available inside the view. But that does not mean that queries cannot be constructed outside of the view. In the implementation below, we have created a ```FetchDescriptor``` inside the Budget class itself, which is later injected into the ```@Query```. 

``` swift 
@Model
final class Budget {
    
    var name: String
    var limit: Double
    
    // other code ....
    
    static var all: FetchDescriptor<Budget> {
        FetchDescriptor(sortBy: [SortDescriptor(\Budget.name, order: .reverse)])
    }
    
}
```

Now inside the view you can use the ```all``` function of the Budget class. 

``` swift 
struct BudgetListScreen: View {   
    @Query(Budget.all) private var allBudgets: [Budget]
}
```

This allows you to move the creation of the query in the model itself instead of the view, allowing you to use the same query in other parts of the application. My recommendation is to start out having the query in the view. If your query is getting complicated and needs to be reused in other views then think about moving it to the model class. 

> At the time of this writing there is also no way to dynamically change the predicate attached with the query. This means you will have to create a new instance of the query and provide the new predicate. In Core Data with ```@FetchRequest``` you were allowed to substitute the predicate with a new one. Maybe this is just a current limitation and will be addressed in the future versions of SwiftData framework.  

### Xcode Previews 

Xcode previews plays a vital role in the development of SwiftUI applications, offering a significant advantage in rapidly iterating over designs and visually validating logic.

In the talk [Build programmatic UI with Xcode Previews](https://developer.apple.com/videos/play/wwdc2023/10252/), Apple engineer mentioned that previews are like tests. They help you to quickly iterate over your user interface design and even UI logic. 

My experience with previews has been the same. I use it extensively for iterating over the app design and user interface logic. If the UI logic is more complicated then those parts can be separated out into independent data structs, where they can be tested individually using XCTest framework. 

> Xcode previews are great for incrementally testing your UI and the logic contained in the UI but they are not a replacement for all different types of tests on your application. You still need to write domain level unit tests, integration tests and end-to-end tests.  

One way to use previews in SwiftData is by implementing a custom ```ModelContainer```. This technique was shown in WWDC video titled [Build an app with SwiftData](https://developer.apple.com/videos/play/wwdc2023/10154/?time=530). The main idea is to create a model container only for the purpose of rendering Xcode previews. The model container can be in-memory containing fake data. The implementation is shown below: 

``` swift 
import Foundation
import SwiftData

@MainActor
let previewContainer: ModelContainer = {
    
    do {
        let container = try ModelContainer(for: Budget.self, ModelConfiguration(inMemory: true))
        SampleData.budgets.enumerated().forEach { index, budget in
            container.mainContext.insert(budget)
            let transaction = Transaction(note: "Note \(index + 1)", amount: (Double(index) * 10), date: Date())
            budget.addTransaction(transaction)
        }
        
        return container
        
    } catch {
        fatalError("Failed to create container.")
    }
}()

struct SampleData {
    static let budgets: [Budget] = {
        return (1...5).map { Budget(name: "Budget \($0)", limit: 100 * Double($0)) }
    }()
}
```

In the above code, we have not only created fake budget objects but also added some transactions to each budget. You can even get more creative and populate fake data through a JSON file.

> It is not mandatory that your model container for previews is always in-memory. You can use an actual persistent model container too. This way your data will be available between preview refreshes. 

Finally, you can use the previewContainer in your view as shown in the implementation below: 

``` swift 
struct BudgetDetailContainerView: View {
    @Query private var budgets: [Budget]
    
    var body: some View {
        NavigationStack {
            BudgetDetailScreen(budget: budgets[0])
        }
    }
}

#Preview { @MainActor in
    BudgetDetailContainerView()
        .modelContainer(previewContainer)
}
```

Please note that the above code will cause a warning ```Converting function value of type '@MainActor () -> any View' to '() -> any View' loses global actor 'MainActor'```. At this point, it is not clear that if this is a bug or expected behavior.  

> #Preview is a new macro introduced in SwiftUI, which automatically writes the necessary code to generate a preview for the view. These kind of macros are also known as freestanding macros. 

Although using ```#Preview``` macro is the preferred choice, you can still use the ```PreviewProvider``` to create macros for your application. This is shown below:  

``` swift 
struct BudgetDetailContainer_Previews: PreviewProvider {
    static var previews: some View {
        BudgetDetailContainerView()
            .modelContainer(previewContainer)
        
    }
}
```

### Migrations 

As your application grows, your database schema also changes. It is important to keep track of these changes so you can revert back to the old schema, if needed. This also allows a new member on the team to quickly setup their environment. All they need to do is run database migrations and they system will be up to date with the current database schema. 

Any changes you do to the schema of the database must be represented by a migration. These changes includes but not limited to: 

- Adding or removing constraints to a column 
- Renaming an existing column 
- Changing the data type of the column 

SwiftData allows you to perform migrations by defining different schema versions. Consider an example where you have a ```Budget``` model defined below: 

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
}
```

The Budget model currently does not have unique constraints on the name. This means you can have duplicate budget names added to your application. What if we want to update our schema to support unique name constraints. We can't just update our Budget model and add the ```@Attribute(.unique)``` macro. The main reason is that we may have existing budget records in the database with the duplicated names and if we try to add the unique constraints then the database will throw an error since the unique constraints will be activated because of the existing duplicate records. 

Since this operation requires the change to the schema, we must write a migration to perform this action. We also must take care of existing duplicate budget records in the database to ensure data integrity. 

Migrations in SwiftData are created using ```VersionedSchema``` protocol. You can create different versions of your schema by conforming to ```VersionedSchema``` protocol. Below, you can see the implementation of our original schema. 

``` swift 
enum SpendTrackerSchemaV1: VersionedSchema {
    
      static var versionIdentifier: String? = "Initial version of the model"
    
    static var models: [any PersistentModel.Type] {
        [Budget.self]
    }
    
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
        
    }
}
```

The ```versionIdentifier``` is used to specify the purpose of the migration. The ```models``` property is used to indicate, which models will be part of this schema. And finally, we have the complete implementation of the Budget model. 

As you can see in the above implementation, we did not have the unique constraint on the name property. This is because in our original implementation, we did not have the unique constraint on the name property. These model/schema changes came later and that is the reason we need to implement a version 2 of the schema. 

In the implementation below we have the V2 of the schema, which includes the unique attribute on the name property. 

``` swift 

enum SpendTrackerSchemaV2: VersionedSchema {
    
    static var versionIdentifier: String? = "Adding unique constraint to the name property"
    
    static var models: [any PersistentModel.Type] {
        [Budget.self]
    }
    
    @Model
    final class Budget {
        
        @Attribute(.unique) var name: String
        var limit: Double
        
        @Relationship(.cascade)
        var transactions: [Transaction] = []
        
        init(name: String, limit: Double) {
            self.name = name
            self.limit = limit
        }
        
    }
}

```

Once, you have implemented the new version of the schema. The next step is to work on a migration plan using ```SchemaMigrationPlan``` protocol. SchemaMigrationPlan provides an interface for describing the evolution of a schema and how to migrate between specific versions.

Since this migration involves preserving the data integrity, it will require a custom migration state. A custom migration is created using the ```custom``` function on the ```MigrationStage``` enum. The function supports the ```willMigrate``` and ```didMigrate``` closures, which are fired at different lifetime of the migration. In our case, we will be using ```willMigrate``` to update the current budget record to make sure the names are unique. 

``` swift 
 static let migrateV1toV2 = MigrationStage.custom(fromVersion: SpendTrackerSchemaV1.self, toVersion: SpendTrackerSchemaV2.self, willMigrate: { context in
        
        guard let budgets = try? context.fetch(FetchDescriptor<Budget>()) else { return }
        
        var duplicates = Set<Budget>()
        var uniqueSet = Set<String>()
        
        for budget in budgets {
            if !uniqueSet.insert(budget.name).inserted {
                duplicates.insert(budget)
            }
        }
        
        // now change the names for duplicates
        for budget in duplicates {
            let budgetToBeUpdated = budgets.first(where: { $0.id == budget.id } )!
            budgetToBeUpdated.name = budgetToBeUpdated.name + " \(generateUniqueRandomNumber())"
        }
        
        // calling save is important
        try? context.save()
        
        
    }, didMigrate: nil)
```

In the above ```willMigrate``` implementation, we first find all the duplicate budgets based on their names. Once we find all the duplicated names we go through them and update their name property to make it unique. And then finally we persist the information to the database by calling ```context.save()``` function. 

**Don't run your app yet!**

Remember that your models are defined as versioned schema and not in the Budget.swift file. Open your Budget.swift file and make a ```typealias``` to point to the correct Budget model version. 

``` swift
typealias Budget = SpendTrackerSchemaV2.Budget
```

> The tip to use ```typealias``` was shared by Pol Piella in his article [Configuring SwiftData in a SwiftUI app](https://www.polpiella.dev/configuring-swiftdata-in-a-swiftui-app). 

Finally, you need to update your SpendTrackerApp file to use the migration plan. This is shown below: 

``` swift 
@main
struct SpendTrackerApp: App {
    
    let container: ModelContainer
    
    init() {
        do {
            container = try ModelContainer(for: [Budget.self], migrationPlan: SpendTrackerMigrationPlan.self, ModelConfiguration(for: [Budget.self]))
        } catch {
            fatalError("Could not initialize the container.")
        }
    }
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                BudgetListScreen()
            }
        }
        .modelContainer(container)
    }
}
```

 Now if you run the app, the migration is going to run and add the unique constraint to the name property. Not only that but your existing budget names will be updated to satisfy the unique constraints.  

### Architecture 

Architecture has always been a topic of hot debate, specially in the SwiftUI community. There are couple of reasons behind the confusion. First, Apple has never advocated openly about a particular architecture to follow when building SwiftUI applications. The primary rationale behind this is that, unlike UIKit applications that typically adhered to the MVC (Model-View-Controller) architecture by default, SwiftUI offers greater flexibility in terms of architectural choices. 

I have written several articles about SwiftUI architectures. Including [Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html) and [Active Record Pattern for Building SwiftUI Apps with Core Data](https://azamsharp.com/2023/01/30/active-record-pattern-swiftui-core-data.html)

> While Apple has not explicitly recommended a particular architecture for building SwiftUI or SwiftData applications, we can acquire valuable insights into the architectural patterns used by Apple through their sample code and WWDC videos.

During [Platforms State of the Union 2023](https://developer.apple.com/videos/play/wwdc2023/102/?time=81), Darin Adler said **"the most natural way to write your code is also the best."**. This was again mentioned by Josh Shaffer (Engineering director with the UIKit and SwiftUI team at Apple) in the Under the Radar podcast [episode 270](https://www.relay.fm/radar/270). 

This can mean different things to different people but for me it simply means that let SwiftUI be SwiftUI. Instead of fighting the framework, try to work with it. 

Since 2019, I have used many different architectural patterns when building SwiftUI applications. This included MVVM, Container pattern, Redux, MV pattern and Active Record Pattern. Apple has a sample SwiftData application called [Backyard Birds: Building an app with SwiftData and widgets](https://developer.apple.com/documentation/swiftui/backyard-birds-sample), which uses a variation of Active Record Pattern. 

I say variation of Active Record Pattern because Apple puts all the logic in their models but still uses the model context for persistence operations like save and delete. This technique allows you to work with Xcode Previews for SwiftData applications since you can easily inject a model container for the previews. I covered working with Xcode previews [earlier](#xcode-previews) in this article.   

Apple introduced ```@FetchRequest``` for Core Data and ```@Query``` for SwiftData. These property wrappers are optimized for working with SwiftUI framework. But sometimes in a quest to satisfy a certain architecture, we ignore SwiftUI built-in features and try to reinvent the wheel. I have seen a lot of developers ignoring the above mentioned property wrappers and manually implementing ```NSFetchedResultsController``` for their SwiftUI applications. I have done the same, I even have a video on it titled [Core Data MVVM in SwiftUI App Using NSFetchedResultsController](https://youtu.be/gGM_Qn3CUfQ). 

Ultimately, my efforts resulted in more lines of code, contributing to an increased burden and liability. The key takeaway from this experience is to embrace SwiftUI as it was intended, avoiding unnecessary complications. Remember that the simplest and most natural approach often yields the best results.  

### Testing

Testing plays a crucial role in software development, serving as a cornerstone of confidence. When dealing with a straightforward domain, I may opt for minimal or even zero tests. However, when tackling a complex business domain, I rely on the assistance of unit tests to ensure accuracy and reliability.

In software development, domain is considered the heart of the application and requires extensive testing.

``` swift 
  func test_should_calculate_budget_total_successfully() {
        
        let budget = Budget(name: "Budger 1", limit: 100)
        let transaction = Transaction(note: "Note 1", amount: 100, date: Date())
        let transaction2 = Transaction(note: "Note 2", amount: 50, date: Date())
        let transaction3 = Transaction(note: "Note 3", amount: 100, date: Date())
        
        budget.addTransaction(transaction)
        budget.addTransaction(transaction2)
        budget.addTransaction(transaction3)

        XCTAssertEqual(250, budget.total)
        
    }
```

Testing is a vast and complicated topic. I have written few articles on my views on testing. You can read it [here](https://azamsharp.com/2012/12/23/pragmatic-unit-testing.html). If you are interested to learn more about testing then check out the book, [Unit Testing Principles, Practices and Patterns by Vladimir Khorikov](https://a.co/d/edDcv2q). 

Sometime back I shared on Twitter that how I consider Xcode Previews to be kind of like tests for your views. This means all the presentation and transformation logic you have in your views can be easily tested using Xcode Previews. 

![Xcode Previews as Tests](/images/xcode-previews-test.png)

Apple also shared similar views in their WWDC 2023 video titled [Build programmatic UI with Xcode Previews](https://developer.apple.com/videos/play/wwdc2023/10252/). 

![Apple Xcode Previews are Tests](/images/apple-xcode-previews.png)

In the end, focus on writing tests for the most important parts of your application. In most cases, it is the domain. This will give you the best return on your investment.    

### SwiftData with UIKit 

SwiftData is primarily designed to work with SwiftUI, but it can be integrated with the UIKit framework. Apple discusses these steps these in the article [Preserving your app’s model data across launches](https://developer.apple.com/documentation/swiftdata/preservingyourappsmodeldataacrosslaunches). 

You will start by setting up the model container for your app. This should be done at the start of your application. The implementation of the container property is shown below: 

``` swift 
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    // other code 
    
    lazy var container: ModelContainer = {
        //let configuration = ModelConfiguration(inMemory: true)
        let container = try! ModelContainer(for: TodoItem.self)
        return container
    }()
}
```

Once the container has been initialized, you can use it in your view controllers, just like you have for SwiftUI applications. Once thing to notice is that when working with SwiftData fro your UIKit apps, you don't have access to the ```@Query``` property wrapper. This means you need to call the fetch function, which is part of the context to manually fetch the records. This is shown in the implementation below: 

``` swift 
 private func populateTodoItems() {
        
        guard let appDelegate = UIApplication.shared.delegate as? AppDelegate else { return }
        let context = appDelegate.container.mainContext
        
        let fetchDescriptor: FetchDescriptor<TodoItem> = FetchDescriptor()
        
        do {
            self.todoItems = try context.fetch(fetchDescriptor)
            tableView.reloadData()
        } catch {
            print(error)
        }
    }
```

> There is no equivalent of ```NSFetchedResultsController``` in SwiftData. This also indicates Apple's intention that SwiftData is primarily created to work with SwiftUI and not with UIKit. Having said that you can, if you want still use SwiftData with UIKit.    

### Resources

- [SwiftData official documentation](https://developer.apple.com/documentation/swiftdata)
- [SwiftData WWDC Videos](https://www.google.com/search?q=wwdc+videos+swiftdata&oq=wwdc+videos+swiftdata&aqs=chrome..69i57j69i60l3.3658j0j7&sourceid=chrome&ie=UTF-8)
- [SwiftData Udemy Course: SwiftData - Declarative Data Persistence for SwiftUI ](https://www.udemy.com/course/swiftdata-declarative-data-persistence-for-swiftui/?referralCode=A1303D0BA99171C90D9B)




