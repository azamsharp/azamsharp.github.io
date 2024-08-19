
# Global Sheets Pattern in SwiftUI   

Managing sheet presentations in SwiftUI can quickly become complex, especially when your app requires multiple sheets across different screens. The default `sheet` view modifier provides a simple way to present modals, but as your app grows, using multiple sheet modifiers or manually controlling their display can lead to scattered, redundant code that’s hard to maintain.

What if there was a way to centralize sheet management, creating a cleaner, more maintainable solution with a user-friendly API? In this article, we’ll explore the Global Sheets Pattern in SwiftUI—a method that simplifies the management of sheets by centralizing their logic. We’ll examine various approaches to displaying and managing sheets and demonstrate how adopting concepts from other platforms can improve your app’s architecture, making sheet handling more efficient and scalable.

> If you're interested in iOS development courses and live workshops, check out [https://azamsharp.school](https://azamsharp.school).

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

Instead of managing multiple sheets, we're now using a single `sheet` modifier bound to the `activeSheet` enum. When the `activeSheet` value changes, the sheet modifier is automatically triggered. This approach allows for dynamic view presentation based on the active sheet, simplifying management and reducing complexity.

While this method is effective, it still involves multiple `sheet` modifiers for each screen. Although this isn't necessarily a problem, there may be opportunities for further improvement in managing sheets more efficiently.

### Global Sheets 

React Hooks are functions that enable you to use state and other React features in functional components, which were previously available only in class components.

A similar approach in SwiftUI involves using custom environment values. This method allows us to achieve a comparable result, enabling more flexible and centralized management of state and actions across different views.

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

Don’t worry if the above code doesn’t work right now due to the absence of `showSheet`. 

Now that we have a clear vision of the final implementation, we can focus on the steps needed to achieve it.

We’ll start by creating a `Sheet` enum and a `SheetAction` struct. The `Sheet` enum will represent the various sheets in the application, while the `SheetAction` struct will handle the actions related to presenting or managing these sheets within a SwiftUI application.

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

> In the above implementation, the `onDismiss` behavior of the sheet is not handled. This will be addressed later in the article.

Next, we will implement `SheetEnvironmentKey` by conforming to the `EnvironmentKey` protocol and add a new custom property, `showSheet`, to the `EnvironmentValues` struct.


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

> In iOS 18, you can use the `@Entry` macro to eliminate the boilerplate code required to create a custom environment value.

Finally, we inject the `showSheet` environment value into the root view of the application, as shown below:

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

Now, we can leverage the `showSheet` environment value in our views to simplify sheet presentation. Here’s how you can use it:

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

Once you call `showSheet`, it sets the `selectedSheet` in the App file, triggering a binding update that causes the `sheet` modifier to display the corresponding sheet.

> Since you're using a single `sheet` view modifier, this approach ensures that only one sheet is displayed at a time, preventing multiple sheets from overlapping.

Additionally, you can refactor the switch case in the App file by moving it into a dedicated view.

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

> You can also conform `Sheet` to the `View` protocol and implement the `body` property to return the appropriate view for each sheet.

### What about onDismiss?

In some cases, you might need to handle the `onDismiss` event of the `sheet` modifier. Our current implementation doesn’t support `onDismiss`, but this can be addressed by adding another closure. The `SheetAction` and `ShowSheetKey` have been updated as shown in the implementation below:

``` swift 
struct SheetAction {
    typealias Action = (Sheet, (() -> Void)?) -> Void
    let action: Action
    
    func callAsFunction(_ sheet: Sheet, _ dismiss: (() -> Void)? = nil) {
        action(sheet, dismiss)
    }
}

struct ShowSheetKey: EnvironmentKey {
    static var defaultValue: SheetAction = SheetAction { _, _ in }
}
```

We also need to update the App file and handle the new ```dismiss``` closure.

``` swift 
@main
struct LearnApp: App {
    
    @State private var selectedSheet: Sheet?
    @State private var onSheetDismiss: (() -> Void)?
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                ContentView()
                    .environment(\.showSheet, SheetAction(action: { sheet, dismiss in
                        selectedSheet = sheet
                        onSheetDismiss = dismiss
                    }))
                    .sheet(item: $selectedSheet, onDismiss: onSheetDismiss) { sheet in
                        SheetView(sheet: sheet)
                    }
            }
        }
    }
}
```

Similar to the `selectedSheet` property, we introduce a separate property for `onSheetDismiss`. The `onSheetDismiss` closure is triggered when the sheet is dismissed. 

The usage is shown below: 

``` swift 
struct ContentView: View {
    
    @Environment(\.showSheet) private var showSheet
    
    private func settingsScreenDismissed() {
        print("settingsScreenDismissed")
    }
    
    var body: some View {
        VStack {
            Button("Show Settings Screen") {
                showSheet(.settings, settingsScreenDismissed)
            }
        }
        .padding()
    }
}
```

The `showSheet` function now supports a second argument for passing a dismiss closure. This allows each view to handle its own dismissal logic, as different views may require specific actions when they are dismissed.

### What about Stacked Sheets? 

In rare cases where you need to display one sheet on top of another, you can still use the default `sheet` modifier to present sheets in a stacked manner.

### Source code: 

You can download the complete source [here](https://gist.github.com/azamsharpschool/9e7732f85f85353de069425454d0e8dc).  

### Conclusion

The Global Sheets Pattern in SwiftUI provides a powerful and streamlined approach to managing multiple sheets across your application. By centralizing sheet management with a single `SheetAction` struct and leveraging custom environment values, you can reduce redundancy, simplify your codebase, and make your sheet presentation logic more scalable.

This pattern not only enhances maintainability but also allows for a more flexible architecture, accommodating various sheets without cluttering your views with multiple modifiers. As demonstrated, adopting ideas from other platforms and implementing them in SwiftUI can lead to a cleaner and more efficient app design. Whether you're managing simple sheets or more complex workflows, this pattern offers a user-friendly API that ensures a smooth and cohesive user experience.