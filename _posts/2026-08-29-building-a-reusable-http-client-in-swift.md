# Building a Reusable HTTP Client in Swift with async/await

Most applications need to communicate with a server. We may need to register a user, log in to an account, load courses, or submit information. Although URLSession provides everything we need to perform these requests, using it directly throughout the application can quickly lead to repeated code.

For every request, we usually perform the same steps. We create a URLRequest, configure the HTTP method and headers, send the request, inspect the status code, and decode the returned data. We must also decide how to handle client errors, server errors, and decoding failures.

Instead of repeating this logic for every endpoint, we can move it into a reusable HTTPClient.

In this article, we will build a small networking layer using URLSession and Swift concurrency. We will create a generic Resource type to describe API requests, represent different HTTP methods, handle network errors, and decode responses into strongly typed Swift models.

By the end, we will use the same HTTPClient to perform both GET and POST requests and integrate it with a SwiftUI application.

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

### Why Generic?

There are several ways to implement an `HTTPClient`. One approach is to create a separate function for every operation:

```swift
func login(...)
func register(...)
func loadCourses(...)
func loadExams(...)
func submitExam(...)
```

This may work for a small application, but it can quickly become a maintenance problem. As the application grows, the `HTTPClient` may end up with hundreds of functions.

Each function will perform many of the same steps. It will create a request, add headers and authentication tokens, encode the request body, send the request, inspect the status code, decode the response, and handle errors.

After writing a few of these functions, you will notice that most of the code is repeated. The main things that change are the endpoint, HTTP method, request body, and expected response type.

Instead of creating a separate networking function for every operation, we can create a single generic function that works with different types of requests and responses:

```swift
func fetch<T>(_ resource: Resource<T>) async throws -> T
```

The `Resource` describes the request, while the generic type `T` represents the response we expect from the server.

For a login request, `T` may be `LoginResponse`:

```swift
let resource = Resource(
    url: Constants.Urls.login,
    method: .post(try loginForm.encoded()),
    responseType: LoginResponse.self
)
```

For a request that loads courses, `T` may be `[Course]`:

```swift
let resource = Resource(
    url: Constants.Urls.courses,
    responseType: [Course].self
)
```

Both requests use the same `fetch` function:

```swift
let loginResponse = try await httpClient.fetch(loginResource)
let courses = try await httpClient.fetch(coursesResource)
```

The requests are different, but the networking steps remain the same. By using generics, we can reuse those steps while still returning strongly typed responses.


### Resource, HTTPMethod and NetworkError

Before implementing `HTTPClient`, we need a few supporting types. These types will describe how the request should be sent, what kind of response we expect, and what can go wrong during the request.

Let’s start with `HTTPMethod`.

```swift
enum HTTPMethod {
    case get
    case post(Data)
    case delete
    case put

    var rawValue: String {
        switch self {
        case .get:
            "GET"
        case .post:
            "POST"
        case .delete:
            "DELETE"
        case .put:
            "PUT"
        }
    }

    var body: Data? {
        switch self {
        case .post(let data):
            data
        default:
            nil
        }
    }
}
```

The `HTTPMethod` enum represents the HTTP method associated with a request. In our application, we currently support `GET`, `POST`, `DELETE`, and `PUT`.

The `post` case contains a `Data` value. This represents the request body that will be sent to the server. For example, when registering a user, we can encode the registration form into JSON and pass the resulting data to the `post` case.

```swift
let data = try registerForm.encoded()
let method = HTTPMethod.post(data)
```

The `rawValue` computed property returns the value expected by `URLRequest`:

```swift
request.httpMethod = resource.method.rawValue
```

The `body` computed property returns the associated data for a `POST` request. For all other request types, it returns `nil`.

```swift
request.httpBody = resource.method.body
```

Next, let’s look at `Resource`.

```swift
struct Resource<T: Decodable> {
    let url: URL
    var method: HTTPMethod = .get
    let responseType: T.Type
}
```

A `Resource` represents an API endpoint. It tells `HTTPClient` three things:

* Where the request should be sent
* Which HTTP method should be used
* What type of response should be returned

The generic type `T` must conform to `Decodable` because the server response will be decoded into that type.

For example, the following resource sends a login request and expects a `LoginResponse`:

```swift
let resource = Resource(
    url: Constants.Urls.login,
    method: .post(try loginForm.encoded()),
    responseType: LoginResponse.self
)
```

The response type is an important part of the resource. It allows the networking layer to remain generic. The same `HTTPClient` can return a `LoginResponse`, a `User`, a `Course`, or an array of courses.

For requests that do not provide a method, `.get` is used by default:

```swift
let resource = Resource(
    url: Constants.Urls.courses,
    responseType: [Course].self
)
```

Finally, we need to think about errors. A network request can fail for several reasons. The server may reject the request, the response may not be valid, or the returned JSON may not match the response type we expected.

We can represent these situations using `NetworkError`.

```swift
enum NetworkError: Error {
    case badRequest(ErrorResponse)
    case invalidResponse
    case decodingFailed(Error)
    case serverError
}
```

Each case represents a different kind of failure.

The `badRequest` case is used when the server returns a client error. This may happen when the user submits an invalid email address, enters an incorrect password, or forgets to provide required information. The associated `ErrorResponse` contains the message returned by the server.

The `invalidResponse` case is used when the response cannot be converted into an `HTTPURLResponse`.

The `decodingFailed` case is used when the server returns data, but the data cannot be decoded into the expected Swift type. We also store the original error, which can be helpful when debugging the problem.

The `serverError` case represents errors generated by the server, typically responses with status codes in the `500...599` range.

At the moment, these errors do not contain messages that are suitable for displaying in the user interface. We can fix that by conforming `NetworkError` to `LocalizedError`.

```swift
extension NetworkError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .badRequest(let response):
            response.errorMessage

        case .invalidResponse:
            "The server returned an invalid response."

        case .decodingFailed:
            "Unable to process the server response."

        case .serverError:
            "The server encountered an error. Please try again later."
        }
    }
}
```

Now the application can use `localizedDescription` to display an appropriate message:

```swift
do {
    let response = try await httpClient.fetch(resource)
} catch {
    showToast(error.localizedDescription)
}
```

At this point, we have everything we need to describe a network request and communicate errors to the user. In the next section, we will use these types to build `HTTPClient`.

### Building the HTTPClient

Now that we have created `Resource`, `HTTPMethod`, and `NetworkError`, we can implement `HTTPClient`.

The job of `HTTPClient` is to create a request, send it to the server, and decode the response into the type specified by the resource.

```swift
struct HTTPClient {
    private let session: URLSession
    private let decoder: JSONDecoder
    private let defaultHeaders: [String: String]

    init(
        session: URLSession = .shared,
        decoder: JSONDecoder = JSONDecoder(),
        defaultHeaders: [String: String] = [
            "Content-Type": "application/json"
        ]
    ) {
        self.session = session
        self.decoder = decoder
        self.defaultHeaders = defaultHeaders
    }

    func fetch<T>(_ resource: Resource<T>) async throws -> T {
        var request = URLRequest(url: resource.url)
        request.httpMethod = resource.method.rawValue
        request.allHTTPHeaderFields = defaultHeaders
        request.httpBody = resource.method.body

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        switch httpResponse.statusCode {
        case 200...299:
            do {
                return try decoder.decode(T.self, from: data)
            } catch {
                print(error.localizedDescription)
                throw NetworkError.decodingFailed(error)
            }

        case 400...499:
            do {
                let errorResponse = try decoder.decode(
                    ErrorResponse.self,
                    from: data
                )

                throw NetworkError.badRequest(errorResponse)
            } catch let error as NetworkError {
                throw error
            } catch {
                throw NetworkError.decodingFailed(error)
            }

        case 500...599:
            throw NetworkError.serverError

        default:
            throw NetworkError.invalidResponse
        }
    }
}
```

The `HTTPClient` depends on `URLSession`, `JSONDecoder`, and a collection of default headers.

```swift
private let session: URLSession
private let decoder: JSONDecoder
private let defaultHeaders: [String: String]
```

`URLSession` is used to send the request, while `JSONDecoder` converts the returned JSON into a Swift type. The default headers are added to every request.

We pass these dependencies through the initializer:

```swift
init(
    session: URLSession = .shared,
    decoder: JSONDecoder = JSONDecoder(),
    defaultHeaders: [String: String] = [
        "Content-Type": "application/json"
    ]
) {
    self.session = session
    self.decoder = decoder
    self.defaultHeaders = defaultHeaders
}
```

Since each dependency has a default value, we can create an instance without providing any arguments:

```swift
let httpClient = HTTPClient()
```

Passing these dependencies through the initializer also makes the client easier to configure and test. For example, we can provide a decoder that supports ISO 8601 dates:

```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

let httpClient = HTTPClient(decoder: decoder)
```

The `fetch` function accepts a resource and returns the response type associated with that resource:

```swift
func fetch<T>(_ resource: Resource<T>) async throws -> T
```

We begin by creating a `URLRequest` using the information stored in the resource:

```swift
var request = URLRequest(url: resource.url)
request.httpMethod = resource.method.rawValue
request.allHTTPHeaderFields = defaultHeaders
request.httpBody = resource.method.body
```

This is one of the main benefits of using `Resource`. The `HTTPClient` does not need to know whether it is sending a login request, registering a user, or loading courses. The resource already contains everything needed to create the request.

Next, we send the request:

```swift
let (data, response) = try await session.data(for: request)
```

The returned response is a `URLResponse`, but we need an `HTTPURLResponse` to access the HTTP status code.

```swift
guard let httpResponse = response as? HTTPURLResponse else {
    throw NetworkError.invalidResponse
}
```

Once we have the status code, we can decide how to process the response.

#### Handling Successful Responses

A status code in the `200...299` range means the request was successful.

```swift
case 200...299:
    do {
        return try decoder.decode(T.self, from: data)
    } catch {
        print(error.localizedDescription)
        throw NetworkError.decodingFailed(error)
    }
```

The response is decoded into `T`. The actual type of `T` depends on the resource.

If the resource expects a `LoginResponse`, `fetch` returns a `LoginResponse`:

```swift
let resource = Resource(
    url: Constants.Urls.login,
    method: .post(try loginForm.encoded()),
    responseType: LoginResponse.self
)

let response = try await httpClient.fetch(resource)
```

If decoding fails, we wrap the original error inside `NetworkError.decodingFailed`.

#### Handling Client Errors

Status codes in the `400...499` range mean there is a problem with the request.

```swift
case 400...499:
    do {
        let errorResponse = try decoder.decode(
            ErrorResponse.self,
            from: data
        )

        throw NetworkError.badRequest(errorResponse)
    } catch let error as NetworkError {
        throw error
    } catch {
        throw NetworkError.decodingFailed(error)
    }
```

The server may return a useful error message when a request fails. For example, it may tell us that an account already exists or that the provided credentials are incorrect.

Instead of throwing a generic error, we decode the response into `ErrorResponse` and include it with `NetworkError.badRequest`.

You may be wondering why we need two `catch` blocks.

After decoding `ErrorResponse`, we intentionally throw `NetworkError.badRequest`. The first `catch` catches that error and throws it again:

```swift
catch let error as NetworkError {
    throw error
}
```

The second `catch` handles an error that occurs while decoding `ErrorResponse`:

```swift
catch {
    throw NetworkError.decodingFailed(error)
}
```

Without the first `catch`, our `badRequest` error would be caught by the general `catch` block and incorrectly changed into a decoding error.

#### Handling Server Errors

A status code in the `500...599` range means something went wrong on the server.

```swift
case 500...599:
    throw NetworkError.serverError
```

There is usually nothing the client can do to fix a server error. We can catch this error in the user interface and ask the user to try again later.

Finally, any status code that does not belong to one of the expected ranges is treated as an invalid response:

```swift
default:
    throw NetworkError.invalidResponse
```

Our `HTTPClient` is now ready to use. It can send different types of requests, decode successful responses, and convert unsuccessful responses into errors that can be handled by the application.

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

### Using the HTTPClient

Now that our `HTTPClient` is ready, let’s use it to send requests to the server.

We will begin with a simple `GET` request that loads a list of courses.

```swift
struct Course: Decodable {
    let id: Int
    let title: String
}
```

Next, we create a resource that describes the request:

```swift
let resource = Resource(
    url: Constants.Urls.courses,
    responseType: [Course].self
)
```

Since we did not provide an HTTP method, the resource uses `.get` by default. We also specified `[Course].self` as the response type because we expect the server to return an array of courses.

We can now pass the resource to `HTTPClient`:

```swift
let httpClient = HTTPClient()
let courses = try await httpClient.fetch(resource)
```

The return type of `fetch` is inferred from the resource. Since the resource expects `[Course]`, the returned value is also `[Course]`.

The complete request looks like this:

```swift
func loadCourses() async {
    let resource = Resource(
        url: Constants.Urls.courses,
        responseType: [Course].self
    )

    do {
        let courses = try await httpClient.fetch(resource)
        print(courses)
    } catch {
        print(error.localizedDescription)
    }
}
```

The networking code does not need to create a `URLRequest`, configure headers, inspect status codes, or decode JSON. All of that work is handled by `HTTPClient`.

#### Sending a POST Request

Let’s look at a more practical example by sending a login request.

We will start with the form that contains the user’s email and password:

```swift
struct LoginForm: Encodable {
    var email = ""
    var password = ""
}
```

We also need a type that represents the server response:

```swift
struct LoginResponse: Decodable {
    let user: User
    let accessToken: String
}
```

Before sending the request, the login form must be converted into `Data`. We can add an `encoded` function to `Encodable`:

```swift
extension Encodable {
    func encoded() throws -> Data {
        try JSONEncoder().encode(self)
    }
}
```

Now we can create the login resource:

```swift
let resource = Resource(
    url: Constants.Urls.login,
    method: .post(try loginForm.encoded()),
    responseType: LoginResponse.self
)
```

The resource contains everything needed to perform the request:

* The URL of the login endpoint
* The encoded login form
* The expected response type

We can send the request by passing the resource to `fetch`:

```swift
let response = try await httpClient.fetch(resource)
```

Since the resource expects `LoginResponse`, the returned value is automatically inferred as `LoginResponse`.

#### Using HTTPClient Inside a Store

In a SwiftUI application, network requests should not be performed directly inside the view. Instead, we can inject `HTTPClient` into a store and allow the store to manage the operation.

For example, the `AuthenticationStore` can use `HTTPClient` to log in the user:

```swift
import Observation

@Observable
class AuthenticationStore {
    private let httpClient: HTTPClient

    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }

    func login(form: LoginForm) async throws {
        let resource = Resource(
            url: Constants.Urls.login,
            method: .post(try form.encoded()),
            responseType: LoginResponse.self
        )

        let response = try await httpClient.fetch(resource)

        print(response.user)
        print(response.accessToken)
    }
}
```

The store does not need to know how `URLSession` works or how the response is decoded. Its responsibility is to create the correct resource and use the returned result.

We can create the store when the application starts:

```swift
@main
struct MathLabClientApp: App {
    @State private var authenticationStore: AuthenticationStore

    init() {
        let httpClient = HTTPClient()

        _authenticationStore = State(
            initialValue: AuthenticationStore(
                httpClient: httpClient
            )
        )
    }

    var body: some Scene {
        WindowGroup {
            LoginScreen()
                .environment(authenticationStore)
        }
    }
}
```

This also means that the same `HTTPClient` instance can be shared with other stores:

```swift
let httpClient = HTTPClient()

let authenticationStore = AuthenticationStore(
    httpClient: httpClient
)

let courseStore = CourseStore(
    httpClient: httpClient
)
```

#### Calling the Store from SwiftUI

The `LoginScreen` can call the store when the user taps the login button:

```swift
struct LoginScreen: View {
    @State private var loginForm = LoginForm()
    @State private var isLoggingIn = false

    @Environment(AuthenticationStore.self)
    private var authenticationStore

    @Environment(\.showToast)
    private var showToast

    var body: some View {
        Form {
            TextField("Email", text: $loginForm.email)
                .textInputAutocapitalization(.never)
                .keyboardType(.emailAddress)

            SecureField("Password", text: $loginForm.password)

            Button("Login") {
                Task {
                    await login()
                }
            }
            .disabled(isLoggingIn)
        }
    }

    private func login() async {
        isLoggingIn = true
        defer { isLoggingIn = false }

        do {
            try await authenticationStore.login(form: loginForm)
        } catch {
            showToast(error.localizedDescription)
        }
    }
}
```

The view is responsible for collecting input, displaying progress, and showing errors. The store performs the login operation, while `HTTPClient` handles the networking details.

This gives us a simple flow:

1. The view creates the user input.
2. The store creates a resource.
3. `HTTPClient` sends the request.
4. The decoded response is returned to the store.
5. The view updates based on the result.

By separating these responsibilities, we can reuse the same `HTTPClient` throughout the application without repeating networking code in every screen.

### Conclusion 
In this article, we created a reusable networking layer using `URLSession`, `async/await`, and Swift generics.

We started by creating `HTTPMethod` to represent the type of request being sent. We then introduced `Resource`, which contains the URL, HTTP method, and expected response type. Finally, we created `NetworkError` to represent the different failures that may occur while communicating with the server.

Using these types, we implemented an `HTTPClient` that can create requests, send them asynchronously, inspect HTTP status codes, and decode successful responses into Swift models. We also used the client inside an `AuthenticationStore`, keeping networking logic out of the SwiftUI view.

The important part of this approach is not the amount of code we wrote. It is the separation of responsibilities. The view collects input and displays the result. The store coordinates the operation. The resource describes the request. The HTTP client handles communication with the server.

As the application grows, the same client can be used for registration, authentication, courses, exams, grades, and other API requests without repeating the underlying networking code.

### Continue Learning with AzamSharp School

Become an AzamSharp School member and get access to more than 250 hours of practical courses covering SwiftUI, SwiftData, testing, architecture, AI, machine learning, and more.

Your membership also includes access to AzamSharp books, live workshops, office hours, and new content added regularly.

[Join AzamSharp School](https://azamsharp.school)

