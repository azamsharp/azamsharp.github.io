# Syncing SwiftData with a Custom Backend Using HistoryObserver 

Many applications need to store data locally while also keeping a copy on a backend server. This is common in business applications where users expect their data to be available across multiple devices or for reporting and analytics. While SwiftData makes local persistence simple, synchronizing that data with your own backend requires additional infrastructure.

Starting with iOS 27, SwiftData introduces `HistoryObserver`, making it much easier to detect changes occurring in your local store. Instead of manually tracking inserts, updates, and deletes throughout your application, you can observe the persistent history and react whenever new transactions are recorded. This provides a clean and centralized place to implement synchronization logic.

In this article, you will learn how to use `HistoryObserver` to build a one way synchronization pipeline between SwiftData and a custom backend. You will see how to observe model changes, process transaction history efficiently, upload only new changes, and handle deletes using a soft delete strategy.

> Note: HistoryObserver supports one way synchronization. It observes changes made to your local SwiftData store, allowing you to synchronize those changes with your backend. However, changes made on the server are not automatically synchronized back to the client.

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

### Scenario 

For this article, we will build a simple room measurement application. The user can create rooms, assign a name, and specify the total square footage. The rooms are stored locally using SwiftData, but we also want every change to be synchronized with our own backend server.

The interesting part is not storing the data locally. SwiftData already makes that easy. The challenge is detecting when the data changes and synchronizing only those changes with the server. This is exactly the problem `HistoryObserver` is designed to solve. 


                  User
                    │
                    ▼
             SwiftUI Application
                    │
                    ▼
               SwiftData Store
                    │
        HistoryObserver detects changes
                    │
                    ▼
             RoomSyncManager
                    │
            Creates RoomPayload[]
                    │
                    ▼
              HTTPClient / REST
                    │
                    ▼
            Node / Express Backend
                    │
                    ▼
                 Database

Figure illustrates the synchronization pipeline. Whenever the user modifies a room, HistoryObserver detects the change, RoomSyncManager converts it into one or more RoomPayload objects, and those payloads are sent to the backend using a REST API.     

### Implementing RoomSyncManager 

The RoomSyncManager is responsible for observing changes in the local SwiftData store and synchronizing those changes with the backend server.

``` swift 
@Observable
class RoomSyncManager {
    
    private var observer: HistoryObserver?
    private let syncService: RoomSyncService

     @ObservationIgnored private var token: ObservationTracking.Token?
    
    init(syncService: RoomSyncService = RoomSyncService(httpClient: HTTPClient())) {
        self.syncService = syncService
    }
}
```

The manager depends on RoomSyncService, which is responsible for communicating with the backend using HTTPClient. It also maintains a reference to HistoryObserver, which watches for changes to the models we are interested in.

The observation is configured in the start function.

``` swift 
 func start(container: ModelContainer) throws {
        observer = try HistoryObserver(observedModels: [Room.self], modelContainer: container)
        
        token = withContinuousObservation(options: .didSet) { [weak self] event in
            print("Observation Fired...")
            _ = self?.observer?.eventCounter
            self?.processChanges(context: container.mainContext)
        }
}
```

The HistoryObserver is configured to monitor only the Room model. Whenever SwiftData records a new transaction for that model, the observer updates its eventCounter.

The eventCounter is incremented whenever the SwiftData model context is saved. In many cases, you do not need to call context.save() yourself because SwiftData automatically saves changes under several conditions, such as when the application moves to the background, when navigating between screens, or after a short period of inactivity.

The call to withContinuousObservation creates a continuous observation that reacts whenever one of its tracked dependencies changes. The returned token is stored in the token property to keep the observation alive for as long as the RoomSyncManager exists. If you don't retain this token, the observation is disposed and no further changes are detected.

Inside the observation closure, we intentionally read the eventCounter property.

`_ = self?.observer?.eventCounter`

This line may look unnecessary, but it is actually the key to making the observation work. Swift's Observation framework only tracks properties that are accessed inside the observation closure. By reading eventCounter, we register it as a dependency. Whenever HistoryObserver increments this value, the observation fires automatically and calls processChanges, where we can fetch the latest transactions and synchronize them with the server.

Next, let's look at how processChanges retrieves the transaction history and prepares the data for synchronization. 

### Implementing processChanges 

The processChanges function is responsible for reading the transaction history from the SwiftData store and preparing those changes to be synchronized with the backend. It receives a ModelContext, which is used to fetch the persistent history maintained by SwiftData.

You can think of the persistent history as a ledger that records every change made to your models. Each transaction contains one or more changes, such as inserts, updates, and deletes. Our initial implementation is shown below. 

``` swift 
    private func processChanges(context: ModelContext) {
        
        do {
            let descriptor = HistoryDescriptor<DefaultHistoryTransaction>()
            let history = try context.fetchHistory(descriptor)
            
            for transaction in history {
                for change in transaction.changes {
                    switch change {
                    case .insert(let insert as DefaultHistoryInsert<Room>):
                        if let room = context.model(for: insert.changedPersistentIdentifier) as? Room {
                            print(room.name, room.area)
                        }
                    case .update(let update as DefaultHistoryUpdate<Room>):
                        if let room = context.model(for: update.changedPersistentIdentifier) as? Room {
                            print(room.name, room.area)
                        }
                    case .delete:
                        break
                    default:
                        break
                    }
                }
            }
            
        } catch {
            print(error.localizedDescription)
        }
        
    }
```

The code fetches the complete transaction history and iterates through every transaction. Each transaction may contain one or more inserts, updates, or deletes. For every insert or update, we retrieve the corresponding Room model and print its contents.

Although this implementation works, it has one major drawback. Every time processChanges is called, it fetches the entire history from the beginning. If your application has accumulated hundreds or even thousands of transactions, all of them are processed again, even if they were already synchronized with the server.

For a logging or auditing system, this behavior may be perfectly acceptable. For a synchronization engine, however, it is inefficient. We only want to process the changes that have occurred since the last successful synchronization.

The solution is to keep track of our position in the transaction history by storing the token of the last processed transaction. Think of this token as a bookmark in the transaction ledger. The next time we fetch history, we use that bookmark to request only the transactions that occurred after it. This allows us to process only the new changes, reduces network traffic, and prevents uploading the same data multiple times.

### Implementing the Last Token

The `lastToken` acts as a bookmark in SwiftData's transaction history. It tells the application which transactions have already been processed.

Without this token, every synchronization cycle would fetch the complete history from the beginning. By storing the token from the most recently processed transaction, we can ask SwiftData to return only the transactions that occurred after it.

The token should also be persisted to disk. This allows the application to continue from the same position even after it has been closed or terminated by the system.

We can store the token in `UserDefaults` using the following property:

```swift
@ObservationIgnored
private var lastToken: DefaultHistoryToken? {
    get {
        guard let data = UserDefaults.standard.data(
            forKey: "RoomSyncManager_LastToken"
        ) else {
            return nil
        }

        return try? JSONDecoder().decode(
            DefaultHistoryToken.self,
            from: data
        )
    }

    set {
        guard let newValue,
              let data = try? JSONEncoder().encode(newValue) else {
            return
        }

        UserDefaults.standard.set(
            data,
            forKey: "RoomSyncManager_LastToken"
        )
    }
}
```

The `lastToken` property is marked with `@ObservationIgnored` because it should not participate in observation tracking.

The `processChanges` function is called from inside `withContinuousObservation`. Any observable property accessed during that call can become part of the observation. If `lastToken` were tracked, updating it after a successful synchronization could cause the observation to fire again.

That could create a loop where processing the history updates the token, updating the token triggers another observation, and the observation starts the synchronization process again.

By marking `lastToken` with `@ObservationIgnored`, we can safely read and update it without triggering another synchronization cycle.

### Creating the Synchronization Payload

Once we can fetch only the latest transactions, we need to decide how those changes should be sent to the server.

One option is to make a separate network request for every insert and update. This approach works, but it can quickly result in a large number of requests.

A better approach is to collect the changes into an array and send them to the server as a single batch. Each item in the batch includes the room data and the action that should be performed.

```swift
enum SyncAction: String, Codable {
    case insert
    case update
    case delete
}

struct RoomPayload: Codable {
    let id: UUID
    let name: String?
    let area: Double?
    let action: SyncAction
}
```

The `SyncAction` tells the server whether it should insert, update, or delete a room. The `RoomPayload` contains the values needed to perform that action.

We can now update `processChanges` to fetch only the transactions that occurred after `lastToken` and convert those changes into payloads.

```swift
private func processChanges(context: ModelContext) {

    do {
        let descriptor: HistoryDescriptor<DefaultHistoryTransaction>

        if let lastToken {
            descriptor = HistoryDescriptor(
                predicate: #Predicate { transaction in
                    transaction.token > lastToken
                }
            )
        } else {
            descriptor = HistoryDescriptor()
        }

        let history = try context.fetchHistory(descriptor)

        var payloadsToSync: [RoomPayload] = []

        for transaction in history {
            for change in transaction.changes {
                switch change {
                case .insert(let insert as DefaultHistoryInsert<Room>):
                    if let room = context.model(
                        for: insert.changedPersistentIdentifier
                    ) as? Room {
                        let payload = RoomPayload(
                            id: room.syncId,
                            name: room.name,
                            area: room.area,
                            action: .insert
                        )

                        payloadsToSync.append(payload)
                    }

                case .update(let update as DefaultHistoryUpdate<Room>):
                    if let room = context.model(
                        for: update.changedPersistentIdentifier
                    ) as? Room {
                        let payload = RoomPayload(
                            id: room.syncId,
                            name: room.name,
                            area: room.area,
                            action: .update
                        )

                        payloadsToSync.append(payload)
                    }

                case .delete:
                    break

                default:
                    break
                }
            }
        }

        // Upload payloads to the server.

    } catch {
        print(error.localizedDescription)
    }
}
```

When `lastToken` is available, the predicate filters the history and returns only transactions with a newer token. If no token has been stored yet, the complete history is fetched because this is the first synchronization cycle.

Each insert and update is converted into a `RoomPayload` and added to `payloadsToSync`. Once all transactions have been processed, the complete batch can be sent to the server using a single request.

One important detail is that `lastToken` should not be updated immediately after fetching the history. It should only be saved after the server confirms that the payloads were uploaded successfully. Otherwise, a failed network request could cause the application to skip changes that were never synchronized.

### Syncing the Payload

Once `payloadsToSync` has been populated, the final step is to send the changes to the backend. If the array contains at least one payload, we call `RoomSyncService` to upload all changes in a single request.

```swift
if !payloadsToSync.isEmpty {
    Task {
        do {
            try await syncService.uploadRooms(payloads: payloadsToSync)

            if let latestToken = history.last?.token {
                self.lastToken = latestToken
            }

            isSyncing = false
        } catch {
            print(error.localizedDescription)
            isSyncing = false
        }
    }
}
```

Notice that `lastToken` is updated only after the upload completes successfully. This is important because if the network request fails, we want those same transactions to be processed again during the next synchronization attempt. Updating the token before a successful upload could result in changes being permanently skipped.

The backend implementation can be written using any language or framework. For this article, I am using a simple Node and Express application. The endpoint accepts an array of room payloads and prints them to the console.

```javascript
app.post('/api/rooms', (req, res) => {
    const rooms = req.body

    console.log(rooms)

    res.status(200).json({ success: true })
})
```

When new rooms are added, the server receives payloads similar to the following.

```javascript
[
  {
    area: 250,
    name: "Kitchen",
    id: "4D9B718D-B802-49EA-B2F4-D557E544588B",
    action: "insert"
  }
]

[
  {
    id: "B646399D-BCE4-4D8F-B2BB-568002FC6E03",
    action: "insert",
    name: "Bedroom",
    area: 300
  }
]
```

At this point, we have implemented synchronization for inserts and updates. You may be wondering why we did not implement deletes in exactly the same way.

The reason is that once a model has been deleted, it no longer exists in the SwiftData store. This means the following code can no longer retrieve the model.

``` swift
context.model(for: delete.changedPersistentIdentifier)
```

Without the model, we no longer have access to values such as the room's identifier or other properties that might be needed by the server to process the deletion.

There are several techniques for solving this problem. One of the most common approaches is to use **soft deletes**, where the model is first marked as deleted instead of being immediately removed from the database. This gives the synchronization engine an opportunity to notify the server before the record is permanently removed from the local store.

### Implementing Soft Deletes (IsDeleted)

The most common way to synchronize deletes is to use **soft deletes**. Instead of immediately removing a model from the local database, we first mark it as deleted using a Boolean flag.

This gives the synchronization engine an opportunity to notify the server before the model is permanently removed. Once the server has successfully processed the delete request, the model can safely be deleted from the local SwiftData store.

To support this behavior, we add an `isDeleted` property to our `Room` model.

```swift
@Model
class Room {
    @Attribute(.unique) var syncId: UUID = UUID()
    var name: String
    var area: Double
    var isDeleted: Bool = false

    init(syncId: UUID = UUID(), name: String, area: Double) {
        self.syncId = syncId
        self.name = name
        self.area = area
    }
}
```

When the user deletes a room, we no longer call `context.delete()`. Instead, we set `isDeleted` to `true` and save the context. Since this is simply another update to the model, `HistoryObserver` records it as an update transaction.

Inside `processChanges`, we can check whether the room has been marked as deleted.

```swift
case .update(let update as DefaultHistoryUpdate<Room>):
    if let room = context.model(for: update.changedPersistentIdentifier) as? Room {

        if room.isDeleted {
            let payload = RoomPayload(
                id: room.syncId,
                name: nil,
                area: nil,
                action: .delete
            )

            payloadsToSync.append(payload)
        } else {
            let payload = RoomPayload(
                id: room.syncId,
                name: room.name,
                area: room.area,
                action: .update
            )

            payloadsToSync.append(payload)
        }
    }
```

If `isDeleted` is `true`, we create a payload with a `.delete` action. Since the server only needs the room identifier to perform the deletion, the `name` and `area` values are set to `nil`. Otherwise, we generate a normal update payload.

After the payload has been uploaded successfully, we can permanently remove the deleted rooms from the local database.

```swift
let deletedIds = payloadsToSync
    .filter { $0.action == .delete }
    .map { $0.id }

if !deletedIds.isEmpty {
    do {
        try context.delete(
            model: Room.self,
            where: #Predicate { room in
                deletedIds.contains(room.syncId)
            }
        )

        try context.save()
    } catch {
        print(error.localizedDescription)
    }
}
```

This approach gives us the best of both worlds. The server is notified about every deletion, and once synchronization succeeds, the room is removed from the local database. As a result, both the local SwiftData store and the backend remain in sync.

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

### The Backend Endpoint

The backend for this sample is intentionally simple. It exposes a single endpoint that accepts an array of `RoomPayload` objects.

```javascript
const express = require("express")
const cors = require("cors")

const app = express()

app.use(cors())
app.use(express.json())

app.post("/api/rooms", (req, res) => {
    const rooms = req.body

    console.log(rooms)

    res.status(200).json({ success: true })
})

app.listen(8080, () => {
    console.log("Server is running...")
})
```

The endpoint currently logs the received payloads and returns a successful response. In a production application, this is where you would persist the changes to your database.

For each payload, the server can inspect the `action` property and determine whether to insert, update, or delete the corresponding room.

```javascript
for (const room of rooms) {
    switch (room.action) {
        case "insert":
            // Insert into database
            break

        case "update":
            // Update existing record
            break

        case "delete":
            // Delete record from database
            break
    }
}
```

The implementation details depend on the language, framework, and database you are using, but the overall workflow remains the same. The client sends a batch of changes, and the server applies each change based on its associated action.

## Source Code

The complete source code for this project is available on GitHub:

**GitHub Repository**
[https://github.com/azamsharpschool/Rooms](https://github.com/azamsharpschool/Rooms)

The repository includes:

* Complete SwiftUI application
* SwiftData models
* `RoomSyncManager` implementation
* `HistoryObserver` integration

Feel free to clone the project, experiment with it, and adapt it for your own applications.

### Conclusion 

`HistoryObserver` provides a clean and efficient way to monitor changes in your SwiftData store without scattering synchronization logic throughout your application. By observing persistent history, tracking the last processed token, batching changes together, and using soft deletes, you can build a reliable synchronization pipeline with relatively little code.

The approach demonstrated in this article is a solid foundation for applications that need to synchronize local data with a custom backend. In a production application, you may also want to add retry logic, background synchronization, conflict resolution, authentication, and server acknowledgements to make the synchronization process even more resilient.

Although `HistoryObserver` currently supports only one way synchronization, it removes much of the complexity involved in detecting local changes and gives you a straightforward way to keep your backend up to date. Combined with a well designed server API, it provides an excellent foundation for building offline first applications powered by SwiftData.





