# The Most Important Concept in Hummingbird (Routing Explained)

In the [previous article](https://azamsharp.com/2026/04/15/getting-started-with-humming-bird.html), you got Hummingbird up and running and saw how simple it is to spin up a Swift server. But a running server by itself is not very useful. It needs to respond to requests.

That is where routing comes in.

Routing is what turns your server into something useful. It decides how your application responds when a request hits a specific URL. Whether you are returning a list of movies, fetching details for a single item, or creating new data, everything starts with a route.

In this article, you will learn how routing works in Hummingbird and how to define your own routes using parameters, query strings, and route groups.

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

### What is Routing?

Routing is how your server decides what code to run when a request comes in.

As an iOS developer, you have already worked with APIs. Every API exposes endpoints, which are simply URLs you call to perform actions like GET, POST, PUT, or DELETE. But when a request hits your server, nothing happens automatically. The server needs a way to figure out what that request actually means and what code should handle it. That is exactly what routing does.

Every request contains two important pieces of information: the path, like `/movies` or `/movies/1`, and the HTTP method, like GET or POST. Routing looks at both of these and maps the request to the correct handler in your application.

So when you call `GET /movies`, the server knows it should return a list of movies. When you call `GET /movies/1`, it returns a specific movie. And when you call `POST /movies`, it knows you are trying to create a new movie. Each of these combinations is handled by different code behind the scenes.

That mapping between the request and the code is routing. Without it, your server is just sitting there with no idea what to do. In the next section, we will start defining routes using Hummingbird.

### Implementing Routes 

When you use Hummingbird template to create a project, it automatically adds a default route to your application. Open `App+Build` file and take a look at the implementation of the `buildRouter` function. You will find the root route as shown below: 

``` swift 
 // Add default endpoint
    router.get("/") { _,_ in
        return "Hello!"
    }
```

Run your server and visit `localhost:8080`. You will see `Hello!` displayed on your page. This route returns a simple string "Hello!" back to the client. **In the future article, you will learn how to return structured data like JSON and even how to use MVC pattern when implementing your backend.**. 

Let's add a custom route to return an array of movies. 

``` swift 
router.get("/api/movies") { request, context in
        return ["Batman", "Spider man", "Lord of the Rings"]
    }
```

Your route can be anything but if you are building an API then it is usually a good practice to include the prefix `/api`. Now if you visit `localhost:8080/api/movies` you will see the following result: 

``` text 
[
  "Batman",
  "Spider man",
  "Lord of the Rings"
]
```

The above example demonstrates a simple GET route but what about POST. POST routes allows users to send in data, which can later be added to the database or any other persistence storage. Below you can find an example of POST request. 

``` swift 
  router.post("/api/movies") { request, context async throws in
        
        let movie = try await request.decode(as: CreateMovieRequest.self, context: context)
        return movie
    }
```

The important thing to note about the above code is how the POST data (JSON) is decoded into a struct called `CreateMovieRequest`. The implementation of `CreateMovieRequest` is shown below: 

``` swift 
struct CreateMovieRequest {
    let name: String
}

extension CreateMovieRequest: ResponseEncodable, Decodable, Equatable { }

```

You can use curl command or tools like [Postman](https://www.postman.com/) to perform the POST request.

``` sh 
curl -X POST http://localhost:8080/api/movies \
-H "Content-Type: application/json" \
-d '{"name": "Batman"}'
```

Going back to GET requests, let’s look at a common scenario.

What if you need to return movies based on their genre? One option is to create separate routes for each genre.

``` 
/api/movies/action  
/api/movies/comedy  
/api/movies/drama  
```

This might work initially, but it does not scale well. Every time you introduce a new genre, you need to add another route. Over time, your API starts growing in ways that are hard to manage and even harder to maintain.

A better approach is to use route parameters. Instead of creating multiple routes, you can define a single route that handles all genres dynamically. Let’s take a look at how route parameters can simplify this.

#### Route Parameters 

Route parameters allow you to pass values directly in the URL while still hitting the same endpoint. This means you don’t need to create multiple routes for each variation. Instead, you can define a single route and let part of the URL change dynamically.

In our case, rather than creating separate routes for each genre, we can use a route parameter to capture the genre from the URL itself. An example is shown below:

`/api/movies/:genre` 

The `:genre` represents the dynamic part of the route. The `:genre` can be subsituted with any value. Let's take a look how you would handle route parameters in code. 

``` swift 
 router.get("/api/movies/:genre") { request, context in
        
        guard let genre = context.parameters.get("genre", as: String.self) else {
            throw HTTPError(.badRequest)
        }
        
        return "You selected genre = \(genre)"
    }
```

The value of `genre` is extracted from the `context.parameters` collection. We also make sure that the value can be converted to the expected type, which in this case is a `String`. If that conversion fails, we return a bad request error.

You are not limited to a single parameter. Route parameters can be combined to create more expressive routes. For example, you can allow users to pass both `genre` and `year` in the URL, giving you more control over how the request is handled.


``` swift 
 router.get("/api/movies/genre/:genre/year/:year") { request, context in
        
        guard let genre = context.parameters.get("genre", as: String.self),
              let year = context.parameters.get("year", as: Int.self) else {
            throw HTTPError(.badRequest)
        }
        
        return "Genre is \(genre) and year is \(year)."
    }
```

> This does not mean you should go crazy with adding parameters to your routes. Keep in mind that someone has to call these endpoints. The more complex your URLs become, the harder they are to use and maintain. Keep it simple.

#### Query Strings

Query strings are not just for simple filters like genre. This is where things start getting really useful.

Let’s say you want to search movies and also support pagination. You don’t create new routes for that. You use query strings instead.

```
/api/movies?search=batman  
/api/movies?page=2  
/api/movies?search=batman&page=2  
```

Notice what is happening here. The route stays exactly the same, but the behavior changes based on the values you pass. That is the real power of query strings.

In Hummingbird, you can read these values directly from the request and use them to shape your response.

```swift
router.get("/api/movies") { request, _ -> [Movie] in
    
    let search = request.uri.queryParameters["search"]
    let page = Int(request.uri.queryParameters["page"] ?? "1") ?? 1
    
    if let search {
        return movieStore.searchMovies(query: search, page: page)
    }
    
    return movieStore.paginatedMovies(page: page)
}
```

Now this single route can handle multiple scenarios. It can return all movies, search movies, paginate results, or even combine search and pagination together.

This is exactly why query strings matter. Without them, you would end up creating a new route for every variation. With them, you keep your API simple and let the request control the behavior.

### Route Groups

As your API grows, you will start noticing something. Your routes begin to repeat themselves.

Everything starts with `/api`. Then maybe `/movies`, `/users`, `/reviews`. Before you know it, you are typing the same prefixes over and over again.

This is where route groups come in.

Route groups allow you to organize related routes under a common path. Instead of repeating the same prefix, you define it once and group everything under it.

For example, instead of writing:

```swift
router.get("/api/movies") { ... }
router.get("/api/movies/:id") { ... }
router.post("/api/movies") { ... }
```

You can group them:

```swift
let api = router.group("api")
let movies = api.group("movies")

movies.get { _, _ -> [Movie] in
    return movieStore.allMovies()
}

movies.get(":id") { request, _ -> Movie? in
    let id = request.parameters.get("id", as: Int.self)
    return movieStore.movieById(id ?? 0)
}

movies.post { request, context -> Movie in
    // create movie

    // return new movie 
    return movie 
}
```

Now everything under `movies` automatically gets `/api/movies` as the base path.

This might not feel like a big deal in a small app, but it makes a huge difference as your project grows. Your routes become easier to read, easier to maintain, and much more organized.

You can also create groups for other features like users or reviews and keep everything nicely separated.

Route groups are not just about saving a few keystrokes. They help you structure your API in a way that actually makes sense.

### Conclusion

Routing is one of those things that looks simple on the surface, but it shapes your entire backend.

In this article, you saw how requests are mapped to code, how to use route parameters to make your endpoints dynamic, how query strings keep your API flexible, and how route groups help you stay organized as your application grows.

The main idea is simple. Keep your routes predictable. Avoid creating unnecessary endpoints. Let the request control the behavior instead of exploding your API surface.

If you get routing right early on, everything else becomes easier. Your API becomes cleaner, your code becomes easier to maintain, and your client applications become much easier to build.

In the next article, we will start building more realistic endpoints and return structured data like JSON instead of simple strings.




