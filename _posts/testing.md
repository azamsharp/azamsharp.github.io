
# Global Sheets Pattern in SwiftUI   

Managing sheet presentations in SwiftUI can quickly become complex, especially when your app requires multiple sheets across different screens. The default `sheet` view modifier provides a simple way to present modals, but as your app grows, using multiple sheet modifiers or manually controlling their display can lead to scattered, redundant code that’s hard to maintain.

What if there was a way to centralize sheet management, creating a cleaner, more maintainable solution with a user-friendly API? In this article, we’ll explore the Global Sheets Pattern in SwiftUI—a method that simplifies the management of sheets by centralizing their logic. We’ll examine various approaches to displaying and managing sheets and demonstrate how adopting concepts from other platforms can improve your app’s architecture, making sheet handling more efficient and scalable.

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

We'll begin by creating a `Sheet` enum and a `SheetAction` struct. The `Sheet` enum will represent all the different sheets in the application, while the `SheetAction` struct will encapsulate the actions related to presenting or managing these sheets within a SwiftUI application.

``` swift 

enum Sheet: Identifiable, Hashable {
    case settings
    case contact(String)
    
    var id: Self { self }
}

struct SheetAction {
    typealias Action = (Sheet) -> Void
    let action: Action
    
    func callAsFunction(_ sheet: Sheet) {
        action(sheet, dismiss)
    }
}
```

> In the above implementation, we are not handling the onDismiss behavior of the sheet. This will be covered later in the article. 

Next we will implement ```SheetEnvironmentKey``` by conforming to ```EnvironmentKey``` protocol and also add a new custom property ```showSheet``` to the ```EnvironmentValues``` struct.  

``` swift 
struct ShowSheetKey: EnvironmentKey {
    static var defaultValue: SheetAction = SheetAction { _ in }
}

extension EnvironmentValues {
    var showSheet: (SheetAction) {
        get { self[ShowSheetKey.self] }
        set { self[ShowSheetKey.self] = newValue }
    }
}
```

> In iOS 18 you can use ```@Entry``` macro to remove boiler plate code needed to create a custom Environment value. 

Finally, we inject the ```showSheet``` environment value to the root view of the application. This is shown below: 

``` swift 
@main
struct LearnApp: App {
    
    @State private var selectedSheet: Sheet?
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                ContentView()
                    .environment(\.showSheet, SheetAction(action: { sheet in
                        selectedSheet = sheet
                    }))
                    .sheet(item: $selectedSheet) { sheet in
                        switch sheet {
                            case .settings:
                                Text("Settings")
                            case .contact(let name):
                                Text("Contacting \(name)")
                        }
                    }
            }
        }
    }
}
```

Now, we can enjoy the fruits of our labor by utilizing the ```showSheet``` environment value in our views. This is shown below: 

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

Once you call `showSheet`, it sets the `selectedSheet` in the App file. This triggers a binding update, causing the `sheet` modifier to display the corresponding sheet.

> Since you're using a single `sheet` view modifier, this approach also ensures that only one sheet can be displayed at a time, preventing multiple sheets from overlapping.

You can also refactor your switch case in the App file and move it into a designated view. 

``` swift 
struct SheetView: View {
    
    let sheet: Sheet
    
    var body: some View {
        switch sheet {
            case .settings:
                Text("Settings")
            case .contact(let name):
                Text("Contacting \(name)")
        }
    }
}

 ContentView()
                    .environment(\.showSheet, SheetAction(action: { sheet in
                        selectedSheet = sheet
                    }))
                    .sheet(item: $selectedSheet) { sheet in
                        SheetView(sheet: sheet)
                    }

```

> You can also conform Sheet to the view and implement the body property to return the appropriate view for the sheet. 

### What about onDismiss?

In certain situations you need to handle the ```onDismiss``` event of the ```sheet``` modifier. Currently, our implementation does not support ```onDismiss```.  

### Conclusion

The Global Sheets Pattern in SwiftUI provides a powerful and streamlined approach to managing multiple sheets across your application. By centralizing sheet management with a single `SheetAction` struct and leveraging custom environment values, you can reduce redundancy, simplify your codebase, and make your sheet presentation logic more scalable.

This pattern not only enhances maintainability but also allows for a more flexible architecture, accommodating various sheets without cluttering your views with multiple modifiers. As demonstrated, adopting ideas from other platforms and implementing them in SwiftUI can lead to a cleaner and more efficient app design. Whether you're managing simple sheets or more complex workflows, this pattern offers a user-friendly API that ensures a smooth and cohesive user experience.