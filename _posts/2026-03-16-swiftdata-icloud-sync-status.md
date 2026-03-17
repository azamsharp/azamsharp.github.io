
# Apple Doesn't Show SwiftData iCloud Sync Status — So Let's Build One

This post was inspired by a question I came across on Reddit. The question was simple but interesting:

Is there a way to show the sync status between SwiftData and iCloud in a SwiftUI app?

At first glance this seems like something Apple would provide out of the box. After all, SwiftData makes enabling iCloud sync incredibly easy. But once you start looking for an API that tells you when syncing starts, finishes, or fails, you quickly realize something surprising.

There is no such API.

SwiftData does a great job syncing data with iCloud behind the scenes, but it does not expose any direct way to observe the sync progress.

Fortunately, the story does not end there. SwiftData is built on top of Core Data with CloudKit integration, and Core Data exposes a set of notifications that tell us when sync events occur. By listening to those notifications we can build a simple but useful sync monitor view.

Let's build one.

> You can watch this [small video](https://x.com/azamsharp/status/2033654042023354622?s=20) of the end result. 

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

### Minimalist SwiftData Application

For this project I created a very small SwiftData application. The goal here is not to build a full Todo app but simply to have something that can create data and trigger CloudKit sync.

The app allows the user to add Todo items and displays them in a list.

Below is the TodoItem model and the ContentView.

``` swift 
@Model
class TodoItem {
    var name: String = ""
    
    init(name: String) {
        self.name = name
    }
}

struct ContentView: View {
    
    @Environment(\.modelContext) private var modelContext
    @Query private var todoItems: [TodoItem]
    
    @State private var name: String = ""
    
    var body: some View {
        NavigationStack {
            List {
                
                Section("Add Todo") {
                    TextField("Name", text: $name)
                    
                    Button("Save") {
                        let todoItem = TodoItem(name: name)
                        modelContext.insert(todoItem)
                        try? modelContext.save()
                        
                        name = ""
                    }
                }
                
                Section("Todo Items") {
                    ForEach(todoItems) { todoItem in
                        HStack {
                            Text(todoItem.name)
                        }
                    }
                }
            }
            .navigationTitle("Todo Items")
        }
    }
}

```

The project is already configured to work with iCloud. This includes:

• CloudKit entitlements
• iCloud container configuration
• background mode for remote notifications

Once everything is configured, syncing works automatically.

If you run the app on a physical device and add a Todo item, the data is uploaded to iCloud. You can verify this by opening the [CloudKit Console](https://developer.apple.com/icloud/cloudkit/) and inspecting your records.

You can even edit the record directly in the CloudKit Console and the change will appear in your app almost immediately without restarting the app.

This is one of the really nice aspects of SwiftData. The iCloud integration is extremely simple to set up.

However, there is one problem.

The user has no idea when syncing is happening.

If data is uploading, downloading, or failing to sync, the UI provides no indication.

To solve that problem we will build a small utility called CloudSyncMonitor.

### Implementing CloudSynMonitor 

The purpose of CloudSyncMonitor is simple. It listens for CloudKit sync events and converts them into a small set of states that our UI can display.

First we define the different statuses our app can report.

``` swift 
@Observable
final class CloudSyncMonitor {
    
    enum Status: Equatable {
        case idle
        case syncing(String)   // "Uploading" / "Downloading" / "Setting up"
        case success
        case failed(String)
    }
}
```

These states represent the current sync activity.

``` swift  
idle
syncing("Uploading to iCloud")
syncing("Downloading from iCloud")
success
failed("Network error")
```

Once we have these states, the next step is to start listening for CloudKit events.

### Implementing the start Function 

The `start` function begins monitoring CloudKit synchronization.

SwiftData does not provide any API that tells us when syncing begins or ends. However, the underlying Core Data stack sends notifications whenever a CloudKit sync event changes.

We can listen to those notifications using NotificationCenter.

``` swift 
 func start() {
        observer = NotificationCenter.default.addObserver(
            forName: NSPersistentCloudKitContainer.eventChangedNotification,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            
            guard
                let event = notification.userInfo?[NSPersistentCloudKitContainer.eventNotificationUserInfoKey]
                    as? NSPersistentCloudKitContainer.Event
            else { return }
            
            let label: String
            switch event.type {
            case .setup:
                label = "Setting up iCloud sync"
            case .import:
                label = "Downloading from iCloud"
            case .export:
                label = "Uploading to iCloud"
            @unknown default:
                label = "Syncing"
            }
            
            // Event started
            if event.endDate == nil {
                self?.status = .syncing(label)
                return
            }
            
            // Event finished with error
            if let error = event.error {
                self?.status = .failed(error.localizedDescription)
                return
            }
            
            // Event finished successfully
            if event.succeeded {
                self?.status = .success
            } else {
                self?.status = .idle
            }
        }
    }
```

The `start` function is responsible for **starting the sync monitor**. In other words, it tells our `CloudSyncMonitor` to begin listening for events emitted by the CloudKit sync engine.

SwiftData itself does not provide an API that tells us when syncing begins or ends. However, under the hood SwiftData relies on **Core Data with CloudKit integration**, and Core Data exposes notifications whenever a sync event changes. The `start` function simply registers a listener for those notifications.

#### Registering the Listener

The first part of the function registers an observer with `NotificationCenter`.

```swift
observer = NotificationCenter.default.addObserver(
    forName: NSPersistentCloudKitContainer.eventChangedNotification,
    object: nil,
    queue: .main
)
```

We are telling the system:

> Whenever the CloudKit sync engine reports a change in sync activity, notify this object.

The notification we are listening for is:

```
NSPersistentCloudKitContainer.eventChangedNotification
```

This notification is triggered whenever CloudKit begins a sync operation, progresses through it, or completes it.

The `queue: .main` parameter ensures that the closure runs on the **main thread**, which is important because the monitor updates UI state.

#### Extracting the CloudKit Event

When the notification fires, it contains additional information inside the `userInfo` dictionary. The most important item is the **CloudKit event object**.

```swift
guard
    let event = notification.userInfo?[NSPersistentCloudKitContainer.eventNotificationUserInfoKey]
        as? NSPersistentCloudKitContainer.Event
else { return }
```

If the event cannot be extracted, we simply return.

The `event` object describes what the sync engine is doing. It contains details such as:

• the type of operation
• when it started
• when it finished
• whether it succeeded
• whether an error occurred

#### Determining the Sync Direction

The next step is determining what type of sync activity is happening.

```swift
switch event.type {
case .setup:
    label = "Setting up iCloud sync"
case .import:
    label = "Downloading from iCloud"
case .export:
    label = "Uploading to iCloud"
@unknown default:
    label = "Syncing"
}
```

CloudKit sync events typically fall into three categories.

`setup`
Occurs when the app initializes its CloudKit environment.

`import`
Occurs when **data is downloaded from iCloud to the device**.

`export`
Occurs when **local data is uploaded from the device to iCloud**.

We convert these event types into human readable labels so they can be displayed in the UI.

#### Detecting When Sync Starts

Next we check whether the event has finished.

```swift
if event.endDate == nil {
    self?.status = .syncing(label)
    return
}
```

If `endDate` is `nil`, the sync operation is **still in progress**.

At this point we update the monitor's status to:

```
.syncing("Uploading to iCloud")
```

or

```
.syncing("Downloading from iCloud")
```

Because the class is marked with `@Observable`, this change will automatically update any SwiftUI view observing the monitor.

#### Detecting Errors

If the event has completed, the next thing we check is whether it finished with an error.

```swift
if let error = event.error {
    self?.status = .failed(error.localizedDescription)
    return
}
```

If CloudKit encountered a problem, the event will contain an error object. In that case we update the status to:

```
.failed(error message)
```

This allows the UI to display a message indicating that the sync operation failed.

#### Detecting Successful Sync

Finally, we check whether the event completed successfully.

```swift
if event.succeeded {
    self?.status = .success
} else {
    self?.status = .idle
}
```

If `event.succeeded` is true, we update the status to `.success`. This indicates that the sync operation completed successfully.

If not, we simply fall back to `.idle`.

#### The Overall Flow

The `start` function effectively converts low level CloudKit events into a simple state machine.

The flow looks like this:

```
CloudKit event occurs
        ↓
NotificationCenter fires notification
        ↓
CloudSyncMonitor receives event
        ↓
Event is analyzed
        ↓
Status is updated
        ↓
SwiftUI view refreshes automatically
```

This is what allows our UI to show a message such as:

```
Uploading to iCloud
Downloading from iCloud
Sync completed
Sync failed
```

Even though SwiftData itself does not expose a direct API for sync progress, this approach gives us a **reasonable approximation of the sync state** using the underlying CloudKit infrastructure.

### Implementing the SyncStatusView 

Now that we have a monitor tracking sync activity, the next step is displaying that information to the user.

We can create a simple SwiftUI view called SyncStatusView.

``` swift 
struct SyncStatusView: View {
    
    let monitor: CloudSyncMonitor
    
    var body: some View {
        HStack(spacing: 8) {
            icon
            Text(title)
                .font(.subheadline)
                .fontWeight(.medium)
        }
        .foregroundStyle(color)
        .padding(.vertical, 8)
        .padding(.horizontal, 12)
        .frame(maxWidth: .infinity, alignment: .leading)
        .background(backgroundColor)
        .clipShape(RoundedRectangle(cornerRadius: 10))
        .animation(.easeInOut, value: monitor.status)
    }

    // other code here 
}
```

This view observes the monitor's status and updates automatically whenever the sync state changes.

### Using the SyncStatusView 

Finally we can integrate the view into our main screen.

``` swift 
struct ContentView: View {
    
    @Environment(\.modelContext) private var modelContext
    @Query private var todoItems: [TodoItem]
    
    @State private var name: String = ""
    @State private var isCompleted: Bool = false
    @State private var syncMonitor = CloudSyncMonitor()
    
    var body: some View {
        NavigationStack {
            List {
                
                Section {
                    SyncStatusView(monitor: syncMonitor)
                }
                
                Section("Add Todo") {
                    TextField("Name", text: $name)
                    
                    Button("Save") {
                        let todoItem = TodoItem(name: name)
                        modelContext.insert(todoItem)
                        try? modelContext.save()
                        
                        name = ""
                    }
                }
                
                Section("Todo Items") {
                    ForEach(todoItems) { todoItem in
                        HStack {
                            Text(todoItem.name)
                        }
                    }
                }
            }
            .navigationTitle("Todo Items")
        }
        .onAppear {
            syncMonitor.start()
        }
    }
}
```

When the view appears, we call:

``` swift 
syncMonitor.start()
```

This activates the sync monitor, and the UI will automatically update whenever CloudKit begins or finishes a sync operation.

### Limitations of This Approach

The technique we used works well for exposing basic sync activity, but it is important to understand that it is **not a perfect solution**. The APIs we are relying on come from the underlying Core Data CloudKit integration, and they were never designed to be a polished, user facing sync status API.

In other words, we are tapping into internal signals that the sync engine emits. That gives us useful visibility, but there are a few limitations worth keeping in mind.

#### No Progress Information

One of the biggest limitations is that we cannot determine **how much of the sync operation has completed**.

The events we receive only tell us when a sync operation **starts** and when it **finishes**. They do not provide incremental progress updates.

This means we cannot build something like a progress bar showing:

```
Uploading 20%
Uploading 60%
Uploading 90%
```

Instead, we can only display high level states such as:

* Uploading to iCloud
* Downloading from iCloud
* Sync completed
* Sync failed

For most applications this is still helpful, but it is not a precise representation of sync progress.

#### Sync Events Do Not Always Match User Actions

Another thing you might notice is that sync events do not always occur immediately after a user performs an action.

When a user inserts or updates a record, SwiftData first writes the change to the **local store**. The upload to iCloud may happen a few seconds later because the sync engine often batches operations together.

Because of this, you might see the syncing indicator appear slightly after the user performs an action.

#### Multiple Events Can Fire Quickly

CloudKit synchronization often consists of several operations happening in sequence. During a single sync cycle you might observe something like this:

```
Uploading to iCloud
Sync completed
Downloading from iCloud
Sync completed
```

This happens because CloudKit may perform both **export and import operations** during the same cycle. For example, it may upload your local changes and then immediately pull down updates from other devices.

For this reason, the status displayed by our monitor should be treated as **informational rather than perfectly precise**.

#### Some Sync Activity Is Invisible

CloudKit also performs internal operations that do not always surface as clear events.

Examples include background scheduling, database maintenance, and conflict resolution. These activities may happen without producing a clean status message that we can display to the user.

#### iCloud and Network Conditions

Finally, this monitor assumes that the device is signed into iCloud and that CloudKit syncing is enabled for the app.

If the user is not logged into iCloud, or if iCloud is disabled for the app, the monitor will not provide much useful information. Similarly, network interruptions can delay synchronization events, which may make the UI appear idle even though the system intends to sync later.

Despite these limitations, this approach still provides **valuable visibility into the SwiftData sync process**. SwiftData makes syncing incredibly easy, but it also hides most of what is happening behind the scenes. By listening to the underlying CloudKit events, we can at least surface some meaningful signals to the user and gain better insight into what the sync engine is doing.

### Demo 

You can watch a small demo video [here](https://x.com/azamsharp/status/2033654042023354622?s=20).

### Source code 

You can download the complete source code using the following Gist link: 
https://gist.github.com/azamsharpschool/f66500e4d6df195802ae9f422ef157bc


### Conclusion 

SwiftData makes enabling iCloud sync surprisingly simple. With just a few configuration steps, your data starts flowing between devices almost automatically. The downside, however, is that the framework keeps most of the syncing process hidden from the developer and the user. When data is uploading, downloading, or even failing to sync, the UI remains completely silent.

In this article we built a small utility called `CloudSyncMonitor` that listens to the underlying CloudKit events exposed by Core Data. Even though SwiftData does not directly expose sync progress APIs, these notifications allow us to build a reasonable approximation of the sync state. By translating those events into a few simple states such as syncing, success, and failure, we can provide meaningful feedback to users and make debugging CloudKit issues much easier.

The `SyncStatusView` we created is intentionally simple, but it demonstrates the core idea. Once you have a monitor observing CloudKit events, you can design the UI in any way that fits your application. Some apps may display a small banner, others may show an icon in the navigation bar, and some may simply log sync activity for debugging purposes.

The key takeaway is that SwiftData iCloud sync does not have to be a complete black box. By tapping into the underlying CloudKit notifications, we can gain visibility into what the sync engine is doing and build better user experiences around it.

As SwiftData continues to evolve, Apple may eventually expose higher level APIs for observing sync activity. Until then, this approach provides a practical way to monitor and surface sync behavior in your SwiftData applications.

<div class="azam-bottom-book-banner" role="region" aria-label="SwiftUI Architecture Book">
  <div class="azam-bottom-book-banner__content">
    <p class="azam-bottom-book-banner__eyebrow">SwiftUI Architecture Book</p>
    <h3 class="azam-bottom-book-banner__title">Patterns and Practices for Building Scalable Applications</h3>
    <p class="azam-bottom-book-banner__text">
      If you enjoy learning about real world SwiftUI architecture, data flow, navigation, environment,
      testing, and scaling apps the right way, check out my book. It is packed with practical examples
      and lessons learned from building SwiftUI applications that go beyond toy projects.
    </p>
    <a
      class="azam-bottom-book-banner__button"
      href="https://azamsharp.school/swiftui-architecture-book.html"
      target="_blank"
      rel="noopener"
    >
      Check out the book
    </a>
  </div>

  <div class="azam-bottom-book-banner__image">
    <img
      src="https://azamsharp.school/images/swiftui-architecture-book-cover.png"
      alt="SwiftUI Architecture book cover"
      loading="lazy"
    />
  </div>
</div>

<style>
  .azam-bottom-book-banner {
    display: grid;
    grid-template-columns: 1.6fr 220px;
    gap: 24px;
    align-items: center;
    margin: 40px 0 20px 0;
    padding: 24px;
    border-radius: 18px;
    border: 1px solid rgba(255, 255, 255, 0.08);
    background:
      radial-gradient(circle at top left, rgba(59, 130, 246, 0.16), transparent 35%),
      radial-gradient(circle at bottom right, rgba(16, 185, 129, 0.14), transparent 35%),
      linear-gradient(135deg, #0f172a, #111827);
    box-shadow: 0 18px 45px rgba(0, 0, 0, 0.22);
    overflow: hidden;
  }

  .azam-bottom-book-banner__content {
    color: rgba(255, 255, 255, 0.94);
  }

  .azam-bottom-book-banner__eyebrow {
    margin: 0 0 8px 0;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: rgba(255, 255, 255, 0.68);
  }

  .azam-bottom-book-banner__title {
    margin: 0 0 10px 0;
    font-size: 26px;
    line-height: 1.2;
    color: #ffffff;
  }

  .azam-bottom-book-banner__text {
    margin: 0 0 18px 0;
    font-size: 15px;
    line-height: 1.7;
    color: rgba(255, 255, 255, 0.78);
    max-width: 60ch;
  }

  .azam-bottom-book-banner__button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    text-decoration: none;
    padding: 12px 18px;
    border-radius: 12px;
    font-size: 14px;
    font-weight: 700;
    color: #0b1220;
    background: linear-gradient(135deg, #6ee7b7, #60a5fa);
    box-shadow: 0 10px 24px rgba(0, 0, 0, 0.25);
    transition: transform 140ms ease, filter 140ms ease;
  }

  .azam-bottom-book-banner__button:hover {
    transform: translateY(-1px);
    filter: brightness(1.03);
  }

  .azam-bottom-book-banner__image {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .azam-bottom-book-banner__image img {
    width: 180px;
    max-width: 100%;
    height: auto;
    border-radius: 14px;
    border: 1px solid rgba(255, 255, 255, 0.12);
    box-shadow: 0 14px 30px rgba(0, 0, 0, 0.35);
    display: block;
  }

  @media (max-width: 768px) {
    .azam-bottom-book-banner {
      grid-template-columns: 1fr;
      padding: 20px;
    }

    .azam-bottom-book-banner__title {
      font-size: 22px;
    }

    .azam-bottom-book-banner__text {
      font-size: 14px;
    }

    .azam-bottom-book-banner__image {
      justify-content: flex-start;
    }

    .azam-bottom-book-banner__image img {
      width: 140px;
    }
  }
</style>