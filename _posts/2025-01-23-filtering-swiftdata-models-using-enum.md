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

# Filtering SwiftData Models Using Enum 
 <div class="share-container">
    <div>
      <!-- Twitter -->
      <a href="https://twitter.com/intent/tweet?url=https://azamsharp.com/2025/01/23/filtering-swiftdata-models-using-enum.html&text=Filtering SwiftData Models Using Enum by @azamsharp"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button twitter">
        Share on Twitter
      </a>
    </div>
    <div>
      <!-- LinkedIn -->
      <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://azamsharp.com/2025/01/23/filtering-swiftdata-models-using-enum.html"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button linkedin">
        Share on LinkedIn
      </a>
    </div>
    
  </div>

# Introduction

Enums are a powerful feature in Swift, allowing developers to define a type with a fixed set of values. When working with `SwiftData`, Apple’s modern persistence framework designed to replace Core Data, integrating enums can simplify code and improve clarity. However, `SwiftData` introduces certain limitations when it comes to querying models using enums directly.

In this article, we’ll explore how to filter `SwiftData` models using an enum. Starting with an example scenario of filtering expense items by type, we’ll address the challenges and limitations of using enums with `SwiftData`, and provide a step-by-step solution to overcome these obstacles. Along the way, we’ll demonstrate how to maintain the advantages of enums, like type safety and computed properties, while ensuring compatibility with `SwiftData`.

By the end of this post, you’ll have a solid understanding of how to work around `SwiftData`’s limitations and implement dynamic filtering using enums effectively. Let’s dive in!


<div style="
    background-color: #f0f8ff;
    border-left: 5px solid #0073e6;
    padding: 20px;
    border-radius: 5px;
    font-family: Arial, sans-serif;
    font-size: 1.1rem;
    color: #333;
    margin: 20px 0;
">
    <strong>Want to become a highly valued iOS developer?</strong> 
    Check out AzamSharp School for comprehensive courses and hands-on learning at 
    <a href="https://azamsharp.school" style="color: #0073e6; text-decoration: none; font-weight: bold;">azamsharp.school</a>.
</div>

### Implementation 

Imagine a scenario where you need to filter expense items by their type. While expense types can vary, for this example, we'll focus on two categories: business and personal. Below is the implementation of the ExpenseItem and ExpenseType structures:

``` swift 
enum ExpenseType: Int, Codable, CaseIterable, Identifiable {
    case business, personal

    var id: Self { self }
    
    var title: String {
        switch self {
        case .business:
            return "Business"
        case .personal:
            return "Personal"
        }
    }
}

@Model
class ExpenseItem: Identifiable {
    var name: String
    var type: ExpenseType
    
    init(name: String, type: ExpenseType) {
        self.name = name
        self.type = type
    }
}
```

The `ExpenseType` enum conforms to `Codable`, as required for inclusion in a SwiftData model.

Next, we define a `previewContainer` to supply data for SwiftUI previews. The implementation is shown below:


``` swift 
@MainActor
var previewContainer: ModelContainer = {
    
    let container = try! ModelContainer(for: ExpenseItem.self, configurations: ModelConfiguration(isStoredInMemoryOnly: true))
    
    for expenseItem in SampleData.expenseItems {
        
        container.mainContext.insert(expenseItem)
        try! container.mainContext.save()

    }
    
    return container
    
}()

struct SampleData {
    
    static var expenseItems: [ExpenseItem] {
        return [
            ExpenseItem(name: "Office Supplies", type: .business),
            ExpenseItem(name: "Groceries", type: .personal),
            ExpenseItem(name: "Client Lunch", type: .business),
            ExpenseItem(name: "Movie Ticket", type: .personal),
            ExpenseItem(name: "Software Subscription", type: .business),
            ExpenseItem(name: "Gasoline", type: .personal),
            ExpenseItem(name: "Conference Fee", type: .business),
            ExpenseItem(name: "Dinner Out", type: .personal),
            ExpenseItem(name: "Marketing Materials", type: .business),
            ExpenseItem(name: "Clothing", type: .personal),
            ExpenseItem(name: "Travel Expenses", type: .business),
            ExpenseItem(name: "Household Repairs", type: .personal),
            ExpenseItem(name: "Consulting Fees", type: .business),
            ExpenseItem(name: "Entertainment", type: .personal),
            ExpenseItem(name: "Equipment Purchase", type: .business),
            ExpenseItem(name: "Personal Care", type: .personal),
            ExpenseItem(name: "Training Course", type: .business),
            ExpenseItem(name: "Hobbies", type: .personal),
            ExpenseItem(name: "Legal Fees", type: .business),
            ExpenseItem(name: "Gifts", type: .personal)
        ]

    }
    
}
```

Next, we implemented the user interface, which includes a segmented control for selecting the expense type and a list that dynamically displays expenses based on the selected type.

``` swift 
struct ContentView: View {
    
    @State private var selectedExpenseType: ExpenseType = .business
    
    var body: some View {
        VStack {
            Picker("Select expense type", selection: $selectedExpenseType) {
               
                    ForEach(ExpenseType.allCases) { expenseType in
                        Text(expenseType.title)
                            .tag(expenseType)
                    }
               
            }.pickerStyle(.segmented)

            ExpenseItemSearchResults(expenseType: selectedExpenseType)
            
        }
        .padding()
    }
}
```

The `ExpenseItemSearchResults` class handles dynamically fetching expense items based on the user's selection. Here's the implementation:

``` swift 
struct ExpenseItemSearchResults: View {
    
    let expenseType: ExpenseType
    
    @Query private var expenseItems: [ExpenseItem]
    
    init(expenseType: ExpenseType) {
        self.expenseType = expenseType
        _expenseItems = Query(filter: #Predicate { $0.type == expenseType })
    }
    
    var body: some View {
        List(expenseItems) { expenseItem in
            HStack {
                Text(expenseItem.name)
                Spacer()
                Text(expenseItem.type.title)
            }
        }
    }
}
```

The core of this dynamic query lies in the initializer, where we assign `_expenseItems` to a new query that updates based on the user's selection. However, if you run this code as is, you'll notice it displays an empty list.

> This behavior is a limitation of SwiftData, as it does not inherently recognize the need to map the `rawValue` of the `expenseType` enum to its corresponding database column.


The solution is to use the rawValue of the `ExpenseType` instead of the enum itself. 

The updated implementation is shown below:

``` swift 
@Model
class ExpenseItem: Identifiable {
    var name: String
    var type: Int
    
    var expenseType: ExpenseType {
        ExpenseType(rawValue: type) ?? .business
    }
    
    init(name: String, expenseType: ExpenseType) {
        self.name = name
        self.type = expenseType.rawValue
    }
}
```

We retained the `expenseType` enum as a computed property in the `ExpenseItem` model. This approach allows seamless access to properties like `title`, which are part of the `ExpenseType` enum.

```ExpenseItemSearchResults``` is also updated as shown below: 

``` swift 
struct ExpenseItemSearchResults: View {
    
    let expenseType: ExpenseType
    
    @Query private var expenseItems: [ExpenseItem]
    
    init(expenseType: ExpenseType) {
        self.expenseType = expenseType
        _expenseItems = Query(filter: #Predicate { $0.type == expenseType.rawValue })
    }
    
    var body: some View {
        List(expenseItems) { expenseItem in
            HStack {
                Text(expenseItem.name)
                Spacer()
                Text(expenseItem.expenseType.title)
            }
        }
    }
}
```

As you can see that now the ```#Predicate``` is based on the ```Int``` properties. This allows the predicate to work as expected and display filtered items based on the user selection. 


### Conclusion 

Enums in Swift provide an elegant way to define and manage a fixed set of values, making them an ideal choice for representing data types like `ExpenseType`. However, when using `SwiftData`, certain limitations arise, such as its inability to directly map an enum’s `rawValue` to the underlying database column during queries.

In this article, we explored these challenges and demonstrated a practical solution by replacing the enum property in the model with its `rawValue` while retaining the enum as a computed property. This approach allows us to leverage the benefits of enums, such as readability and type safety, while ensuring seamless integration with `SwiftData`.

By combining this solution with dynamic filtering and SwiftUI’s declarative design, we created an interactive and user-friendly interface for displaying filtered results. This method not only resolves the compatibility issues with `SwiftData` but also maintains clean and efficient code.

As SwiftData continues to evolve, we can hope for better support for enums in the future. Until then, the techniques demonstrated here provide a reliable way to work around its current limitations. With this knowledge, you’re well-equipped to build robust and dynamic data-driven applications in Swift.
