<style>
    .share-container {
      display: flex;
      gap: 10px; /* Spacing between buttons */
      margin-bottom: 20px; 
    }
    .share-button {
      background-color: #0077b5; /* Default LinkedIn blue */
      color: white;
      border: none;
      padding: 10px 15px;
      font-size: 14px;
      border-radius: 5px;
      text-decoration: none;
      cursor: pointer;
      text-align: center;
    }
    .twitter { background-color: #1da1f2; }
    .linkedin { background-color: #0077b5; }
    .bluesky { background-color: #353c63; }
    .share-button:hover {
      opacity: 0.8;
    }
  </style>

# SwiftData Architecture – Patterns and Practices 

 <div class="share-container">
    <div>
      <!-- Twitter -->
      <a href="https://twitter.com/intent/tweet?url=https://azamsharp.com/2025/03/28/swiftdata-architecture-patterns-and-practices.html&text=SwiftData Architecture - Patterns and Practices by @azamsharp"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button twitter">
        Share on Twitter
      </a>
    </div>
    <div>
      <!-- LinkedIn -->
      <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://azamsharp.com/2025/03/28/swiftdata-architecture-patterns-and-practices.html"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button linkedin">
        Share on LinkedIn
      </a>
    </div>
    
  </div>

SwiftData, announced at WWDC 2023, marks a significant evolution in data persistence for SwiftUI applications. Built from the ground up with Swift-native syntax and a declarative approach, SwiftData aims to replace Core Data with something that feels more intuitive, modern, and better integrated into the SwiftUI ecosystem.

But as with any new framework, the big question remains: *how should you architect your app around it?* Should you fully embrace SwiftData’s built-in conveniences like `@Model` and `@Query`, or should you abstract them behind protocols for flexibility? Where should business logic live? And how can you structure your code so that it remains testable, maintainable, and easy to reason about?

In this article, I’ll walk you through the patterns and practices I’ve developed for building real-world SwiftData applications. Using a budget tracking app as an example, we’ll explore:

- How to structure your data models
- Where to place business logic and validation
- When to use DTOs (and when not to)
- How to write meaningful unit tests
- How to set up effective Xcode previews
- How to work with CloudKit integration
- How to future-proof your app against changes in persistence layers

Whether you’re just getting started with SwiftData or looking to refine your architecture, this guide will help you make informed, pragmatic decisions and get the most out of Apple’s latest persistence framework.

<div style="
    background: linear-gradient(to right, #e6f2ff, #f9fbff);
    border-left: 6px solid #0073e6;
    padding: 24px 28px;
    border-radius: 8px;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
    font-size: 1.1rem;
    color: #2c3e50;
    margin: 24px 0;
    box-shadow: 0 4px 10px rgba(0, 115, 230, 0.08);
">
    <strong style="font-size: 1.2rem;">🚀 Want to become a highly valued iOS developer?</strong><br><br>
    Explore expert-led courses and hands-on learning at 
    <a href="https://azamsharp.school" style="color: #0073e6; text-decoration: none; font-weight: 600;">
        AzamSharp School
    </a>.
</div>


## Architecture

Over the years, I’ve experimented with several different patterns for building SwiftData applications. But I’ve always found myself asking the same question: How can I make this simpler—while still keeping the code scalable, testable, and easy to maintain?

In this article, I’ll demonstrate how to build a SwiftData application using a budget tracking app as an example. The main reason for choosing a budget tracking application is that most people are already familiar with the concepts of budgeting and expenses, making it easier to focus on the architecture rather than the domain.

Let's start by creating our model to represent a Budget. The implementation is shown below: 

``` swift 
import SwiftData

@Model
class Budget {
    var name: String
    var limit: Double
    
    init(name: String, limit: Double) {
        self.name = name
        self.limit = limit
    }
}
```


The `Budget` model is quite straightforward—it consists of just two properties: `name` and `limit`. Next, let’s look at how we can persist a new budget to the database using the SwiftData framework. Below is the implementation of the `BudgetListScreen`.

``` swift 
struct BudgetListScreen: View {
    
    @State private var name: String = ""
    @State private var limit: Double?
    
    @Environment(\.modelContext) private var context
    @Query private var budgets: [Budget] = []
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            TextField("Limit", value: $limit, format: .currency(code: "USD"))
            Button("Save") {
                
                // validate the name and limit
                guard let limit = limit else { return }
                
                let budget = Budget(name: name, limit: limit)
                context.insert(budget)
                
            }
            
            if !budgets.isEmpty {
                Section("Budgets") {
                    List(budgets) { budget in
                        Text(budget.name)
                    }
                }
            } else {
                ContentUnavailableView("No budgets available", systemImage: "chart.pie.fill")
            }
        }
    }
}

#Preview {
    BudgetListScreen()
        .modelContainer(for: Budget.self, inMemory: true)
}

```

There are several key aspects to note about the implementation above. First, for Xcode Previews, we’ve injected a `modelContainer` with `inMemory` explicitly set to `true`. This ensures that preview data is not persisted to the actual database, allowing for safe and isolated testing within the preview environment.

When the user taps the button, a new `Budget` object is instantiated, its values are assigned, and it is inserted into the model context. There's no need to explicitly call the `save()` function on the model context, as SwiftData automatically persists changes in response to certain app events—such as navigation or when the app moves to the background.

SwiftData also introduces the `@Query` macro, which enables seamless tracking and fetching of records from the database. It functions similarly to the `@FetchRequest` property wrapper in Core Data, which internally relies on `NSFetchedResultsController` to monitor changes and update the UI accordingly. When a new budget is added, the `@Query` macro automatically fetches the updated list of budgets and reflects the changes on the screen.  

Currently, our application doesn't implement any business rules. So far, we've focused solely on persisting budgets and retrieving them. In the next section, we’ll explore how to incorporate domain logic and business rules into your SwiftData application.

### Business Rules 

To ensure that each budget has a unique name—meaning no two budgets can share the same name—we need to introduce a validation step. While SwiftData provides the `#Unique` macro for enforcing uniqueness, it doesn't throw an error or offer a clean way to handle violations when the rule is broken.

As a workaround, we can implement an exists function in the Budget model that checks whether a budget with the given name already exists in the database.

Although it's possible to place this logic directly in the BudgetListScreen, a better approach is to encapsulate it within the Budget class. This not only makes the code more reusable and easier to maintain but also enables unit testing the logic independently of the UI layer.

The implementation is shown below: 

``` swift 
@Model
class Budget {
    var name: String
    var limit: Double
    
    init(name: String, limit: Double) {
        self.name = name
        self.limit = limit
    }
    
    private func exists(context: ModelContext, name: String) throws -> Bool {
       
        let predicate = #Predicate<Budget> { budget in
            budget.name.localizedStandardContains(name)
        }

        let fetchDescriptor = FetchDescriptor<Budget>(predicate: predicate)
        let results: [Budget] = try context.fetch(fetchDescriptor)
        return !results.isEmpty
    }
    
    func save(context: ModelContext) throws {
        
        if try exists(context: context, name: name) {
            throw BudgetError.duplicateName
        }
        
        context.insert(self)
    }
}
```

We’ve added our business logic directly inside the Budget model. At the moment, we have a single rule: to prevent users from adding duplicate budget names to persistent storage. This rule is enforced through the exists function, as shown in the implementation above.

The exists function is marked as private, making it inaccessible from outside the Budget class. However, you can still unit test this logic indirectly by using the publicly available save function, which internally relies on exists to enforce the uniqueness rule.

This approach of embedding business rules directly within your model class keeps things clean and straightforward, without the need to wrap your objects in additional design patterns.

We can also validate the budget limit by ensuring it is greater than zero. The save function has been updated to enforce this rule.

``` swift 
  func save(context: ModelContext) throws {
        
        guard limit > 0 else {
            throw BudgetError.invalidLimit
        }
        
        if try exists(context: context, name: name) {
            throw BudgetError.duplicateName
        }
        
        context.insert(self)
    }
```

Business rules aren’t limited to the Budget class alone—they can also apply to relationships defined within the model. In the example below, we've added a one-to-many relationship between Budget and Expense, meaning a single budget can be associated with multiple expenses.

``` swift 

@Model
class Budget {
    var name: String
    var limit: Double
    
    @Relationship(deleteRule: .cascade)
    var expenses: [Expense] = []

    // other code 
}

@Model
class Expense {
    var name: String
    var amount: Double
    var quantity: Int
    var budget: Budget?
    
    var total: Double {
        amount * Double(quantity)
    }
    
    init(name: String, amount: Double, quantity: Int = 1) {
        self.name = name
        self.amount = amount
        self.quantity = quantity
    }
}

```

Now, we can enhance our Budget class by adding computed properties spent and remaining, which dynamically calculate the total amount spent and the remaining budget based on associated expenses.

``` swift 
@Model
class Budget {
    var name: String
    var limit: Double
    
    @Relationship(deleteRule: .cascade)
    var expenses: [Expense] = []
    
    var spent: Double {
        expenses.reduce(0) { result, expense in
            result + expense.total
        }
    }
    
    var remaining: Double {
        limit - spent
    }

    // other code 
}
```

Properties like total in the Expense model, as well as spent and remaining in the Budget model, represent business logic. As such, it’s important to take the time to write unit tests for them to ensure they behave as expected under various scenarios.

## Should we create separate layer for business logic? 

I tend to avoid introducing extra layers into my application unless they’re absolutely necessary. At this stage, I don’t see a strong need for a separate business logic layer, as the Budget and Expense models are perfectly capable of handling their own logic.

That said, there are certainly cases where creating services, managers, or other abstractions makes sense—particularly when importing data from a JSON API and persisting it to storage. In those scenarios, separating concerns can help keep the codebase clean and maintainable.

Here’s the outline of the Importer, a component responsible for fetching expenses from a JSON API. The imported data can then be persisted into the local database using the SwiftData framework.

``` swift 
import Foundation

struct Importer {
    
    let httpClient: HTTPClient
    
    func importExpenses() -> [Expense] {
        
        // use HTTPClient to fetch the expenses from JSON API
        
        // return the expenses
        return []
    }
}
```

> You may have noticed that `importExpenses` returns the actual SwiftData model rather than a DTO. Whether to return a model or a DTO depends on the specific use case, and I’ll dive deeper into this decision in the **Handling JSON Responses** section of the article.

## What if I want to use a different persistent storage in the future? 

Apple designed SwiftData to integrate seamlessly with SwiftUI—which is evident from the fact that the @Query macro can only be used inside SwiftUI views. While this tight integration offers convenience, it also introduces a major trade-off: coupling your views directly to SwiftData. 

> In SwiftUI, views are the view model. 

Now, imagine you've been building your application for a few months and later decide to switch to a different persistence solution like GRDB or Realm. Because your views rely on SwiftData-specific features like @Model and @Query, migrating becomes difficult. These constructs are unique to SwiftData and don’t translate cleanly to other data persistence frameworks, making your architecture harder to adapt or extend over time.

A possible solution is to create a custom data access layer and define a protocol to abstract away the underlying data provider. In the implementation below, I’ve introduced a `DataAccess` protocol that includes `getBudgets` and `addBudget` functions. Currently, there's a single concrete implementation called `BudgetSwiftDataAccess`, but the architecture allows for easily adding alternative implementations in the future, such as for testing or switching to a different data source.


``` swift 

@MainActor
protocol DataAccess {
    func getBudgets() throws -> [BudgetPlain]
    func addBudget(name: String, limit: Double)
}

@MainActor
class BudgetSwiftDataAccess: DataAccess {
    
    var container: ModelContainer
    var context: ModelContext
    
    @MainActor
    init(container: ModelContainer = ModelContainer.default()) {
        self.container = container
        self.context = container.mainContext
    }
    
    func getBudgets() throws -> [BudgetPlain] {
        let budgets = try context.fetch(FetchDescriptor<Budget>())
        return budgets.map(BudgetPlain.init)
    }
    
    func addBudget(name: String, limit: Double) {
        let budget = Budget(name: name, limit: limit)
        context.insert(budget)
    }
    
}
```

One important thing to note is that the `getBudgets` and `addBudget` functions in the `DataAccess` protocol operate on plain types. They are not tied to any specific persistence framework like SwiftData, Core Data, or Realm. The `BudgetPlain` type is a simple `struct` containing just `name` and `limit` properties.

Each data layer is responsible for converting its specific model into a `BudgetPlain` instance. This ensures that the view layer interacts only with simple, framework-agnostic types, rather than being coupled to specialized models from SwiftData, Core Data, or Realm.

Next, you can access your data access layer in your view through the use of Environment values. This is shown below: 

``` swift 
struct BudgetListScreen: View {
    
    @State private var name: String = ""
    @State private var limit: Double?
    @State private var budgets: [BudgetPlain] = []
    
    @Environment(\.dataAccess) private var dataAccess

    // other code ...
}
```

So, the million dollar question is that should you use APIs provided by SwiftData or should you always create data access layers. There is no right answer to this what-if question—it all comes down to trade-offs. If you choose to use SwiftData’s APIs such as @Query, @Model, #Predicate, and others, you gain tight integration with SwiftUI and a faster development experience. However, you also introduce coupling that makes it harder to switch to another persistence layer down the road.

On the other hand, if you build a more abstract architecture—using plain Swift models, protocol-based repositories, and custom query layers—you’ll gain flexibility and testability, but at the cost of added complexity and boilerplate.

In the end, it’s about choosing the right balance for your project’s scope, longevity, and team preferences.

## Queries 

SwiftData provides the `@Query` macro, which allows you to fetch records directly from persistent storage. Beyond just fetching, `@Query` also automatically triggers a view update whenever a record is added, updated, or deleted—*as long as* a property from that record is used within the view’s body.

This behavior is similar to how `NSFetchedResultsController` works in Core Data, and I wouldn’t be surprised if SwiftData uses it behind the scenes to achieve this reactivity.

The `@Query` macro is only available within SwiftUI views. In our implementation, we've used it inside the `BudgetListScreen`, as shown below:


``` swift 
@Query private var budgets: [Budget] = []
```

When developers see this code, their immediate reaction often centers around *separation of concerns*. The instinct is to push back—arguing that placing data access logic directly in the view is unacceptable.

However, as I mentioned earlier, in SwiftUI, the **view also acts as the view model**. Of course, that doesn’t mean you should start dumping all of your business logic into the view. But it *is* an appropriate place for presentation logic, data transformation, and mapping.

Additionally, separation of concerns still exists—it’s just structured differently. Instead of being divided across layers (like in MVC or MVVM), responsibilities are now **organized by feature or view**. For example, all logic related to listing budgets lives inside `BudgetListScreen`, while everything related to adding a new expense resides in `AddExpenseScreen`.

So, the principle of separation of concerns hasn’t been lost—it has simply **shifted in alignment with SwiftUI’s component-driven architecture**.

Now, you might be thinking about the *massive views* problem. If a view starts getting too large, there’s nothing stopping you from breaking it down into smaller, reusable views. SwiftUI makes it easy to extract and compose views, helping you keep your codebase clean, modular, and maintainable.

Let’s get back to the `@Query` macro. In most cases, it’s recommended to use `@Query` directly inside the view for clarity and simplicity. However, if you prefer, you can also move the query definition into the model itself. This approach helps centralize query logic and can make your views cleaner. Here's an example demonstrating how to implement this:

``` swift 
@Model
class Budget {
    var name: String
    var limit: Double

    static var all: FetchDescriptor<Budget> {
        FetchDescriptor<Budget>()
    }

    // other code ...
}
```

And now, you can use it in the view as shown below: 

``` swift 
struct BudgetListScreen: View {
    
    @State private var name: String = ""
    @State private var limit: Double?
    
    @Environment(\.modelContext) private var context
    
    @Query(Budget.all) private var budgets: [Budget] = []

    // other code 
}
```

This approach also makes it easier to write unit tests for your queries and promotes reusability. If you find yourself needing the same query in multiple places, extracting it into a shared model function is helpful. That said, if you're reusing the **exact same query and presentation**, it may be even better to make the **view itself reusable** so it can be embedded in different parts of your application with minimal duplication.

But what about **dynamic queries**? How do you implement queries in SwiftData that depend on a runtime value? Unfortunately, there’s no straightforward way to change the predicate once `@Query` has been defined.

One approach—demonstrated in [Apple’s sample code](https://developer.apple.com/documentation/swiftui/backyard-birds-sample)—involves creating a **subview** and passing the filter or sort order as a parameter. Inside the subview's initializer, you can then define the `@Query` using that dynamic value. This technique allows for more flexible, runtime-driven queries, as shown below:

``` swift 
struct BudgetListView: View {
    
    let sortOrder: SortOrder
    
    @Query private var budgets: [Budget] = []
    
    init(sortOrder: SortOrder) {
        self.sortOrder = sortOrder
        
        let sortDescriptor = SortDescriptor<Budget>(\.name, order: sortOrder == .asc ? .forward: .reverse)
        _budgets = Query(sort: [sortDescriptor])
    }
    
    var body: some View {
        List(budgets) { budget in
            Text(budget.name)
        }
    }
}
```


The `BudgetListView` encapsulates the logic for displaying sorted budgets. The `@Query` is constructed within the initializer based on the value passed in—such as `sortOrder` in this case. This approach demonstrates how you can implement **dynamic queries** in SwiftData by generating the query at runtime through subview initialization. This technique does require you to break your views based on the dynamic parameter. Depending on the scenario, that can actually be a good thing. Since SwiftUI’s dependency graph is view-based, it will only re-evaluate the views whose input parameters—or dependencies—have changed. This can lead to more efficient UI updates and better performance, especially in complex view hierarchies.

## Testing 

In the last section, we talked about architecture and business logic and how placing the rules right in the SwiftData model class allows us to easily write unit tests. In this section, we are going to implement couple of unit tests for our model classes. 

When writing unit tests, it's important to focus on tests that provide real value and a return on investment. After all, tests are code too—and poorly written or unnecessary tests can add to the complexity of your codebase rather than reduce it.

Take a look at the following unit test. 

``` swift 
  @Test func user_save_budget_successfully() throws {
        
        let budget = Budget(name: "Groceries", limit: 500)
        context.insert(budget)
        
        // fetch the newly saved budget
        
        let fetchDescriptor = FetchDescriptor<Budget>()
        let budgets = try context.fetch(fetchDescriptor)
        
        let savedBudget = budgets[0]
        #expect(savedBudget.name == "Groceries")
        #expect(savedBudget.limit == 500)
    }
```

The test creates an instance of the `Budget` model and uses the SwiftData model context to insert it into the in-memory database. It then verifies the save operation by fetching the model from storage.

While this test may pass, it doesn't provide meaningful value—because it ends up testing the SwiftData framework itself, rather than our custom business logic.

Now, let's take a look at a test that provides much better return on our investment. 

``` swift 
  @Test func throw_exception_when_inserting_budgets_with_duplicate_name() throws {
        
        let budget = Budget(name: "Vacation", limit: 100)
        try budget.save(context: context)
        
        #expect(throws: BudgetError.duplicateName, "Duplicate name exception was not thrown.", performing: {
            
            let anotherBudget = Budget(name: "Vacation", limit: 500)
            try anotherBudget.save(context: context)
            
            // also check that in the database there is only one instance
            let budgets = try context.fetch(Budget.all)
            #expect(budgets.count == 1)
        })
        
    }
```

The test above ensures that an exception is thrown when a user attempts to save a budget with a duplicate name. It also verifies that the storage contains only a single budget record, confirming that duplicates are not persisted.

In similar fashion you can write high quality tests that provide value to your codebase and protect your against regression. 

``` swift 
 @Test func calculate_spent_amount_for_budget() {
        
        let budget = Budget(name: "Vacation", limit: 500)
        
        budget.expenses.append(Expense(name: "Rental car", amount: 200))
        budget.expenses.append(Expense(name: "Airfare", amount: 120))
        
        #expect(budget.spent == 320)
    }
```

In the last section, we talked about architecture and business logic and how placing the rules right in the SwiftData model class allows us to easily write unit tests. In this section, we are going to implement couple of unit tests for our model classes. 

When writing unit tests, it's important to focus on tests that provide real value and a return on investment. After all, tests are code too—and poorly written or unnecessary tests can add to the complexity of your codebase rather than reduce it.

Take a look at the following unit test. 

``` swift 
  @Test func user_save_budget_successfully() throws {
        
        let budget = Budget(name: "Groceries", limit: 500)
        context.insert(budget)
        
        // fetch the newly saved budget
        
        let fetchDescriptor = FetchDescriptor<Budget>()
        let budgets = try context.fetch(fetchDescriptor)
        
        let savedBudget = budgets[0]
        #expect(savedBudget.name == "Groceries")
        #expect(savedBudget.limit == 500)
    }
```

The test creates an instance of the `Budget` model and uses the SwiftData model context to insert it into the in-memory database. It then verifies the save operation by fetching the model from storage.

While this test may pass, it doesn't provide meaningful value—because it ends up testing the SwiftData framework itself, rather than our custom business logic.

Now, let's take a look at a test that provides much better return on our investment. 

``` swift 
  @Test func throw_exception_when_inserting_budgets_with_duplicate_name() throws {
        
        let budget = Budget(name: "Vacation", limit: 100)
        try budget.save(context: context)
        
        #expect(throws: BudgetError.duplicateName, "Duplicate name exception was not thrown.", performing: {
            
            let anotherBudget = Budget(name: "Vacation", limit: 500)
            try anotherBudget.save(context: context)
            
            // also check that in the database there is only one instance
            let budgets = try context.fetch(Budget.all)
            #expect(budgets.count == 1)
        })
        
    }
```

The test above ensures that an exception is thrown when a user attempts to save a budget with a duplicate name. It also verifies that the storage contains only a single budget record, confirming that duplicates are not persisted.

In similar fashion you can write high quality tests that provide value to your codebase and protect your against regression. 

``` swift 
 @Test func calculate_spent_amount_for_budget() {
        
        let budget = Budget(name: "Vacation", limit: 500)
        
        budget.expenses.append(Expense(name: "Rental car", amount: 200))
        budget.expenses.append(Expense(name: "Airfare", amount: 120))
        
        #expect(budget.spent == 320)
    }
```


In the above test, we validated that the `spent` property on the `Budget` instance works as expected. You can apply the same approach to test the `remaining` property and ensure it returns the correct value based on the budget's limit and associated expenses.

By focusing on tests that validate your business logic—like enforcing unique budget names or correctly calculating totals—you’re not just testing code, you’re safeguarding the behavior of your application. These kinds of tests are resilient, easy to understand, and provide long-term value as your app evolves.

Rather than testing what frameworks already guarantee, invest your time in writing unit tests that cover **what your app is responsible for**. This ensures your business rules remain intact, your logic remains sound, and your codebase remains trustworthy.

As you continue building with SwiftData, remember: it's not about testing everything—it's about testing the **right** things.

## Previews 

As one Apple engineer put it during a WWDC session, *“Previews are like tests.”* Previews allow us to quickly visualize the appearance of our user interface without the need to launch the simulator every time we make a change. As an iOS developer, I rely heavily on Xcode Previews and truly believe they make SwiftUI development faster, more efficient, and a lot more enjoyable.

The BudgetListScreen uses the following code to construct a preview. 

``` swift 
#Preview {
    NavigationStack {
        BudgetListScreen()
    }.modelContainer(previewContainer)
}
```

An important detail to note is that for the `modelContainer` argument, we passed in the `previewContainer` property. This property is a specialized instance of `ModelContainer` designed specifically for use in previews. It provides an isolated, in-memory context that allows your SwiftData models to function properly within Xcode Previews without affecting your main app data.

``` swift 
@MainActor
var previewContainer: ModelContainer = {
    
    let container = try! ModelContainer(for: Budget.self, configurations: ModelConfiguration(isStoredInMemoryOnly: true))
    
    for budget in SampleData.budgets {
        container.mainContext.insert(budget)
    }
    
    return container
    
}()

struct SampleData {
    
    static var budgets: [Budget] {
        return [Budget(name: "Groceries", limit: 400), Budget(name: "Vacation", limit: 2000)]
    }
    
}
```


The `previewContainer` also gives you the flexibility to insert default values into the in-memory storage. These values are then used to populate and display meaningful content in the `BudgetListScreen`, making your previews more realistic and useful during development.

There are some scenarios where you need to first fetch data using @Query and then pass an individual item to the screen. This is shown below: 

``` swift 
    BudgetDetailScreen(budget: budgets[0])
```

You may try to write the following preview: 

``` swift 
#Preview {
    
    @Previewable @Query var budgets: [Budget] = []
    
    NavigationStack {
        BudgetDetailScreen(budget: budgets[0])  
    } .modelContainer(previewContainer)
}
```

> @Previewable macro allows you to use dynamic properties inline in previews. 


Unfortunately, the code above will throw an **"index out of range"** exception. This happens because the `previewContainer` has not been properly initialized before attempting to access the `budgets` collection.

One way to resolve this issue is to manually create a **container view**—much like we used to do before `@Query`—and use `@Query` in the **parent view**, then pass the resulting data down to the child view. This approach ensures the data is available when the child view is rendered in the preview.


``` swift 
struct BudgetDetailScreenContainer: View {
    
    @Query(sort: \Budget.name, order: .forward) private var budgets: [Budget]
    
    var body: some View {
        BudgetDetailScreen(budget: budgets[0])
    }
}

#Preview { @MainActor in
    NavigationStack {
        BudgetDetailScreenContainer()
    }.modelContainer(previewContainer)
}
```

We created the `BudgetDetailScreenContainer` specifically to support previews. In the preview provider, we set up the model container with mock data. The `BudgetDetailScreenContainer` then uses `@Query` to fetch items from the in-memory database and passes the relevant data down to `BudgetDetailScreen`. This approach ensures that the preview has access to real data, enabling us to visualize the UI in a meaningful and realistic way.

Another option introduced in the WWDC 2024 session titled [**What’s New in SwiftData**](https://developer.apple.com/videos/play/wwdc2024/10137/) is the use of `PreviewModifier` and `PreviewTrait`. These new tools allow you to inject model data directly into your previews—eliminating the need to create a custom parent container view just for previewing purposes.  

The following example, taken from Apple’s WWDC 2024 sample code, demonstrates how to use `PreviewModifier` and `PreviewTrait` to simplify preview setup:


``` swift 
import SwiftUI
import SwiftData

struct SampleData: PreviewModifier {
    static func makeSharedContext() throws -> ModelContainer {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        let container = try ModelContainer(for: Trip.self, configurations: config)
        Trip.makeSampleTrips(in: container)
        return container
    }
    
    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}


extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleData: Self = .modifier(SampleData())
}

struct ContentView: View {
    @Query
    var trips: [Trip]


    var body: some View {
        ...
    }
}


#Preview(traits: .sampleData) {
    ContentView()
}
```

The PreviewModifier can also be used for previews of views taking a model as parameter:

``` swift 
// Create a preview query using @Previewable

import SwiftUI
import SwiftData

#Preview(traits: .sampleData) {
    @Previewable @Query var trips: [Trip]
    BucketListItemView(trip: trips.first)
}
```

Moving forward, you should prefer using `PreviewModifier` and `PreviewTrait` to provide sample data to your previews. This approach leads to cleaner, more maintainable code by reducing the need for custom container views and boilerplate setup.

Xcode Previews are an essential tool in every SwiftUI developer’s workflow, and with the enhancements introduced in WWDC 2024—such as `PreviewModifier` and `PreviewTrait`—creating rich, data-driven previews is now easier and more scalable than ever. These tools allow you to build realistic previews with minimal effort, making it simpler to catch UI issues early, prototype quickly, and deliver polished interfaces with confidence.

While custom container views still have their place in certain scenarios, embracing `PreviewModifier` and `PreviewTrait` will help streamline your preview setup, improve reusability, and keep your preview code clean and focused.

By making previews an integral part of your development process—and powering them with real, testable data—you’ll not only speed up development but also improve the quality of your SwiftUI apps.

## Handling JSON Responses 

When building SwiftData applications, there are times when you need to fetch data from a remote source and insert it into your local database. In my course, [**Build Gardening App Using SwiftUI & SwiftData**](https://azamsharp.teachable.com/p/build-gardening-app-using-swiftui-swiftdata), I demonstrate this by downloading a list of vegetables from an API and allowing the user to add them to their garden based on their interaction.

In scenarios like this, most developers instinctively create DTO (Data Transfer Object) types to represent the server's response. While this is often a good approach, the need for DTOs really depends on a few important factors.

For instance—what if you control the server and the response format? Or what if the server response is already flat and closely resembles your SwiftData model? In such cases, introducing a separate DTO layer may be unnecessary.

In this section, we’ll explore different use cases to determine when a DTO is beneficial and when it’s reasonable to skip it and map the response directly to your SwiftData model.

Consider a scenario where you're building a vegetable gardening application, and you receive the following response from your own server—one that you have complete control over:

``` swift 
[
  {
    "vegetableId": 1,
    "name": "Carrot",
    "body": "A root vegetable, usually orange in color, rich in beta-carotene."
  },
  {
    "vegetableId": 2,
    "name": "Spinach",
    "body": "A leafy green vegetable high in iron and vitamins."
  },
  {
    "vegetableId": 3,
    "name": "Tomato",
    "body": "A red, juicy fruit often used as a vegetable in cooking."
  }
]
```

The response is quite straightforward and closely aligns with our data model. In this case, we can map the response directly to a SwiftData model without introducing a separate DTO layer. This keeps the code simple and reduces unnecessary abstraction. The implementation of the SwiftData model is shown below:

``` swift 
@Model
class Vegetable: Decodable {

    var vegetableId: Int
    var name: String
    var body: String
    
    private enum CodingKeys: String, CodingKey {
        case vegetableId, name, body
    }
    
    required init(from decoder: any Decoder) throws {
        
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.vegetableId = try container.decode(Int.self, forKey: .vegetableId)
        self.name = try container.decode(String.self, forKey: .name)
        self.body = try container.decode(String.self, forKey: .body)
    }
}
```

But what about if response is not controlled by you and it is nested, complicated or all of the above. In those cases, it is a good idea to implement a DTO and let DTO handle all the mapping etc. Here is one example: 

Below you can find the response from [JSONPlaceHolder /users endpoint](https://jsonplaceholder.typicode.com/users). 

``` json 
[{
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "suite": "Apt. 556",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": {
        "lat": "-37.3159",
        "lng": "81.1496"
      }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
      "name": "Romaguera-Crona",
      "catchPhrase": "Multi-layered client-server neural-net",
      "bs": "harness real-time e-markets"
    }
  }]
```

As you can see, the response is nested across multiple levels. While it's technically possible to map this response directly to our SwiftData `User` model, doing so would introduce a significant amount of boilerplate code into the model itself. Additionally, the structure of our model may differ from the structure of the actual API response.  

A better approach is to use dedicated DTO (Data Transfer Object) types to handle the JSON decoding. These lightweight structures allow us to cleanly map the API response and then convert the data into our SwiftData model as needed.

The implementation of the DTO objects is shown below:

``` swift 
import Foundation

struct GeoDTO: Codable {
    let lat: String
    let lng: String
}

struct AddressDTO: Codable {
    let street: String
    let suite: String
    let city: String
    let zipcode: String
    let geo: GeoDTO
}

struct CompanyDTO: Codable {
    let name: String
    let catchPhrase: String
    let bs: String
}

struct UserDTO: Codable {
    let id: Int
    let name: String
    let username: String
    let email: String
    let address: AddressDTO
    let phone: String
    let website: String
    let company: CompanyDTO
}

```

And the implementation of SwiftData model `User` is shown below: 

``` swift 
import Foundation
import SwiftData

@Model
class User {
    @Attribute(.unique) var id: Int
    var name: String
    var username: String
    var email: String
    var phone: String
    var website: String
    var address: String
    var company: String

    init(id: Int, name: String, username: String, email: String, phone: String, website: String, address: String, company: String) {
        self.id = id
        self.name = name
        self.username = username
        self.email = email
        self.phone = phone
        self.website = website
        self.address = address
        self.company = company
    }

    static func fromDTO(_ dto: UserDTO) -> User {
        let addressString = "\(dto.address.street), \(dto.address.suite), \(dto.address.city), \(dto.address.zipcode)"
        let companyString = dto.company.name
        return User(
            id: dto.id,
            name: dto.name,
            username: dto.username,
            email: dto.email,
            phone: dto.phone,
            website: dto.website,
            address: addressString,
            company: companyString
        )
    }
}

```

As you can see, the decision to use a DTO depends heavily on the complexity and ownership of the data source. When the server response is flat and predictable—as in the case of our gardening app—you can safely map the data directly to your SwiftData model and avoid unnecessary layers.

However, when working with more complex or nested responses—especially from third-party APIs—it’s often best to introduce DTOs. They help isolate the decoding logic, keep your models clean, and make your application more maintainable and testable.

Ultimately, there's no one-size-fits-all answer. The key is to evaluate each situation carefully and choose the approach that keeps your codebase simple, scalable, and easy to reason about.

## CloudKit 

One of the key benefits of using **SwiftData** with **SwiftUI** is its seamless integration with **CloudKit**. In most cases, no additional configuration is needed—your local SQLite database will automatically sync with the user’s **private CloudKit database**.

> Note: At the time of this writing, SwiftData only supports syncing with the user's **private** CloudKit database.

Most CloudKit-related issues are surfaced in the **Xcode output window**, which can help guide you in resolving them. The most common problems typically include not being signed into iCloud on the device or simulator, or failing to provide **default or optional values** for properties in your SwiftData models.

As shown in the example below, I’ve provided default values for all model properties and marked relationships as optional to ensure smooth syncing with CloudKit:


``` swift 
@Model
class Budget {
    
    var name: String = ""
    var limit: Double = 0.0
    @Relationship(deleteRule: .cascade)
    var expenses: [Expense]? = []
```


Additionally, if you're integrating with CloudKit, **you cannot use unique constraints** (e.g., `#Unique<Budget>([\.name])`) on your model’s properties. CloudKit does not support enforcing uniqueness at the schema level, and including such constraints in your SwiftData model will result in sync failures.

One other interesting thing to note about ...

The following code works perfectly fine on a single device with CloudKit integration. Unfortunately, when tested on multiple devices for CloudKit sync then it becomes clear that the changes are not being propagated and the view is not getting refreshed. 

``` swift 

import SwiftUI
import SwiftData

struct BudgetDetailScreen: View {
    
    @Bindable var budget: Budget
    @Environment(\.modelContext) private var context
    
    var body: some View {
        
        VStack {

           // other code
            
            Form {
                Section("Budget") {
                     TextField("Budget name", text: $budget.name)
                     TextField("Budget limit", value: $budget.limit, format: .currency(code: Locale.currencyCode))
                }
                
                Section("Expenses") {
                    List {
                        
                        ForEach(budget.expenses) { expense in
                            ExpenseCellView(expense: expense)
                        }.onDelete(perform: deleteExpense)
                    }
                }
                
            }.navigationTitle(budget.name)
        }
    }
}

```

CloudKit sends **silent push notifications** to notify the app of changes, but these do not automatically trigger view updates. That’s because the tracked entity—in this case, `budget`—hasn’t technically changed. Since `Budget` is a reference type and still points to the same memory location, SwiftUI doesn’t see it as modified, and therefore the view isn’t re-rendered. As a result, properties like `budget.expenses` are not re-evaluated when new data is synced from CloudKit.

One effective way to solve this issue is by tracking **expenses directly** using the `@Query` macro. Instead of accessing expenses through the `budget` model, you can construct and execute a dynamic `@Query` to fetch all relevant expenses. This allows SwiftUI to re-render when the underlying expense data changes, ensuring that your UI stays in sync with the latest CloudKit updates.

The implementation is shown below: 

``` swift 
struct BudgetDetailScreen: View {
    
    @Bindable var budget: Budget
    @Environment(\.modelContext) private var context
    @Query private var expenses: [Expense] = []
    
    init(budget: Budget) {
        self.budget = budget
        let budgetId = self.budget.persistentModelID
        
        let predicate = #Predicate<Expense> {
            if let budget = $0.budget {
                return budget.persistentModelID == budgetId
            } else {
                return false
            }
        }
        
        _expenses = Query(filter: predicate)
    }
```

A custom initializer has been added to `BudgetDetailScreen`, which is responsible for constructing a dynamic `@Query`. This query fetches all expenses associated with the provided budget. By using `@Query`, SwiftUI is able to **track changes** to the underlying expense data, allowing the view to re-render correctly when updates—such as CloudKit syncs—occur. The updated implementation is shown below:

``` swift 
Section("Expenses") {
                    List {
                        
                        // use expenses instead of budget.expenses 
                        ForEach(expenses) { expense in
                            ExpenseCellView(expense: expense)
                        }.onDelete(perform: deleteExpense)
                    }
                }
```


By shifting from `budget.expenses` to a dynamic `@Query`, we ensure that SwiftUI is properly notified when new data arrives—whether it's created locally or synced silently from CloudKit. This approach not only solves the view refresh issue but also aligns well with SwiftData’s design, where `@Query` plays a central role in tracking changes and driving UI updates.

While CloudKit integration with SwiftData feels seamless on the surface, subtle nuances—like view re-rendering and data observation—require careful handling. Leveraging tools like dynamic `@Query` helps you build more robust, reactive interfaces that stay in sync across devices and platforms.

As SwiftData continues to evolve, we can expect improvements and more predictable syncing behavior. Until then, strategies like these can help you bridge the gap and deliver a smooth user experience with real-time data updates.

## Conclusion 

SwiftData brings a fresh, modern approach to data persistence for SwiftUI apps, streamlining everything from model definition to syncing with CloudKit. With features like `@Model`, `@Query`, and seamless integration into the SwiftUI lifecycle, SwiftData simplifies what used to be a complex part of iOS development.

Throughout this article, we explored a practical architecture for building SwiftData applications—starting with modeling your data, incorporating business rules, handling JSON responses, writing meaningful tests, and creating efficient, real-world previews. We also covered how to structure your code in a way that remains flexible, testable, and maintainable—whether you're building a side project or scaling a production app.

SwiftData works best when you embrace its strengths, but it's also important to be mindful of its limitations—especially around flexibility and framework coupling. Whether you choose to build tightly around SwiftData or abstract your persistence layer behind protocols, the key is making thoughtful decisions that align with your project’s goals.

In the end, a well-architected SwiftData app is one that balances simplicity with structure, leverages previews and unit tests to catch issues early, and keeps the business logic close to where it matters most—your models.

Thanks for reading. I hope this article helps you build better, cleaner, and more enjoyable SwiftUI applications using SwiftData.

<div style="font-family: Arial, sans-serif; line-height: 1.6; max-width: 600px; margin: 20px auto; padding: 20px; border: 1px solid #ddd; border-radius: 10px;">
  <h2 style="color: #2c3e50; text-align: center;">🚀 Interested in Learning SwiftData?</h2>
  <p style="font-size: 16px; color: #333; text-align: center;">Check out my top-rated courses and live workshop below:</p>

  <ul style="list-style-type: none; padding-left: 0;">
    <li style="margin: 10px 0;">
      <a href="https://azamsharp.teachable.com/p/swiftdata-build-apple-reminders-clone" target="_blank" style="text-decoration: none; color: #2980b9; font-weight: bold;">
        📌 Build Reminders App Clone Using SwiftData
      </a>
    </li>
    <li style="margin: 10px 0;">
      <a href="https://azamsharp.teachable.com/p/swiftdata-bootcamp" target="_blank" style="text-decoration: none; color: #2980b9; font-weight: bold;">
        🧠 SwiftData Bootcamp - A Comprehensive Guide to Building Data Driven Applications
      </a>
    </li>
    <li style="margin: 10px 0;">
      <a href="https://azamsharp.teachable.com/p/build-gardening-app-using-swiftui-swiftdata" target="_blank" style="text-decoration: none; color: #2980b9; font-weight: bold;">
        🌿 Build Gardening App Using SwiftUI & SwiftData
      </a>
    </li>
    <li style="margin: 10px 0;">
      <a href="https://azamsharp.teachable.com/p/swiftdata-fundamentals-workshop" target="_blank" style="text-decoration: none; color: #2980b9; font-weight: bold;">
        🎓 SwiftData Fundamentals Live Workshop
      </a>
    </li>
  </ul>
</div>


