# Embracing Core Data in SwiftUI 

Core Data framework allows the user to persist an object tree to several different stores including SQLite, XML, binary and in-memory. SwiftUI team has worked really hard to make sure that SwiftUI works nicely with Core Data. In this post, we will be building a small budget app using SwiftUI and Core Data. We will discuss how to use @FetchRequest property wrapper and how to setup one-to-many relationship in Core Data. 

>> The complete app is part of my course [MV Design Pattern in iOS - Build SwiftUI Apps Apple's Way](https://www.udemy.com/course/mv-design-pattern-in-ios-for-swiftui/?couponCode=780EABCF3E4DC632EC9F)

## Setting Up the Core Data Stack 

The first step in using Core Data is to setup the Core Data stack. This means you have a basic Data Model file ready, which can be loaded using APIs provided by Core Data. In the implementation below you can see the CoreDataManager, which is responsible for setting up the Core Data stack. 

>> You can download the complete project [here](https://github.com/azamsharp/BudgetApp) and check out the Core Data model file. 

The implementation of CoreDataManager is shown below: 

```swift 
import Foundation
import CoreData

class CoreDataManager {
    
    static let shared = CoreDataManager()
    let persistentContainer: NSPersistentContainer
    
    init() {
        persistentContainer = NSPersistentContainer(name: "BudgetDataModel")
        persistentContainer.loadPersistentStores { description, error in
            if let error {
                fatalError("Unable to load Core Data Model (\(error))")
            }
        }
    }
    
}
```

Once, the Core Data stack has been initialized you can inject the NSManagedObject into the @Environment of the app. 

>> This step is very important because the SwiftUI property wrappers like @FetchRequest, @SectionedFetchRequest will look for the viewContext inside the @Environment. If they don't find a value for managedObjectContext then your code will not work as expected. 

The implementation is shown below: 

```swift 
@main
struct BudgetApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView().environment(\.managedObjectContext, CoreDataManager.shared.persistentContainer.viewContext)
        }
    }
}
```

Now, you are ready to use Core Data inside your View. 

>> View in SwiftUI is also a View Model. Anytime in this article we refer to a view we are talking about a SwiftUI view which is also a view model. Check out the articles discussed in resources section below to learn more about it. 

## Displaying Budget Categories 

SwiftUI includes few property wrappers, which makes it super easy to retrieve information through Core Data and display it on the screen. One of those property wrappers is @FetchRequest. @FetchRequest allows you to perform a query from right within the view and then display the information on the screen. 

>> @FetchRequest property wrapper is only available inside the View

Here is a simple example that shows how to perform a request to fetch all budget categories from the database. 

``` swift 
@FetchRequest(sortDescriptors: [SortDescriptor(\.title)]) private var budgetCategoriesResults: FetchedResults<BudgetCategory>
```

The FetchedResults represents a collection of results retrieved from the Core Data store. The main purpose of FetchedResults is to display the results in a view. This is main reason that it conforms to the RandomAccessCollection protocol. 

You can use FetchedResults directly in the view as shown below: 
```swift 
    @FetchRequest(sortDescriptors: [SortDescriptor(\.title)]) private var budgetCategoriesResults: FetchedResults<BudgetCategory>
    
    var body: some View {
        NavigationStack {
            List(budgetCategoryResults) { budgetCategory in
                

                    HStack {
                        Text(budgetCategory.title ?? "")
                        Spacer()
                        Text(budgetCategory.amount as NSNumber, formatter: NumberFormatter.currency)
                    }
                
               
            }
        }
    }
```

As you can see with only few lines of code we were able to display the budget categories on the view. 

>> @FetchRequest will only work if it can find the managedObjectContext in the @Environment. This was covered earlier in this post. 

One of the main objection of using @FetchRequest inside a view is that by doing so, it makes it difficult to reuse the same request in other views. Although in this app we don't plan to reuse the same request again but if we did we can always move it to a separate file. This is shown in the implementation below. 

```swift 
@objc(BudgetCategory)
public class BudgetCategory: NSManagedObject {
    
    static var all: NSFetchRequest<BudgetCategory> {
        let request = BudgetCategory.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "title", ascending: true)]
        return request
    }
}
```

We used the "Category/Extension" option for "Codegen" in Xcode for the data model file. This puts all the properties in category or extension and give us control of the entity class. For BudgetCategory we have added the ```all``` function, which returns the ```NSFetchRequest<BudgetCategory>```. Now, we can easily use this in any view we want as shown below. 

``` swift 
 @FetchRequest(fetchRequest: BudgetCategory.all) private var budgetCategoryResults: FetchedResults<BudgetCategory>
```

## Adding Transactions for Budget Category 

Each budget category will consists of a list of transactions. The relationship between a budget category and transaction is one to many. This relationship is created in the Core Data data model designer as shown below: 

![Core Data Model Diagram](/images/core-data-relationship.png)

When a user selects a budget, we want to display all the transactions associated with that budget. We also want the user to add transactions to a selected budget. 

### Saving a Transaction 

In order to save a transaction, we need to know the budget category. In our application budget category is passed to the BudgetDetailView, where the transactions can be added and viewed. The BudgetDetailView requires that the budget category is passed as an argument. 

```swift 
struct BudgetDetailView: View {
    
    @Environment(\.managedObjectContext) private var viewContext
    @ObservedObject var budgetCategory: BudgetCategory
}
```

One thing to note is the usage of ```@ObservedObject```. The ObservedObject is going to make sure that the BudgetDetailView is reevaluated and rerendered (if necessary), when the budget category changes. 

Once the user has filled out the transaction details, they can press the save button to add the transaction to the existing budget category. The implementation of the saveTransaction is shown below: 

```swift 
 private func saveTransaction() {
       
        let transaction = Transaction(context: viewContext)
        transaction.title = title
        transaction.amount = Double(amount)!

        budgetCategory.addToTransactions(transaction)
        
        try? viewContext.save()
}
```

The ```addToTransactions``` method is added by Core Data, which allows you to add a transaction to a budget category. 

## Displaying Transactions 

The next step is to display all the transactions associated with the budget category. Due to one to many relationship between budget category and transaction, there is already a transactions property on the budget object. Unfortunately, the transactions property is of NSSet type, which does not conform to RandomAccessCollection. 

```swift 
 List(budgetCategory.transactions) { transaction in
                Text(transaction.title)
 }

 Initializer 'init(_:rowContent:)' requires that 'NSSet?' conform to 'RandomAccessCollection'
```

We can solve this problem by converting our NSSet to an array and iterating over the array. In the code below we have added the transactionsArray function to our existing BudgetCategory class. 

```swift 
 var transactionsArray: [Transaction] {
        
        guard let transactions = transactions else { return [] }
        return transactions.compactMap { $0 as? Transaction }
    }
```

Now we can display transactions in our view. 

```swift 
  List(budgetCategory.transactionsArray) { transaction in
                Text(transaction.title ?? "")
    }
```

If you add a transaction, it will instantly show up on the screen. Although this works, we can refactor the displaying of transactions into a separate view called TransactionListView. This is implemented below: 

```swift 
struct TransactionListView: View {
    
    let transactions: [Transaction]
    
    var body: some View {
        List(transactions) { transaction in
            Text(transaction.title ?? "")
        }
    }
}
```

You can now use the TransactionListView inside your BudgetView as shown below: 

```swift 
 TransactionListView(transactions: budgetCategory.transactionsArray)
```

Unfortunately, you will notice that when the transaction list will not get updated when you add a new transaction. 

>> TransactionListView is not rerendered again with the updated list of transactions because budgetCategory is an instance of BudgetCategory class, which is a reference type. So, from SwiftUI point of view, the instance is still pointing to the same location in memory and hence does not needs to be rerendered. 

One way to solve this problem is by passing a FetchRequest to the TransactionListView. The FetchRequest will be invoked when the BudgetDetailView is (r)evaluated. The request will get all the transactions based on the selected budget category and then displays them on the screen. This is shown in the implementation below: 

``` swift 
struct TransactionListView: View {
    
    @FetchRequest var transactions: FetchedResults<Transaction>
    
    init(request: NSFetchRequest<Transaction>) {
        _transactions = FetchRequest(fetchRequest: request)
    }
    
    var body: some View {
        List {
            ForEach(transactions) { transaction in
                HStack {
                    Text(transaction.title ?? "")
                    Spacer()
                    Text(transaction.amount as NSNumber, formatter: NumberFormatter.currency)
                }
            }
        }
    }
}
```

Inside the BudgetDetailView, we call the TransactionListView as shown below: 

```swift 
  TransactionListView(request: budgetCategory.transactionsByCategoryRequest)
```

The implementation of ```transactionsByCategoryRequest``` is inside the BudgetCategory class is shown below: 

```swift 
 lazy var transactionsByCategoryRequest: NSFetchRequest<Transaction> = {
        let request = Transaction.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "title", ascending: true)]
        request.predicate = NSPredicate(format: "budgetCategory = %@", self)
        return request
    }()
```

You can even expose this property as a static function like shown below: 

```swift 
 static func transactionsByCategoryRequest(_ category: BudgetCategory) -> NSFetchRequest<Transaction> {
        let request = Transaction.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "title", ascending: true)]
        request.predicate = NSPredicate(format: "budgetCategory = %@", category)
        return request
    }
```
## Source Code 
- [Download the complete source code](https://github.com/azamsharp/BudgetApp)

## Resources

- [MV Design Pattern - Build SwiftUI Apps Apple's Way](https://www.udemy.com/course/mv-design-pattern-in-ios-for-swiftui/?referralCode=4627986F77F533DEF0C7)
- [Core Data in iOS](https://www.udemy.com/course/core-data-in-ios/?referralCode=F87F4552453DA9E776FE)
- [@FetchRequest in SwiftUI](https://youtu.be/AhT1dJM8WvY)
- [@SectionedFetchRequest in SwiftUI](https://youtu.be/HMt9R4tYkXY)
- [SwiftUI View is also a View Model](https://azamsharp.com/2022/07/21/view-is-the-view-model.html)

## Conclusion 

Apple engineers have done a lot of work to make Core Data work seamlessly with SwiftUI framework. Property wrappers like @FetchRequest and @SectionedFetchRequest are optimized to work with SwiftUI framework and they are only available in the View (View is also a ViewModel in SwiftUI). 

I hope you enjoyed this article! 
