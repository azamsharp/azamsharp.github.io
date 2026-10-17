# Simplifying Sorting, Filtering, and Pagination in SwiftUI Using HotelQuery

In the previous articles, we implemented server-side sorting, filtering, and pagination in our SwiftUI hotel application. Each feature works independently, but they also work together to control which hotels are returned from the server.

As we continue adding features, we need to make sure that our code remains simple and easy to maintain. One area that can benefit from improvement is how we pass sorting, filtering, and pagination options to our `HotelStore`.

Currently, our `loadHotels` function accepts several individual parameters. Although this approach works, it can become difficult to manage as we introduce more options.

In this article, we will simplify our implementation by grouping these options into a single `HotelQuery` structure. We will also see how this change makes our `HotelStore` API cleaner and reduces the amount of state we need to manage in our SwiftUI views.

## Prerequisites

This article builds on our previous implementations of server-side sorting, filtering, and pagination in SwiftUI. If you are new to this series, I recommend reading the following articles before continuing:

- [Implementing Server-Side Sorting in SwiftUI](https://azamsharp.com/2026/10/13/implementing-server-side-sorting-in-swiftui.html) — Implement sorting options and send the selected sort order to an Express.js server.
- [Implementing Server-Side Filtering in SwiftUI](https://azamsharp.com/2026/10/15/implementing-server-side-filtering-in-swiftui.html) — Filter hotels based on selected amenities using query string parameters.
- [Implementing Server-Side Pagination in SwiftUI](https://azamsharp.com/2026/10/16/implementing-server-side-pagination-in-swiftui.html) — Load hotels incrementally as the user scrolls through the list.

In this article, we will build on these implementations and simplify how sorting, filtering, and pagination options are managed by introducing a `HotelQuery` structure.

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

## Grouping Options into HotelQuery

At present, we have `sortOption`, `filters`, `page`, and `limit` passed to the `loadHotels` function as individual parameters. There is nothing inherently wrong with this approach, but as the number of parameters increases, the function becomes harder to use.

Instead of passing these values individually, we can group the related options into a common structure called `HotelQuery`.

The implementation of `HotelQuery` is shown below:

```swift
struct HotelQuery: Equatable {
    var sort: SortOption?
    var filters: Set<FilterOption> = []
    var pageSize: Int = 10
}
```

The `HotelQuery` structure represents the options that determine which hotels should be returned from the server.

The `sort` property determines how the hotels should be ordered. The `filters` property contains the selected filtering options, and `pageSize` determines the maximum number of hotels returned in a single request.

You may have noticed that we did not include the current page number in `HotelQuery`.

This is intentional.

The current page number is not really a selection made by the user. It is part of the pagination state managed by `HotelStore`. The user can change the sorting options, apply filters, or select a different page size, but the store is responsible for keeping track of which page should be loaded next.

By keeping the page number inside `HotelStore`, we can separate the options selected by the user from the internal state required to perform pagination.

Another important detail is that `HotelQuery` conforms to `Equatable`. This allows us to compare two queries and determine whether the user has changed any of the options.

For example:

```swift
let query1 = HotelQuery()
let query2 = HotelQuery(pageSize: 20)

print(query1 == query2) // false
```

This comparison becomes particularly useful when implementing pagination because changing any query option should cause the store to load hotels from the first page.

**Note:** For synthesized `Equatable` conformance to work, `SortOption` and `FilterOption` must also conform to `Equatable`. In our implementation, `FilterOption` is also `Hashable` because it is stored in a `Set`.

## Updating HotelStore to Use HotelQuery

Now that we have introduced `HotelQuery`, we can update our `HotelStore` implementation to accept a single query argument.

The updated `loadHotels` function is shown below:

```swift
func loadHotels(_ query: HotelQuery) async throws {
    guard !isLoading else { return }

    let queryChanged = self.query != query
    let page = queryChanged ? 1 : currentPage

    guard queryChanged || hasMoreHotels else { return }

    isLoading = true
    defer { isLoading = false }

    let resource = Resource(
        url: Constants.URLs.hotels(query: query, page: page),
        responseType: [Hotel].self
    )

    let newHotels = try await httpClient.fetch(resource: resource)

    if page == 1 {
        hotels = newHotels
    } else {
        hotels.append(contentsOf: newHotels)
    }

    self.query = query
    currentPage = page + 1
    hasMoreHotels = newHotels.count == query.pageSize
}
```

As you can see, we have replaced multiple arguments with a single `HotelQuery` argument. This may seem like a small change, but it simplifies the API and makes it easier to introduce additional query options in the future.

Let's take a closer look at how the function works.

### Detecting Changes to the Query

One of the first things we do inside `loadHotels` is compare the incoming query with the previously used query.

```swift
let queryChanged = self.query != query
let page = queryChanged ? 1 : currentPage
```

If the query has changed, we reset the page number to `1`. Otherwise, we continue loading from the current page.

For example, imagine that the user is browsing hotels sorted by price. After scrolling through several pages, the user decides to sort the hotels by guest rating instead.

Since the sorting option has changed, we should not continue loading hotels from the previous page number. Instead, we need to start from the first page using the newly selected sorting option.

The same behavior applies when the user changes filters or the page size.

This is one of the main benefits of using `HotelQuery`. Instead of checking each option individually, we can simply compare the entire query.

### Preventing Unnecessary Requests

Next, we check whether another request is already in progress.

```swift
guard !isLoading else { return }
```

This prevents multiple requests from being started at the same time.

We also check whether we need to load more hotels.

```swift
guard queryChanged || hasMoreHotels else { return }
```

If the query has changed, we need to load hotels from the first page, even if the previous query had already reached the end of its results.

If the query has not changed, we only continue when more hotels are available.

This allows the same function to handle both loading a new set of results and loading additional pages.

### Replacing or Appending Hotels

Once we receive the response from the server, we decide whether to replace the existing hotels or append the new results.

```swift
if page == 1 {
    hotels = newHotels
} else {
    hotels.append(contentsOf: newHotels)
}
```

When the page number is `1`, we replace the existing hotels. This happens when the query changes or when hotels are loaded for the first time.

For subsequent pages, we append the newly fetched hotels to the existing collection.

Finally, we update the pagination state.

```swift
self.query = query
currentPage = page + 1
hasMoreHotels = newHotels.count == query.pageSize
```

The `hasMoreHotels` property is determined by comparing the number of returned hotels with the requested page size.

If the server returns fewer hotels than requested, we assume that there are no additional pages.

Keep in mind that receiving exactly `pageSize` hotels does not guarantee that another page exists. It only means that another request may be necessary. A more sophisticated API could return additional pagination metadata, such as `hasNextPage` or `totalCount`, to make this determination more accurate.

Also, the current implementation prevents overlapping requests by returning when `isLoading` is true. For this simple application, that is a reasonable starting point. In a production application, we may also want to handle a query change that occurs while an earlier request is still in progress, so that the latest selection is not ignored.

## Updating the URL Builder

Now that we have introduced `HotelQuery`, we can also simplify our URL builder. Previously, we passed sorting, filtering, and pagination options as individual arguments. We can now pass a single `HotelQuery` along with the page number.

The updated implementation is shown below:

```swift
struct Constants {

    struct URLs {

        static let base = URL(string: "http://localhost:8080/api/")!

        static func hotels(query: HotelQuery, page: Int = 1) -> URL {

            let url = base.appending(path: "hotels")

            var queryItems: [URLQueryItem] = [
                URLQueryItem(name: "page", value: String(page)),
                URLQueryItem(name: "limit", value: String(query.pageSize))
            ]

            if let sort = query.sort {
                queryItems.append(
                    URLQueryItem(
                        name: "sort",
                        value: sort.rawValue
                    )
                )
            }

            for option in query.filters {
                queryItems.append(
                    URLQueryItem(
                        name: option.rawValue,
                        value: "true"
                    )
                )
            }

            return url.appending(queryItems: queryItems)
        }
    }
}
```

As you can see, the URL builder now accepts `HotelQuery` and uses its properties to construct the query string.

We start by adding the page number and page size. If a sorting option is selected, we append it to the query string. Finally, we loop through the selected filters and append each one as a separate query parameter.

For example, if the user selects sorting by price and enables breakfast and parking filters, the resulting URL may look like this:

```text
http://localhost:8080/api/hotels?page=1&limit=10&sort=priceLowToHigh&breakfast=true&parking=true
```

Notice that we are still passing the page number separately. This is because the page number is managed internally by `HotelStore`, while `HotelQuery` represents the options selected by the user.

Another benefit of this approach is that the URL builder does not need to know anything about `HotelStore` or `HotelListScreen`. It simply accepts a query and returns the appropriate URL.

If we decide to add additional query options in the future, we can update `HotelQuery` and the URL builder without changing the signature of our `loadHotels` function.

## Simplifying HotelListScreen

Now that `HotelStore` accepts a `HotelQuery`, we can simplify how we manage sorting and filtering options inside `HotelListScreen`.

Previously, we maintained separate state properties for sorting and filtering.

```swift
@State private var sortOption: SortOption?
@State private var filters: Set<FilterOption> = []
```

There is nothing wrong with maintaining these properties separately, but since they are related to the same hotel query, we can group them into a single state property.

```swift
@State private var query = HotelQuery()
```

This means that `HotelListScreen` now has a single source of truth for the options used to fetch hotels.

Let's see how this affects our sheet implementation.

```swift
struct HotelListScreen: View {

    @Environment(HotelStore.self) private var hotelStore

    @State private var activeSheet: Sheet?
    @State private var query = HotelQuery()

    // other code

    var body: some View {
        // List and other UI

        .sheet(item: $activeSheet) { sheet in
            switch sheet {
            case .filter:
                FilterView(initialFilters: query.filters) { filters in
                    query.filters = filters
                    Task { await loadHotels() }
                }
                .presentationDetents([.fraction(0.25)])

            case .sort:
                SortView(initialSelection: query.sort) { option in
                    query.sort = option
                    Task { await loadHotels() }
                }
                .presentationDetents([.fraction(0.25)])
            }
        }
    }

    private func loadHotels() async {
        do {
            try await hotelStore.loadHotels(query)
        } catch {
            print(error.localizedDescription)
        }
    }
}
```

*The code above focuses on the query-related implementation. The remaining list UI and sheet definitions are omitted for brevity.*

The interesting part is how the selected sorting and filtering options are updated.

When the user confirms their filtering options, we update the `filters` property of `HotelQuery`.

```swift
query.filters = filters
Task { await loadHotels() }
```

Similarly, when the user confirms a sorting option, we update the `sort` property.

```swift
query.sort = option
Task { await loadHotels() }
```

Both operations call the same `loadHotels` function, which passes the complete query to `HotelStore`.

```swift
private func loadHotels() async {
    do {
        try await hotelStore.loadHotels(query)
    } catch {
        print(error.localizedDescription)
    }
}
```

The view does not need to determine whether sorting or filtering has changed. It simply passes the current query to the store.

The store is responsible for comparing the query, resetting pagination when necessary, and fetching the appropriate hotels.

This is a much cleaner separation of responsibilities.

## Why Do We Have HotelQuery in Both HotelListScreen and HotelStore?

You might have noticed that we have a `HotelQuery` property in both `HotelListScreen` and `HotelStore`.

In `HotelListScreen`, we have the following:

```swift
@State private var query = HotelQuery()
```

And in `HotelStore`, we also maintain a query:

```swift
private var query = HotelQuery()
```

At first glance, this may look like we are duplicating state. But these two properties serve different purposes.

The `HotelQuery` inside `HotelListScreen` represents the user's current selections. When the user changes sorting, applies filters, or modifies the page size, we update this query and pass it to `HotelStore`.

The `HotelQuery` inside `HotelStore`, on the other hand, is used to keep track of the query associated with the previously loaded hotels.

This allows us to compare the incoming query with the stored query.

```swift
let queryChanged = self.query != query
let page = queryChanged ? 1 : currentPage
```

If the query has changed, we start loading hotels from the first page. Otherwise, we continue loading the next page.

Once the hotels have been successfully fetched, we update the stored query.

```swift
self.query = query
```

Notice that we only update the stored query after the request succeeds. This is important because if the request fails, we don't want to update the query associated with our existing results.

Another important detail is that the query inside `HotelStore` is marked as `private`. This means that the view cannot directly modify it. The only way to provide new query options is through the `loadHotels` function.

I like this approach because it keeps the responsibilities clear. The view manages the user's selections, while the store keeps track of which query was used to load the hotels and manages the pagination state.

Although both properties use the same `HotelQuery` type, they represent different information and are used for different purposes.

## Making HotelQuery Easier to Extend

Another benefit of introducing `HotelQuery` is that it gives us a convenient place to add new options.

For example, suppose we decide to introduce a search feature that allows users to search hotels by name.

We can simply add a new property to `HotelQuery`.

```swift
struct HotelQuery: Equatable {
    var searchText: String = ""
    var sort: SortOption?
    var filters: Set<FilterOption> = []
    var pageSize: Int = 10
}
```

Since `HotelQuery` conforms to `Equatable`, changing `searchText` will automatically cause the new query to compare differently from the previous one.

This means our existing pagination logic can detect the change and start loading hotels from the first page.

We would still need to update the URL builder and the server to support the search parameter, but we would not need to change the signature of `HotelStore.loadHotels`.

```swift
func loadHotels(_ query: HotelQuery) async throws
```

This is one of the advantages of grouping related parameters into a dedicated type. We can extend the query without continually changing the function signatures throughout our application.

It is also worth mentioning that `HotelQuery` is specific to our hotel application. If we were building a product catalog, we could introduce a `ProductQuery` with properties relevant to products.

Although both queries may share sorting, filtering, and pagination concepts, that does not necessarily mean we should immediately introduce a generic query abstraction.

I prefer starting with a simple, concrete implementation and introducing abstractions only when there is a clear need for them.

## Conclusion

In this article, we simplified our SwiftUI hotel application by introducing a `HotelQuery` structure to group sorting, filtering, and pagination options.

Instead of passing several individual parameters to `HotelStore`, we now pass a single query that represents the user's current selections.

This small change provides several benefits. It simplifies the `HotelStore` API, reduces the number of state properties in `HotelListScreen`, and makes it easier to introduce additional query options in the future.

More importantly, it allows us to clearly separate the responsibilities of our application. The view manages the selected options, the store manages data loading and pagination, and the URL builder converts the query into parameters understood by the server.

As you continue building SwiftUI applications, look for opportunities to group related values into meaningful types. Not every collection of parameters needs its own structure, but when those parameters represent a single concept, introducing a dedicated type can make the code easier to understand and maintain.

## Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school)