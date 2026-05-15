# Swift Is Great for Apps, Not for Training ML Models 

A few years back, I conducted a couple of workshops on building machine learning models using Create ML. One workshop was hosted at SwiftCraft and another at try! Swift NYC. The sessions went great, and people were excited to see machine learning workflows built entirely with Swift and Apple’s tooling.

But after spending more time with Create ML, I started noticing something frustrating.

Simple machine learning tasks that felt effortless in Python suddenly became awkward and verbose in Swift. Data cleanup, preprocessing, feature engineering, encoding, normalization, and experimentation all required significantly more effort than I was used to with tools like pandas, scikit-learn, and TensorFlow.

To be clear, I like Swift. Swift is one of my favorite languages for building applications. But after working with Create ML workflows for a while, I came to a realization.

Swift is great for deploying machine learning models.
It is not great for training them.

Over time, I found myself spending more energy fighting the tooling than actually solving machine learning problems.

In this article, we will look at some of the challenges I encountered while training machine learning models using Swift and Create ML, why I eventually switched back to Python for training workflows, and how to convert Python trained models into Core ML models that can be deployed in iOS apps or even hosted on the server using Swift frameworks like Vapor and Hummingbird.

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

### Where Things Started Breaking Down

Let's say that we are working on the [Carvana dataset](https://www.kaggle.com/datasets/ravishah1/carvana-predict-car-prices) for car pricing and the "Year" column in the dataset has dirty values. Instead of values like 2017, you may sometimes see values like 20173. A simple fix is to take the first four characters from the year value.

In Swift, using TabularData, you may write something like this:

``` swift 
var df = try DataFrame(contentsOfCSVFile: fileURL, options: options)

df.transformColumn("Year") { (year: Int) -> Int in
    let yearStr = String(year)
    if let cleanerYear = Int(yearStr.prefix(4)) {
        return cleanerYear
    }
    
    return year
}
```

And in Python you need the following: 

``` python 
df["Year"] = df["Year"].astype(str).str[:4].astype(int)
```

Both examples are doing exactly the same thing, but the Python version is much more compact and easier to understand. And honestly, this matters a lot in machine learning because training a model is rarely just about calling fit().

The majority of your time is usually spent before training even begins. You clean messy datasets, inspect columns, deal with missing values, encode categories, normalize features, remove outliers, split data into training and testing sets, evaluate results, tweak preprocessing steps, and repeat the whole process again and again. Machine learning is extremely iterative, and that is where Python really shines.

The other major issue I kept running into with the Create ML APIs was stability, especially when working inside macOS Playgrounds. The frustrating part was that the issues were not even related to my machine learning code. The failures were coming from linker and runtime problems inside the Playground environment itself. 

``` swift 
error: Failed to load linked library cups of module Cocoa - errors:
Failed to find "libcups.dylib" in paths:
  /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/swift/macosx
  /usr/lib/swift
```

> The recommendation was to stop using Playgrounds and move the code to a macOS Command Line Tool project.

As your machine learning workflows become more complex, you start spending more time fighting the tooling instead of actually training models. You deal with linker issues, Playground instability, missing APIs, awkward preprocessing workflows, and limited ecosystem support. Eventually, it starts slowing down experimentation, which is the exact opposite of what you want in machine learning.

Python also provides excellent support for building custom machine learning pipelines. The pipeline allows you to create machine learning workflow that can be composed of preprocessing, model training etc. 

For example, you can create preprocessing stages for standardization, one hot encoding, scaling, feature selection, and then pass the transformed data directly into the model.

The really nice part is that everything becomes part of one consistent pipeline.

The caller does not need to manually standardize values or build large arrays representing one hot encoded categories. The pipeline takes care of those transformations automatically before the data reaches the model. This makes the code cleaner, easier to maintain, and much easier to evolve as your preprocessing logic grows over time.

The overall workflow is also very straightforward, especially when using tools like scikit-learn.

If you want to build similar workflows in Swift, you usually end up working with Create ML Components. Unfortunately, the experience is nowhere near as smooth. The APIs are much harder to use, require significantly more code, and the documentation is still pretty limited compared to the Python machine learning ecosystem.

At that point, I decided to go a different route.

Instead of using Create ML for training, I switched back to the more mature and battle tested ecosystem built around Python. Tools like pandas, scikit-learn, and TensorFlow simply provide a much smoother experience for data cleanup, preprocessing, feature engineering, training, and experimentation.

In the next section, you will learn how to take a model trained in Python and convert it into a Core ML model using Core ML Tools.

### Python to the Rescue

Converting a Python trained model to a Core ML model is actually pretty straightforward. The biggest challenge is usually making sure that your Python version and the machine learning libraries you used are compatible with Core ML Tools.

One thing I quickly discovered is that Core ML Tools can be extremely picky about version compatibility. For example, if your model was trained using a newer or unsupported version of scikit-learn, the conversion process may fail with compatibility errors.

Once you have the correct versions installed, converting the model is surprisingly simple.

``` python 
coreml_model = ct.converters.sklearn.convert(model, ["Year", "Miles", "Name"], "Price")
coreml_model.save("Carvana.mlmodel")
```

That is pretty much it.

You can now drag the generated .mlmodel file directly into your iOS application and use it as an on-device machine learning model with Core ML.

A lot of developers assume that Core ML models can only run on-device, but that is not entirely true. You can also host Core ML models on the server using Swift backend frameworks like Vapor or Hummingbird and expose them through an API.

This approach gives you a lot more flexibility. Instead of shipping a brand new version of your app every time your model changes, you can update the model on the server and immediately make the latest predictions available to all clients.

# Hosting Core ML Model Using Vapor 

One of the biggest benefits of hosting the model on your server is flexibility. If the model changes, you can update it on the server without submitting a new version of your app to the App Store.

Vapor and Hummingbird are two popular options for building server-side Swift applications, and both can be used to expose your Core ML model through an API.

That said, you are not restricted to a Swift-based backend. If your model was trained in Python and you prefer staying in that ecosystem, you can also use Flask, Django, or FastAPI. In that case, you may not even need to convert the model to Core ML, because your Python backend can use the original scikit-learn model directly.

One important thing to keep in mind is that if you want to run a Core ML model on the server, the server must run macOS. Core ML is an Apple framework, so you cannot deploy this setup to a standard Linux-based DigitalOcean droplet or AWS EC2 instance. You would need a Mac cloud provider, such as MacStadium, or your own Mac mini acting as the server.

Below is the implementation of the `CarPricePredictor` actor. This actor is responsible for loading the compiled Core ML model and performing the actual prediction.


``` swift 
import Foundation

actor CarPricePredictor {
    
    let model: Carvana
    
    init() throws {
        let url = Bundle.module.url(forResource: "Carvana", withExtension: "mlmodelc")!
        model = try Carvana(contentsOf: url)
    }
    
    // predict
    func predict(year: Double, miles: Double, nameEncodedValue: Double) throws -> Double {
        
        let prediction = try model.prediction(Year: year, Miles: miles, Name_encoded: nameEncodedValue)
        return prediction.Price
    }
    
}
```

The `CarPricePredictor` actor is used inside the Vapor routes to perform the actual prediction. The route receives the request from the client, extracts the required values, passes them to the `predict` function, and then returns the prediction result back to the caller.

This keeps the prediction logic separated from the networking layer and makes the code easier to maintain as the application grows.

> In a real world application, it is usually a good idea to move the route logic into controllers and follow a proper architectural pattern such as MVC. This keeps your routes lightweight and prevents business logic from spreading across the application.


``` swift 
func routes(_ app: Application) throws {
    
    let carPricePredictor = try CarPricePredictor()
        
    // /api/car-price
    app.post("api", "car-price") { req async throws -> PricePredictionResponse in
        
        // miles, year, encodedNameValue
        let pricePredictionRequest = try req.content.decode(PricePredictionRequest.self)
        
        let price = try await carPricePredictor.predict(year: pricePredictionRequest.year, miles: pricePredictionRequest.miles, nameEncodedValue: pricePredictionRequest.nameEncodedValue)
        
        return PricePredictionResponse(price: price)
        
    }

}
```

The client can now call the endpoint above, provide the required input values, and receive the predicted car price as a response.

One thing I really like about this approach is that the mobile app no longer needs to know anything about how the model works internally. The app simply sends the input values to the backend and receives the prediction result.

This also gives you much more flexibility over time. If you retrain the model, improve the preprocessing logic, or completely replace the underlying machine learning algorithm, the client application does not need to change. As long as the API contract remains the same, you can continue evolving the model entirely on the server.

### Resources

- [Deploying Machine Learning Models with Vapor and Core ML](https://azamsharp.teachable.com/p/machine-learning-with-python-core-ml-and-vapor)

### Conclusion 

Swift and Core ML are excellent technologies for deploying machine learning models on Apple platforms. The integration with iOS, macOS, watchOS, and visionOS is incredibly powerful, and the performance of on-device inference is impressive.

But training machine learning models is a completely different problem.

Modern machine learning workflows involve heavy experimentation, preprocessing, feature engineering, encoding, normalization, debugging, and working with constantly changing datasets. This is where Python shines. The ecosystem around Pandas, scikit-learn, TensorFlow, PyTorch, and Jupyter notebooks is simply more mature, expressive, and productive.

After spending time with Create ML, I realized that I was fighting the tooling more than solving the actual machine learning problem. Simple transformations required verbose code, Playground support was unreliable, and many workflows felt restrictive compared to Python.

That does not mean Swift has no place in machine learning. Quite the opposite. Swift is excellent for deploying and consuming models using Core ML. In fact, using Swift on the server with frameworks like Vapor or Hummingbird opens interesting opportunities for hosting Core ML powered APIs.

But for training models, preprocessing data, and experimenting with machine learning pipelines, Python remains the better tool for the job.

My preferred workflow today looks like this:

Python → Train Model → Convert with CoreMLTools → Deploy with Swift/Core ML

And honestly, that combination gives you the best of both worlds.