# Protecting SwiftUI Views with Authentication 

Most applications contain a mix of public and protected content. A user may be able to explore the Home or Contact screen without signing in, but accessing Courses or Profile requires authentication.

One way to handle this is to add authentication checks directly inside every protected screen. This works, but it also spreads the same logic throughout the application.

In this article, we will build a reusable RequiresAuthentication container view that keeps this decision in one place. We will start by displaying the login screen directly, then present it as a sheet and handle what happens when the user dismisses it without logging in. 

Finally, we will look at a view modifier as an alternative implementation.

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

### Inspiration from React 

React, Flutter, Jetpack Compose, and SwiftUI are all declarative UI frameworks. They share many of the same ideas, which is why I like to keep a close eye on what developers in these communities are building.

The inspiration for this article came from my work in React. I was using JSON Web Tokens to secure server routes, but I also needed a way to prevent unauthenticated users from accessing protected routes in the React application.

In React, this is commonly handled using a wrapper component called ProtectedRoute. This made me wonder: what would the SwiftUI version of this pattern look like?

It turns out that we can apply the same idea to SwiftUI by creating a reusable container view that displays its content only when the user is authenticated.

### Requires Authentication View 

The `RequiresAuthentication` view acts as a gatekeeper for protected content. It reads the current authentication state from the environment and decides which view to display.

If the user is authenticated, it displays the content passed to the container. Otherwise, it displays the login screen.

``` swift 
struct RequiresAuthentication<Content: View> : View {
    
    @Environment(AuthenticationStore.self) private var authenticationStore
    
    @ViewBuilder
    let content: () -> Content
    
    var body: some View {
        if authenticationStore.isAuthenticated {
            content()
        } else {
            LoginScreen()
        }
    }
}
```

The Content generic parameter allows the container to work with any SwiftUI view, while `@ViewBuilder` allows us to pass one or more views using trailing-closure syntax.

Here is how we can use the `RequiresAuthentication` container inside a TabView:

``` swift 
struct TabRootScreen: View {
    var body: some View {
        TabView {
            Tab("Home", systemImage: "house") {
                Text("Home")
            }

            Tab("Courses", systemImage: "book") {
                RequiresAuthentication {
                    Text("Courses")
                }
            }

            Tab("Profile", systemImage: "person") {
                RequiresAuthentication {
                    Text("Profile")
                }
            }
        }
    }
}
```

The Home tab is available to everyone, while the Courses and Profile tabs require authentication. When an unauthenticated user selects either protected tab, the `RequiresAuthentication` container displays LoginScreen. After the user successfully logs in, the authentication state changes and SwiftUI automatically displays the requested content.

This allows users to explore the public areas of the application without signing in while still protecting content that requires an account. In the next section, we will discuss how our flow will change when the login screen is displayed in a sheet. 

### Presenting the Login Screen as a Sheet

Displaying the login screen directly in place of the protected content works, but sometimes we may want to present it as a sheet instead.

This introduces an important question: what should happen if the user dismisses the sheet without logging in?

Since the protected content cannot be displayed, we can return the user to the previously selected tab. This allows public tabs such as Home and Contact to remain accessible without requiring authentication.

First, we will update RequiresAuthentication to present LoginScreen in a sheet:

``` swift 
struct RequiresAuthentication<Content: View>: View {
    
    @Environment(AuthenticationStore.self) private var authenticationStore
    @State private var isPresented: Bool = false
    let onAuthenticationDismissed: () -> Void
    
    @ContentBuilder
    let content: () -> Content
    
    var body: some View {
        
        Group {
            if authenticationStore.isAuthenticated {
                content()
            } else {
                Color.clear
            }
        }.onChange(of: authenticationStore.isAuthenticated, initial: true) {
            isPresented = !authenticationStore.isAuthenticated
        }
        .sheet(isPresented: $isPresented, onDismiss: {
            if !authenticationStore.isAuthenticated {
                onAuthenticationDismissed()
            }
        }) {
            LoginScreen()
        }
    }
}
```

The `isPresented` property controls the presentation of the sheet. When the view first appears, or when the authentication state changes, it is updated using the inverse of isAuthenticated.

If the user successfully logs in, isAuthenticated becomes true and the sheet is dismissed. The protected content is then displayed automatically.

If the user dismisses the sheet without logging in, `onAuthenticationDismissed` is called. The container does not decide where the user should go. Instead, it reports the event to its parent and allows the parent to make that decision.

Next, we will keep track of the selected and previously selected tabs:

``` swift 
enum AppTab {
    case home
    case courses
    case profile
    case contact
}

struct TabRootScreen: View {
    
    @State private var selectedTab: AppTab = .home
    @State private var previouslySelectedTab: AppTab = .home
    
    var body: some View {
        TabView(selection: $selectedTab) {
            Tab("Home", systemImage: "house", value: .home) {
                Text("Home")
            }
            
            Tab("Courses", systemImage: "book", value: .courses) {
                RequiresAuthentication(
                    onAuthenticationDismissed: {
                        selectedTab = previouslySelectedTab
                    }
                ) {
                    Text("Courses")
                }
            }
            
            Tab("Profile", systemImage: "person", value: .profile) {
                RequiresAuthentication(
                    onAuthenticationDismissed: {
                        selectedTab = previouslySelectedTab
                    }
                ) {
                    Text("Profile")
                }
            }
            
            Tab("Contact", systemImage: "heart", value: .contact) {
               Text("Contact")
            }
        }.onChange(of: selectedTab) { oldValue, _ in
            previouslySelectedTab = oldValue
        }
    }
}
```

Whenever the selected tab changes, we save the old value in previouslySelectedTab.

For example, if the user moves from Contact to Profile, the login sheet is displayed. If they dismiss the sheet without logging in, the closure changes selectedTab back to Contact. If they successfully log in, they remain on Profile and the protected content is displayed.

This keeps the authentication container reusable. It handles authentication and presentation, while TabRootScreen remains responsible for tab selection.

### Using a View Modifier

We can also implement the same behavior using a custom view modifier:

``` swift 
struct RequiresAuthenticationModifier: ViewModifier {
    @Environment(AuthenticationStore.self)
    private var authenticationStore

    func body(content: Content) -> some View {
        if authenticationStore.isAuthenticated {
            content
        } else {
            LoginScreen()
        }
    }
}
```

``` swift 
extension View {
    func requiresAuthentication() -> some View {
        modifier(RequiresAuthenticationModifier())
    }
}
```

This allows us to protect a view using standard SwiftUI modifier syntax:

``` swift 
Tab("Courses", systemImage: "book") {
    Text("Courses")
        .requiresAuthentication()
}
```

The modifier produces a cleaner call site, but I prefer the container approach:

``` swift 
RequiresAuthentication {
    Text("Courses")
}
```

The container makes the authentication boundary more explicit. It clearly communicates that the enclosed content will only be displayed after the user is authenticated.

The modifier remains a perfectly valid alternative, especially if you prefer the fluent syntax commonly used throughout SwiftUI.

### Conclusion 

SwiftUI’s composition system makes it easy to build reusable authentication boundaries without introducing a complicated routing or navigation architecture.

The RequiresAuthentication container has a simple responsibility: display the requested content when the user is authenticated and display the login experience when they are not. The parent view remains responsible for navigation decisions, such as returning the user to the previously selected tab when login is canceled.

You can implement the same idea using a view modifier, but I prefer the container because it makes the authentication boundary explicit and easy to recognize at the call site.

Keep in mind that this approach only controls what is displayed in the SwiftUI application. Your server must still validate every protected request. Hiding a screen is part of the user experience, not a replacement for securing the backend.

### Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school)
