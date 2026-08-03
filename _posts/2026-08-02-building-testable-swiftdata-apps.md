# Building Testable SwiftData Applications

> **About this chapter**
>
> This chapter is from my upcoming book, **Testing in iOS**, which is currently a work in progress. I'm publishing selected chapters on azamsharp.com as I write the book to gather feedback from the community. If you have suggestions or comments, I'd love to hear them.

SwiftData makes it incredibly easy to create, save, and fetch models. That convenience can also make it tempting to write unit tests that simply verify those operations work. While those tests may increase your test count, they often provide very little confidence in your application because they're testing SwiftData rather than your own code.

In this chapter, we'll focus on writing unit tests that protect your application's business logic. You'll learn when to use an in-memory store, what kinds of tests are worth writing, and how to recognize tests that don't add much value. 

We'll also look at testing logic that lives outside of your SwiftData models and see how the new ResultsObserver API in iOS 27 makes those types easier to test. By the end of this chapter, you'll have a better understanding of how to build a test suite that gives you confidence when making changes without wasting time maintaining tests that don't matter.

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


### In-Memory vs. Persistent Stores 

Before we start writing unit tests for our domain logic, we need to decide which type of SwiftData store to use. SwiftData supports two types of stores: an in-memory store and a persistent store.

A persistent store saves data to disk. This is what you'll use in a production app because the data is still there the next time the app launches. While this is exactly what users expect, it isn't ideal for unit tests. If one test leaves data behind, another test may see that data and produce unexpected results.
An in-memory store works differently. It keeps all of the data in memory and never writes anything to disk. As soon as the model container is deallocated, the database disappears. This means every test can start with a fresh, empty database.

Throughout this chapter, we'll use an in-memory store. Since each test gets its own database, our tests remain isolated, repeatable, and easy to reason about.
An in-memory store isn't the right choice for every type of test. For example, if you're testing database migrations or verifying that data is restored when the app launches, you'll need to use a persistent store. Those scenarios depend on data being written to disk. For unit tests that focus on business logic, however, an in-memory store is usually the better choice.

## What Not to Test 

Over the years, I've reviewed many different codebases. One of the most common mistakes I see developers make is writing unit tests that verify third-party frameworks instead of their own code. The same thing happens in SwiftData applications.

Instead of testing their business rules and domain logic, developers end up writing tests that verify SwiftData's insert, save, and fetch operations. These tests don't provide much value because they aren't testing code you wrote. They are simply confirming that SwiftData works as documented.

Of course, if you work on the SwiftData team at Apple, these tests are incredibly valuable because they verify the framework itself. Likewise, if you're building your own framework or library, you should thoroughly test every feature you provide. But when you're building a business application, your focus should be on testing the behavior and business rules that make your application unique.

Let's look at an example of a test that does not provide any value to the overall codebase. 

``` swift 
  @Test
    func save_budget_successfully() throws {
        
        let budget = Budget(name: "Groceries", limit: 200)
        context.insert(budget)
	   try context.save() 
        
        let budgets = try context.fetch(FetchDescriptor<Budget>())
        
        #expect(budgets.count == 1)
        #expect(budgets.first?.name == "Groceries")
        #expect(budgets.first?.limit == 200)
    }
``` 

At first glance, this looks like a perfectly reasonable test. It creates a Budget, saves it to the database, and then fetches it back to verify that it was persisted successfully.

The problem is that this test isn't exercising any business logic that you wrote. It's simply verifying that SwiftData can insert, save, and fetch a model. Unless you've customized that behavior, you're testing Apple's framework instead of your application.

A more valuable unit test would verify one of your business rules. For example, does the application prevent duplicate budget names? Does it reject an empty budget name? Does it enforce a minimum spending limit? Those are behaviors implemented by your code, and those are the behaviors that deserve unit tests.

Unfortunately, I've seen these kinds of tests in real production applications. Developers spend time writing tests that verify insert, save, fetch, and delete, while the business rules that actually matter remain untested. On paper, the project looks like it has excellent test coverage. In reality, it doesn't.

These tests also create technical debt. Every unit test you write becomes part of your codebase and has to be maintained for the foreseeable future. As your models evolve, these tests need to be updated even though they aren't providing any meaningful confidence in your application. You're paying the cost of maintaining them without getting much value in return.

Write tests that protect your business logic, not tests that prove SwiftData is working. Apple already tests SwiftData. Your responsibility is to make sure your application's behavior is correct.

### Writing Good Unit Tests 

A good unit test verifies the behavior of your application. It doesn't verify that SwiftData can insert, save, or fetch models. Apple already has thousands of tests that do that. Your responsibility is to make sure the business rules in your application are working correctly.

Every application has business rules. These rules define what users can and cannot do. In our budgeting application, one of those rules is that two budgets cannot have the same name. Another rule might require that a budget name cannot be empty or that the spending limit must be greater than zero. These are the kinds of behaviors that deserve unit tests because they represent code that you wrote.

Let's look at a better unit test.
``` swift 
@MainActor
struct BudgetTests {

    let container: ModelContainer
    let context: ModelContext
    
    init() throws {
        container = try ModelContainer(for: Budget.self, configurations: ModelConfiguration(isStoredInMemoryOnly: true))
        context = container.mainContext
    }
    
    @Test(arguments: [
        ("Groceries", "groceries"),
        ("Rent", "RENT"),
        ("Entertainment", "entertainment")
    ])
    func `Saving budget with duplicate name throws error`(
        firstName: String,
        secondName: String
    ) throws {

        let firstBudget = Budget(name: firstName, limit: 100)
        try firstBudget.save(context: context)

        let secondBudget = Budget(name: secondName, limit: 200)

        #expect(throws: BudgetError.duplicateName) {
            try secondBudget.save(context: context)
        }
    }
}
```

This test isn't interested in whether SwiftData can save a model. Instead, it verifies that our application enforces the rule that budget names must be unique. If someone accidentally removes that validation six months from now, this test will fail immediately and alert us to the problem.

That's what makes a good unit test. It protects your business logic. As your application grows and changes, these tests continue to provide value because they verify the behavior that makes your application unique.
Let’s take a look at some other good unit tests. 

``` swift
  @Test
    func `Budget spent returns total of all expenses`() {
        
        let budget = Budget(name: "Groceries", limit: 100)
        
        let expense1 = Expense(name: "Milk", amount: 3.50, quantity: 5)
        let expense2 = Expense(name: "Eggs", amount: 4.00, quantity: 1)
        
        budget.expenses.append(expense1)
        budget.expenses.append(expense2)
        
        #expect(budget.spent == 21.50)
    }
```
The test above verifies that the spent computed property returns the correct total for all expenses in the budget. This is an important business rule because the rest of the budgeting experience depends on this value being accurate.

By surrounding this behavior with a unit test, we protect it from future changes. Someone may later update the expense model, change how quantities are handled, or refactor the calculation. If that change accidentally breaks the total, this test will catch the problem immediately.

Here is another example of a good unit test. 
``` swift 
  @Test
    func `Budget remaining returns budget balance`() {
        
        let budget = Budget(name: "Groceries", limit: 100)
        
        let expense1 = Expense(name: "Milk", amount: 3.50, quantity: 5)
        let expense2 = Expense(name: "Eggs", amount: 4.00, quantity: 1)
        
        budget.expenses.append(expense1)
        budget.expenses.append(expense2)
        
        #expect(budget.remaining == 78.50)
    }
```
This test verifies another important business rule. A budget's remaining balance should always reflect the budget limit after all expenses have been deducted. If this calculation is incorrect, users will no longer have an accurate view of how much money is left in their budget.

Like the previous example, this test protects an important piece of business logic. If someone later changes how expenses are calculated or introduces a bug while refactoring the Budget model, this test will fail and alert the developer that the remaining balance is no longer being calculated correctly.

### Refactoring Unit Tests

The unit tests you write are part of your application. They are checked into source control alongside your production code and will need to be maintained as your application evolves. Just because it's test code doesn't mean it gets a free pass. Test code should be held to the same quality standards as the rest of your codebase.

Right now, our unit tests are small and easy to read. Each test only requires creating a Budget and a couple of Expense objects. For tests like these, I would not create a helper function. The setup is simple enough that keeping it inside the test makes it easy to understand what's going on.

But what happens when the setup becomes more complicated?
Imagine you're testing an account balance. Before you can even write your assertion, you need to create a user, an account, a category, and one or more transactions.

``` swift 
let user = User(name: "John")
let account = Account(owner: user, balance: 500)
let category = Category(name: "Food")
let transaction = Transaction(amount: 50, category: category, account: account)
``` 

Now the setup is getting in the way. Instead of focusing on the behavior being tested, the reader has to work through several lines of object creation before reaching the assertion.

In situations like this, a helper function can make the test much easier to read.

``` swift 
let account = makeAccountWithTransactions()
#expect(account.balance == 450)
```

The test now gets straight to the point. The setup is still there, but it has been moved behind a descriptive helper function. When someone reads the test, their attention is immediately drawn to the behavior being verified rather than the mechanics of creating objects.
Helper functions also make your tests easier to maintain.

Imagine that ten different tests all require the same account with the same set of transactions. If the Account initializer changes tomorrow, you'll have to update all ten tests. If the setup lives in a helper function, you only need to update it in one place.

Another benefit is that helper functions make it easy to create variations of the same object. Let's say most of your tests need an active user, but a few require an inactive user or an administrator. Instead of repeating the entire setup, you can provide sensible defaults and override only the values that matter.

``` swift
func makeUser(
    name: String = "John",
    role: Role = .customer,
    isActive: Bool = true
) -> User {
    User(
        name: name,
        role: role,
        isActive: isActive
    )
}
```

Your tests become much more expressive.

``` swift 
let activeUser = makeUser()
let inactiveUser = makeUser(isActive: false)
let adminUser = makeUser(role: .admin)
```

Notice how easy it is to understand each scenario. You don't have to read through a long initializer to figure out what makes one user different from another.

Does that mean every repeated line of setup belongs in a helper function? Absolutely not. A few duplicated lines are often easier to understand than another layer of abstraction. My rule of thumb is simple: if the setup is short, keep it in the test. As the setup becomes more complex or starts appearing in multiple tests, that's usually a good indication that it's time to extract a helper function.

Like most things in software development, there isn't a hard rule. Use your judgment and choose the approach that makes your tests the easiest to read and maintain.
Testing Logic Outside SwiftData Models 

So far, the business logic we've tested has naturally belonged inside our SwiftData models. But that won't always be the case.

Imagine you're building a screen that displays the combined budget limit across all budgets. Calculating this value requires adding the limit of every Budget in the collection and displaying the result to the user.

Where should this logic live?

The most straightforward solution is to keep it inside the view.

``` swift 
struct BudgetListScreen: View {
    
    @Query private var budgets: [Budget]
    
    private var totalBudgets: Double {
        budgets.reduce(0) { $0 + $1.limit }
    }
    
    var body: some View {
        VStack {
            Text(totalBudgets, format: .currency(code: "USD"))
        }
        .padding()
    }
}
```

There is nothing wrong with this approach. In fact, this is probably how I would implement it in a real application. The calculation is short, easy to read, and only used by this screen.

One mistake I see developers make is extracting every computed property into a separate type just so it can be unit tested. More code does not automatically mean a better design.

The downside is that totalBudgets is difficult to unit test. Since it is a private computed property backed by @Query, you can't simply instantiate the view and call the property from your test.

To verify this calculation, you would need to create a SwiftData container, insert test data, construct the view, and inspect the rendered output using a library such as ViewInspector. You could also write a UI test that launches the application, inserts several budgets, navigates to the screen, and verifies the displayed value.

Both approaches work, but they require considerably more setup than the calculation itself.
Another option is to extract the calculation into a separate type.

``` swift 
struct BudgetSummary {
    
    func totalLimit(for budgets: [Budget]) -> Double {
        budgets.reduce(0) { $0 + $1.limit }
    }
}
```

Since BudgetSummary doesn't contain any mutable state, there is no reason to make it an @State property. We can simply create an instance and use it from the view.

``` swift 
struct BudgetListScreen: View {
    
    @Query private var budgets: [Budget]
    private let summary = BudgetSummary()
    
   
    var body: some View {
        VStack {
            Text(summary.totalLimit(for: budgets), format: .currency(code: "USD"))
        }
        .padding()
    }
}
```

Now writing a unit test becomes straightforward.

``` swift 
@Test
    func `Total limit returns sum of all budget limits`() {
        let groceries = Budget(name: "Groceries", limit: 100)
        let rent = Budget(name: "Rent", limit: 1000)

        let summary = BudgetSummary()

        let total = summary.totalLimit(for: [groceries, rent])

        #expect(total == 1100)
    }

```

Does this mean you should always extract calculations like this into a separate type?

Absolutely not.

If this is the only summary calculation your application needs, I would probably leave it inside the view. The original implementation is shorter, easier to read, and perfectly reasonable.

On the other hand, applications tend to grow over time. Today you only need the combined budget limit. Tomorrow you may also need the total amount spent, the remaining balance, the average budget limit, or a list of overspent budgets. At that point, a dedicated BudgetSummary type starts to make sense because it groups related calculations in one place.

The fact that BudgetSummary is easy to unit test is simply an added benefit. The real reason for introducing it is that it represents a meaningful concept in your application rather than existing solely to satisfy your test suite.

Like most architectural decisions, there isn't a single right answer. Start with the simplest solution that works. As your application grows, don't be afraid to refactor when the design calls for it.

In the next section, we'll look at ResultsObserver, introduced in iOS 27, which allows you to observe SwiftData changes outside of SwiftUI views. This is useful when your logic doesn't naturally belong in a view but still needs to react whenever the underlying data changes.

<!-- Book Banner: SwiftData Architecture Book -->
<div class="azam-book-banner" role="region" aria-label="SwiftData Architecture Book Banner">
  <div class="azam-book-banner__inner">
    <div class="azam-book-banner__cover">
      <img
        src="https://azamsharp.school/images/swiftdata-3d-cover.png"
        alt="SwiftData Architecture book cover"
        loading="lazy"
      />
    </div>
    <div class="azam-book-banner__content">
      <h3 class="azam-book-banner__title">
        SwiftData Architecture - Patterns and Practices for Building Scalable Applications
      </h3>
      <p class="azam-book-banner__subtitle">
        Learn SwiftData architecture, ModelContext, relationships, queries,
        migrations, CloudKit, testing, performance, custom stores, and the latest
        iOS 27 features including ResultsObserver, Compound Queries, Sectioned Queries,
        and the new .codable attribute.
      </p>
      <div class="azam-book-banner__actions">
        <a
          class="azam-book-banner__button"
          href="https://azamsharp.school/swiftdata-architecture.html"
          target="_blank"
          rel="noopener"
        >
          Get the book
        </a>
      </div>
    </div>
  </div>
</div>
<style>
  .azam-book-banner {
    --bg1: #07111f;
    --bg2: #111827;
    --text: rgba(255, 255, 255, 0.94);
    --muted: rgba(255, 255, 255, 0.74);
    --border: rgba(255, 255, 255, 0.12);
    --shadow: 0 18px 45px rgba(0, 0, 0, 0.28);
    --accent: #8fd6ff;
    --accent2: #9d7cff;
    margin: 22px 0;
    color: var(--text);
    border: 1px solid var(--border);
    border-radius: 16px;
    overflow: hidden;
    background:
      radial-gradient(1000px 520px at 10% 0%, rgba(143, 214, 255, 0.22), transparent 60%),
      radial-gradient(900px 520px at 90% 30%, rgba(157, 124, 255, 0.22), transparent 60%),
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
    max-width: 68ch;
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
    filter: brightness(1.04);
  }
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

## ResultsObserver: Observing SwiftData Outside SwiftUI Views

So far, we have used @Query whenever we needed to observe SwiftData models inside a SwiftUI view. When the underlying data changes, @Query refreshes the view automatically.

This works well for displaying data, but it becomes limiting when the logic you want to test does not naturally belong inside the view.

Consider a dashboard that displays the total budget, total spending, remaining balance, and a list of overspent budgets. You could calculate all of these values directly inside the view, but that would make the logic difficult to test without involving SwiftUI.
A better approach is to move the calculations into a dedicated store.

In iOS 27, ResultsObserver allows an @Observable type to observe SwiftData changes outside a SwiftUI view. This makes it a good fit for our BudgetSummaryStore.

``` swift 
@Observable
final class BudgetSummaryStore {

    private let observer: ResultsObserver<Budget, Never>?
    @ObservationIgnored private var token: ObservationTracking.Token?

    var totalBudget: Double = 0
    var totalSpent: Double = 0
    var remaining: Double = 0
    var overspentBudgets: [Budget] = []

    init(modelContext: ModelContext) throws {
        observer = try ResultsObserver(modelContext: modelContext)

        token = withContinuousObservation(options: [.didSet]) { [weak self] _ in
            self?.updateSummary()
        }
    }

    private func updateSummary() {
        guard let observer else { return }

        let budgets = observer.results

        totalBudget = budgets.reduce(0) { $0 + $1.limit }

        totalSpent = budgets.reduce(0) { result, budget in
            result + budget.expenses.reduce(0) { $0 + $1.amount }
        }

        remaining = totalBudget - totalSpent

        overspentBudgets = budgets.filter {
            spentAmount(for: $0) > $0.limit
        }
    }

    private func spentAmount(for budget: Budget) -> Double {
        budget.expenses.reduce(0) { $0 + ($1.amount * Double($1.quantity)) }
    }
}
```

The important part is that BudgetSummaryStore now owns the calculated values. A SwiftUI view can display those values, but it is no longer responsible for producing them.

More importantly, we can create the store inside a unit test, insert models into an in-memory SwiftData container, and verify the results without creating a view.

``` swift 
@MainActor
@Test
func `Budget summary store calculates correct aggregate totals`() async throws {
    let container = try ModelContainer(
        for: Budget.self,
        configurations: ModelConfiguration(
            isStoredInMemoryOnly: true
        )
    )

    let context = container.mainContext

    let food = Budget(
        name: "Food",
        limit: 500
    )

    food.expenses.append(
        Expense(
            name: "Groceries",
            amount: 125
        )
    )

    let travel = Budget(
        name: "Travel",
        limit: 1500
    )

    travel.expenses.append(
        Expense(
            name: "Hotel",
            amount: 600
        )
    )

    context.insert(food)
    context.insert(travel)

    try context.save()

    let store = try BudgetSummaryStore(
        modelContext: context
    )

    #expect(store.totalBudget == 2000)
    #expect(store.totalSpent == 725)
    #expect(store.remaining == 1275)
    #expect(store.overspentBudgets.isEmpty)
}
```

The purpose of this test is not to verify that SwiftData can insert or fetch models. Apple already tests that behavior.

We are testing our own business logic. Given a collection of budgets and expenses, does BudgetSummaryStore calculate the correct totals?
That is the main benefit of moving this logic outside the view. The view remains responsible for presentation, while the store contains business behavior that can be tested independently.

## Conclusion 

Writing unit tests for a SwiftData application isn't about proving that insert, save, or fetch work correctly. Apple has already invested countless hours testing those APIs. Your responsibility is to protect the business rules and behaviors that make your application unique.

In this chapter, you learned why an in-memory store is the preferred choice for most unit tests, how to identify tests that provide little value, and how to focus your efforts on testing business logic instead of framework behavior. You also saw how moving calculations into dedicated models or observable types makes them easier to test, and how ResultsObserver enables unit testing for logic that depends on collections of SwiftData models.

As your application grows, your tests should evolve alongside it. A well-designed test suite gives you the confidence to refactor, add new features, and fix bugs without worrying about introducing regressions. The goal isn't to have the most tests—it's to have the right tests. Those are the tests that continue to provide value long after they're written.
