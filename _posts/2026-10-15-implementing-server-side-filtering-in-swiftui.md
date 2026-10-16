# Implementing Server-Side Filtering in SwiftUI

In the [previous](https://azamsharp.com/2026/10/13/implementing-server-side-sorting-in-swiftui.html) article, we learned how to implement server-side sorting in a SwiftUI application. Sorting allows users to organize hotels based on price, ratings, and other criteria. But what if users only want to see hotels that provide breakfast, parking, or free Wi-Fi?

In this article, we will extend our existing application to support server-side filtering. We will start by updating our Express.js server to filter hotels based on the provided query string parameters. Next, we will implement a `FilterView` in SwiftUI, allowing users to select multiple filters and apply them to the hotel listings.

We will also see how to combine filtering with our existing sorting functionality so users can narrow down the results and sort them based on their preferences.

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

# Building the Server Filtering 

Let's start with our server, which is already configured to perform sorting. We will extend the existing implementation to support filtering. Users will be able to filter hotels based on different criteria, such as whether the hotel provides breakfast, parking, Wi-Fi, or airport shuttle services.

All these options will be passed to the server using query string parameters.

For example, the following URL will return hotels that provide breakfast, parking, and Wi-Fi.

`http://localhost:8080/api/hotels?breakfast=true&parking=true&wifiIncluded=true`

Keep in mind that only hotels that satisfy all three conditions will be returned. If a hotel provides breakfast and parking but does not offer Wi-Fi, it will not be included in the results.

Below, you can find the server implementation:

``` js 
const FILTER_OPTIONS = [
    "breakfast",
    "parking",
    "wifiIncluded",
    "airportShuttle"
]

app.get("/api/hotels", (req, res) => {

    // get the sort options 
    const { sort } = req.query

    let filteredHotels = filterHotels(hotels, req.query)
    filteredHotels = sortHotels(filteredHotels, sort)

    res.json(filteredHotels)
})

const filterHotels = (hotels, query) => {

    let results = [...hotels]

    FILTER_OPTIONS.forEach(filter => {
        if (query[filter] === "true") {
            results = results.filter(hotel => hotel[filter])
        }
    })

    return results
}
```

The filterHotels function is responsible for filtering the hotels based on the query string parameters. We start by creating a copy of the hotels array and then iterate through the FILTER_OPTIONS array to check which filters have been provided.

If the value of a filter is "true", we apply that filter to the results. Since we are updating the results after each filter, only hotels that satisfy all the selected conditions will be returned.

The nice thing about this approach is that we don't have to write separate filtering logic for each option. If we decide to support additional filters in the future, we can simply add them to the FILTER_OPTIONS array, provided the corresponding property exists on our hotel model.

Once the filtering is complete, we pass the filtered hotels to the sortHotels function so the results can also be sorted based on the user's selection.

> For larger applications, it would be a good idea to organize the server-side code using the MVC (Model View Controller) pattern. For this article, I have kept everything inside the app.js file to keep things simple.

Now that our server supports filtering, let's move to the SwiftUI client and see how we can allow users to select and apply filters.

### Adding Filtering to SwiftUI Client 

Now that our server supports filtering, let's move to the SwiftUI client and implement the user interface for selecting filters.

We will create a FilterView that displays all the available filtering options and allows users to select multiple filters. Just like our SortView, we will also provide an option to close the view without applying any changes.

Here is the complete implementation of FilterView.

``` swift 
struct FilterView: View {
    
    @State private var selectedFilters: Set<FilterOption>
    let onDone: (Set<FilterOption>) -> Void
    
    @Environment(\.dismiss) private var dismiss
    
    init(initialFilters: Set<FilterOption>, onDone: @escaping (Set<FilterOption>) -> Void) {
        _selectedFilters = State(initialValue: initialFilters)
        self.onDone = onDone
    }
    
    var body: some View {
        VStack(alignment: .leading) {
            
            HStack {
                Button("Close") {
                    dismiss()
                }
                
                Spacer()
                
                Button("Clear") {
                    selectedFilters = []
                }
                
            }.padding([.bottom], 10)
            
            ForEach(FilterOption.allCases) { option in
                HStack {
                    Image(systemName: selectedFilters.contains(option) ? "checkmark.square.fill": "square")
                    Text(option.title)
                }.contentShape(Rectangle())
                .onTapGesture {
                    if selectedFilters.contains(option) {
                        selectedFilters.remove(option)
                    } else {
                        selectedFilters.insert(option)
                    }
                }
            }
            
            Button("Done") {
                onDone(selectedFilters)
                dismiss()
            }.buttonStyle(.borderedProminent)
            
        }.padding()
    }
}
```

Just like SortView, we pass initialFilters from the parent so that FilterView can initialize its state with the previously selected filters. We are also using a closure to send the selected filters back to the parent when the user taps the Done button. This allows users to cancel their selections without affecting the filters already applied.

One important thing to note is that we are using a Set to represent the selected filters. Since users can select multiple filters, we could have used an array, but a Set is more suitable for this scenario because it does not allow duplicate values. It also makes it easy to check whether a particular filter is selected using the contains function.

> Just like SortView, our FilterView also includes Close and Clear buttons. Whether these buttons should be part of the individual views or handled by the parent depends on how you plan to use these views in your application.

Now, let's update our HotelStore to support the selected filters.

``` swift 
 func loadHotels(sortOption: SortOption? = nil, filters: Set<FilterOption>? = nil) async throws {
    let resource = Resource(url: Constants.URLs.hotels(sort: sortOption, filters: filters), responseType: [Hotel].self)
    hotels = try await httpClient.fetch(resource: resource)
    }
```

We have added filters as a second optional argument to the loadHotels function. The selected filters, along with the sort option, are passed to the URL generator, which creates a URL containing the appropriate query string parameters.

This allows us to use the same endpoint for both sorting and filtering without making any additional changes to the server.

Below, you can see the updated implementation of the hotels function inside the Constants.URLs struct.

``` swift 
struct Constants {

    struct URLs {

        static let base = URL(string: "http://localhost:8080/api/")!

        static func hotels(sort: SortOption? = nil,filters: Set<FilterOption>? = nil) -> URL {

            let url = base.appending(path: "hotels")

            var queryItems: [URLQueryItem] = []

            if let sort {
                queryItems.append(
                    URLQueryItem(
                        name: "sort",
                        value: sort.rawValue
                    )
                )
            }

            if let filters {

                for option in filters {
                    queryItems.append(
                        URLQueryItem(
                            name: option.rawValue,
                            value: "true"
                        )
                    )
                }
            }

            return url.appending(queryItems: queryItems)
        }
    }
}
```

Finally, in HotelListScreen, we have replaced the showSortSheet boolean with an enum since we now have two different sheets to display: SortView and FilterView. Using an enum allows us to manage multiple sheets using a single .sheet modifier.

``` swift 
  .sheet(item: $activeSheet, content: { sheet in
            switch sheet {
                case .filter:
                FilterView(initialFilters: filters) { filters in
                    self.filters = filters
                    Task { await loadHotels() }
                }.presentationDetents([.fraction(0.25)])
                case .sort:
                SortView(initialSelection: sortOption) { option in
                    sortOption = option
                    Task { await loadHotels() }
                }.presentationDetents([.fraction(0.25)])
            }
        })
```

When the user taps the Done button, the closure is called with the selected filters or sort option. We update the corresponding property in HotelListScreen and then call the loadHotels function to fetch the updated results from the server.

The loadHotels function uses the current filters and sort option to construct the URL with the appropriate query string parameters. This allows us to apply both sorting and filtering in a single HTTP request.

One thing you might be wondering is why we are explicitly calling loadHotels after updating the filters or sort option. We could also use the .task(id:) modifier to automatically reload the hotels when these values change. Both approaches work, but for this example, I prefer to explicitly call loadHotels when the user confirms their selection by tapping the Done button.

### Source Code 

You can download the source code from GitHub: 
[https://github.com/azamsharpschool/HotelApp](https://github.com/azamsharpschool/HotelApp)

### Conclusion 

In this article, we learned how to implement server-side filtering in a SwiftUI application. We started by updating our Express.js server to accept filter options through query string parameters. We then implemented a `FilterView`, allowing users to select multiple filters and apply them to the hotel listings.

One interesting part of our implementation is the use of a `Set` to manage the selected filters. Since users can select multiple options, a `Set` works nicely because it automatically prevents duplicate values. We also used a closure to communicate the selected filters back to the parent view, allowing users to cancel their selections without affecting the existing filters.

Finally, we updated our `HotelStore` and URL generation logic to send the selected filters to the server. The nice thing about this approach is that filtering and sorting can work together without requiring separate endpoints.

In the next article, we will extend our application even further by implementing server-side pagination, allowing users to load hotels in smaller batches instead of fetching all the results at once.

### Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school)

