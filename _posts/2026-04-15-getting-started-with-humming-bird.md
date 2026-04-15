# Getting Started with Hummingbird 

I still remember the late 90s when I installed Internet Information Services on my machine. It allowed me to run Classic ASP pages connected to a Microsoft SQL Server database. That was my first real introduction to backend development.

Fast forward a few decades, and I still enjoy backend development just as much. It gives you a level of control that is hard to achieve when relying entirely on backend-as-a-service platforms.

Over the years, I have worked with a variety of backend technologies, including ASP.NET Web Forms, ASP.NET MVC, ExpressJS, Flask, Vapor, and now Hummingbird.

What I have learned is that once you understand the fundamentals, moving between frameworks becomes much easier. Recently, I have been spending time exploring [Hummingbird](https://hummingbird.codes/), and in this article, I will walk you through a simple introduction to get started.

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

### What is Hummingbird? 

Hummingbird is a lightweight web framework for building backend applications using Swift. It focuses on simplicity, performance, and giving you full control over your server-side code. Unlike heavier frameworks, Hummingbird does not hide too much behind abstractions. It embraces Swift’s async/await from the ground up, which makes writing concurrent code feel natural and clean. If you already understand backend fundamentals like routing, request handling, and working with databases, you will find Hummingbird straightforward and easy to pick up.

### Why Hummingbird? Why not Vapor?

This is not about one being better than the other. It is about what you value as a developer.

Hummingbird is a great choice if you want something lightweight and closer to how Swift actually works today. It embraces async/await from the ground up and avoids hiding things behind too many abstractions. You write more of the plumbing yourself, but in return you get clarity and control. If you already understand backend fundamentals, Hummingbird feels very natural.

Vapor, on the other hand, is a more mature and full featured framework. It gives you a lot out of the box like authentication, ORM, middleware, and a well established ecosystem. That is great if you want to move fast or build production systems without reinventing common pieces.

The trade off is that Vapor can sometimes feel heavier. It was built before Swift’s modern concurrency model, so some parts feel adapted rather than designed for async/await from the beginning.

If you want control, simplicity, and a framework that aligns closely with modern Swift, go with Hummingbird. If you want a complete ecosystem with batteries included and less setup, Vapor is a solid choice.

In the end, once you understand the fundamentals, moving between the two is not that difficult.

### Creating Your First Hummingbird Project 

The easiest way to get started with Hummingbird is by using the official template. It gives you a clean starting point so you do not have to set everything up from scratch.

Start by cloning the template repository on your machine:

``` swift 
git clone https://github.com/hummingbird-project/template
```

Once the template is downloaded, you can use it to generate your project. Run the following command to launch the setup script:

``` swift 
./template/configure.sh MyNewProject
```

This will walk you through a few configuration options like packages and features. If you are just getting started, you can safely accept all the defaults by pressing the return key.

After the project is created, navigate into the MyNewProject folder and run:

`swift run`

This will build and start your server. The first build may take a few minutes since Swift Package Manager needs to download and compile dependencies.

Once everything is running, open your browser and go to 127.0.0.1:8080. You should see a simple "Hello" response, which confirms that your Hummingbird server is up and running.

### Understanding the Starter Template 

You can open your Hummingbird project in any editor you prefer. If you want to use Xcode, simply run open Package.swift from the project folder. This will open the project in Xcode and automatically resolve and download all the required dependencies.

Inside the Sources/App folder, you will find the App.swift file. This file contains the main function, which serves as the entry point of your application. This is where your app starts executing, and it is responsible for setting up configuration, building the application, and starting the server. The code for the main function is shown below:

``` swift 
@main
struct App {
    static func main() async throws {
        // Application will read configuration from the following in the order listed
        // Command line, Environment variables, dotEnv file, defaults provided in memory 
        let reader = try await ConfigReader(providers: [
            CommandLineArgumentsProvider(),
            EnvironmentVariablesProvider(),
            EnvironmentVariablesProvider(environmentFilePath: ".env", allowMissing: true),
            InMemoryProvider(values: [
                "http.serverName": "MyNewProject"
            ])
        ])
        let app = try await buildApplication(reader: reader)
        try await app.runService()
    }
}
```

The `main` function is the entry point of your Hummingbird application, and it is responsible for three main tasks: reading configuration, building the application, and starting the server. The `@main` attribute tells Swift where the program begins, and the `async throws` signature allows the app to perform asynchronous work and handle errors gracefully, which aligns with how Hummingbird is built around Swift’s async/await model.

The first part of the function creates a `ConfigReader`, which is used to load configuration values for your application. These values can include things like the server name, port, or environment settings. What makes this powerful is that Hummingbird reads configuration from multiple sources in a specific order: command line arguments, environment variables, a `.env` file, and finally in-memory defaults. This means you can override values easily depending on how you run your application. For example, a value defined in the command line will take precedence over the same value defined in a `.env` file.

Once the configuration is loaded, the `buildApplication` function is called. This function uses the configuration reader to construct your application, set up routes, middleware, and any services your app depends on. You can think of this step as wiring everything together before the app starts running.

Finally, `app.runService()` starts the server and begins listening for incoming HTTP requests. At this point, your application is fully up and running, ready to handle requests from the browser or any client.

The next file is App+Build.swift file. Below you can find the complete code: 

``` swift 
import Configuration
import Hummingbird
import Logging

// Request context used by application
typealias AppRequestContext = BasicRequestContext

///  Build application
/// - Parameter reader: configuration reader
func buildApplication(reader: ConfigReader) async throws -> some ApplicationProtocol {
    let logger = {
        var logger = Logger(label: "MyNewProject")
        logger.logLevel = reader.string(forKey: "log.level", as: Logger.Level.self, default: .info)
        return logger
    }()
    let router = try buildRouter()
    let app = Application(
        router: router,
        configuration: ApplicationConfiguration(reader: reader.scoped(to: "http")),
        logger: logger
    )
    return app
}

/// Build router
func buildRouter() throws -> Router<AppRequestContext> {
    let router = Router(context: AppRequestContext.self)
    // Add middleware
    router.addMiddleware {
        // logging middleware
        LogRequestsMiddleware(.info)
    }
    // Add default endpoint
    router.get("/") { _,_ in
        return "Hello!"
    }
    return router
}

```

This part of the code is responsible for **building your application and defining how it handles incoming requests**. It is where most of your backend setup lives.

The `buildApplication` function takes a `ConfigReader` as input and returns an instance of your application. The first thing it does is configure a logger. The logger is created using the Swift Logging system, and its log level is read from the configuration. This means you can control how much information gets logged (debug, info, error) without changing your code, simply by updating configuration values.

Next, the function calls `buildRouter()`, which is where all your routes and middleware are defined. Think of the router as the part of your application that decides how to respond to different URLs. After the router is created, it is passed into the `Application` along with configuration and the logger. The configuration is scoped to `"http"`, which means it will only use HTTP-related settings from your configuration sources.

The `buildRouter` function is where you define how your app behaves. It starts by creating a router with a request context, which in this case is `BasicRequestContext`. This context carries information about each incoming request. Middleware is then added to the router. In this example, a logging middleware is used to log every incoming request, which is useful for debugging and monitoring.

Finally, a simple route is defined using `router.get("/")`. This means when a user visits the root URL (`/`), the server will respond with `"Hello!"`. This is your first endpoint, and it shows how easy it is to define routes in Hummingbird.

Overall, this setup keeps things clean and explicit. You configure logging, define your routes, and wire everything together in one place, giving you full control over how your backend behaves.

At this point, go ahead and change the route to return `Hello World` instead of just `Hello`. If you refresh your browser with 127.0.0.1:8080, you will notice that it still says `Hello` instead of `Hello World`. 

For this to work, you need to stop the server `Control C` and then start the server again `swift run`. Now, when you visit the route you will get the updated information. 

Although it works fine but it can get quite annoying if you need to restart the server, each time you make a small change. What if the server can start automatically when it detects that any Swift file is changed. 

### Automatically Restarting Server Using watchexec

If you have worked with Node, you are probably familiar with tools like nodemon that automatically restart your server whenever a file changes. This kind of workflow makes development much smoother since you do not have to manually stop and start the server every time you make a small change.

You can achieve the same behavior in your Hummingbird projects using a tool called [watchexec](https://github.com/watchexec/watchexec). It watches your files and reruns a command whenever it detects changes.

To install watchexec, run:

```swift id="g0c0g6"
brew install watchexec
```

Once installed, navigate to your Hummingbird project folder and run:

```swift id="m3v7rc"
watchexec -e swift --restart swift run
```

This command tells watchexec to watch for changes in Swift files and automatically restart your server by running `swift run`.

This small setup can save you a lot of time. Instead of constantly restarting the server manually, you can focus on writing code and see your changes reflected immediately. For me, watchexec has become a must-have tool when working on backend projects.

## Conclusion

Hummingbird gives you a clean and simple way to build backend applications using Swift without hiding too much behind abstractions. If you understand the fundamentals of backend development, you will feel right at home. You define your routes, wire up your services, and control how everything behaves. That is the real power.

At the end of the day, frameworks will come and go, but the fundamentals stay the same. Once you understand how requests flow through your application, how routing works, and how to structure your code, switching between frameworks becomes much easier. 

Hummingbird is just another tool, but it is a tool that aligns really well with modern Swift. If you enjoy building things from the ground up and want more control over your backend, it is definitely worth exploring.

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