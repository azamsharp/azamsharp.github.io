
# Dependency Management in SwiftUI   

Managing dependencies is an essential part of building scalable and testable SwiftUI applications. Dependencies such as API clients, stores, and services form the backbone of your app’s data flow. If handled carelessly, they can lead to tightly coupled code that is difficult to test, reuse, or extend.

SwiftUI, being a declarative framework, provides multiple strategies for injecting and managing dependencies. The most common approaches include constructor injection, where dependencies are passed directly into views or services, and environment values/objects, which allow dependencies to be shared across the view hierarchy without explicitly threading them through every initializer.

By understanding how to leverage these techniques effectively, you can design SwiftUI apps that remain modular, test-friendly, and easy to maintain as your project grows. In this article, we will explore three key patterns of dependency management in SwiftUI:

**Constructor Dependency** – Directly passing dependencies into views.

**Environment Values** – Using SwiftUI’s built-in dependency injection system for stateless services.

**Environment Objects** – Sharing observable state across multiple views without manual propagation.

Each approach has its strengths and trade-offs, and knowing when to use one over the other is the key to building clean, maintainable SwiftUI architectures.

 <!-- AzamSharp School promo (drop anywhere in your article) -->
<section class="azamsharp-promo" role="complementary" aria-label="AzamSharp School">
  <div class="azamsharp-card">
    <div class="azamsharp-icon" aria-hidden="true">
      <!-- simple inline Swift-like swirl -->
      <svg viewBox="0 0 64 64" width="28" height="28" xmlns="http://www.w3.org/2000/svg">
        <path d="M48 14c2 10-5 26-18 30 4-6 3-12 1-16-5-9-14-14-17-16 6 11 12 17 12 17-7-4-12-9-15-12 3 7 8 13 13 17-5-1-10-3-14-6 6 9 15 15 25 16 16 2 25-10 25-22 0-3-1-6-2-8z" fill="currentColor"/>
      </svg>
    </div>
    <div class="azamsharp-content">
      <div class="azamsharp-kicker">AzamSharp School</div>
      <h3 class="azamsharp-title">Level up your Swift &amp; SwiftUI skills</h3>
      <p class="azamsharp-copy">
        Hands-on courses, live workshops, and personalized 1-on-1 coaching.
      </p>
      <div class="azamsharp-actions">
        <a class="azamsharp-button" href="https://azamsharp.school/" target="_blank" rel="noopener noreferrer">
          Explore courses
        </a>
        <span class="azamsharp-note"></span>
      </div>
    </div>
  </div>
</section>

<style>
  .azamsharp-promo { margin: 1.25rem 0; }
  .azamsharp-card {
    --bg: #ffffff; --fg: #0f172a; --muted:#475569; --ring1:#7c3aed; --ring2:#06b6d4;
    display:flex; gap:1rem; align-items:flex-start; padding:1rem 1.125rem;
    border-radius:16px; border:1px solid transparent;
    background:
      linear-gradient(var(--bg),var(--bg)) padding-box,
      linear-gradient(135deg,var(--ring1),var(--ring2)) border-box;
    box-shadow: 0 6px 18px rgba(2,6,23,.06);
  }
  .azamsharp-icon { display:grid; place-items:center; width:44px; height:44px;
    border-radius:12px; background:linear-gradient(135deg,var(--ring1),var(--ring2)); color:#fff; flex:0 0 auto; }
  .azamsharp-content { color:var(--fg); }
  .azamsharp-kicker { font-size:.75rem; letter-spacing:.06em; text-transform:uppercase; color:var(--muted); }
  .azamsharp-title { margin:.1rem 0 .4rem; font-size:1.05rem; line-height:1.35; }
  .azamsharp-copy { margin:0 0 .75rem; color:var(--muted); }
  .azamsharp-actions { display:flex; flex-wrap:wrap; gap:.6rem .9rem; align-items:center; }
  .azamsharp-button {
    display:inline-block; padding:.55rem .85rem; border-radius:10px; font-weight:600; text-decoration:none;
    background:var(--fg); color:#fff; transition:transform .06s ease, opacity .2s ease;
  }
  .azamsharp-button:hover { transform:translateY(-1px); opacity:.95; }
  .azamsharp-note { font-size:.9rem; color:var(--muted); }

  @media (prefers-color-scheme: dark) {
    .azamsharp-card { --bg:#0b1220; --fg:#e5e7eb; --muted:#94a3b8; box-shadow: 0 8px 24px rgba(0,0,0,.35); }
    .azamsharp-button { background:#e5e7eb; color:#0b1220; }
  }

  @media (max-width: 520px) {
    .azamsharp-card { align-items: center; }
    .azamsharp-title { font-size:1rem; }
    .azamsharp-copy { font-size:.95rem; }
  }
</style>

### Constructor Dependency 

A straightforward and widely used approach to dependency injection is constructor injection. Constructor injection keeps dependencies explicit and makes testing easier by allowing you to substitute different implementations as needed.

For example, imagine a SwiftUI view responsible for displaying a list of customers retrieved from a JSON API. In this case, we can inject an HTTPClient instance through the view’s constructor and then call its methods to load the customer data:

``` swift 
struct ContentView: View {
    
    let httpClient: HTTPClientProtocol
    @State private var customers: [String] = []
    
    var body: some View {
        List(customers, id: \.self) { customer in
            Text(customer)
        }.task {
            do {
                customers = try await httpClient.loadCustomers()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}

#Preview {
    ContentView(httpClient: MockHTTPClient())
}

```

> In this example, the HTTPClient is injected directly into the view. Alternatively, you could encapsulate it within a Store (an Observable object), which can then be shared across multiple views.

The ContentView now explicitly depends on HTTPClientProtocol. In the preview, we provide a MockHTTPClient so that no actual network requests are made.

> If your server is a managed dependency, mocking may not always be necessary. You can often call a real API instance running locally or in a controlled environment, as long as it responds quickly and is isolated from production.

However, constructor injection can quickly become repetitive. For example, if we later introduce a new screen that allows users to add customers, we’ll need to pass the same HTTPClient dependency into that screen as well. The snippet below demonstrates this pattern when presenting an AddCustomerScreen inside a sheet:

``` swift 
VStack {
            List(customers, id: \.self) { customer in
                Text(customer)
            }.task {
                do {
                    customers = try await httpClient.loadCustomers()
                } catch {
                    print(error.localizedDescription)
                }
            }
            Button("Add Customer") {
                isPresented = false
            }
        }.sheet(isPresented: $isPresented) {
            AddCustomerScreen(httpClient: httpClient)
        }
```

This approach requires every screen that relies on HTTPClient to receive it through its constructor. While this works, it quickly becomes repetitive and cumbersome as more screens in the app need access to the same dependency for making JSON calls.

> Although there are third-party dependency management libraries that simplify this with property wrappers and other abstractions, here we’ll focus on plain vanilla SwiftUI—without introducing external dependencies.

In the next section, we’ll explore how SwiftUI’s built-in Environment system can act as a lightweight dependency injection framework, reducing boilerplate and making shared services easier to access.

### Environment Values 

SwiftUI relies heavily on Environment values to provide system-level dependencies that can be accessed directly inside views. Common examples include colorScheme, dismiss, locale, and scenePhase.

Beyond these built-in options, SwiftUI also empowers developers to define their own custom Environment values. This makes it easy to share services or utilities across the view hierarchy without the overhead of threading them through every initializer.

For instance, the code below shows how you can extend the EnvironmentValues struct to add a custom httpClient dependency:

``` swift 
extension EnvironmentValues {
    @Entry var httpClient = HTTPClientFactory.create()
}
```

With this setup, the httpClient dependency can now be accessed directly inside our views. The updated ContentView implementation below demonstrates how this works:

``` swift 
struct ContentView: View {
    
    @Environment(\.httpClient) private var httpClient
    @State private var customers: [String] = []
    @State private var isPresented: Bool = false
    
    var body: some View {
        VStack {
            List(customers, id: \.self) { customer in
                Text(customer)
            }.task {
                do {
                    customers = try await httpClient.loadCustomers()
                } catch {
                    print(error.localizedDescription)
                }
            }
            Button("Add Customer") {
                isPresented = false
            }
        }.sheet(isPresented: $isPresented) {
            AddCustomerScreen()
        }
    }
}

#Preview {
    ContentView()
}
```

As you can see, we no longer need to pass the httpClient dependency through the ContentView constructor. Instead, it’s accessed directly from the Environment. The same applies to AddCustomerScreen, which can now retrieve the dependency without requiring it in its initializer.

That said, this approach doesn’t eliminate initializer-based dependency injection—and it shouldn’t. Initializer injection remains essential when passing data between views. For example, in the code below, a CustomerCellView is introduced that requires a specific customer to be passed in. This is a perfect use case for initializer injection:

``` swift 
 List(customers, id: \.self) { customer in
                CustomerCellView(customer: customer)
            }
```

The dependencies we’ve discussed so far are the kinds of services that many views in an app will need to use.

You might be wondering: why choose Environment values over Environment objects? The main reason in this case is that we’re working with a stateless service. Since the service doesn’t maintain state, it’s better represented as a value type accessed through the Environment.

On the other hand, if we were dealing with stateful entities—such as Stores that hold and publish data changes—we’d use Environment objects, which conform to ObservableObject or leverage the @Observable macro.

In the next section, we’ll build a Store and inject it as an Environment object to see how this approach works in practice.

### Environment Object 

Environment objects act as a form of shared, observable state that can be accessed by any view within the scope where they are injected. It’s important to note that shared here does not mean singleton—the availability of an Environment object depends entirely on where it is provided in the view hierarchy.

Here’s an example implementation of a Store as an Environment object:

``` swift 
@Observable
class Store {
    
    var customers: [String] = []
    let httpClient: HTTPClientProtocol
    
    init(httpClient: HTTPClientProtocol) {
        self.httpClient = httpClient
    }
    
    func loadCustomers() async throws {
        customers = try await httpClient.loadCustomers()
    }

    // other functions 
    
}
```

You can inject the Store into the Environment at any level of your view hierarchy. In practice, it’s often best to provide it at the root of the application, so that all screens have access to it.

When I refer to screens, I mean container views. Their job is to coordinate data loading through services or stores and then pass the necessary data down to their child views. Child views, on the other hand, should remain free of Environment dependencies whenever possible to keep them simple, reusable, and easy to test.

The ContentView below has been updated to demonstrate how an Environment object is used in this setup:

``` swift 
struct ContentView: View {

    @Environment(Store.self) private var store
    @State private var customers: [String] = []
    @State private var isPresented: Bool = false
    
    var body: some View {
        VStack {
            List(store.customers, id: \.self) { customer in
                CustomerCellView(customer: customer)
            }.task {
                do {
                    try await store.loadCustomers()
                } catch {
                    print(error.localizedDescription)
                }
            }
            Button("Add Customer") {
                isPresented = false
            }
        }.sheet(isPresented: $isPresented) {
            AddCustomerScreen()
        }
    }
}

#Preview {
    ContentView()
        .environment(Store(httpClient: HTTPClientFactory.create()))
}
```

Instead of passing the Store directly into ContentView through its initializer, we’ve injected it into the Environment once, making it available to any screen that needs it. This greatly reduces boilerplate and simplifies dependency management.

That said, even though Environment objects can now be accessed anywhere in the app (since they’re injected at the root level), it’s best practice to use them primarily in container screens. These higher-level views can then pass the necessary data down to their child views. This pattern keeps child views simple, reusable, and independent of global dependencies, while also ensuring SwiftUI can track changes efficiently and re-render views only when necessary.

A typical SwiftUI App file might look like this:

``` swift 
@main
struct LearnApp: App {
    
    let httpClient = HTTPClientFactory.create()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                ContentView()
            }.environment(Store(httpClient: httpClient))
        }
    }
}
```

If your app is structured with multiple NavigationStacks, such as in a TabView, you have flexibility in how you provide the Store. You can either inject a single Store at the root level, making it available across all tabs, or inject a dedicated Store at the root of each tab if they manage different domains of data.

Ultimately, the right choice depends on the design of your app and its specific requirements. The key is to keep dependencies scoped appropriately—broad enough to avoid repetition, but narrow enough to maintain clear boundaries between different parts of your app.

### Conclusion 

Dependency management in SwiftUI is about choosing the right tool for the right job. Constructor injection works best when passing data or explicit dependencies into individual views, keeping them clear and testable. Environment values are a great fit for stateless services that need to be shared across many parts of the app without repetitive boilerplate, while Environment objects provide a powerful way to share and observe state across multiple screens. Rather than competing, these approaches complement each other, and by applying them thoughtfully you can build SwiftUI applications that remain scalable, maintainable, and easy to extend as they grow in complexity.

<!-- AzamSharp School promo (drop anywhere in your article) -->
<section class="azamsharp-promo" role="complementary" aria-label="AzamSharp School">
  <div class="azamsharp-card">
    <div class="azamsharp-icon" aria-hidden="true">
      <!-- simple inline Swift-like swirl -->
      <svg viewBox="0 0 64 64" width="28" height="28" xmlns="http://www.w3.org/2000/svg">
        <path d="M48 14c2 10-5 26-18 30 4-6 3-12 1-16-5-9-14-14-17-16 6 11 12 17 12 17-7-4-12-9-15-12 3 7 8 13 13 17-5-1-10-3-14-6 6 9 15 15 25 16 16 2 25-10 25-22 0-3-1-6-2-8z" fill="currentColor"/>
      </svg>
    </div>
    <div class="azamsharp-content">
      <div class="azamsharp-kicker">AzamSharp School</div>
      <h3 class="azamsharp-title">Master Swift &amp; SwiftUI — faster</h3>
      <p class="azamsharp-copy">
        Learn by building: concise lessons, live workshops, and 1-on-1 coaching.
        <span class="azamsharp-note">150+ hours of practical iOS training.</span>
      </p>
      <div class="azamsharp-actions">
        <a class="azamsharp-button" href="https://azamsharp.school/" target="_blank" rel="noopener noreferrer">
          Start learning
        </a>
      </div>
    </div>
  </div>
</section>
