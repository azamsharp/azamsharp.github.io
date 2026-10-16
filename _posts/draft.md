# Implementing Infinite Scrolling in SwiftUI + ExpressJS 

When working with a large dataset, it is usually not a good idea to download all the records in a single request. Not only does this increase the amount of data transferred over the network, but it can also negatively impact the performance of the application.

One way to solve this problem is through pagination, where the server returns a small subset of records instead of the entire dataset. On the client side, we can take this a step further by implementing infinite scrolling, which allows us to fetch additional records as the user scrolls through the list.

In this article, we will implement infinite scrolling in our SwiftUI hotel application using an ExpressJS backend. We will also learn how to handle pagination when the user changes the sorting and filtering options.

### Prerequisites

This article builds on our previous implementations of server-side sorting and filtering in SwiftUI. If you have not followed along with those articles, I recommend checking them out before continuing.

- **[Implementing Server-Side Sorting in SwiftUI](https://azamsharp.com/2026/10/13/implementing-server-side-sorting-in-swiftui.html)** — Learn how to sort hotels using an ExpressJS backend and manage sorting preferences in SwiftUI.

- **[Implementing Server-Side Filtering in SwiftUI](https://azamsharp.com/2026/10/15/implementing-server-side-filtering-in-swiftui.html)** — Learn how to filter hotels based on amenities and combine filtering with sorting to display relevant results.

In this article, we will extend the same hotel application by adding pagination and infinite scrolling, allowing users to load additional hotels as they scroll through the list.

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

### Implementing Pagination on the Server

For infinite scrolling to work, we first need to implement pagination on the server. The server is responsible for extracting the page and limit query string parameters and returning a subset of hotels based on those values.

Let's start by implementing the paginateHotels function, as shown below:

``` swift 
const paginateHotels = (pageNumber, pageLimit, hotels) => {

    let startIndex = (pageNumber - 1) * pageLimit
    let endIndex = startIndex + pageLimit
    
    const paginatedHotels = hotels.slice(startIndex, endIndex)
    return paginatedHotels
}
```

The implementation is quite straightforward. We calculate the starting index based on the requested page number and page limit. We then use the JavaScript slice function to return only the hotels that fall within that range.

For example, consider the following URL:

`http://localhost:8080/api/hotels?page=3&limit=5`

In this case, we are requesting page 3 with a limit of 5 hotels per page. This means the server will return records 11–15, assuming there are enough records available.

Now, let's take a look at the complete implementation of the endpoint:

``` swift 
app.get("/api/hotels", (req, res) => {

    // get the sort options 
    const { sort, page = 1, limit = 10 } = req.query

    const pageNumber = Number(page)
    const pageLimit = Math.min(Number(limit), 50)

    let filteredHotels = filterHotels(hotels, req.query)
    filteredHotels = sortHotels(filteredHotels, sort)
    
    // paginate 
    filteredHotels = paginateHotels(pageNumber, pageLimit, filteredHotels)
    res.json(filteredHotels)
})

const paginateHotels = (pageNumber, pageLimit, hotels) => {

    let startIndex = (pageNumber - 1) * pageLimit
    let endIndex = startIndex + pageLimit
    
    const paginatedHotels = hotels.slice(startIndex, endIndex)
    return paginatedHotels
}
```

Notice that we first apply the filters and sorting options before performing pagination. This is important because we want to paginate the filtered and sorted results, not the original collection of hotels.

We also limit the maximum number of hotels returned in a single request to 50. This prevents the client from requesting an unnecessarily large number of records.

For a production application, we should also validate that page and limit are positive integers before using them.

Now that our server supports pagination, let's move to the client side and see how we can use it to implement infinite scrolling in SwiftUI.

### Handling Pagination on the Client Side

Infinite scrolling works by downloading additional data as the user scrolls down the screen. You can think of it as on-demand loading, where we only fetch more hotels when they are needed instead of downloading the entire collection in the initial request.

Infinite scrolling is commonly used by social media applications like Twitter, Facebook, Instagram, and TikTok, where additional content is continuously loaded as the user scrolls through the feed.

The first thing we need to figure out is when to make the next request to the server. Ideally, we want to fetch additional hotels when the user reaches the end of the currently loaded list.

One way to accomplish this is by comparing the current hotel's ID with the ID of the last hotel in the collection. If both IDs match, we know that the last hotel cell has appeared, and we can request the next page.

This is shown in the implementation below:

``` swift 
 ForEach(hotelStore.hotels) { hotel in
                HotelCellView(hotel: hotel)
                    .task {
                        if hotel.id == hotelStore.hotels.last?.id {
                            await loadHotels()
                        }
                    }
            }
```

As you can see, we have attached a task modifier to HotelCellView. SwiftUI executes the task when the view appears. Inside the task, we check whether the current hotel is the last hotel in the collection. If it is, we call the loadHotels function to fetch the next page from the server.

Keep in mind that the task modifier is tied to the view's lifecycle and may execute more than once as views appear again. This means our store should prevent duplicate requests.

When using a List, SwiftUI creates and manages rows lazily, so it does not need to create every row at once. This helps improve performance when displaying large collections of hotels. However, it does not mean that only the rows currently visible on the screen are instantiated.

Now, let's take a look at the updated loadHotels function in HotelStore.

``` swift 
func loadHotels(sortOption: SortOption? = nil, filters: Set<FilterOption>? = nil, pageSize: Int = 10) async throws {
        
         let queryChanged = (self.sortOption != sortOption) || (self.filters != filters) || (self.pageSize != pageSize)
        let page = queryChanged ? 1: currentPage
        
        guard queryChanged || hasMoreHotels else { return }
        
        let resource = Resource(url: Constants.URLs.hotels(sort: sortOption, filters: filters, page: page, pageSize: pageSize), responseType: [Hotel].self)
        print(resource.url)
        let newHotels = try await httpClient.fetch(resource: resource)
        
        if page == 1 {
            hotels = newHotels
        } else {
            hotels.append(contentsOf: newHotels)
        }
        
        self.sortOption = sortOption
        self.filters = filters
        
        currentPage = page + 1
        hasMoreHotels = newHotels.count == pageSize
    }
```

There are a few interesting things happening in this implementation.

First, we check whether the sorting, filtering or page size have changed. If they have, we reset the page number to 1. This is important because the new sorting, filtering or page size options may produce a completely different collection of hotels, and we want to start displaying those results from the beginning.

Next, we check the hasMoreHotels property to determine whether we should make another request. If there are no additional hotels to fetch and the query has not changed, we simply return without making a network request.

Once the server returns the hotels, we check the page number. If the page number is 1, we replace the existing hotels with the new results. Otherwise, we append the newly downloaded hotels to the existing collection.

Finally, we increment currentPage so the next request fetches the following page. We also update hasMoreHotels by comparing the number of returned hotels with the requested page size.

If the server returns fewer hotels than the requested page size, we assume that there are no additional hotels available.

There are also two improvements worth considering before using this implementation in a production application. The first is to track whether a request is already in progress using an isLoading property. This prevents multiple requests from being triggered when the last hotel cell appears more than once. The second is to include pageSize when checking whether the query has changed. Otherwise, changing the page size could result in skipped or duplicated hotels.

With these pieces in place, we have the basic functionality needed to support infinite scrolling in our SwiftUI application.

### Source Code 

You can download the source code from GitHub: 
[https://github.com/azamsharpschool/HotelApp](https://github.com/azamsharpschool/HotelApp)

### Conclusion 

Infinite scrolling is a great way to load data on demand without requiring users to manually navigate between pages. By implementing pagination on the server and detecting when the user reaches the end of the list in SwiftUI, we can create a seamless scrolling experience.
We also learned how to reset pagination when sorting or filtering options change, ensuring that the user always sees the correct results. This approach is not limited to hotels and can be applied to products, recipes, social media feeds, or any application that displays a large collection of records.

### Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school)