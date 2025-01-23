# Filtering SwiftData Models Using Enum 

SwiftData is Apple’s modern on-device persistence framework, designed to replace Core Data. As a relatively new addition, it still has some bugs and limitations. In this post, I’ll address one of the most frequently asked questions about SwiftData: *How can I perform a query using an enum?*

<div class="share-container">
    <div>
      <!-- Twitter -->
      <a href="https://twitter.com/intent/tweet?url=https://azamsharp.com/2024/12/18/the-ultimate-guide-to-validation-patterns-in-swiftui.html&text=The Ultimate Guide to Validation Patterns in SwiftUI by @azamsharp"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button twitter">
        Share on Twitter
      </a>
    </div>
    <div>
      <!-- LinkedIn -->
      <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://azamsharp.com/2024/12/18/the-ultimate-guide-to-validation-patterns-in-swiftui.html"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button linkedin">
        Share on LinkedIn
      </a>
    </div>
    
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


The solution is to replace the `ExpenseType` property in the `ExpenseItem` model with an `Int` to directly store the raw value of the enum. The updated implementation is shown below:


Let me know if you'd like the example code included as well!

``` swift 
@Model
class ExpenseItem: Identifiable {
    var name: String
    var type: Int
    
    var expenseType: ExpenseType {
        ExpenseType(rawValue: type) ?? .business
    }
    
    init(name: String, type: Int) {
        self.name = name
        self.type = type
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


