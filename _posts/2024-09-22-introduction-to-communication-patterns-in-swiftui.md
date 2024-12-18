
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

# Introduction to Communication Patterns in SwiftUI

 <div class="share-container">
    <div>
      <!-- Twitter -->
      <a href="https://twitter.com/intent/tweet?url=https://azamsharp.com/2024/09/22/introduction-to-communication-patterns-in-swiftui.html&text=Introduction to Communication Patterns in SwiftUI by @azamsharp"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button twitter">
        Share on Twitter
      </a>
    </div>
    <div>
      <!-- LinkedIn -->
      <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://azamsharp.com/2024/09/22/introduction-to-communication-patterns-in-swiftui.html"
         target="_blank" 
         rel="noopener noreferrer" 
         class="share-button linkedin">
        Share on LinkedIn
      </a>
    </div>
    
  </div>

SwiftUI provides a powerful and declarative way to build UIs, allowing views to react to state changes automatically. However, managing communication between views, especially when passing data or events from one view to another, can be challenging if not handled properly. In this article, we'll explore several communication patterns in SwiftUI that enable seamless data flow between views, ensuring that updates occur efficiently and in a way that aligns with SwiftUI’s architecture.

We will dive into a practical scenario: starting from a list screen, where a user can tap a button to open a sheet to add a new item. The focus of the article will be on different approaches that allow the newly added item to communicate back to the list screen and update it accordingly. By exploring these patterns—such as closures, bindings, and `@Environment` objects—you’ll learn best practices for ensuring your SwiftUI views interact harmoniously, without workarounds like boolean flags to force view refreshes.

### Option 1: Callback Using Closures 

One of the simplest approaches to communication between views is using closures as callbacks. In this case, `UserListScreen` can pass a closure to `AddUserScreen`, which `AddUserScreen` will call after collecting and validating the necessary information. This allows the new data to be passed back to `UserListScreen` through the closure.

Here’s how the implementation of `AddUserScreen` looks:

``` swift 
struct AddUserScreen: View {
    
    @Environment(\.dismiss) private var dismiss
    
    @State private var name: String = ""
    let onUserAdd: (User) -> Void
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            Button("Save") {
                let user = User(name: name)
                onUserAdd(user)
                dismiss()
            }
        }
    }
}
```

The `AddUserScreen` requires a mandatory `onUserAdd` closure, which gets triggered when the save button is tapped. This closure allows the newly created user to be passed back to `UserListScreen`.

Below is the implementation of the calling screen, `UserListScreen`:

``` swift 
struct UserListScreen: View {
    
    @Environment(\.httpClient) private var httpClient
    
    @State private var users: [User] = []
    @State private var isPresented: Bool = false
    
    private func handleUserAdd(user: User) {
        users.append(user)
    }
    
    var body: some View {
        List(users) { user in
            Text(user.name)
        }
        .sheet(isPresented: $isPresented, content: {
            AddUserScreen(onUserAdd: handleUserAdd)
        })
        .task {
            // fetch the users 
        }
        .toolbar(content: {
            ToolbarItem(placement: .topBarTrailing) {
                Button("Add New User") {
                    isPresented = true
                }
            }
        })
       
        .navigationTitle("Users")       
    }
}
```

There are a couple of interesting points to highlight in `UserListScreen`. First, we access the `HTTPClient` directly within the view using custom `@Environment` values. Additionally, `UserListScreen` maintains a private state for the `users` array, which allows it to manage user data locally.

> Choosing to maintain private state for model objects within a view depends on the specific use case. Local state is ideal when the data is only relevant to that view and does not need to be shared across other parts of the application.

The `handleUserAdd` method, defined inside `UserListScreen`, is responsible for updating the `users` array by appending the new user to the list.  

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

### Option 2: @Binding 

Another technique is to pass ```@Binding``` from ```UserListScreen``` to ```AddUserScreen```. This way ```AddUserScreen``` directly modify the users array, without needing a callback function. 

The implementation is shown below: 

``` swift 
struct AddUserScreen: View {
    
    @Environment(\.dismiss) private var dismiss
    @State private var name: String = ""
    
    @Binding var users: [User]
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            Button("Save") {
                let user = User(name: name)
                users.append(user)
                dismiss()
            }
        }
    }
}
```

The SwiftUI `@Binding` property wrapper allows a child view to communicate changes back to its parent view. Although a model might not technically be considered a child view, `@Binding` still enables it to modify data passed from the parent, in this case, the `UserListScreen`. This creates a two-way data flow, where changes made in the child view can directly update the state in the parent view.

Next, inside the ```UserListScreen``` we can pass the binding as shown below: 

``` swift 
 var body: some View {
        List(users) { user in
            Text(user.name)
        }
        .sheet(isPresented: $isPresented, content: {
            AddUserScreen(users: $users)
        })
```

While this approach works, it feels inefficient to pass the entire array to another view just to add a single item. A more elegant solution is to use `@Environment`, which allows us to manage shared state more cleanly across multiple views. We'll explore this alternative approach next.

### Option 3: @Environment 

The `@Environment` object provides a way to share state across different views in your application. It allows you to inject global state into any view that needs it, without making it a singleton. The key distinction is that `@Environment` objects are dependent on where they are injected, meaning their scope is determined by the view hierarchy. Additionally, you can have multiple `@Environment` objects throughout your app, each managing different aspects of your application's state.

To address our needs, we can create an `@Environment` object called `UserStore` that manages user-related state and functionality. This `UserStore` will handle tasks such as adding, updating, and deleting users, as well as fetching, sorting, and searching through user data. This centralized state management simplifies interactions with user data across the app.

The implementation is shown below: 

``` swift 
@Observable
class UserStore {
    
    // make HTTPClient protocol if needed
    let httpClient: HTTPClient
    private(set) var users: [User] = []
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
        self.users = users
    }
    
    func addUser(_ user: User) {
        users.append(user)
    }
    
    func loadUsers() async throws {
        do {
            users = try await httpClient.fetchUsers()
        } catch {
            print(error)
        }
    }
}
```

The `UserStore` is marked as an `ObservableObject` and depends on an `HTTPClient` to handle network operations. It maintains an array of users, managing all the user-related state and functionality.

The `UserListScreen` can access `UserStore` via the environment. To enable this, the `UserStore` needs to be injected into the environment, as shown below:

``` swift 
#Preview {
    NavigationStack {
        UserListScreen()
    }
    .environment(UserStore(httpClient: HTTPClient()))
}
```

Next, ```UserListScreen``` can be updated to use ```UserStore``` as shown below: 

``` swift 
struct UserListScreen: View {
    
    @Environment(UserStore.self) private var userStore
    @State private var isPresented: Bool = false
    
    var body: some View {
        List(userStore.users) { user in
            Text(user.name)
        }
        .sheet(isPresented: $isPresented, content: {
            AddUserScreen()
        })
        .task {
            do {
                try await userStore.loadUsers()
            } catch {
                print(error)
            }
        }
       
        .navigationTitle("Users")
    }
}
```

We have removed the local state from the view and replaced it with `@Environment`, allowing `UserStore` to manage the state. Now that `UserStore` is accessible through `@Environment`, `AddUserScreen` can directly interact with it to make changes. Here's how that works:

``` swift 
struct AddUserScreen: View {
    
    @Environment(\.dismiss) private var dismiss
    @State private var name: String = ""
    @Environment(UserStore.self) private var userStore
    
    var body: some View {
        Form {
            TextField("Name", text: $name)
            Button("Save") {
                let user = User(name: name)
                userStore.addUser(user)
                dismiss()
            }
        }
    }
}
```

Using this technique we did not have to pass any closure or binding. ```AddUserScreen``` access the store through the environment and adds a new user. 

One of the main benefits of this approach is that all the views that are interested in users array will automatically get updated. This means you don't have to fight SwiftUI to manually cause a refresh. 

> If you're relying on custom boolean flags to trigger view refreshes in SwiftUI, it likely indicates you're not leveraging SwiftUI's declarative nature correctly.

### Conclusion

In this article, we explored various communication patterns in SwiftUI that enable seamless interaction between views. From using closures as callbacks, to leveraging `@Binding` for direct data manipulation, and finally utilizing the power of `@Environment` objects for shared state management, each method has its own strengths and use cases. 

Closures offer simplicity and control, while `@Binding` enables a two-way data flow. However, `@Environment` objects provide a more scalable and clean solution for managing state globally, especially when multiple views need access to the same data. By using the right approach for the given scenario, you can avoid common pitfalls like relying on manual refresh mechanisms and instead let SwiftUI’s declarative design do the heavy lifting. 

Choosing the correct communication pattern will lead to more maintainable, efficient, and readable SwiftUI applications.

