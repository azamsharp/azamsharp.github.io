# Implementing Server-Side Sorting in SwiftUI

When building applications, it is common to display a list of items and allow users to sort them based on different criteria. For example, in a hotel booking application, users may want to sort hotels by price or guest ratings.

There are different ways to implement sorting. One approach is to fetch all the records from the server and sort them locally on the client. This works fine for smaller datasets, but as the number of records grows, fetching everything just to sort it is not a good idea. A better approach is to let the server handle sorting and return the results in the requested order.

In this article, we will learn how to implement server-side sorting using Express.js and SwiftUI. We will start by building an API that supports different sorting options through query parameters. Next, we will implement the SwiftUI client using the @Observable macro to manage our hotel data. Finally, we will create a sorting interface that allows users to select, clear, and confirm their sorting preferences.

Let's get started!

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

### Building the Sorting API with Express.js

These days, you have countless options when it comes to implementing a server. Even in the Swift ecosystem, you have frameworks like Vapor and Hummingbird. My personal favorite is Express.js, mainly because I have taught it for a long time and I also enjoy the dynamic nature of JavaScript.

One thing I like about using JavaScript on the server is that I don't have to create separate DTO objects for every response. I can simply return anonymous objects. I personally like this flexibility.

For our hotel application, we will implement an API endpoint that allows clients to sort hotels based on different criteria, such as price and guest ratings. The client will specify the sorting option using a query string parameter, and the server will be responsible for sorting the results before returning them.

Below, you can find the implementation of the /api/hotels route, which supports different sorting options.

``` swift 
app.get("/api/hotels", (req, res) => {

    // get the sort options 
    const { sort } = req.query
    // hotels is a hard-coded list 
    const sortedHotels = sortHotels(hotels, sort)
    res.json(sortedHotels)
})

const sortHotels = (hotels, sort) => {
    switch (sort) {
        case "priceLowToHigh":
            hotels.sort((a, b) => a.price - b.price)
            break
        case "priceHighToLow":
            hotels.sort((a, b) => b.price - a.price)
            break
        case "guestRating":
            hotels.sort((a, b) => b.guestRating - a.guestRating);
            break
    }

    return hotels 
}

```

The query string parameter determines how the hotels are sorted. For example, http://localhost:8080/api/hotels?sort=priceLowToHigh will return hotels sorted by price in ascending order, from lowest to highest.

Now that our server is working as expected, we can move on to implementing the SwiftUI client.

### Building the SwiftUI Hotel App

We will start by implementing our HotelStore, which will be responsible for managing the state of hotels along with the business and domain logic associated with them.

> Unlike traditional View Models, you don't need to create a separate HotelStore for each screen. HotelStore is designed around the bounded context of hotel management and can be shared across multiple screens that work with hotel-related entities and operations.

The implementation of HotelStore is shown below:

``` swift 
@Observable
class HotelStore {
    
    private(set) var hotels: [Hotel] = []
    let httpClient: HTTPClient
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func loadHotels(sortOption: SortOption? = nil) async throws {
        let resource = Resource(url: Constants.URLs.hotels(sort: sortOption), responseType: [Hotel].self)
        hotels = try await httpClient.fetch(resource: resource)
    }
}
```

HotelStore has a dependency on HTTPClient, which is responsible for performing network requests. The loadHotels function also accepts an optional SortOption, allowing us to construct the appropriate URL based on the selected sorting option.

The Constants.URLs.hotels function is responsible for constructing the URL with the necessary query string parameters. In a future article, we will extend this function to support filtering and pagination.

Inside our SwiftUI view, we can access HotelStore through the environment.

> Another option is to pass HotelStore as a dependency through the initializer. I personally prefer using the environment because I often end up using the same store across multiple screens. Passing the store through the initializer of every screen can become cumbersome.

The implementation of HotelListScreen is shown below:

``` swift 
struct HotelListScreen: View {
    
    @Environment(HotelStore.self) private var hotelStore
    @State private var showSortSheet: Bool = false
    
    @State private var sortOption: SortOption?
    
    var body: some View {
        List {
            
            HStack {
                Spacer()
                Button("Sort") {
                    showSortSheet = true
                }
            }
            
            ForEach(hotelStore.hotels) { hotel in
                HotelCellView(hotel: hotel)
            }
        }
        .sheet(isPresented: $showSortSheet, content: {
            SortView(initialSelection: sortOption, onDone: { option in
                sortOption = option
                Task { await loadHotels() }
            }).presentationDetents([.fraction(0.25)])
        })
        .task {
            await loadHotels()
        }.navigationTitle("Hotels")
    }
    
    private func loadHotels() async {
        do {
            try await hotelStore.loadHotels(sortOption: sortOption)
        } catch {
            print(error.localizedDescription)
        }
    }
}
```

You might be wondering why we pass sortOption to the loadHotels function instead of creating a separate function for sorting. You can definitely take that approach, but I personally prefer this design, which is inspired by the Sequelize ORM in JavaScript.

The idea is simple. You can call loadHotels without passing any options to fetch the default set of hotels (limited to 10 records by default), or you can provide options to customize the results. As we add more features, the same function will support sorting, filtering, and pagination without requiring separate functions for each operation.

The user selects the sorting option through SortView. The implementation is shown below:

``` swift 
struct SortView: View {
    
    @State private var selectedSortOption: SortOption?
    @Environment(\.dismiss) private var dismiss
    
    let onDone: (SortOption?) -> Void
    
    init(initialSelection: SortOption? = nil, onDone: @escaping (SortOption?) -> Void) {
        _selectedSortOption = State(initialValue: initialSelection)
        self.onDone = onDone
    }
    
    var body: some View {
        VStack {
            
            HStack {
                Button("Close") {
                    dismiss()
                }
                
                Spacer()
                
                Button("Clear") {
                    selectedSortOption = nil
                }
                
            }.padding([.bottom], 10)
            
            ForEach(SortOption.allCases) { sortOption in
                Button {
                    selectedSortOption = sortOption
                } label: {
                    HStack {
                        Image(systemName: selectedSortOption == sortOption
                              ? "largecircle.fill.circle" : "circle")

                        Text(sortOption.title)
                    }
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .contentShape(Rectangle())
                }
                .buttonStyle(.plain)
            }
            
            Button("Done") {
                onDone(selectedSortOption)
                dismiss()
                
            }.frame(maxWidth: .infinity, alignment: .leading)
            .padding([.top], 20)
            .buttonStyle(.borderedProminent)
            
        }.padding()
    }
}
```

The interesting part of this implementation is the use of a closure instead of passing an @Binding from the parent. The main reason I decided not to use @Binding is that I want users to be able to cancel their sorting selections without affecting the currently applied sort option.

If we directly bind the selected sort option to the parent's state, every selection will immediately update that state, even if the user decides to cancel.

One way to solve this is to introduce a separate draftSortOption in the parent and pass it as a binding to SortView. That approach works perfectly fine, but I prefer a simpler solution.

In our implementation, SortView manages its own temporary selection using @State. When the user taps the Done button, we pass the selected sort option back to the parent through a closure. If the user closes the sheet without tapping Done, the original sort option remains unchanged.

### Resources 

- [Hummingbird Fundamentals - Build REST APIs in Swift](https://azamsharp.teachable.com/p/hummingbird-fundamentals)

- [Full Stack iOS Development Using Swift & Vapor](https://azamsharp.teachable.com/p/swiftui-vapor-full-stack-ios-development-bootcamp)

### Conclusion 

In this article, we learned how to implement server-side sorting in a SwiftUI application. Instead of fetching all the records and sorting them on the client, we delegated the sorting responsibility to the server. This approach becomes especially useful when working with large datasets.

We also explored how to use @Observable to manage our hotel data and how to build a sorting interface using SwiftUI sheets. One interesting design decision was using a closure instead of @Binding, allowing users to confirm or cancel their selections without immediately updating the parent state.

In the next article, we will extend our implementation to support server-side filtering and pagination. This will allow users to narrow down their search results and load hotels in smaller batches instead of fetching everything at once.

### Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school)



