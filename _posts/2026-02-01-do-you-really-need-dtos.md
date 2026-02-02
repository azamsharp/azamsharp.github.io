
# Do You Really Need DTOs or Are You Just Copying JSON

One of the most common tasks in iOS applications is consuming a JSON API. An app receives a JSON response, maps it into client side models, and then uses those models to drive the UI.

The question that often comes up is whether we need a separate layer of DTOs (Data Transfer Objects) between the JSON response and our client side models, or if we can decode the response directly into the models we use throughout the app.

As with most things in software development, the answer is it depends. In this article, we will look at different ways to map JSON responses to client side models and explore the trade offs involved in each approach.

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

### TLDR

You do not always need DTOs. If you control both the server and the client, decoding JSON directly into client side models can be a perfectly valid and simple approach. When you consume third party APIs or work with response structures you do not control, DTOs provide an important boundary that protects the rest of your app from change.

DTOs should be plain and reflect the server contract, while client side or domain models should own business logic and meaning. When used intentionally, DTOs help isolate API changes, support different shapes for GET and POST requests, and keep your UI and business rules stable as your app grows.

### JSON Response Objects as Client Side Models 

Sometimes, it is not necessary to introduce a separate DTO layer. In some cases, the JSON response can be decoded directly into client side models. I used this approach in my My Veggie Garden app, where the app fetches vegetable data from a server I fully control and decodes it straight into SwiftData models on the client. These SwiftData models also contain business logic, such as when a plant can be harvested and how much progress it has made.

The main reason this approach worked well is simple. I control both the server and the client. That gives me the flexibility to shape the server response specifically for the app’s needs, even if that response does not match the exact structure of a database record. On the server, I can return an anonymous object using JavaScript and include only the properties the client actually needs.

However, this level of control is not always possible. In many real world scenarios, you are consuming a third party API where you must follow a predefined contract. In those cases, you cannot freely change the response shape to fit your client side models. In the next section, we will look at how DTOs can act as a boundary between the API response and your client side models, helping you absorb change without spreading it throughout your app. 

### Data Transfer Objects 

The main purpose of DTOs is to reflect the JSON response coming from the server. DTOs are plain data structures with no business logic. Their job is to match the shape of the API response as closely as possible and nothing more.

In the example below, I’ve defined two DTOs, one for Product and one for Category. Both are designed to mirror the exact JSON structure returned by the server.

> Keep in mind that in this scenario, we do not control the server. This detail matters, and we will come back to it later.

``` swift 
struct ProductDTO: Decodable {
    let id: Int
    let title: String
    let slug: String
    let price: Double
    let description: String
    let category: CategoryDTO
    let images: [URL]
}

struct CategoryDTO: Decodable {
    let id: Int
    let name: String
    let image: URL
    let slug: String
}
```

As you can see, ProductDTO and CategoryDTO are just plain value type structs.

The reason DTOs should not contain business logic is straightforward. They are not your business. DTOs exist to represent whatever the server happens to send at a given point in time, and that can change without warning. Fields can become optional, names can change, and entire response structures can be reshaped. When business logic lives inside DTOs, your rules start bending around those changes, and the code slowly becomes defensive and harder to reason about.

Your business rules deserve a stable home. Client side or domain models represent what your application actually cares about and what it believes to be true. By keeping DTOs plain and moving business logic into your own models, you create a clear boundary between data transport and business meaning. This separation makes your code easier to read, easier to test, and much easier to evolve as the API inevitably changes.

Now, let’s shift our focus to the client side models.

``` swift 
struct Product: Identifiable, Equatable {
    let id: Int
    let title: String
    let price: Double
    let description: String
    let category: Category
    let images: [URL]

    init(_ productDTO: ProductDTO) {
        self.id = productDTO.id
        self.title = productDTO.title
        self.price = productDTO.price
        self.description = productDTO.description
        self.category = .init(productDTO.category)
        self.images = productDTO.images
    }
    
    // client side rules
    var isAffordable: Bool {
        price < 100
    }
    
    // other business rules 
}

struct Category: Equatable {
    let id: Int
    let name: String
    let imageURL: URL?
    
    init(_ categoryDTO: CategoryDTO) {
        self.id = categoryDTO.id
        self.name = categoryDTO.name
        self.imageURL = categoryDTO.image
    }
}
```

The first thing you might notice is that the Product model looks almost identical to ProductDTO, and the same is true for Category and CategoryDTO. At first glance, this can feel like unnecessary duplication. Apart from a few business rules in the Product model, it looks like the same data is being represented twice.

> Since client side model and DTO are exactly same, you can make a choice of not implementing DTOs and mapping the response directly to the client side models. 

However, this approach is not about optimizing for today. It is about planning for what happens next. The real value shows up when the server changes its response or introduces a new structure. To see why this matters, let’s assume the server starts sending the price in a different format, like this:

``` swift 
"price": { "amount": 89.99, "currency": "USD" }
```

This change would immediately cause a decoding error at the DTO layer, since ProductDTO expects price to be a simple value and not a nested object. The important part is where this failure happens. We catch the issue at the boundary, fix the DTO, and the rest of the application remains untouched. Once the DTO is updated, it can continue mapping cleanly to the Product model, and the UI, services, and business logic do not need to change at all.

Now, imagine the same scenario without a DTO layer, where the JSON response is decoded directly into the client side models. The decoding would fail for the same reason, but the impact would be much larger. Every place in the app that depends on price, including views, services, and business logic, would now need to be updated to handle the new structure. What was once a localized fix becomes a widespread change.

You might argue that we could simply update the Product model in the same way we updated ProductDTO. That is certainly possible. However, doing so pushes parsing and mapping logic directly into the client side model. Over time, the Product model starts doing double duty, acting both as a domain model and as a transport model, which is exactly the coupling we are trying to avoid.

Another place where DTOs provide clear value is when sending data back to the server. In many cases, the structure required for a POST request does not match the structure used for a GET response. This often leads to separate DTOs for reading and writing data. In the next example, we introduce a CreateProductDTO and compare it with ProductDTO to illustrate this difference.

``` swift 
struct CreateProductDTO: Encodable {
    let title: String
    let price: Double
    let description: String
    let categoryId: Int
    let images: [String]
}
```

As you can see, we cannot send Product or ProductDTO back to the server to create a new product because the server expects a different structure for a POST request. This is exactly why it is important to have separate DTOs that conform to what the server accepts, rather than trying to reuse the same type everywhere.

Below is the implementation of AddProductScreen. This screen allows the user to create a new product and then send it to the server for persistence. 

``` swift 
struct AddProductScreen: View {
    
    @State private var title: String = ""
    @State private var price: Double?
    @State private var description: String = ""
    @State private var categoryId: Int?
    @State private var images: [URL]?
    
    @Environment(PlatziStore.self) private var platziStore
    
    var body: some View {
        Button("Save Product") {
            
            guard let price = price, let categoryId = categoryId else { return }
            // get the category
            let category = Category(CategoryDTO(id: 1, name: "Clothes", image: URL(string: "")!, slug: "clothes"))
            
            // create the product
            let product = Product(id: 1, title: title, price: price, description: description, category: category, images: [])
            
            Task { try await platziStore.createProduct(product) }
        }
    }
}
```

First, we construct our client side Product model and pass it to platziStore.createProduct. This is the point where the mapping from the client side model to the appropriate DTO can happen before handing the request off to the HTTPClient.

``` swift 
 func createProduct(_ product: Product) async throws {
        
        let productDTO = CreateProductDTO(title: product.title, price: product.price, description: product.description, categoryId: product.category.id, images: product.images)
        
    // httpClient.saveProduct(product: productDTO)
}
```

> For larger apps, you can also introduce a dedicated mapping layer. This moves the responsibility of converting between DTOs and client side models out of the Store and into a focused layer whose only job is mapping.

### Mapping Layer 

Sometimes, DTOs and client side models live in different modules. In those situations, you want to avoid tightly coupling them by introducing direct dependencies between the two. One way to solve this is by adding a mapping layer that sits in between and handles the conversion from DTOs to client side models and back.

This mapping layer can live as a standalone module or as part of your application layer. Its responsibility is simple and focused: take one representation of data and convert it into another without leaking concerns across module boundaries.

> This kind of mapping layer is conceptually similar to AutoMapper in .NET development.

### Should I Always Use DTOs 

No.

Everything in software development is a tradeoff, and data transfer objects are no exception. Sometimes they help absorb the impact of changing API responses, and other times they simply add unnecessary code and complexity.

In this article, CreateProductDTO is a good example of where a DTO makes sense. It is a true DTO in that it exists solely to match the structure expected by the server. It does not try to represent business meaning or client side behavior. It simply does its job and gets out of the way.

The takeaway is simple. Use DTOs where they add value, and do not introduce them just because they are expected.

### Conclusion 

There is no single right answer when it comes to using DTOs. Like most architectural decisions, it depends on your constraints, your team, and how much control you have over the system. If you control both the server and the client, decoding JSON directly into client side models can be perfectly reasonable and highly productive. It keeps things simple and avoids unnecessary layers.

However, when you are consuming third party APIs or working with systems you do not control, DTOs become an important boundary. They protect the rest of your application from changes in response structure, optional fields, and evolving contracts. By keeping DTOs plain and free of business logic, you ensure that your domain and UI models remain focused on meaning rather than transport details.

Client side models should represent what your application believes to be true and should be the home of business rules and behavior. DTOs should simply describe what the server sends or expects. Mapping between the two may feel like extra work at first, but it gives you flexibility, clarity, and long term stability as your app grows.

In the end, DTOs are a tool, not a rule. Use them where they add value, avoid them where they do not, and always be intentional about where responsibilities live. That mindset will take you much further than blindly following any pattern.