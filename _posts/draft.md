# The Ultimate Guide to the Foundation Models Framework 

At WWDC 2025, Apple unveiled the Foundation Models framework, an on-device LLM (Large Language Model). Foundation Models allows developers to utilize powerful generative AI capabilities directly on Apple devices — enabling natural language understanding, content generation, summarization, and more — all while maintaining user privacy and performance by running entirely on-device.

In this article, we will walk through how to get started with Apple’s Foundation Models framework and explain the core concepts you need to understand in order to take full advantage of its powerful on-device AI capabilities.


## Requirements

Let's first start with the requirements. In order to use Foundation Models framework you need to satisfy number of requirements. 

1. macOS Tahoe 
2. Xcode 26
3. Apple Silicon 

You can visit [Apple Developer Portal](https://developer.apple.com/download/all/?q=Xcode) to download all the needed tools, included the operating system. 

> It is recommended to not install beta operating system on your primary machine. If you have a spare Apple silicon MacBook or a similar machine then you can definitly use that to try out all the new features. 

## Getting Started 

The best way to get started with Foundation Models is by using the new `#Playground` macro in your SwiftUI application. The `#Playground` macro allows you to embed playgrounds directly alongside your SwiftUI views, making it an excellent way to quickly test and iterate on your code.

Below you can find, what I like to call is hello world of Foundation Models framework. 

``` swift 
import Playgrounds
import FoundationModels

#Playground {
    let session = LanguageModelSession()
    let response = try await session.respond(to: "List all states in USA.")
    print(response.content)
}
```

We start by importing Playgrounds since we're using the #Playground macro. Next, we import FoundationModels to enable communication with the on-device LLM.

Interaction with the model begins by creating an instance of LanguageModelSession. This session maintains state between requests, allowing it to retain the context of the conversation or prompts.

We then call the respond function on the session object and pass in a prompt. A prompt is a request or query from the user—essentially, a question being asked. In this case, we're asking the model to list all the states of the USA.

Finally, we print the string response using the response.content property. The result is shown below:

![Listing all states in USA](/images/all-50-states.png)

It’s truly impressive how simple the Foundation Models API is to work with. With just a few lines of code, we were able to send a request and receive a complete response.

-- make it better from here and fix spelling and grammer 

One thing you may have noticed is that the user had to wait for the entire response to be available. This added latency and delay from the user's perspective. A better approach would be to stream the response so it is available as soon as possible.

In the implementation below, we are streaming the response as soon as it is available. 

``` swift 
#Playground {
    let session = LanguageModelSession()
    let stream = session.streamResponse(to: "Write a short story on a cat lost in the big city.")
    
    for try await partialResponse in stream {
        print(partialResponse)
    }
}
```

As soon as the partial response is available it is displayed on the screen. This is an excellent way to keep user engaged, without waiting for the complete response to be available. If you had used a non-stream approach to execute the above prompt, it would have taken at least 20-30 seconds.

### Guided Generation 

In the previous section you learn how to get a response from the Foundation Model. The responses we received were plain string. But what if we wanted more structured response. 

We can try to play smart and specify in our prompt that we are looking for a JSON response. Unfortunately, this can get really complicated. The main reason is that the model will mostly never return the same exact structure. You will always have to parse the response and remove the unwanted text. And for nested and more complicated responses, it becomes extremely hard to maintain. 

Fortunately, Foundation Models framework solves this problem by a concept known as **Guided Generation**. Guided generation allows the model to return structured responses. This means you will always get the same exact structure no matter what. 

You start by implementing a struct to represent the response. This is shown below: 

``` swift 
@Generable()
struct Recipe {
    @Guide(description: "The name of the recipe.")
    let name: String
    @Guide(description: "The description of the recipe.")
    let description: String
}
```

The struct is decorated with ```@Generable``` macro, indicating that this struct is part of the guided generation. You can also decorate individual properties of the struct with ```@Guide``` macro. This influences the allowed values for the properties of a generable type. 

Now, we can use our guided generation type to receive a structured output. 

``` swift 
#Playground {
    
    let session = LanguageModelSession()
    let response = try await session.respond(to: "Suggest a tradional Pakistani recipe", generating: Recipe.self)
    
    // response.content is of type Recipe
    print(response.content)
    print(response.content.name) // name of the recipe
    print(response.content.description) // description of the recipe
}
```

In the above code when the model generate a recipe and it uses out ```@Generable``` type to populate the result. Instead of getting a string response, we receive a ```Recipe``` object. 

You can also add constraints to your ```@Generable``` models. In the code below, we are adding a constraint on a newly added property ```prepTimeMinutes```. 

``` swift 
@Generable()
struct Recipe {
    @Guide(description: "The name of the recipe.")
    let name: String
    @Guide(description: "The description of the recipe.")
    let description: String
    @Guide(description: "The prep time in minutes.", .range(0...20))
    let prepTimeMinutes: Int
}
```

The constraint ```.range``` makes sure that the generated value is between 0 and 20. Now, when the model generates the ```Recipe``` struct, it will assign a random value within range to ```prepTimeMinutes``` property. 