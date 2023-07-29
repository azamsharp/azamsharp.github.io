
When I started learning SwiftUI in 2019, I adopted MVVM pattern for my SwiftUI architecture. Most of my apps were client/server based, which means they were consuming a JSON API.

Each time I added a view, I also added a view model for that view. Even though the source of truth never changed. The source of truth was still the server. This is a very important point as source of truth plays an important role in SwiftUI applications. As new screens were added, new view models were also added for each screen. And before I know it, I was dealing with dozens of view models, each still communicating with the same server and retrieving the information. The actual request was initiated and handled by the HTTPClient/Webservice layer. 

Even with a medium sized apps with 10-15 screens, it was becoming hard to manage all the view models. I was also having issues with access values from EnvironmentObjects. This is because ```@EnvironmentObject``` property wrapper is not available inside the view models. 

After a lot of research, experimentation I later concluded that view models for each screen is not required when building SwiftUI applications. If my view needs data then it should be given to the view directly. There are many ways to accomplish it. Below, my view is consuming the data from a JSON API. The view uses the HTTPClient to fetch the data. HTTPClient is completely stateless, it is just used to perform a network call, decode the response and give the results to the caller. 

 This is shown in the implementation below: 


``` swift
struct ConferenceListScreen: View {
    
    @Environment(\.httpClient) private var httpClient
    @State private var conferences: [Conference] = []
    
    private func loadConferences() async {
        let resource = Resource(url: Constants.Urls.conferences, modelType: [Conference].self)
        do {
            conferences = try await httpClient.load(resource)
        } catch {
            // show error view
            print(error.localizedDescription) 
        }
    }
     
    var body: some View {
        
        Group {
            if conferences.isEmpty {  
                ProgressView()
            } else { 
                List(conferences) { conference in
                    NavigationLink(value: conference) {
                        ConferenceCellView(conference: conference)
                    }
                }
                .listStyle(.plain)
            }
        }
        .task {
            await loadConferences()
        }
    }
}
```

Sometimes, we need to fetch the data and then hold on to it so other views can also access and even modify the data. For those cases, we can use @Binding to send the data to the child view or even put the data in @EnvironmentObject. The @EnvironmentObject implementation is shown below: 

``` swift
class StoreModel: ObservableObject {
    
    private var storeHTTPClient: StoreHTTPClient
    
    init(storeHTTPClient: StoreHTTPClient) {
        self.storeHTTPClient = storeHTTPClient
    }
    
    @Published var products: [Product] = []
    @Published var categories: [Category] = []
    
    func addProduct(_ product: Product) async throws {
         try await storeHTTPClient.addProduct(product)
    }
    
    func populateProducts() async throws {
        self.products = try await storeHTTPClient.loadProducts()
    }
}
```

Inject it into the root view as shown below: 

``` swift 
@main
struct StoreAppApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(StoreModel(client: StoreHTTPClient()))
            
        }
    }
}
```

And then access it in the view as shown below: 

``` swift
struct ContentView: View {

    @EnvironmentObject private var model: StoreModel
    
    var body: some View {
        ProductListView(products: model.products)
            .task {
                do {
                    try await model.populateProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
    }
}
```

Now, when most readers read the above code they say "Oh you change the name of view model to StoreModel or DataStore etc and thats it". NO! Look carefully. We are no longer creating view model per screen. We are creating a single thing that maintains an entire state of the application. I am calling that thing StoreModel (E-Commerce) but you can call it anything you want. You can call it DataStore etc. 

The main point is that when working with SwiftUI applications, the view is already a view model so you don't have to add another layer of redirection. 

Your next question might be what about larger apps! Great question! I have written a very detailed article on SwiftUI Architecture that you can read below: 

https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html

I have also written a detailed article on SwiftData. The same concepts can be applied when building Core Data applications.  

https://azamsharp.com/2023/07/04/the-ultimate-swift-data-guide.html

NOTE: Appeloper is 100% correct. Don'nt needlessly add layers to your app. It will only make your app more complicated. 

