
# SwiftUI Architecture Lessons I Wish I Knew Earlier

SwiftUI makes it very easy to build user interfaces.

You can put together a screen in minutes and everything feels simple at first. But as your app grows, that simplicity starts to fade. More screens, more state, more logic. Now you are asking different questions.

Where should this logic go.
How do I share data across screens,
Why is this view updating when it should not.

At this point, the problem is no longer SwiftUI. The problem is architecture.

After building real world apps and teaching SwiftUI to thousands of developers, one thing becomes very clear. Most issues come from unclear boundaries and mixing responsibilities.

This article is a collection of practical tips that I use in production apps. Nothing theoretical. Just patterns that work.

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

## Start with clear boundaries

If you take away one thing from this article, it should be this.

Keep responsibilities separate.

A simple pattern that works very well is to split your views into containers and presenters.

Containers are your screens. They fetch data, talk to stores or services, and prepare everything needed for rendering.

Presenters focus only on UI. They take data and display it. No networking. No business logic.

This small separation removes a lot of confusion. When something breaks, you know exactly where to look.

## Keep SwiftUI state where it belongs

SwiftUI has its own system for managing updates. Property wrappers like `@State`, `@Binding`, `@Environment`, and `@Query` are part of that system.

Do not move them out of views.

Trying to push them into view models or other layers usually leads to problems. Updates stop behaving correctly. Bugs become harder to track.

Keep UI state in the view. Move business logic somewhere else.

## Use composition aggressively

SwiftUI is built for composition.

If a view starts getting large, break it apart. Smaller views are easier to read, easier to test, and easier to reuse.

Another benefit is performance. When you pass only the data a subview needs, SwiftUI can update only the parts that actually changed.

Think in small building blocks, not big screens.

## Keep logic simple and close to the UI

Not all logic needs to be extracted.

Simple things like formatting text, toggling flags, or enabling a button can stay in the view. This keeps your code straightforward and avoids unnecessary abstractions.

But there is a line.

When logic starts growing or getting reused, move it out into a helper or a separate type. Do not let your views turn into mini view models.


## Introduce Stores when you need them

Stores are a great way to organize business logic.

They keep your data in one place and make it easier to keep your UI in sync. With the new observation system, using `@Observable` makes this even cleaner.

Start simple. You do not need five stores on day one.

As your app grows, split your stores based on domains. A user store, a product store, an inventory store. Each store should have a clear responsibility.

## Share state the right way

When multiple screens need access to the same store, do not pass it through every view manually.

Inject it at a higher level using the environment.

For example, placing a store at the NavigationStack level makes it available to every screen in that stack. This keeps your code clean and avoids unnecessary wiring.

## Be intentional about data flow

There is no single correct way to communicate between parts of your app.

For simple cases, closures work great. If a child view needs to send one result back, a closure is the simplest solution.

When interactions grow, use enums to represent events instead of passing multiple closures. This keeps your APIs clean.

For more complex scenarios, you can use observation, streams, or Combine. But do not reach for these tools too early.

Start simple and scale when needed.

## Use the environment for more than just data

The environment is not just for passing values. It can also be used to expose actions.

Things like navigation, showing sheets, or displaying toasts can be wrapped in environment values.

This allows any view to trigger these actions without knowing how they are implemented.

It keeps your views clean and your architecture flexible.

## Manage sheets and navigation cleanly

If you find yourself creating multiple boolean flags to control sheets, stop.

Use a single enum to represent all possible sheets and drive your UI from that. It scales much better and is easier to maintain.

Also, avoid stacking multiple sheets on top of each other. It makes the user experience confusing and the code harder to manage.

## Stateless services 

Networking, file uploads, and data imports should live in stateless services.

These services should not know anything about your UI. Their job is simple. Perform work and return results.

This makes them easy to reuse and easy to test.

In smaller apps, it is perfectly fine to call these services directly from a view. Just inject them through the environment.

## Use Swift concurrency correctly

Async work should follow the lifecycle of your views.

Use `.task` for loading data and async functions for background work. Update your state when results arrive.

This keeps things predictable and avoids issues with outdated data updating your UI.

## Handle validation without overengineering

For simple forms, keep validation inside the view.

A computed property like `isValid` is often enough to enable or disable a button.

When validation becomes more complex or reusable, extract it into a separate type or use property wrappers.

Keep it simple until it is not.

## Be mindful of performance details

SwiftUI re-evaluates views often. That is normal.

The important part is that it only re-renders what actually changes. You help this process by passing minimal data and keeping your views small.

Also, remember that `AsyncImage` does not cache images. For real apps, you will need a proper caching solution.

## Test what matters

Focus on testing your business logic. Stores and services are where bugs matter most.

For critical user flows, add end to end tests. That is where testing gives you real value.

## Final thought

Do not chase perfect architecture.

Start simple. Let your app grow. Refactor when needed.

The best architecture is not the most clever one. It is the one your team can understand and maintain.

Clarity always wins.

### Conclusion 

SwiftUI does not fail you. Your architecture does.

Most problems developers face are not because SwiftUI is hard. They come from unclear boundaries, too much responsibility in views, and trying to force patterns that do not fit.

The lessons in this article are not rules. They are guardrails.

Start with simple decisions. Keep responsibilities clear. Let SwiftUI do what it is good at. As your app grows, evolve your architecture instead of overengineering it from day one.

You do not need the perfect setup. You need a setup that your future self can understand in five minutes.

If your code is easy to read, easy to change, and easy to debug, you are on the right track.

That is what good SwiftUI architecture looks like.

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