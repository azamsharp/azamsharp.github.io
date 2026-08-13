# Unit Testing Navigation Logic in SwiftUI 

> NOTE: This article is part of my upcoming book on Testing in iOS, where I cover practical > techniques for testing SwiftUI applications, navigation, networking, persistence, external > dependencies, and more.

Navigation is an important part of almost every SwiftUI application. We use NavigationStack to push views, present different screens, and move users through various flows in the application.

But navigation is not always just about moving from one screen to another. In many applications, the destination depends on business rules.

For example, after registration, an in-state student may navigate directly to the courses screen, while an out-of-state or international student may need to review an agreement first. A faculty member may navigate to a completely different dashboard.

These are navigation decisions, but they are also application logic.

If this logic lives directly inside SwiftUI views, testing it usually means testing the view or relying on UI tests. A better approach is to separate navigation decisions from the view and represent navigation using routes that can be easily inspected and tested.

In this article, we will build a simple router, move our navigation decisions into the router, and use Swift Testing to verify that users are sent to the correct destination.

<!-- Book Banner: SwiftUI Architecture Book -->
<div class="azam-book-banner" role="region" aria-label="SwiftUI Architecture Book Banner">
  <div class="azam-book-banner__inner">
    <div class="azam-book-banner__cover">
      <img
        src="https://azamsharp.com/images/swiftui-book-1.png"
        alt="SwiftUI Architecture book cover"
        loading="lazy"
      />
    </div>
    <div class="azam-book-banner__content">
      <p class="azam-book-banner__eyebrow">SwiftUI Architecture Book</p>
      <h3 class="azam-book-banner__title">Patterns and Practices for Building Scalable Applications</h3>
      <p class="azam-book-banner__subtitle">
        A practical guide to building SwiftUI apps that stay clean as they grow.
      </p>
      <div class="azam-book-banner__actions">
        <a class="azam-book-banner__button" href="https://azamsharp.school/swiftui-architecture-book.html" target="_blank" rel="noopener">
          Get the book
        </a>
      </div>
    </div>
  </div>
</div>

<style>
  .azam-book-banner {
    --bg1: #0b1220;
    --bg2: #111a2d;
    --text: rgba(255, 255, 255, 0.92);
    --muted: rgba(255, 255, 255, 0.74);
    --border: rgba(255, 255, 255, 0.12);
    --shadow: 0 18px 45px rgba(0, 0, 0, 0.28);
    --accent: #6ee7b7; /* tweak to match your brand */
    --accent2: #60a5fa;

    margin: 22px 0;
    color: var(--text);
    border: 1px solid var(--border);
    border-radius: 16px;
    overflow: hidden;
    background: radial-gradient(1200px 600px at 10% 0%, rgba(96, 165, 250, 0.22), transparent 60%),
                radial-gradient(900px 500px at 90% 30%, rgba(110, 231, 183, 0.18), transparent 60%),
                linear-gradient(135deg, var(--bg1), var(--bg2));
    box-shadow: var(--shadow);
  }

  .azam-book-banner__inner {
    display: grid;
    grid-template-columns: 132px 1fr;
    gap: 18px;
    padding: 18px;
    align-items: center;
  }

  .azam-book-banner__cover {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .azam-book-banner__cover img {
    width: 132px;
    height: auto;
    border-radius: 12px;
    border: 1px solid rgba(255,255,255,0.14);
    box-shadow: 0 14px 28px rgba(0,0,0,0.35);
    background: rgba(255,255,255,0.04);
  }

  .azam-book-banner__eyebrow {
    margin: 0 0 6px 0;
    font-size: 12px;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--muted);
  }

  .azam-book-banner__title {
    margin: 0 0 8px 0;
    font-size: 18px;
    line-height: 1.25;
  }

  .azam-book-banner__subtitle {
    margin: 0 0 14px 0;
    font-size: 14px;
    line-height: 1.55;
    color: var(--muted);
    max-width: 62ch;
  }

  .azam-book-banner__actions {
    display: flex;
    gap: 12px;
    align-items: center;
    flex-wrap: wrap;
  }

  .azam-book-banner__button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 10px 14px;
    border-radius: 12px;
    font-weight: 700;
    font-size: 14px;
    color: #071018;
    text-decoration: none;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border: 0;
    box-shadow: 0 10px 22px rgba(0,0,0,0.28);
    transition: transform 140ms ease, filter 140ms ease;
  }

  .azam-book-banner__button:hover {
    transform: translateY(-1px);
    filter: brightness(1.02);
  }

  .azam-book-banner__link {
    font-size: 14px;
    color: rgba(255,255,255,0.86);
    text-decoration: none;
    border-bottom: 1px solid rgba(255,255,255,0.22);
    padding-bottom: 2px;
    transition: border-color 140ms ease, color 140ms ease;
  }

  .azam-book-banner__link:hover {
    color: rgba(255,255,255,0.95);
    border-color: rgba(255,255,255,0.45);
  }

  /* Mobile */
  @media (max-width: 520px) {
    .azam-book-banner__inner {
      grid-template-columns: 1fr;
      text-align: left;
    }

    .azam-book-banner__cover {
      justify-content: flex-start;
    }

    .azam-book-banner__cover img {
      width: 120px;
    }
  }
</style>

### Designing the Route Structure 

Since our application supports both students and faculty, we can organize our navigation routes based on each role. Instead of placing every possible destination inside a single large enum, we can create separate routes for students and faculty.

The parent Route enum defines two cases, student and faculty. Each case wraps its corresponding route type, StudentRoute or FacultyRoute.

Both StudentRoute and FacultyRoute also expose a destination property. This property is responsible for returning the SwiftUI view associated with a particular route. The parent Route simply delegates the destination to the appropriate nested route.

This approach keeps the navigation structure organized and makes it easier to add new routes as the application grows.

The implementation is shown below:

``` swift 
enum Route: Hashable {
    
    case student(StudentRoute)
    case faculty(FacultyRoute)
    
    @ViewBuilder
    var destination: some View {
        switch self {
        case .student(let studentRoutes):
            studentRoutes.destination
        case .faculty(let facultyRoutes):
            facultyRoutes.destination
        }
    }
    
    enum StudentRoute {
        case courses
        case profile
        case agreement
        
        @ViewBuilder
        var destination: some View {
            switch self {
            case .courses:
                Text("Courses")
            case .profile:
                Text("Profile")
            case .agreement:
                Text("Agreement")
            }
        }
        
    }
    
    enum FacultyRoute {
        case dashboard
        case profile
        
        @ViewBuilder
        var destination: some View {
            switch self {
            case .dashboard:
                Text("Dashboard")
            case .profile:
                Text("Profile")
            }
        }
    }
}
```

One of the benefits of modeling navigation this way is that routes are represented as values. This becomes especially useful when we start unit testing our navigation logic. Instead of testing whether a particular SwiftUI screen was rendered, we can test whether our application produced the correct Route for a given action or business rule.

Now, let's see how our Router can use these routes. 

### Implementing the Router 

To keep things simple, our Router will manage a single collection of routes. The routes array represents the current navigation path and is updated whenever we navigate to a new destination.

If your application uses a TabView where each tab maintains its own NavigationStack, you will need to adjust this implementation so that each tab can maintain its own collection of routes. For now, we will focus on a single navigation stack.

The implementation is shown below:

``` swift 
@Observable
class Router {
    
    var routes: [Route] = []
    
    func navigate(to route: Route) {
        routes.append(route)
    }
    
}
```

The navigate(to:) function simply appends the new route to the routes array. Since Router uses the Observation framework, SwiftUI can automatically respond to changes in the navigation path. 

In the next section, we will learn how to start using Router in our view. 

### Using the Router in the View 

Before we can use the Router inside our views, we need to create it and inject it into the SwiftUI environment.

A good place to do this is at the root of the application. We create the Router using @State and connect its routes collection to the NavigationStack.

``` swift 
@main
struct AZSchoolApp: App {
    
    @State private var router = Router()
    
    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $router.routes) {
                RootScreen()
                    .navigationDestination(for: Route.self) { route in
                        route.destination
                    }
            }.environment(router)
        }
    }
}
```

The important part is the following line:

`.environment(router)`

This places the Router in the SwiftUI environment, making the same router instance available to views further down the view hierarchy.

We also bind the router's routes collection to the NavigationStack:

`NavigationStack(path: $router.routes)`

Now, whenever a route is added through `router.navigate(to:)`, the navigation path changes and NavigationStack presents the corresponding destination.

Since the Router is injected at the root level, technically any view in the hierarchy can access it. However, this does not mean that every view should depend on the Router.

A good practice is to access the Router from parent or container views and keep smaller child views independent of navigation whenever possible. This makes child views easier to reuse because they do not need to know how navigation is implemented in the application.

For example, a reusable registration form could simply communicate that registration has completed. The parent view can then decide how to respond to that event, including performing any necessary navigation.

With the router available through the environment, a view such as RegisterScreen can access it without receiving it explicitly through its initializer:

``` swift 
struct RegisterScreen: View {
    
    @Environment(Router.self) private var router
    
    var body: some View {
        Button("Register") {
            register()
        }
    }
    
    private func register() {
        // Register the user 
        // authenticationStore.register(...)
        
        // navigate to the courses screen 
        router.navigate(to: .student(.courses))
    }
}
```

At this point, navigation works, but our view is also making the decision about where the user should go. Next, we will look at why that can become a problem when navigation depends on business rules and how we can make that logic easier to unit test.

### Moving Navigation Decisions to the Router

In the previous implementation, `RegisterScreen` was responsible for deciding which screen should be displayed after registration. That approach works for simple cases, but it becomes harder to maintain once navigation depends on business rules.

For example, a student may navigate to different screens depending on their status. An in-state student can go directly to the courses screen, while an out-of-state or international student may need to review an agreement first. Faculty members have a completely different destination.

Instead of placing this logic inside the view, we can move it into the `Router`.

```swift
@Observable
class Router {
    
    var routes: [Route] = []
    
    func navigate(to route: Route) {
        routes.append(route)
    }
    
    func onRegister(_ user: User) {
        switch user.role {
        case .student:
            switch user.studentStatus {
            case .inState:
                navigate(to: .student(.courses))
                
            case .outOfState, .international:
                navigate(to: .student(.agreement))
                
            case .none:
                navigate(to: .student(.profile))
            }
            
        case .faculty:
            navigate(to: .faculty(.dashboard))
        }
    }
}
```

The `onRegister(_:)` function now contains the navigation rules associated with registration. It examines the registered user and determines the appropriate route.

This also makes `RegisterScreen` much simpler:

```swift
private func register() {
    
    // Register the user
    
    let user = User(
        name: "John Doe",
        role: .student,
        studentStatus: .inState
    )
    
    router.onRegister(user)
}
```

The view no longer needs to know whether an in-state student should go to the courses screen or whether an international student should go to the agreement screen. It simply tells the router that registration has completed.

More importantly, this gives us something concrete that we can unit test. Since `onRegister(_:)` changes the router's `routes` collection based on the supplied user, we can test these navigation rules without launching the application, rendering a SwiftUI view, or writing a UI test.

### Unit Testing the Navigation Logic

Now that the navigation decisions have been moved out of the SwiftUI view and into the `Router`, we can test those rules using regular unit tests.

This is one of the main advantages of separating navigation logic from the view. We do not need to launch the application, interact with buttons, or verify which screen is currently visible. Instead, we can provide the router with a user and verify that it produces the expected route.

The following test verifies that a faculty member is sent to the dashboard after registration:

```swift
import Testing
@testable import AZSchool

struct RouterTests {

    @Test
    func `When faculty registers successfully, navigate to the dashboard screen`() {
        
        let router = Router()
        let user = User(
            name: "John Doe",
            role: .faculty
        )
        
        router.onRegister(user)
        
        #expect(router.routes == [.faculty(.dashboard)])
    }
}
```

The test follows a simple pattern. We create a `Router`, create a user representing the scenario we want to test, call `onRegister(_:)`, and then verify the resulting navigation path.

```swift
#expect(router.routes == [.faculty(.dashboard)])
```

We are not testing SwiftUI here. We are testing a navigation rule: **when a faculty member registers successfully, the next route should be the faculty dashboard.**

We can use the same approach for students. In our application, both out-of-state and international students should navigate to the agreement screen. Since both statuses have the same expected behavior, this is a good use case for a parameterized test.

```swift
@Test(arguments: [
    StudentStatus.outOfState,
    StudentStatus.international
])
func `When Out-of-State or International student registers successfully, navigate to the agreement screen`(
    studentStatus: StudentStatus
) {
    
    let router = Router()
    
    let user = User(
        name: "John Doe",
        role: .student,
        studentStatus: studentStatus
    )
    
    router.onRegister(user)
    
    #expect(router.routes == [.student(.agreement)])
}
```

Instead of writing two nearly identical tests, Swift Testing runs this test once for each value supplied through `@Test(arguments:)`.

The important point is that our navigation behavior is now represented as data that can be inspected:

```swift
router.routes
```

This makes navigation logic much easier to test than navigation decisions buried inside button actions or SwiftUI view bodies.

We can continue adding tests for the remaining rules, such as verifying that an in-state student navigates to the courses screen and a student without a status navigates to the profile screen. At that point, the important registration navigation paths can be covered with fast unit tests instead of slower UI tests.

### Scaling Navigation Logic with a Registration Coordinator

For this example, keeping `onRegister(_:)` inside the `Router` works well. The navigation rules are small, easy to understand, and easy to test.

As the application grows, however, we should be careful not to move every navigation decision into the `Router`. Otherwise, the router can slowly become responsible for registration, authentication, onboarding, checkout, subscriptions, and many other workflows.

In a larger application, feature-specific navigation rules can be moved into a coordinator.

For example, a `RegistrationCoordinator` can determine where the user should go after registration:

```swift
struct RegistrationCoordinator {
    
    func destination(for user: User) -> Route {
        switch user.role {
        case .student:
            switch user.studentStatus {
            case .inState:
                return .student(.courses)
                
            case .outOfState, .international:
                return .student(.agreement)
                
            case .none:
                return .student(.profile)
            }
            
        case .faculty:
            return .faculty(.dashboard)
        }
    }
}
```

The `Router` can then remain focused on managing the navigation path:

```swift
@Observable
class Router {
    
    var routes: [Route] = []
    
    func navigate(to route: Route) {
        routes.append(route)
    }
}
```

The `RegisterScreen` can use the coordinator to determine the next destination and then ask the router to perform the navigation:

```swift
struct RegisterScreen: View {
    
    @Environment(Router.self) private var router
    
    private let registrationCoordinator = RegistrationCoordinator()
    
    var body: some View {
        Button("Register") {
            register()
        }
    }
    
    private func register() {
        
        // Register the user
        
        let user = User(
            name: "John Doe",
            role: .student,
            studentStatus: .inState
        )
        
        let route = registrationCoordinator.destination(for: user)
        router.navigate(to: route)
    }
}
```

This gives each type a more focused responsibility. The `RegistrationCoordinator` decides **where the user should go after registration**, while the `Router` is responsible for **performing the navigation**.

It also makes the registration navigation rules easy to test independently:

```swift
@Test
func `Faculty registration destination is dashboard`() {
    
    let coordinator = RegistrationCoordinator()
    let user = User(
        name: "John Doe",
        role: .faculty
    )
    
    let route = coordinator.destination(for: user)
    
    #expect(route == .faculty(.dashboard))
}
```

This does not mean every application needs a coordinator. For a smaller application, keeping a method such as `onRegister(_:)` directly on the `Router` can be perfectly reasonable.

The coordinator becomes useful when the navigation rules for a feature start growing and you want to prevent the router from becoming responsible for too many application workflows.

### Conclusion 

Navigation is often treated as a UI concern, but the decisions that determine where a user should go can be part of our application logic.

By representing destinations as routes, we turn navigation into something that can be inspected and tested. Instead of launching the application and verifying every navigation scenario through UI tests, we can provide the appropriate input and verify that the expected route was produced.

For smaller applications, keeping navigation decisions directly in the Router can work perfectly well. As those decisions become more complex, they can be moved into feature-specific coordinators, allowing the router to remain focused on managing the navigation path.

This does not mean that every navigation action needs additional abstractions or unit tests. If tapping a student always opens the student details screen, calling router.navigate(to:) directly may be all you need.

The value of testing navigation becomes much more apparent when the destination depends on user roles, permissions, account status, registration state, or other business rules. In those situations, separating the navigation decision from the UI gives us code that is easier to understand, easier to change, and most importantly, easier to test.

### Learn More at AzamSharp School

If you enjoyed this article and want to continue learning, check out **AzamSharp School**.

You will find practical courses, live workshops, books, and one-on-one coaching covering SwiftUI, SwiftData, iOS architecture, testing, AI, machine learning, and more.

Visit [AzamSharp School](https://azamsharp.school) to explore all available resources.
