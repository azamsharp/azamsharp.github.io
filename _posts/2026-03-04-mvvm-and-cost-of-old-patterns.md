

# MVVM and the Cost of Carrying Old Patterns Forward

This chapter is part of my book *SwiftUI Architecture: Patterns and Practices for Building Scalable Applications*.

More details: [https://azamsharp.school/swiftui-architecture-book.html](https://azamsharp.school/swiftui-architecture-book.html)

I first came across the MVVM pattern around 2008 or 2009 while working for a very large oil and gas company. At the time, we were building a WPF application used by oil workers on offshore rigs to order parts and equipment. Our team had six to eight developers, and the application was deployed across most of the rigs owned by the company. This was not a small internal tool. It was a real production system that people depended on every single day, often in harsh and unpredictable environments where reliability was not optional.

A few years later, I joined another company, this time in the computer retail business. It is one of the top five hardware providers in the world. Their project was an ASP.NET web application built using MVC, but it also borrowed several ideas from MVVM. I worked on some of the core parts of that system and gained a broader perspective on how MVVM could exist outside the desktop world.

Beyond those two projects, I worked on several other systems that used MVVM in different forms. I genuinely enjoyed working with the pattern, especially when it was paired with WPF. The combination of XAML views and C# view models felt natural. It encouraged cleaner code, clearer responsibilities, and systems that were easier to understand and maintain over time.

MVVM was popularized and formally named by John Gossman, a Microsoft architect working on WPF. However, the ideas behind MVVM existed even earlier. A closely related pattern called Presentation Model was described by Martin Fowler in a 2004 article.

In that article, Fowler talked about manually synchronizing view properties with a presentation model and then keeping everything in sync as the application evolved. He also shared his frustration with that approach and wrote:

> "Probably the most annoying part of Presentation Model is the synchronization between Presentation Model and view. It's simple code to write, but I always like to minimize this kind of boring repetitive code. Ideally some kind of framework could handle this, which I'm hoping will happen some day with technologies like .NET's data binding."

Fowler turned out to be right. In 2006, Microsoft released Windows Presentation Foundation, which introduced powerful data binding features between XAML views and their associated C# view models. These capabilities removed much of the repetitive synchronization code developers previously had to write by hand. For the first time, MVVM felt natural instead of forced.

Years later, in 2013, React was announced and quickly changed how many developers thought about building user interfaces on the web. Today, it is consistently ranked as one of the most loved frameworks by web developers. That was not always the case. Early on, many developers were uncomfortable with React because it mixed HTML, CSS, and JavaScript in the same file. To them, this felt like a violation of separation of concerns.

Over time, the community began to see things differently. React still respected separation of concerns, but it organized code around behavior instead of file types. Everything related to login lived in a Login component. Everything related to photos lived in a Photos component. This shift took time, but once it clicked, it changed how many developers approached application architecture.

One of the most common patterns used in React applications is the Container and Presenter pattern, which we will discuss later in this book. While MVVM can technically be applied in React, it is rarely used in practice. After talking with many React developers over the years, I have yet to meet one who actively uses MVVM in a React codebase. The reason is simple. React components already manage state and binding directly, which makes a separate ViewModel layer unnecessary in most cases.

So why spend so much time talking about WPF and React in a chapter about MVVM and SwiftUI?

Because context matters. More importantly, understanding why a pattern exists and what problem it was designed to solve matters even more.

In the next section, I will walk through my early attempts to apply MVVM in SwiftUI and explain why it often felt like I was fighting the framework instead of working with it.

<!-- Book Banner: SwiftUI Architecture Book -->
<div class="azam-book-banner" role="region" aria-label="SwiftUI Architecture Book Banner">
  <div class="azam-book-banner__inner">
    <div class="azam-book-banner__cover">
      <!-- Replace the src with your real book cover image -->
      <img
        src="https://azamsharp.school/images/swiftui-architecture-book-cover.png"
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

### MVVM in SwiftUI

SwiftUI was announced at WWDC 2019, and my first attempt to apply MVVM started on day one. I honestly do not remember why I chose MVVM at the time. Most likely it was because everyone else was doing it, and I followed along without questioning it. For the next three years, I primarily used MVVM with SwiftUI.

My process was simple. For each screen, I created a separate view model. If the view model had dependencies, I passed them through the initializer. If I was building a movies app with screens like MovieListScreen, AddMovieScreen, and MovieDetailScreen, I would end up with MovieListViewModel, AddMovieViewModel, and MovieDetailViewModel.

The term screen is important here. A screen is technically a SwiftUI view, but it is used as a container rather than something meant for reuse. If you come from a web background, you can think of a screen as a page. Smaller sections inside that page are views. Screens are not meant to be reusable, but views inside them can be, if they are designed correctly.

You could argue that not every screen needs a view model. But if you are following classical MVVM, the relationship between a screen and its view model is typically one to one. While it is possible to share a view model across multiple screens, most of the time you end up creating a separate one per screen. The reason is that the view model is expected to host all presentation logic for that screen.

Going back to our movies example, let us look at `MovieListViewModel`, which is used by `MovieListScreen` to populate the list of movies.

```swift
@Observable
class MovieListViewModel {
    
    var movies: [Movie] = []
    let httpClient: HTTPClient
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func loadMovies() async throws {
        movies = try await httpClient.loadMovies()
    }
}
````

This is a fairly standard view model implementation. `MovieListScreen` receives `MovieListViewModel` through initializer injection. Since `MovieListViewModel` depends on `HTTPClient`, that dependency also needs to be created.

```swift
struct MovieListScreen: View {
    
    let vm: MovieListViewModel
    
    var body: some View {
        List(vm.movies) { movie in
            Text(movie.name)
        }.task {
            do {
                try await vm.loadMovies()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}

#Preview {
    MovieListScreen(vm: MovieListViewModel(httpClient: HTTPClient()))
}
```

`MovieListScreen` depends on `MovieListViewModel`, and `MovieListViewModel` depends on `HTTPClient`.

If you plan to mock `HTTPClient`, introducing a protocol is a good idea.

At this point, everything works. The list loads correctly. You can add search, sorting, or any other presentation logic to `MovieListViewModel`.

Now the app needs a way to add new movies. To support this, we introduce `AddMovieScreen` and a new view model called `AddMovieViewModel`. `AddMovieViewModel` is responsible for validating user input and sending a request to create a movie. Once the movie is created, it needs to communicate back to `MovieListScreen` so the list can update. This communication can be handled using closures, bindings, or other mechanisms.

Below you can find the implementation of the `AddMovieViewModel`:

```swift
@Observable
class AddMovieViewModel {
    
    let httpClient: HTTPClient
    
    var name: String = ""
    var description: String = ""
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    var isFormValid: Bool {
        !name.isEmptyOrWhitespace && !description.isEmptyOrWhitespace
    }
    
    func saveMovie() async throws -> Movie? {
        let movie = Movie(name: name, description: description)
        return try await httpClient.createMovie(movie)
    }
}
```

`AddMovieViewModel` also depends on `HTTPClient` because it performs a POST request. Once the server creates the movie, it returns a new instance with an id, which is then passed back to the caller.

```swift
struct AddMovieScreen: View {
    
    @Bindable var vm: AddMovieViewModel
    let onSave: (Movie) -> Void
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        Form {
            TextField("Title", text: $vm.name)
            TextField("Description", text: $vm.description)
        }
        .toolbar {
            ToolbarItem(placement: .topBarTrailing) {
                Button("Done") {
                    Task {
                        if let movie = try await vm.saveMovie() {
                            onSave(movie)
                            dismiss()
                        }
                    }
                }.disabled(!vm.isFormValid)
            }
        }
    }
}
```

`AddMovieScreen` binds directly to `AddMovieViewModel` using the `Bindable` property wrapper. This allows the view to connect directly to the view model’s state.

Finally, `MovieListScreen` presents `AddMovieScreen` and wires up all the dependencies.

```swift
struct MovieListScreen: View {
    
    let vm: MovieListViewModel
    @State private var isPresented: Bool = false
    
    var body: some View {
        List(vm.movies) { movie in
            Text(movie.name)
        }.task {
            do {
                try await vm.loadMovies()
            } catch {
                print(error.localizedDescription)
            }
        }.toolbar {
            ToolbarItem(placement: .topBarTrailing) {
                Button("Add Movie") {
                    isPresented = true
                }
            }
        }.sheet(isPresented: $isPresented) {
            NavigationStack {
                AddMovieScreen(vm: AddMovieViewModel(httpClient: vm.httpClient)) { movie in
                    vm.movies.append(movie)
                }
            }
        }
    }
}
```

This works. But this is the moment where you need to step back and ask an important question.

Right now, the app has three to five screens, and there is already a noticeable amount of code dedicated to view models, dependency passing, and coordination between screens. What happens when the app grows to thirty or fifty screens? Does that mean thirty or fifty view models? And if so, how do those view models communicate with each other in a way that remains understandable?

I can tell you from experience that this quickly becomes difficult to manage. After a while, you lose track of which view model is the actual source of truth. The code becomes harder to reason about, and small changes start to have unexpected side effects.

I have often seen developers welcome complexity instead of actively trying to remove it. Maybe it is because complexity makes software development feel more valuable. Maybe it creates a false sense of control. Whatever the reason, complexity always comes with a cost.

To work around these issues, I have seen developers try to access environment objects inside view models so they can share global state. But the environment is not designed for that. It is only available inside views. Passing environment objects into view models just brings you back to the same problem you were trying to solve.

At this point, you might wonder whether view models themselves should be injected into the environment. Technically, this works. View models are observable, so they can live in the environment. But this goes directly against the core idea of MVVM, where a view model is tied to a specific screen.

Yes, there are exceptions. Authorization state is a good example. But most screens have very different needs. You would not reuse your LoginViewModel, which contains all the presentation logic for a login screen, inside a registration screen. The responsibilities are different.

These issues do not exist because SwiftUI is limited. They exist because of how we try to use it. We bring our past experience with us and force the framework to fit those expectations. I did exactly that. I posted countless questions on Stack Overflow about views not updating even though the data had changed. Instead of understanding how SwiftUI was designed to work, I added refresh flags and toggles to force updates.

Those were not solutions. They were workarounds. It is like owning a Ferrari and insisting it should be pulled by donkeys. Let the Ferrari be fast. Learn how to drive it.

### Apple’s Intentional Design for SwiftUI

Apple has been investing heavily in SwiftUI since its original release in 2019. This includes new views, property wrappers, macros, and even new frameworks like SwiftData, which are built primarily for SwiftUI. When we step back and look at all these additions, a pattern starts to emerge. Most of these features only work inside a SwiftUI view.

We already know that `@State`, `@Binding`, `@Environment`, and `@Bindable` are limited to views. But the same is true for other property wrappers such as `@AppStorage`, `@FocusState`, `@FetchRequest`, `@SectionedFetchRequest`, and `@Query`.

For Core Data and SwiftData, you can often find ways to move these into classes if you really want to. But at that point, you are no longer using the platform as it was designed. You are working around it.

That raises an important question. Why did Apple make these decisions? Why not allow these features to be used anywhere?

The answer becomes clearer when you look at SwiftUI as a whole. SwiftUI is opinionated. It expects state, identity, and lifecycle to live close to the view. These property wrappers are not just storage. They are hooks into the framework’s rendering, invalidation, and update system. That only works when SwiftUI fully controls the context in which they run.

SwiftData is a good example of this thinking. The `@Query` property wrapper is only available inside a view. Yes, you can use a `FetchDescriptor` in a class and return the results to a view, but once you do that, you lose automatic tracking. You are then responsible for keeping the user interface in sync with the data yourself.

Both `@Query` and `@FetchRequest` are highly optimized for SwiftData and Core Data. They are designed to keep data and UI aligned with minimal effort. Once you move that logic outside the view, you step off the fast path.

There was a talk at NSSpain by developers from SoundCloud where they shared a similar lesson. They had built their app using VIPER along with an internal framework. The app was rejected by Apple because it was significantly slower than competing apps. After removing their custom layers and adopting SwiftUI property wrappers directly, performance improved dramatically.

Once they started using SwiftUI the way it was designed to be used, the performance problems disappeared.

So what is Apple’s recommended approach?

Based on their code samples, documentation, WWDC sessions, and conversations with Apple engineers, a clear pattern emerges. SwiftUI views are expected to host presentation logic. Observable objects and stateless services are expected to host business logic. The framework handles the rest.

That simplicity is not accidental. It is the design.

### But What About Testing

One of the biggest arguments for using the MVVM pattern in SwiftUI is testing. Instead of letting the view host presentation logic, we move that logic into a view model so it can be tested. On the surface, this sounds reasonable.

Unlike React and Flutter, Apple does not provide an easy way to test logic that lives inside a view. In React, you can mount a view and directly test the logic it contains. Flutter supports widget testing as well. SwiftUI, however, does not offer any out of the box solution for testing view logic.

There are third party tools such as ViewInspector that make this possible, but they come at a cost. You are adding more complexity and another dependency to your application, all for the purpose of testing something that SwiftUI was never designed to expose directly.

So how do we test the logic inside a view? To answer that, we first need to ask what kind of logic actually belongs in a SwiftUI view. As mentioned earlier and as Apple has shown many times, views are meant to host presentation logic. That means the discussion is really about testing presentation logic.

Whether presentation logic should be tested depends on the kind of behavior the view is responsible for. In many cases, this behavior can be verified using Xcode previews. Apple has even suggested in WWDC sessions that previews should be treated as tests. Previews allow you to quickly validate different states and edge cases without writing a single line of test code.

If the logic inside a view starts to grow or becomes difficult to reason about, it should be extracted. This does not automatically mean introducing a view model. Often, the logic can live in a simple struct, function, or extension and be tested in isolation. A common example is form validation. Validation rules can be moved into a String extension and tested without any dependency on SwiftUI.

But what about application flow? How do we test navigation, presenting sheets, or making sure the right view is visible at the right time? The most reliable way to verify this type of behavior is through end to end or UI testing. This is the only approach that truly confirms the application behaves correctly from the user’s perspective.

A common criticism of end to end tests is that they are slow. To a degree, this is true. UI tests are slow because they exercise the application the same way a real user does. They tap buttons, navigate screens, and trigger real side effects. That work takes time. The key is to focus these tests on user stories rather than individual interface elements. As the flow is tested, the correctness of the underlying UI is validated naturally along the way.

We will explore testing strategies in much more detail in the Testing in SwiftUI chapter later in this book.

### Conclusion

SwiftUI applications can be written in many different ways. As a software developer, your primary responsibility is to reduce complexity. The architecture or pattern that helps you achieve simplicity, clarity, and long term maintainability is the one that makes the most sense.

If you believe you can achieve those goals using the traditional MVVM pattern in SwiftUI, then there is no reason to stop using it.

However, if creating separate view models for each screen, moving presentation logic into those view models, and injecting view models into views while also injecting the networking layer as a dependency into the view model starts to feel like added complexity, then it may be time to look at a different approach.

In the next chapter, we will cover the container and presenter pattern, which is the default pattern used when building React applications.

This chapter is part of my book *SwiftUI Architecture: Patterns and Practices for Building Scalable Applications*.  

Learn more about the book here: [https://azamsharp.school/swiftui-architecture-book.html](https://azamsharp.school/swiftui-architecture-book.html)