# Syncing SwiftData with Custom Backend Using HistoryObserver 

// Introduction 

> HistoryObserver provides a unidirectional sync support. This means the data from SwiftData store can be sycned with your custom server, but changes from your server are not automatically sycned back to the client. 

### Scenario 

For this article we will consider a situation where a user is measuring different rooms in their house. Each room consists of a name and the total square feet. The room data will be stored on the device using SwiftData framework and then also synced to a custom server. 

### Implementing RoomSyncManager 

The RoomSyncManager is responsible for observing the on-device store and then triggering the server to sync the newly available data. The implementation is shown below: 

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

RoomSyncManager has a dependency on RoomSyncService, which uses HTTPClient to perform the actual HTTP calls to the server. The HistoryObserver is represented by the observer object and it is responsible for setting up observation on the selected models. The observation is initialized using the start function shown below: 

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

> The eventCounter is incremented when SwiftData model context is saved. You don't have to explicity call the save function on context, it is called by SwiftData automatically under various conditions. Few of those conditions includes, app goes to the background, navigating to a different screen and it even calls by itself after few seconds. 

As you can see the HistoryObserver is indicating that it is only going to observe the Room models. Once the observer is initialized, the observation is setup using the withContinuousObservation closure. This closure returns a token, which is assigned to the token property. The token is responsible for making sure that the reference to the observation is alive and not disposed. Inside the withContinuousObservation closure, we activate the observation on properties that are accessed in the closure. This includes eventCounter property on the observer object and all the properties accessed inside the processsChanges function. 

The main purpose of the eventCounter is to provide an observable hook that Swift's Observation framework can track. Because Swift Observation only monitors properties that are explicitly read inside the tracking closure, referencing _ = self?.observer?.eventCounter registers it as a dependency. Whenever SwiftData records new transactions in the persistent history, the HistoryObserver increments this counter, which automatically triggers the observation closure to fire and calls processChanges to execute your sync logic.

Next, let's check out the implementation of the processChanges function. 

### Implementing processChanges 

The function processChanges is our own custom function, which gather all the changes (insert, delete, update) and then sends them over to the server using the RoomSyncService. It has a dependency on the ModelContext, which is used to fetch history. Fetching the history means to fetch all the transactions for Room model that has happened on the data store. You can think of it like a ledger, which maintains a list of all the events. The initial implementation of the processChanges function is shown below:  

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

The code fetches the history using the ModelContext and then goes through all the transaction. Each transaction is identified by insert, delete or update. The main issue with the above code is that it always fetches all the transactions. This means if your app has done 10 inserts, 5 deletes and 2 updates then each time the observation is fired, all those transactions are returned. 

This is fine, if you are building some sort of logging system, but for our application we only need to sync the newest, unprocessed changes that have occurred since our last successful sync.

To achieve this, we need to track our current position in the transaction ledger using a persistent token (like lastToken). By saving the token of the last processed transaction to disk and passing it as a predicate inside our HistoryDescriptor, we can filter out the historical transactions we have already processed. This allows us to fetch only the "delta" (the new changes), keeping our network payloads minimal and preventing duplicate uploads to the server.

### Implementing the Last Token 

The main idea behind the last token is to act as a persistent bookmark or cursor within SwiftData's chronological transaction log.

By saving the token of the latest processed transaction (typically to a persistent store like UserDefaults), the application keeps track of exactly how far it has read through the database's history. When the next sync cycle is triggered, instead of scanning the entire database ledger from the very beginning of time, the RoomSyncManager uses this saved token to query only the transactions that have occurred after that specific point.

This approach guarantees that:

- Syncing is highly efficient: The app only processes the "delta" (new changes) rather than wasting CPU cycles and memory re-evaluating historical data that has already been processed.

- Network overhead is minimized: It prevents the app from sending redundant payloads or duplicate API requests to your backend server.

- State survives app termination: Because the token is saved to disk rather than kept as an in-memory variable, the sync manager can safely pick up exactly where it left off, even if the user closes the app, the system terminates it, or the device is rebooted.

Let's start with implementing our lastToken property, which can persist the token to UserDefaults. 

``` swift 
@ObservationIgnored private var lastToken: DefaultHistoryToken? {
        get {
            // get the value from user defaults
            guard let data = UserDefaults.standard.data(forKey: "RoomSyncManager_LastToken") else { return nil }
            return try? JSONDecoder().decode(DefaultHistoryToken.self, from: data)
            
        }
        set {
            
            if let newValue = newValue {
                guard let data = try? JSONEncoder().encode(newValue) else { return }
                UserDefaults.standard.set(data, forKey: "RoomSyncManager_LastToken")
            }
        }
    }
```

The lastToken property is also using the @ObservationIgnored macro since we don't want the lastToken property to take part in the observation tracking process.

If lastToken were not ignored, any read operation on it inside processChanges would automatically register it as a dependency of the withContinuousObservation closure. Consequently, the moment you update lastToken at the end of a successful sync cycle, the Observation framework would detect this mutation and immediately trigger the observation block again. This would result in an infinite loop where saving the token triggers a new sync, which updates the token, which triggers another sync endlessly. Marking it with @ObservationIgnored keeps this property isolated from the reactive UI and observation loop, ensuring it can be read and updated safely without side effects.

Apart from the lastToken, we also need a way to batch process the transactions. If we are just iterating over the transaction.changes and performing POST request to the server then we will end up with dozens of calls for every single transaction. Instead of bombarding server with individual calls, we can store the room payloadin an array called `payloadsToSync`. For this we will implement RoomPayload and SyncAction struct. This is shown below: 

``` swift 
enum SyncAction: String, Codable {
    case insert
    case delete
    case update
}

struct RoomPayload: Codable {
    let id: UUID
    let name: String?
    let area: Double?
    let action: SyncAction
}
```

Now, we can update the processChanges function to use the lastToken and payloadsToSync array. 

``` swift 
 private func processChanges(context: ModelContext) {
        
        do {
            let descriptor: HistoryDescriptor<DefaultHistoryTransaction>
            
            if let lastToken {
                descriptor = HistoryDescriptor<DefaultHistoryTransaction>(predicate: #Predicate { transaction in
                    transaction.token > lastToken
                })
            } else {
                descriptor = HistoryDescriptor<DefaultHistoryTransaction>()
            }
            
            let history = try context.fetchHistory(descriptor)
            
                 for transaction in history {
                
                for change in transaction.changes {
                    switch change {
                        case .insert(let insert as DefaultHistoryInsert<Room>):
                            if let room = context.model(for: insert.changedPersistentIdentifier) as? Room {
                                let payload = RoomPayload(id: room.syncId, name: room.name, area: room.area, action: .insert)
                                payloadsToSync.append(payload)
                            }
                        
                        case .update(let update as DefaultHistoryUpdate<Room>):
                            if let room = context.model(for: update.changedPersistentIdentifier) as? Room {
                                    let payload = RoomPayload(id: room.syncId, name: room.name, area: room.area, action: .update)
                                    payloadsToSync.append(payload)
                                 }
                            }
                        
                        case .delete:
                                break
                            
                        default:
                            break
                    }
                }
            }
            
            // upload Rooms to the server
            
            if let latestToken = history.last?.token {
                self.lastToken = latestToken
            }
            
        } catch {
            print(error.localizedDescription)
        }
```

### Syncing the Payload 

Once our payloadsToSync array is populated with changes, we can call our sync service to upload rooms to the server. This is shown in the implementation below: 

``` swift 
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

You can implement your server in any language and framework you want. I have implemented a basic endpoint using Node and ExpressJS. The endpoint simply gets the posted data and then logs it to the terminal. 

``` js 
app.post('/api/rooms', (req, res) => {
    const rooms = req.body
    console.log(rooms)
    res.status(200).json({ success: true })
})
```

Here is one of the sample outputs from the terminal: 

``` js 
[
  {
    area: 250,
    name: 'Kitchen',
    id: '4D9B718D-B802-49EA-B2F4-D557E544588B',
    action: 'insert'
  }
]
[
  {
    id: 'B646399D-BCE4-4D8F-B2BB-568002FC6E03',
    action: 'insert',
    name: 'Bedroom',
    area: 300
  }
]
```

At this point you might be wondering but what about delete. We have implemented insert and update but why not implement delete the same way. The main reason, delete cannot be implemented in a similar way is because we will not be able to use `context.model(for: insert.changedPersistentIdentifier)` to fetch the model after it has been deleted. And the reason is that it no longer exists. There are number of techniques you can use to fix this issue in the next section, we will cover a common technique used in these scenarios. 

### Implementing Soft Deletes (IsDeleted)

A common and highly effective technique to handle deletions in a synchronized database is to use a soft delete. Instead of immediately purging the record from your local database using context.delete(), you mark the record as "deleted" using a boolean flag.

This flags the record as inactive, allowing your UI to instantly hide it while preserving the actual data on disk just long enough for your sync engine to read it, capture its unique identifier, and replicate the deletion to your server.

To implement this, we can introduce a boolean property isDeleted to our Room model:

``` swift 
@Model
class Room {
    @Attribute(.unique) var syncId: UUID = UUID()
    var name: String
    var area: Double
    var isDeleted: Bool = false // The soft-delete flag
    
    init(syncId: UUID = UUID(), name: String, area: Double) {
        self.syncId = syncId
        self.name = name
        self.area = area
    }
}
```

Now, when a user deletes a room in your interface, you simply set isDeleted to true and save the context. Inside our RoomSyncManager, we can handle this change inside the update case. This is shown below: 

``` swift 
   case .update(let update as DefaultHistoryUpdate<Room>):
                        if let room = context.model(for: update.changedPersistentIdentifier) as? Room {
                            
                            if room.isDeleted {
                                let payload = RoomPayload(id: room.syncId, name: nil, area: nil, action: .delete)
                                payloadsToSync.append(payload)
                            } else {
                                let payload = RoomPayload(id: room.syncId, name: room.name, area: room.area, action: .update)
                                payloadsToSync.append(payload)
                            }
                            
                        }
```

If the room is marked with isDeleted = true then we create a RoomPayload with delete action, we also don't need to send the name and area for deleting so we set those values to be nil.  



