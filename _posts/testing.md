
# Global Sheets Pattern in SwiftUI   

SwiftUI’s ```sheet``` view modifier provides a straightforward way to present modals, but when your app requires multiple sheets across different screens, managing them can become cumbersome. Typically, developers resort to using multiple sheet modifiers or controlling the display with an enum. However, this approach often leads to redundant code scattered across various views.

What if you could centralize the management of sheets, offering a cleaner, more maintainable solution with a user-friendly API?

In this article, we’ll explore different patterns for displaying and managing sheets in SwiftUI. We'll also look at how adopting ideas from other platforms can enhance your app’s architecture, making sheet management more efficient and scalable.

### Displaying a Basic Sheet  

There are various ways of displaying sheets in SwiftUI. The simplest approach is to use the ```isPresented``` argument to control when the sheet is shows. In the code below we are displaying a sheet when the ```isPresented``` property is changed.       

``` swift 
struct ContentView: View {
    
    @State private var isPresented: Bool = false
    
    var body: some View {
        Button("Show Sheet") {
            isPresented = true
        }.sheet(isPresented: $isPresented, content: {
            Text("Sheet")
        })
    }
}
```

If you want to display multiple sheets then you can use multiple sheet modifiers, dependent on separate properties. This is shown in the implementation below:  

``` swift 
struct ContentView: View {
    
    @State private var isPresented: Bool = false
    @State private var isChatPresented: Bool = false
    
    var body: some View {
        VStack {
            Button("Show Sheet") {
                isPresented = true
            }
            Button("Show Chat") {
                isChatPresented = true
            }
        }.sheet(isPresented: $isPresented, content: {
            Text("Sheet")
        })
        .sheet(isPresented: $isChatPresented, content: {
            Text("Chat Presented")
    })
    }
}
```

Managing multiple sheets can become cumbersome over time, even if the current approach works. To simplify this complexity, you can introduce a private enum that encapsulates all the sheet types that can be presented within a screen. This allows for cleaner, more organized code and makes it easier to manage and extend the available sheet types.

### Enum Based Sheets

Instead of using multiple sheet view modifiers, we can make use of an enum with all the different options for a particular screen. The enum can be set to private since it will be for a particular screen. This is shown below: 

``` swift 
struct CustomerListScreen: View {
    
    private enum Sheet: Identifiable {
        
        case add
        case update(Customer)
        
        var id: String { String(describing: self) }  
    }
}
```

The `Sheet` enum conforms to the `Identifiable` protocol to participate in binding operations, ensuring each case can be uniquely identified within SwiftUI views. Now we can start using our Sheet enum as shown below: 

``` swift 
struct CustomerListScreen: View {
    
    // Sheet definition 
    
    @State private var activeSheet: Sheet?
    
    var body: some View {
        VStack {
            
            Button("Add Customer") {
                activeSheet = .add
            }
            
            Button("Update Customer") {
                let customer = Customer(name: "John Doe")
                activeSheet = .update(customer)
            }
        }.sheet(item: $activeSheet) { activeSheet in
            switch activeSheet {
                case .add:
                    Text("Add Sheet")
                case .update(let customer):
                    Text("Updating \(customer.name)")
            }
        }
    }
}
```

Instead of managing multiple sheets, we're now using a single sheet modifier that binds to the ```activeSheet``` enum. When the value of ```activeSheet``` changes, the sheet modifier is automatically triggered. This approach allows you to dynamically return the view associated with the active sheet, streamlining the process and reducing the complexity of handling multiple sheets.

Although this works but we still end up with multiple sheet modifiers for each screen. This is not a problem per se but maybe there is a room for improvement. 

### Global Sheets 

React Hooks are special functions in React that allow you to use state and other React features in functional components, which were previously only available in class components.

One way to implement a similar behavior is by using custom Environment values in SwiftUI. This will allow us to reach a the following result. 

``` swift 
struct ContentView: View {
    
    @Environment(\.showSheet) private var showSheet
    
    var body: some View {
        VStack {
            Button("Show Settings Screen") {
                showSheet(.settings)
            }
            
            Button("Show Contact Screen") {
                showSheet(.contact("John Doe"))
            }
        }
        .padding()
    }
}
```

Don't worry the above code will not work right now since there is no such thing called ```showSheet```. 

Now that we have a clear vision of our final implementation, we can focus on the steps needed to achieve it.

We will start by creating a custom Environment value. 




