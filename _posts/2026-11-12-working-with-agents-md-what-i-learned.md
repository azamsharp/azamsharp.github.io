# Working with AGENTS.md: What I Learned

An `AGENTS.md` file contains the conventions and guidelines that you want your AI coding agent to follow. It is a living document, which means you can update it as your project evolves.

The file is usually placed at the root of your project. You can also have multiple `AGENTS.md` files, each containing instructions for a particular directory or module. But I think it is a good idea to start with a single file and add more when needed.

In this article, I am going to use Codex integrated into Xcode to see how it follows the instructions defined in `AGENTS.md`. I will also explore what happens when we update those instructions during an active conversation and whether Codex can apply new conventions to existing code.

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

## Creating Our First AGENTS.md File

Let's start with a simple example. Create an `AGENTS.md` file at the root of your Xcode project and add the following instruction.

```md
# Foundation

- Every function should follow the snake_case naming convention.
```

Yes, I know this is silly because Swift does not use snake_case for function names. It uses camelCase. But I wanted to see whether the agent would follow my guidelines even when they were different from the standard Swift conventions.

I opened a new conversation in Xcode using Codex and entered the following prompt:

> Write a function in ContentView so it can toggle isOn state.

Here is the generated code:

```swift
struct ContentView: View {
    @State private var isOn = false

    private func toggle_is_on() {
        isOn.toggle()
    }

    var body: some View {
        VStack {
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundStyle(.tint)

            Text("Hello, world!")
        }
        .padding()
    }
}
```

As you can see, Codex followed the instructions from `AGENTS.md` and created a function called `toggle_is_on`. Of course, this is not following the standard Swift naming convention, so I updated the file to use camelCase instead.

```md
# Foundation

- Every function should follow the camelCase naming convention.
```

Since I updated the file during an active conversation, I asked Codex to reread it and update the existing code to follow the new guidelines. The expected result was to rename `toggle_is_on` to `toggleIsOn`.

One thing to keep in mind is that `AGENTS.md` is not a compiler or a rule enforcement system. It provides instructions that compatible AI coding agents can follow, but that does not mean the agent will always follow them. You still need to review the generated code.

Also, you don't have to explain basic Swift syntax or standard naming conventions in `AGENTS.md`. Modern coding assistants already understand Swift and SwiftUI. I think the real value of this file is communicating architectural decisions and conventions that are specific to your project.

## Adding Architectural Guidelines

In my SwiftUI applications, I prefer using the MV (Model View) pattern. Views manage their own presentation state, stores manage shared application state, and services handle operations such as networking. I want Codex to follow these conventions instead of introducing additional architectural layers that I don't need.

Here is an example of the guidelines I use:

```md
## Core Principles

1. No ViewModel classes. Views hold their own
   presentation state.

2. Use @State for local state, @Binding for
   parent-owned state, and @Observable for
   shared observable state.

3. Container views (*Screen) access stores.
   Child views (*View) receive data through
   parameters.

4. Stores are @Observable @MainActor classes
   with private(set) on mutable state.
   Commands are methods and queries are
   computed properties.

5. Services are stateless structs with no
   mutable stored properties.

6. Inject stores using .environment() at
   the narrowest useful scope.

7. Prefer @Observable and @Environment over
   ObservableObject and @EnvironmentObject
   for new code.

8. When a view exposes three or more event
   closures, consolidate them into an
   enum-based event pattern.

   Reference:
   References/EnumBasedEventPattern.md
```

These are just some of the conventions I prefer when building SwiftUI applications. Your guidelines may be different depending on your architecture and the requirements of your project.

Notice the last instruction about grouping multiple view events into an enum. Instead of explaining the entire pattern inside `AGENTS.md`, I referenced another file called `References/EnumBasedEventPattern.md`. This file contains an example of how to implement the pattern, so the agent can read it when needed.

I think this is a good approach because it allows us to keep `AGENTS.md` relatively small while providing detailed examples in separate files.

## Does Codex Read AGENTS.md on Every Request?

One question I had was whether Codex automatically rereads `AGENTS.md` every time I submit a new prompt. To test this, I added the following instruction:

```md
## IMPORTANT

- Make sure to read the AGENTS.md file on each
  coding assistant request/prompt and comply
  with the latest guidelines and conventions.
```

Of course, just because I added this instruction does not mean Codex will always reread the file. If the agent has not loaded the latest version, it may not even know that the instruction exists. But I wanted to see how it would behave.

I also added another naming convention:

```md
9. Swift struct names must begin with the letter S.
```

Yes, another silly convention. But that was the whole point of the experiment. I wanted to see whether Codex would follow instructions that were different from normal Swift conventions.

I asked Codex to add a function to `ContentView` to toggle the `isLightOn` state. Codex inspected the project guidelines and recognized the new naming convention, but it did not rename `ContentView` to `SContentView`.

Instead, it responded:

> The latest project rules add an S prefix convention for Swift structs. This request doesn't introduce or rename a struct, so I'll limit the edit to the requested state and function inside the existing ContentView.

This was interesting because Codex clearly understood the convention but decided not to apply it to the existing struct. When you think about it, this makes sense. I only asked it to add a function, not rename existing types. In most cases, this is actually the behavior we want because changing unrelated code can introduce unnecessary problems.

But what would happen if Codex had to create a new struct?

## Testing New SwiftUI Views

I asked Codex to create a new SwiftUI view that displays a Toggle for enabling dark mode. Here is the code it generated:

```swift
struct SettingsView: View {
    @State private var isDarkModeEnabled = false

    var body: some View {
        Form {
            Toggle(
                "Dark Mode",
                isOn: $isDarkModeEnabled
            )
        }
        .preferredColorScheme(
            isDarkModeEnabled ? .dark : .light
        )
    }
}
```

Notice that Codex created `SettingsView` instead of `SSettingsView`. This time, it was creating a new struct, so the previous explanation about preserving existing code did not apply. It simply did not follow the naming convention.

I started another conversation and explicitly asked Codex to read `AGENTS.md` and follow all project naming conventions. This time, it created `SDarkModeView`, which followed the instructions.

This made me wonder whether we always need to remind Codex to read `AGENTS.md`. To test this, I started another request without mentioning the file or any naming conventions.

My prompt was:

> Create a SwiftUI view that displays a list of books.

Codex inspected the project conventions and responded:

> The project requires every Swift struct name to begin with S, so I'll follow that convention for both the book model and views.

It then created a view called `SBookListView`. This time, Codex followed the convention without me explicitly asking it to reread `AGENTS.md`.

At this point, I had seen different results. Sometimes Codex followed the instructions, and sometimes it didn't. But I wanted to test one more thing. What happens if I change the naming convention during the same conversation?

## Updating AGENTS.md During an Active Conversation

I updated the naming convention from requiring an `S` prefix to requiring a `D` prefix.

```md
10. Swift struct names must begin with the letter D.
```

Without starting a new conversation, I asked Codex to create a new view called `HelloWorldView` that displays Hello World.

Here is the generated code:

```swift
import SwiftUI

struct DHelloWorldView: View {

    var body: some View {
        Text("Hello World")
    }
}

typealias HelloWorldView = DHelloWorldView

#Preview {
    HelloWorldView()
}
```

Codex created `DHelloWorldView` instead of `SHelloWorldView`, which means it was following the updated convention. But notice that it also introduced a typealias.

My prompt specifically asked for a view called `HelloWorldView`, while `AGENTS.md` required every Swift struct to begin with `D`. Codex tried to satisfy both requirements by creating `DHelloWorldView` and exposing it as `HelloWorldView` through a typealias.

Technically, this works, but I don't think I would want the agent creating typealiases just to satisfy naming conventions. I would rather have it use the correct name directly.

What I found even more interesting was the activity log in Xcode. Codex explicitly mentioned that it was going to reread the project conventions, and the log showed **AGENTS — 19 lines read**. It then explained that the current convention required every Swift struct to begin with `D`, and the project built successfully.

So, in this experiment, Codex did read the updated `AGENTS.md` file and followed the new convention without requiring a new conversation. This does not mean it will always reread the file on every request, but at least we know it can pick up changes during an active conversation.

## Can AGENTS.md Update Existing Code?

There was still one thing that bothered me. If the latest convention requires every Swift struct to begin with `D`, why didn't Codex update existing structs such as `ContentView`, `SettingsView`, or `SBookListView`?

My initial thought was that Codex was limiting its changes to the current request. So I decided to make the instructions more explicit.

I updated `AGENTS.md` with the following guidelines:

```md
## IMPORTANT

- Read AGENTS.md at the beginning of every coding
  request and follow the latest guidelines and conventions.

- Apply all conventions to both new and existing code.

- Before implementing a request, inspect the relevant
  existing code for violations of these conventions.

- Update existing components, structs, functions, and
  other declarations to comply with the latest conventions,
  even if they were created before the rules were added.

- When renaming or modifying existing code, update all
  affected references to prevent compilation errors.

- After making changes, build the project and resolve
  any errors introduced by the changes.
```

I thought this would be enough because I was explicitly telling Codex to update existing code, even if it was created before the new rules were added.

I asked Codex to add a function to `ContentView` to toggle the `isLightOn` state. Codex inspected the code and discovered that the function already existed.

```swift
private func toggleIsLightOn() {
    isLightOn.toggle()
}
```

It responded that no code changes were needed, built the project successfully, and finished. But it still did not rename `ContentView` to `DContentView`.

Maybe the problem was that the function already existed, so Codex decided there was nothing to do. To eliminate that possibility, I tried another prompt:

> Add a new @State property called isLoading to ContentView.

This request required an actual code change. I wanted to see whether Codex would also rename `ContentView` to `DContentView` to comply with the latest convention.

But it didn't. Codex handled the requested change without performing the broader naming updates.

At this point, I had explicitly instructed Codex through `AGENTS.md` to update existing code, but it was still not doing it during normal feature requests. I wanted to see whether it would behave differently if I asked it to perform a project audit.

## Asking Codex to Audit the Entire Project

Instead of asking Codex to add another feature, I entered the following prompt:

> Audit the entire project against AGENTS.md. Fix all violations, including renaming existing Swift structs to begin with D. Update all references and build the project.

And this time, it worked! Codex updated the existing structs to follow the naming convention.

I think this was one of the most interesting results from my experiments. I had already instructed Codex through `AGENTS.md` to apply conventions to both new and existing code, but it did not perform the broader updates when I asked it to add a feature. When I explicitly asked it to audit the project, it performed the refactoring.

This suggests that Codex treats the instructions in `AGENTS.md` as guidelines for completing the current task. Even when those guidelines mention updating existing code, it may still limit its changes to the work you explicitly requested.

One thing to keep in mind is that `AGENTS.md` does not automatically enforce conventions across your entire project. It provides instructions to the agent, but if you want existing code throughout your application to follow new conventions, you may need to explicitly ask for that work.

Also, just because your project builds successfully does not mean it follows all the conventions defined in `AGENTS.md`. A successful build tells us that the code compiles, not whether it follows your architectural decisions or naming conventions.

## What Did I Learn?

After running these experiments, I learned that Codex can follow conventions defined in `AGENTS.md`, even when those conventions go against standard Swift practices. I also found that Codex can reread the file during an active conversation and pick up updated instructions without requiring a new conversation.

But the results were not always consistent. In one experiment, Codex created `SettingsView` instead of `SSettingsView`. In another experiment, it correctly created `SBookListView`. It also introduced a typealias when trying to satisfy both my naming convention and the name I requested in the prompt.

Another important observation was that Codex considers the scope of the task. When I asked it to add a function or a state property to `ContentView`, it did not rename the existing struct, even though my `AGENTS.md` explicitly instructed it to update existing code. But when I asked it to audit the entire project, it performed the refactoring.

Now, these are just my observations from a few experiments. I am not saying that Codex will behave exactly the same way every time. But I think they demonstrate why we still need to review the code generated by AI agents instead of assuming they will always follow our guidelines.

## Do We Really Need to Document Everything in AGENTS.md?

One thing I have been exploring recently is whether we really need to document every architectural convention in `AGENTS.md`.

For example, let's say I scaffold my SwiftUI application myself. I create the models, stores, views, function signatures, and comments describing what each function should do. If my project already contains working examples of the architectural patterns I prefer, shouldn't the AI agent be able to infer those patterns from the existing code?

I think it should be able to, as long as it examines the relevant code and has enough context to understand the patterns.

This raises an interesting question. **If the code itself communicates the architecture, how much additional value does AGENTS.md provide?**

For example, if my project already uses the MV pattern, contains `@Observable` stores, and uses enum-based events for reusable views, do I need to repeat all of those conventions in `AGENTS.md`? Or can the agent figure them out by looking at the existing code?

This is something I have been researching, and I plan to explore it further through experiments. For now, I prefer keeping `AGENTS.md` focused on guidelines that may not be obvious from the existing codebase.

## Final Thoughts

Keep in mind that `AGENTS.md` should reflect your own guidelines and architectural preferences. If you're working with a team, it is a good idea to discuss these conventions and make sure everyone is on the same page.

You can include guidelines related to architecture, testing, deployment, third party libraries, error handling, and other aspects of your application. You can also reference existing code files or documentation to help the agent understand how a particular pattern should be implemented.

My recommendation is to start small. You don't need to create a large `AGENTS.md` file before writing your first line of code. As you work with your AI coding agent, you may notice certain patterns or conventions that it does not follow. Those are good candidates to add to your `AGENTS.md` file.

But don't assume that adding a rule guarantees the agent will follow it every time. And if you want the agent to update existing code to follow new conventions, don't assume it will automatically do that either. As my experiments showed, you may have to explicitly ask it to audit and update the project.

The goal is not to write as many instructions as possible. The goal is to provide enough guidance so the agent can work within the architecture you have defined. And most importantly, just because AI can generate code does not mean we should stop understanding the code it produces.

## SwiftUI Architecture Book

If you're looking for an `AGENTS.md` file for your SwiftUI projects, check out my **SwiftUI Architecture: Patterns and Practices for Building Scalable Applications** book.

The book covers practical architectural patterns for building SwiftUI applications that remain maintainable as they grow. It also comes with a downloadable `AGENTS.md` file that you can use as a starting point for your own projects.

[Get the SwiftUI Architecture Book](https://azamsharp.school/swiftui-architecture-book.html)