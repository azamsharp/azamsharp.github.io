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




## Debugging 

## Testing 

## Previews 

## Serialization 

## Conclusion 