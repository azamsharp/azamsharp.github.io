

# If You’re Not Versioning Your SwiftData Schema, You’re Gambling 

Let me say this clearly. If you are building a SwiftData app and you are not versioning your schema, you are gambling. It may not hurt you today. It may not hurt you next week. But eventually it will. Everything works when the database is empty. You add a model, you run the app, you insert some data, and when something breaks you just delete the app and reinstall. That is development mode. That is not production reality.

Reality begins the moment real users have real data stored on their devices. Budgets. Transactions. Notes. Profiles. Anything that matters. Now imagine you ship an update where you add one small property. Or maybe you enforce a uniqueness constraint. Or maybe you split one field into two because the domain became clearer. It feels harmless. It feels like a small refactor. But underneath, you just changed the contract between your code and the data already sitting on disk. If that change is not handled properly, your app will not “adjust.” It will crash.

Schema evolution is not an advanced topic reserved for large scale apps. It is a day one concern the moment you persist data beyond a single launch. The second your app stores something that survives reinstalling, you have made a long term commitment to that structure. And structures evolve. Requirements change. Models mature. Relationships grow more complex. What started as a simple Budget with a name and limit suddenly needs a description, categories, constraints, maybe even Cloud sync compatibility. If you did not plan for evolution, you are hoping your assumptions hold forever. They will not.

SwiftData actually gives you excellent tools to handle this responsibly. VersionedSchema. SchemaMigrationPlan. Custom migration stages with willMigrate and didMigrate. These are not optional extras. They are the mechanisms that let your app grow without breaking the trust of your users. Ignoring them and hoping automatic migration covers every scenario is not engineering discipline. It is rolling dice in production and hoping nothing explodes.

In this article, we are going to walk through what schema versioning looks like in a real SwiftData application, why automatic migration is not always enough, and how to implement custom migrations in a way that feels controlled and intentional. Because evolving your data model should feel like architecture, not damage control.

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

### Initial Schema 

Imagine you are building a budget tracking application. One of the core features is allowing users to create multiple budgets, each with its own spending limit. In the beginning, the model might be very simple. Just a name and a limit. Nothing fancy. Something like this:

``` swift 
    @Model
    class Budget {
        
        var name: String = ""
        var limit: Double = 0.0
        
        init(name: String, limit: Double) {
            self.name = name
            self.limit = limit
        }
    }
```

This works perfectly fine for version one of your app. You can create budgets, persist them, and everything behaves as expected. But here is the part many developers overlook. The moment your app ships, your schema is no longer just code. It becomes stored data on real devices. And once data is out there, changing the model is no longer trivial.

That is why it is always a good idea to version your schema from day one. Even if your model looks simple. Even if you think it will never change. Because it will.

SwiftData provides the `VersionedSchema` protocol to formally define and version your model schema. Instead of defining Budget at the top level, we wrap it inside a schema version like this: 

``` swift 
enum BudgetAppSchemaV1: VersionedSchema {
    
    static let versionIdentifier: Schema.Version = .init(1, 0, 0)
    static var models: [any PersistentModel.Type] { [BudgetAppSchemaV1.Budget.self] }
    
    @Model
    class Budget {
        
        var name: String = ""
        var limit: Double = 0.0 
        
        init(name: String, limit: Double) {
            self.name = name
            self.limit = limit
        }
    }
}
```

Now the Budget model lives inside `BudgetAppSchemaV1`. This makes it clear that this structure represents version 1.0.0 of your data model. When future changes come, you can introduce V2, V3, and so on, without breaking existing user data.

If you plan to sync your data with iCloud, there is another important rule. All stored properties must either have a default value or be optional. Otherwise CloudKit will complain. This is not optional advice. It is a requirement.

One small inconvenience of nesting the model inside the schema is that you now have to reference it as `BudgetAppSchemaV1.Budget`. That can get verbose. A simple typealias keeps your code clean:

``` swift 
typealias Budget = BudgetAppSchemaV1.Budget
```

Now the rest of your codebase can continue using Budget without caring which schema version it belongs to. This keeps your implementation flexible while your app continues to evolve.

Let’s say a few weeks later a new requirement comes in. Each budget must now include a description property. Not only that, but for all existing budgets, the description should be automatically generated based on the budget name. Maybe a “Groceries” budget gets a default description like “Monthly grocery allocation.” Maybe “Travel” gets something different.

If the requirement was simply to add an optional description, SwiftData would handle it automatically. Optional properties are migrated for you. No custom migration required.

But that is not what we are doing here.

We are adding a new stored property and assigning it meaningful default values based on existing data. That changes everything. This is no longer a lightweight migration. This requires a custom migration strategy.

Let’s implement that in the next section.

### Adding Budget Description with Default Values

So the first step is creating version 2 of our schema. 

``` swift 
enum BudgetAppSchemaV2: VersionedSchema {
    
    static let versionIdentifier: Schema.Version = .init(2, 0, 0)
    static var models: [any PersistentModel.Type] { [BudgetAppSchemaV2.Budget.self] }
    
    @Model
    class Budget {
        
        var name: String = ""
        var limit: Double = 0.0 
        var desc: String = ""
        
        init(name: String, limit: Double, desc: String = "") {
            self.name = name
            self.limit = limit
            self.desc = desc
        }
    }
}
```

Notice that we added a new stored property `desc`. I gave it a default empty string value. You could also make it optional if your use case allows that. But in our case, we are not leaving it empty. During migration we will assign a meaningful value derived from the budget name.

At this point, defining V2 alone is not enough. SwiftData needs to know how to move existing data from V1 to V2. That is where a migration plan comes in.

``` swift 

enum BudgetMigrationPlan: SchemaMigrationPlan {
    
    static var schemas: [any VersionedSchema.Type] { [BudgetAppSchemaV1.self, BudgetAppSchemaV2.self] }
    
    static var stages: [MigrationStage] { [migrateV1toV2] }
    
    static let migrateV1toV2 = MigrationStage.custom(fromVersion: BudgetAppSchemaV1.self, toVersion: BudgetAppSchemaV2.self) { context in
        
        print("willMigrate migrateV1toV2")
        
    } didMigrate: { context in
        
        print("didMigrate start")
        let budgets = try context.fetch(FetchDescriptor<BudgetAppSchemaV2.Budget>())
        
        for budget in budgets {
            budget.desc = "This is description for \(budget.name)"
        }
        
        try context.save()
        print("didMigrate migrateV1toV2")
    }
    
}
``` 

To perform a custom migration, we conform to `SchemaMigrationPlan`.

There are three important parts here.

First, the schemas property. This tells SwiftData which schema versions are involved in this migration. In our case, we are moving from V1 to V2, so we list both.

Second, the `stages` property. This defines the sequence of migration stages. Right now we only have one stage, migrateV1toV2, but in larger applications you may have several.

Third, the custom migration stage itself.

A custom stage gives you two closures:

- `willMigrate`
- `didMigrate`

Think of `willMigrate` as the before phase. It runs before the migration happens. This is useful if you need to inspect or prepare old data before the schema changes.

`didMigrate` runs after the migration completes. At this point, you are working with the new schema. This is exactly what we need, because the desc property only exists in V2.

Inside `didMigrate`, we fetch all budgets using the V2 model type. Then we assign a new description based on the existing name property. Finally, we call `context.save()` to persist those changes.

Do not forget to save. Migration changes are not automatically persisted unless you explicitly call save().

This is the key moment of the migration. We are not just adding a column. We are shaping the data into its new form.

``` swift 
import SwiftUI
import SwiftData

@main
struct LearnSwiftDataMigrationsApp: App {
    
    let container: ModelContainer
    
    init() {
        self.container = try! ModelContainer(for: Budget.self, migrationPlan: BudgetMigrationPlan.self, configurations: ModelConfiguration(isStoredInMemoryOnly: false))
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .modelContainer(container)
        }
    }
}
```

Before you run the app make sure to update the `typealias` to point to the new Budget schema. 

``` swift 
typealias Budget = BudgetAppSchemaV2.Budget
```

Now run the app.

If you already had budgets stored on disk using version 1 of the schema, SwiftData will detect the version mismatch and automatically trigger the migration. Behind the scenes, it moves your data from V1 to V2, executes your custom migration stage, and updates every existing record. When the app finishes launching, each budget will now contain a populated desc property derived from its name.

This completes our first real migration. We added a brand new stored property to an existing model and populated it with meaningful data during the upgrade process. This is exactly how production apps evolve. Requirements change. Models grow. Data needs to be reshaped.

In the next section, we will look at another common migration scenario that you will almost certainly face in a real SwiftData application, and how to handle it safely and confidently.

### Adding Unique Constraint to Budget Name 

At this point your app is stable. The new `desc` field is working. Customers are happy. Everything feels complete.

And then a new complaint shows up.

Users are creating multiple budgets with the same name. Two “Groceries” budgets. Three “Travel” budgets. From a database perspective that might be fine. From a user perspective it is confusing. They cannot tell which one is which. Now what felt like a harmless flexibility becomes a usability problem.

This is where you move from “it works” to “it behaves correctly.”

Your job now is twofold. First, prevent future duplicates. Second, clean up the duplicates that already exist in production. You cannot just enforce a constraint and hope for the best. If duplicate names are already stored, the migration will fail. And failing a migration means a broken app.

SwiftData makes enforcing uniqueness straightforward. You can decorate the field with the `#Unique` attached macro. This tells SwiftData that values for this property must be unique across all instances.

Here is how the new schema version looks with the uniqueness constraint applied:

``` swift 
enum BudgetAppSchemaV3: VersionedSchema {
    
    static let versionIdentifier: Schema.Version = .init(3, 0, 0)
    
    static var models: [any PersistentModel.Type] { [BudgetAppSchemaV3.Budget.self] }
    
    @Model
    class Budget {
        
        #Unique<Budget>([\.name])
        var name: String = ""
        var limit: Double = 0.0
        var desc: String = ""
        
        init(name: String, limit: Double, desc: String = "") {
            self.name = name
            self.limit = limit
            self.desc = desc
        }
    }
}
```

Most of the implementation is identical to version 2. The only real change is the addition of the `#Unique` macro on the name property. That one line changes the rules of the database. It tells SwiftData that no two budgets are allowed to share the same name.

From this point forward, duplicates are blocked.

But here is the real question.

What about the budgets that are already sitting on disk with duplicate names?

If you attempt to apply this as a lightweight migration, it will fail. SwiftData will attempt to enforce the uniqueness constraint, detect the existing duplicates, and stop the migration. That means your app will not launch. And that is not something you want your users discovering after an update.

This is why uniqueness constraints are not just a schema change. They are a data correction problem.

Before the new constraint can be enforced, the existing data must be cleaned up. Every duplicate must be resolved so that when the constraint is applied, the database is already in a valid state.

And that means we need another custom migration stage.

Below you can see the full migration plan, including the earlier V1 to V2 migration as well as the new V2 to V3 stage that resolves duplicates before the uniqueness rule is enforced.

``` swift 
enum BudgetMigrationPlan: SchemaMigrationPlan {
    
    static var schemas: [any VersionedSchema.Type] { [BudgetAppSchemaV1.self, BudgetAppSchemaV2.self] }
    
    static var stages: [MigrationStage] { [migrateV1toV2] }
    
    static let migrateV1toV2 = MigrationStage.custom(fromVersion: BudgetAppSchemaV1.self, toVersion: BudgetAppSchemaV2.self) { context in
        
        print("willMigrate migrateV1toV2")
        
    } didMigrate: { context in
        
        print("didMigrate start")
        let budgets = try context.fetch(FetchDescriptor<BudgetAppSchemaV2.Budget>())
        
        for budget in budgets {
            budget.desc = "This is description for \(budget.name)"
        }
        
        try context.save()
        print("didMigrate migrateV1toV2")
    }
    
    static let migrateV2toV3 = MigrationStage.custom(fromVersion: BudgetAppSchemaV2.self, toVersion: BudgetAppSchemaV3.self) { context in
        
        print("migrateV2toV3")
        
        let budgets = try context.fetch(FetchDescriptor<BudgetAppSchemaV2.Budget>())
        
        func normalized(_ s: String) -> String {
            s.trimmingCharacters(in: .whitespacesAndNewlines).lowercased()
        }
        
        let grouped = Dictionary(grouping: budgets) { budget in
            normalized(budget.name)
        }
        
        let duplicates = grouped.filter { _, items in
            items.count > 1
        }
        
        for (_, items) in duplicates {
            for (index, budget) in items.enumerated() where index > 0 {
                let baseName = budget.name.trimmingCharacters(in: .whitespacesAndNewlines)
                budget.name = "\(baseName) (\(index + 1))"
            }
        }
        
        try context.save()
        
        
    } didMigrate: { context in
        print("didMigrate V2toV3")
    }
}
```

Let’s zoom in on migrateV2toV3.

This is the migration that makes the uniqueness constraint possible. In this stage, we do the cleanup work before SwiftData applies the V3 schema rules. That is why the logic lives in willMigrate. We want to fix the data while we are still working with the V2 model, because once V3 is in effect, any duplicates would immediately violate the #Unique constraint and the migration would fail.

The idea is simple. We fetch all existing budgets, group them by name (after trimming whitespace and normalizing case), and then rename the duplicates so every budget ends up with a unique name. For example, if there are three “Groceries” budgets, we keep the first one as “Groceries” and rename the others to something like “Groceries (2)” and “Groceries (3)”. After that, we save the changes, and now the database is in a valid state for V3.

Before running the app, update your typealias so the rest of your code uses the latest schema:

``` swift 
typealias Budget = BudgetAppSchemaV3.Budget
```

Now run the app again. If you had duplicate budget names in the existing database, they will be automatically renamed during migration, and from this point forward SwiftData will prevent new duplicates from ever being created.

### Conclusion 

If you step back and look at what we just did, none of it was complicated. We did not write hundreds of lines of defensive code. We did not build a custom persistence engine. We simply respected one truth.

Data lives longer than your code.

We started with a simple model. Then we added a new property and transformed existing data. Then we introduced a uniqueness constraint and cleaned up duplicates before enforcing it. Each change felt small in isolation. But every one of those changes altered the structure of data already stored on disk. That is where most apps break. Not because SwiftData is fragile, but because developers assume their models will not evolve.

Versioning your schema is not about over engineering. It is about acknowledging that your app will grow. Requirements will change. Your understanding of the domain will mature. Fields will be renamed. Constraints will be added. Relationships will shift. And every time that happens, your job is to evolve the data safely, not just the code.

SwiftData gives you the tools to do this properly. VersionedSchema gives you structure. SchemaMigrationPlan gives you control. Custom migration stages let you reshape data intentionally instead of hoping automatic migration handles everything. When you use these tools, migrations stop feeling scary. They become part of your architecture.

And that is really the point.

Schema evolution should feel deliberate. It should feel controlled. It should feel like engineering. Not like patching something after it explodes in production.

If you are not versioning your SwiftData schema, you are gambling. You are hoping that your assumptions about the future will hold. But if you treat your schema as a first class architectural component from day one, your app can grow without breaking trust.

Your users will never notice your migrations.

And that is exactly how it should be.

