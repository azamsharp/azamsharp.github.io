
# Effective Communication Between Observable Stores in SwiftUI

Modern SwiftUI applications often rely on observable stores to manage state and business logic. As apps grow in complexity, these stores need to communicate efficiently—whether reacting to user actions, synchronizing data, or triggering side effects. This article explores practical patterns for inter-store communication, from direct method calls to event-driven approaches like Combine publishers and Swift Concurrency’s `AsyncStream`.  

We’ll examine the trade-offs of each technique, including:  
- **Direct View Coordination**: Simple but tightly couples UI to business logic.  
- **Delegate Pattern**: Works for one-to-one communication but lacks scalability.  
- **Combine Publishers**: Decouples producers and consumers, ideal for reactive workflows.  
- **AsyncStream**: A lightweight, concurrency-native alternative to Combine.  

By aligning stores with **bounded contexts** (e.g., `UserStore`, `InsuranceStore`) and adopting the right communication strategy, you can keep your codebase modular, testable, and free from spaghetti dependencies. Whether you’re building a small app with a single store or a large-scale system with many interconnected domains, this guide provides actionable insights to streamline store interactions while keeping SwiftUI views lean and focused.

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


## What is a Store? 

A **Store** is a class that either conforms to the `ObservableObject` protocol or uses the `@Observable` macro. Unlike view models, you don’t create a separate store for each screen. Instead, stores are organized around the **bounded contexts** of your application. In other words, a store represents a logical slice of your app’s data and business rules, not just a single view.

For example, a `ProductStore` could power multiple views such as `ProductListScreen`, `AddProductScreen`, and `ProductDetailScreen`. The same store instance ensures consistency across these screens.

Stores primarily manage **business logic**, not presentation logic. Any presentation-specific behavior can live inside SwiftUI views themselves or, if complex, be extracted into dedicated value types (structs).

Stores can also have **dependencies**, such as a networking layer. They use these dependencies to fetch or persist data, while holding only the minimal state required by the view.

> For instance, even if the server holds 100,000 records, your store might fetch only 50 at a time using pagination and expose that slice to the view.

Over the years, I’ve come across many different implementations of what I now call *stores*. Depending on the team or the codebase, they’ve been labeled as *services*, *controllers*, or even *view models*.

## A Single Store

Depending on your app and its requirements, you may start with a **single store** that manages the entire application state. This approach works well for small to medium-sized apps. Personally, I prefer beginning with a single store and introducing additional stores only when the complexity of the app demands it.

Here’s a snippet from my `PlatziStore` implementation:


``` swift 
@MainActor
@Observable
class PlatziStore {
    
    let httpClient: HTTPClient
    var categories: [Category] = []
    var locations: [Location] = []
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func loadCategories() async throws {
        let resource = Resource(url: Constants.Urls.categories, modelType: [Category].self)
        categories = try await httpClient.load(resource)
    }
    
    func createCategory(name: String) async throws {
       // code ...
    }
    
    func fetchProductsBy(_ categoryId: Int) async throws -> [Product] {
       // code ...
    }
    
    func createProduct(title: String, price: Double, description: String, categoryId: Int, images: [URL]) async throws -> Product {
        // code ...
    }
    
    func deleteProduct(_ productId: Int) async throws -> Bool {
       // code ...
    }
    
    func loadLocations() async throws {
       // code ...
    }
    
}
```

Stores can be injected either at the root of your application or at any specific node in the SwiftUI view hierarchy. Once injected, they become accessible through the `@Environment` property wrapper.

A good practice is to limit direct store access to **container views**. Child views should instead receive only the data they need via their initializers. This keeps child views lightweight, reusable, and free from unnecessary dependencies. 

This also improves SwiftUI’s performance, since views are only re-evaluated and re-rendered when their dependencies actually change.

The following implementation demonstrates this approach:

``` swift 

    @Environment(PlatziStore.self) private var store 

    var body: some View {
        ProductListView(products: store.products)
        LocationsView(locations: store.locations)
    }

```


A single store may be sufficient for small to medium-sized applications, but as your app grows larger, it becomes important to split responsibilities across multiple stores.

## Multiple Stores 

I prefer to design and divide stores based on the bounded context of the application. A bounded context defines a clear boundary around a specific domain or responsibility, ensuring each store manages only the data and logic relevant to that area. For example, in an e-commerce app:

* **`InsuranceStore`** manages insurance rates, insurance plans etc. 
* **`CatalogStore`** handles categories, filters, and sorting options.
* **`InventoryStore`** tracks stock levels and availability.
* **`UserStore`** manages user profiles.
* **`FulfillmentStore`** oversees shipping, delivery, and returns.

By aligning stores with bounded contexts rather than screens, the codebase remains modular, scalable, and easier to reason about.

A store does not operate in isolation—it often needs to communicate with other stores to retrieve or share information. This becomes especially important when an action in one store triggers a sequence of events that other stores must handle. In the next section, we’ll explore different strategies for propagating and handling domain events across stores.

## Handling Store Events in the View 

Consider a scenario in a healthcare or insurance app: whenever a user adds a new dependent, the insurance rates must be recalculated. In this setup, a `UserStore` is responsible for managing user information (including dependents), while an `InsuranceStore` handles insurance options, premiums, and rates for each user.

The question is: **how should the insurance rates be updated when a new dependent is added?**

One straightforward approach is to give the view access to both `UserStore` and `InsuranceStore`. After successfully adding a dependent through `UserStore`, the view can directly trigger a recalculation of insurance rates by calling the appropriate function on `InsuranceStore`.

The following implementation demonstrates this approach:

``` swift 
struct ContentView: View {
    
    @Environment(UserStore.self) private var userStore
    @Environment(InsuranceStore.self) private var insuranceStore
    @State private var insuranceRate: InsuranceRate?
    
    let userId = UUID()
    
    private func addDependent() async {
        let dependent = Dependent(id: UUID(), name: "Nancy", dob: Date())
        do {
            // add dependent to user management store
            try await userStore.addDependent(dependent, to: userId)
            insuranceRate = try await insuranceStore.calculateInsuranceRates(userId: userId)
        } catch {
            print(error.localizedDescription)
        }
    }
}
```

One major drawback of this approach is that any additional side effects must also be triggered directly from the view. For example, just as we manually called `insuranceStore.calculateInsuranceRates` after adding a dependent, we would need to add more calls in the view as new side effects are introduced. Over time, this makes the view more complex, tightly coupled to business logic, and harder to test.

## Handling Store Events Through Delegate 

Although I don’t typically use the delegate pattern in SwiftUI, it can still be applied if you want to forward an event to a single store. The first step is to define a delegate protocol, as shown in the implementation below:

``` swift 
@MainActor
protocol UserStoreDelegate: AnyObject {
    func dependentAdded(dependent: Dependent, userId: UUID) async
}
```

The `UserStore` is then updated to include a `delegate` property. At composition time, this property can be assigned to another store, which will act as the delegate and handle events emitted by `UserStore`.


``` swift 
@MainActor
@Observable
class UserStore {
    
    let httpClient: HTTPClient
    weak var delegate: (any UserStoreDelegate)?
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    // add a dependent to the user
    func addDependent(_ dependent: Dependent, to userId: UUID) async throws {
        
        guard var user = try await getUser(by: userId) else {
            throw UserError.userNotFound
        }
        
        user.dependents.append(dependent)
        await delegate?.dependentAdded(dependent: dependent, userId: userId)
    }
    
}
```

The `Insurance` becomes conforms to the delegate and implements the `dependentAdded` function. 

``` swift 
@MainActor
@Observable
class InsuranceStore: UserStoreDelegate {
    
    let httpClient: HTTPClient
    var insuranceRate: InsuranceRate?
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func dependentAdded(dependent: Dependent, userId: UUID) {
        // calculate rates
        calculateInsuranceRates(dependent: dependent, userId: userId)
        print("dependentAdded InsuranceStore")
    }
    
    private func calculateInsuranceRates(dependent: Dependent, userId: UUID) {
        insuranceRate = InsuranceRate(
            monthlyPremium: Decimal(Double.random(in: 300...600)),
            deductible: Decimal(Double.random(in: 500...2000)),
            coverageAmount: Decimal(Double.random(in: 10_000...50_000))
        )
    }
}
```

Make sure to hook up the delegate in the App file. This is shown below: 

``` swift 
@main
struct LearnApp: App {
    
    @State private var userStore: UserStore
    @State private var insuranceStore: InsuranceStore
    
    init() {
        let http = HTTPClient()
        let userStore = UserStore(httpClient: http)
        let insurance = InsuranceStore(httpClient: http)
        userStore.delegate = insurance   // wire them here
        _userStore = State(initialValue: userStore)
        _insuranceStore = State(initialValue: insurance)
    }
```


Now, whenever the `UserStore` adds a new dependent, the delegate’s method—`InsuranceStore.dependentAdded`—is invoked automatically.

However, a key limitation of this approach is that the delegate pattern only supports a **single listener**. In other words, if another store—such as `DocumentStore`—also needs to react when a new dependent is added, this setup won’t work.

In the following sections, we’ll explore alternative techniques that make it possible to notify **multiple listeners** when such events occur.

## Handling Store Events Using Combine Publishers 

Stores can also broadcast events using the publisher–subscriber pattern. `UserStore` exposes a publisher that any interested consumer can subscribe to. In the example below, `dependentAddedPublisher` emits an event whenever a new dependent is successfully added, allowing multiple listeners to react without coupling the view to those side effects.

``` swift 
@MainActor
@Observable
class UserStore {
    
    var users: [User] = []
    
    private let dependentAddedSubject = PassthroughSubject<(Dependent, UUID), Never>()
    
    var dependentAddedPublisher: AnyPublisher<(Dependent, UUID), Never> {
        dependentAddedSubject.eraseToAnyPublisher()
    }
    
    func addDependent(_ dependent: Dependent, to userId: UUID) async throws {
        guard let index = users.firstIndex(where: { $0.id == userId }) else { return }
        users[index].dependents.append(dependent)
        
        dependentAddedSubject.send((dependent, userId))
    }

}
``` 

`InsuranceStore` subscribes to `UserStore`’s events **in its initializer**. During setup, it listens to `dependentAddedPublisher` and, whenever a new dependent is added, it asynchronously recalculates the user’s insurance rate. Errors from that recalculation are caught and stored (e.g., in `lastError`) without blocking the UI.

Because the store is annotated with `@MainActor`, updates to observable state like `insuranceRate` are safely performed on the main thread. The subscription closure captures `self` with `[weak self]`, preventing retain cycles—so there’s no need to keep a separate (weak) property reference to `UserStore`. This wiring keeps **views** free of orchestration logic and allows **multiple consumers** to react to the same event stream independently. If you expect bursts of events, you can layer in operators like `removeDuplicates`, `debounce`, or `throttle` to control recalculation frequency.


``` swift 
@MainActor
@Observable
class InsuranceStore: Store {
    
    var insuranceRate: InsuranceRate?
    
    private var cancellables = Set<AnyCancellable>()
    private weak var userStore: UserStore?
    
    init(userStore: UserStore) {
        super.init()
        self.userStore = userStore
        self.userStore?.dependentAddedPublisher
            .sink { [weak self] dependent, userId in
                Task {
                    do {
                        try await self?.calculateInsurance(for: userId)
                        self?.lastError = nil
                    } catch {
                        self?.lastError = error
                    }
                }
            }.store(in: &cancellables)
    }
}
```

With this wiring, `UserDetailScreen` stays lean—it just adds the dependent. After that, `UserStore` emits a `dependentAdded` event, and every subscriber (e.g., `InsuranceStore`, `DocumentStore`) receives it and runs its own handler (recalculate rates, regenerate docs, etc.). No imperative calls from the view, no tight coupling, and side effects remain fully decoupled from UI code.

``` swift 
 Button("Add Dependent") {
                    Task {
                        let dependent = Dependent(id: UUID(), name: String.randomPersonName())
                        do {
                            try await userStore.addDependent(dependent, to: user.id)
                        } catch {
                            print(error.localizedDescription)
                        }
                    }
                }
```


By moving side effects out of the view and into subscribers, we keep `UserDetailScreen` simple and let each store own its domain logic. `UserStore` emits a single, typed event; `InsuranceStore`, `DocumentStore`, and any future consumers react independently—no tight coupling, no chains of imperative calls.

With this pattern, your UI stays predictable, tests get easier, and adding a new reaction (analytics, audit logs, notifications) is as simple as adding another subscriber—no changes to the view or the producer.

## Handling Store Events Using AsyncStream 

Swift Concurrency gives you a lightweight, Combine-free way to broadcast store events with `AsyncStream`. It fits naturally with `async/await`, keeps views thin, and scales to multiple listeners without wiring delegates.

We’ll begin by defining a typed `UserEvent` enum that represents all events the `UserStore` can emit. This gives us a clear, extensible, and type-safe way to model store events.

``` swift 
enum UserEvent {
        case dependentAdded(dependent: Dependent, userId: UUID)
        case dependentRemoved(dependent: Dependent, userId: UUID)
    }
```

Next, declare a `continuations` dictionary that holds one `AsyncStream<UserEvent>.Continuation` per active subscriber. A continuation is the write end of the stream—you use it to `yield` new events to listeners.

``` swift 
 // one continuation per subscriber
    private var continuations: [UUID: AsyncStream<UserEvent>.Continuation] = [:]
```

Next, when a consumer calls `events()`, we create and return a fresh `AsyncStream` and **register its continuation**. We also set `continuation.onTermination` to remove that entry when the subscriber stops listening—preventing leaks. The registration runs inside `Task { @MainActor in … }` because the `AsyncStream` builder isn’t main-actor–isolated, while the store’s state (the `continuations` dictionary) is.

Finally, the `emit(_:)` function fan-outs an event to every active subscriber. It iterates over the registered continuations and calls `yield(event)` on each, delivering the same event to all listeners.

``` swift 
private func emit(_ event: UserEvent) {
        for c in continuations.values { c.yield(event) }
    }
```

`UserStore.addDependent` is now wired to `AsyncStream`: after updating state, it **emits** a typed `UserEvent` on the stream, so all subscribers are notified immediately.

``` swift 
 func addDependent(_ dependent: Dependent, to userId: UUID) async throws {
        guard let index = users.firstIndex(where: { $0.id == userId }) else { return }
        users[index].dependents.append(dependent)
        emit(.dependentAdded(dependent: dependent, userId: userId))
    }

``` 

The `startListening(_:)` method in `InsuranceStore` subscribes to the `AsyncStream<UserEvent>` and reacts to events. The store keeps a cancellable `Task` so you can safely start/stop listening (and avoid multiple concurrent loops). When it receives `.dependentAdded`, it recalculates the user’s insurance rate.

```swift
@MainActor
@Observable
final class InsuranceStore {
    var insuranceRate: InsuranceRate?
    private var listener: Task<Void, Never>?

    func startListening(to events: AsyncStream<UserEvent>) {
        // Cancel any previous subscription
        listener?.cancel()

        listener = Task { [weak self] in
            guard let self else { return }
            for await event in events {
                switch event {
                case let .dependentAdded(userId, _):
                    do { try await self.calculateInsurance(for: userId) }
                    catch { /* log or emit error event */ }
                }
            }
        }
    }

    func calculateInsurance(for userId: UUID) async throws {
        // fetch & update insuranceRate
    }

    deinit { listener?.cancel() }
}
```

You can call `startListening(to: userStore.events())` in your composition root (e.g., `App.init()`), not from a view. This keeps the view simple, ensures a single subscription per store, and makes teardown deterministic. The implementation is shown below: 

``` swift 
@main
struct LearnApp: App {
    @State private var userStore: UserStore
    @State private var insuranceStore: InsuranceStore
    @State private var documentStore: DocumentStore

    init() {
        let user = UserStore()
        let insurance = InsuranceStore()
        let docs = DocumentStore()

        // Wire listeners once at startup
        insurance.startListening(to: user.events())
        docs.startListening(to: user.events())

        _userStore = State(initialValue: user)
        _insuranceStore = State(initialValue: insurance)
        _documentStore = State(initialValue: docs)
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(userStore)
                .environment(insuranceStore)
                .environment(documentStore)
        }
    }
}

```

## Conclusion

At the end of the day, observable stores work best when each one owns a clear slice of your app’s world and keeps the business logic where it belongs—inside the store—so your SwiftUI views can stay light and focused. When stores need to react to each other, choose the simplest communication style that fits: calling directly from the view is quick but doesn’t age well; a **delegate** is great when there’s exactly one listener; and for multiple, independent reactions, reach for **Combine** or **AsyncStream** to broadcast events without coupling your UI to side effects.

Keep the wiring tidy by composing everything at the app root (e.g., `App.init()` or a small coordinator), not inside views. Let producers emit events without knowing who’s listening, annotate UI-facing stores with `@MainActor`, avoid retain cycles, cancel listeners on teardown, and handle errors locally (log them or emit short-lived error events instead of a global “last error”). Type your events as an enum so changes stay compiler-checked.

In practice, start simple, evolve when the architecture asks for it, and favor patterns that make adding new reactions—like analytics, audit logs, or notifications—as easy as adding one more subscriber. That way, your UI stays predictable, your tests stay focused, and your codebase stays flexible as your app grows.

