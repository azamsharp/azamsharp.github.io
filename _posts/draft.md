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




### Business Rules 




## Debugging 

## Testing 

## Previews 

## Serialization 

## Conclusion 