
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

