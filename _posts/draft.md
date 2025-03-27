# SwiftData Architecture – Patterns and Practices 

SwiftData, introduced at WWDC 2023, is a modern, Swift-native framework built to simplify data persistence in SwiftUI applications. Designed as a successor to Core Data, it offers a more declarative and intuitive approach tailored to Swift developers.

When it comes to app architecture, there’s no shortage of opinions. Developers often debate how to strike the right balance between flexibility, simplicity, and long-term maintainability. In this article, I’ll walk you through the architecture I personally use for building SwiftData-powered apps—an approach that prioritizes clarity, simplicity, and ease of reasoning.

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

To ensure that each budget has a unique name—meaning no two budgets can share the same name—we need to introduce a validation step. While SwiftData provides the #Unique macro for enforcing uniqueness, it doesn't throw an error or offer a clean way to handle violations when the rule is broken.

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

> You may have noticed that importExpenses returns the actual SwiftData model rather than a DTO. Whether to return a model or a DTO depends on the specific use case, and I’ll dive deeper into this decision in the Serialization section of the article.

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

But what about dynamic queries? How will you implement queries in SwiftData that depends on a value.

## Testing 

## Previews 

## Serialization 

## CloudKit 

## Conclusion 