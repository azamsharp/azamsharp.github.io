# Articles 

[Home](https://azamsharp.github.io)
[Courses](https://azamsharp.school/)
[Books](/books)
[Articles](/articles)
[Newsletter](https://azamsharp.teachable.com/p/newsletter)
[YouTube](https://www.youtube.com/channel/UCKvDySsrOVgUgRLhWHeyHJA?view_as=subscriber)
[Speaking](/speaking)
[Contact](/contact)

# [StoreKit Subscriptions: A Practical Guide Part 2: Soft vs Hard Paywalls](_posts/2025-12-27-storekit-subscriptions-soft-vs-hard-paywalls.md) 

Choosing the right paywall strategy is one of the most important decisions you will make when monetizing an iOS app. It affects not only revenue, but also user trust, retention, and long term sustainability. Let’s break down the most common approaches and how they fit together.

### [StoreKit Subscriptions: A Practical Guide Part 1: Understanding Different Monetization Models](_posts/2025-12-26-storekit-subscriptions-understanding-monetization-models.md) 

I started iOS development back in 2010. My first app was a kids game called ABC Pop, which I built with my wife. She recorded all the audio for it. After that, I went on to release dozens of apps on the App Store, most notably Vegetable Tree Gardening Guide.

Vegetable Tree was featured by Apple multiple times and was also mentioned in several online magazines. Looking back, all of my apps used a one time purchase model. You paid once and got access to the app for life.

In this article, we are going to go over different monetization models available with Apple. 


### [What I Learned While Building My Veggie Garden](_posts/2025-11-19-lessons-learned-veggie-garden.md)

I recently released a new Vegetable Gardening app called **My Veggie Garden**. It is built with SwiftUI, SwiftData, and Foundation Models, and it has been one of the most rewarding projects I have worked on. Along the way I made mistakes, rewrote things, simplified things, and discovered a few lessons that I wish I had known earlier.

### [Why 90% of SwiftUI Apps Get Dependency Injection Wrong](_posts/2025-09-08-why-90-percent-swiftui-apps-gets-dependency-injection-wrong.md)

Managing dependencies is an essential part of building scalable and testable SwiftUI applications. Dependencies such as API clients, stores, and services form the backbone of your app’s data flow. If handled carelessly, they can lead to tightly coupled code that is difficult to test, reuse, or extend.

SwiftUI, being a declarative framework, provides multiple strategies for injecting and managing dependencies. The most common approaches include constructor injection, where dependencies are passed directly into views or services, and environment values/objects, which allow dependencies to be shared across the view hierarchy without explicitly threading them through every initializer.

### [Effective Communication Between Observable Stores in SwiftUI](/_posts/2025-08-17-effective-communication-between-observable-stores.md)

Modern SwiftUI applications often rely on observable stores to manage state and business logic. As apps grow in complexity, these stores need to communicate efficiently—whether reacting to user actions, synchronizing data, or triggering side effects. This article explores practical patterns for inter-store communication, from direct method calls to event-driven approaches like Combine publishers and Swift Concurrency’s `AsyncStream`. 

### [How Vibe Coding is Hurting Your Critical Thinking](/_posts/2025-08-03-how-vibe-coding-is-hurting-your-critical-thinking.md)

Back when I used to commute home from a client’s office, I would call a friend almost every day to keep me company during the long drive. For the entire hour, we would chat about nothing in particular, just aimless conversation, until I reached my destination.

If you asked me what I saw during that drive, I wouldn’t be able to tell you. Not the road. Not the traffic. Not even the turns I took.

That is exactly how vibe coding feels.

### [The Ultimate Guide to the Foundation Models Framework](_posts/2025-06-18-the-ultimate-guide-to-the-foundation-models-framework.md) 

Apple’s Foundation Models framework, unveiled at WWDC 2025, brings powerful on-device generative AI to Apple platforms—enabling private, fast, and capable language-based features like summarization, content generation, and structured data output. This guide introduces key components such as `LanguageModelSession`, real-time streaming, and `@Generable`-powered guided generation. Through a practical SwiftUI recipe app example, it also covers tool integration, response persistence with SwiftData, and performance best practices. Ideal for developers building intelligent, privacy-focused iOS apps, this article serves as a hands-on introduction to Apple’s new on-device AI capabilities.


### [SwiftData Architecture - Patterns and Practices](_posts/2025-03-28-swiftdata-architecture-patterns-and-practices.md) 

SwiftData, introduced at WWDC 2023, is Apple’s modern, Swift-native framework for data persistence in SwiftUI applications. Designed to replace Core Data, it offers a declarative API that aligns closely with SwiftUI’s architecture. This article explores practical architectural patterns and best practices for building real-world SwiftData applications. Using a budget tracking app as a case study, it covers topics such as model structuring, embedding business logic, working with DTOs, writing unit tests, creating effective Xcode previews, integrating with CloudKit, and designing flexible data access layers. The goal is to help developers adopt SwiftData in a scalable, maintainable, and testable way.

### [Filtering SwiftData Models Using Enum](_posts/2025-01-23-filtering-swiftdata-models-using-enum.md)

SwiftData, Apple’s modern persistence framework, provides a streamlined approach to managing on-device data. However, it introduces challenges when working with enums, particularly when filtering data using their `rawValue`. This article explores the limitations of querying `SwiftData` models with enums and offers a practical solution. By replacing the enum property with its `rawValue` in the model and retaining the enum as a computed property, we achieve compatibility with SwiftData while preserving the advantages of enums, such as type safety and clarity. Through a step-by-step implementation, including dynamic queries and a SwiftUI interface, this article provides a comprehensive guide to effectively handling enum-based filtering in SwiftData applications.

### [The Ultimate Guide to Validation Patterns in SwiftUI](_posts/2024-12-18-the-ultimate-guide-to-validation-patterns-in-swiftui.md)

Validation is a crucial component of app development, ensuring that only accurate and meaningful data is processed by your application. As the saying goes in software development, **Garbage in, garbage out.** If your app allows users to submit invalid data, it will inevitably result in unreliable and flawed outputs.

In this article, we will explore various techniques for implementing robust validation in your SwiftUI apps to enhance data integrity and user experience.

### [Deep Dive into Environment in SwiftUI](_posts/2024-11-18-deep-dive-into-environment-in-swiftui.md)
SwiftUI simplifies app development with its declarative syntax and reactive data handling, making it easy to build dynamic and responsive UIs. Central to this framework is the `@Environment` property wrapper and its tools, enabling seamless shared state management across views.  

This article explores state management in SwiftUI, from `@EnvironmentObject` and `ObservableObject` to the new `@Observable` and `@Bindable` macros in iOS 17. You’ll learn how to inject and access global state, optimize state propagation, and streamline complex view hierarchies.  

By the end, you’ll master using SwiftUI’s environment tools to create scalable, maintainable, and modular components. Whether you’re new to SwiftUI or refining your skills, this guide has you covered. Let’s dive in!

### [Simplifying List Sorting in SwiftUI: A Guide to Custom Environment Values](_posts/2024-10-27-simplifying-list-sorting-in-swiftui-a-guide-to-custom-environment-values.md)
In React, **hooks** allow function components to access state and lifecycle features, with **custom hooks** enabling the reuse of common logic. Examples include `useFetch` for fetching data, `useLocalStorage` for persisting state, and `useSorting` for sorting lists. In SwiftUI, while there is no direct equivalent to hooks, similar functionality can be achieved using **custom Environment values** like `dismiss` and `colorScheme`. This article demonstrates how to create a custom Environment value in SwiftUI to implement reusable sorting logic, akin to React’s custom hooks.

### [Introduction to Communication Patterns in SwiftUI](_posts/2024-09-22-introduction-to-communication-patterns-in-swiftui.md)
SwiftUI provides a powerful and declarative way to build UIs, allowing views to react to state changes automatically. However, managing communication between views, especially when passing data or events from one view to another, can be challenging if not handled properly. In this article, we'll explore several communication patterns in SwiftUI that enable seamless data flow between views, ensuring that updates occur efficiently and in a way that aligns with SwiftUI’s architecture.

### [The Hidden Cost of AI-Generated Unit Tests: Sacrificing Domain Knowledge](_posts/2024-09-30-hidden-cost-of-ai-unit-tests.md)
Over the past 20 years, I have worked with a diverse range of large organizations across various industries, including retail, healthcare, oil and gas, and energy. The biggest challenge I encountered wasn't the programming languages, technologies, or platforms—it was the domain knowledge. To build high-quality software, it's crucial to fully grasp the business rules and understand the intricate processes that drive operations behind the scenes.


### [Global Sheets Pattern in SwiftUI](_posts/2024-08-18-global-sheets-pattern-swiftui.md)
Managing multiple sheets in SwiftUI can become cumbersome with traditional approaches. This article introduces the Global Sheets Pattern, a streamlined method for centralizing sheet management using a single `SheetAction` struct and custom environment values. 

By exploring various techniques for presenting sheets and leveraging ideas from other platforms, we demonstrate how this pattern simplifies sheet handling, improves code maintainability, and enhances app architecture.

### [Navigation Patterns in SwiftUI](_posts/2024-07-29-navigation-patterns-in-swiftui.md)
Navigation has often been a challenge in SwiftUI applications. Initially, SwiftUI introduced NavigationView, which was later replaced by NavigationStack in iOS 16.

NavigationStack enhanced navigation by enabling dynamic and programmatic routing, and it also offered ways to centralize routes for the entire application. In this article, I'll explore common navigation patterns that can be employed when building SwiftUI applications.

### [Newsletter: Stop Testing Your UI Using View Models ](_posts/2024-01-22-stop-testing-your-ui-using-view-models.md) 
A few months ago, I attended iOSDevHappyHour and engaged in a discussion with a young man. He was discussing how he tested his SwiftUI interface using view models. I asked him for clarification, and he explained that instead of writing UI tests, he simply wrote tests for the view models. According to him, if the view model tests passed, then the UI was working as expected.

### [Newsletter: Is MVVM Dead in SwiftUI?](_posts/2024-01-09-is-mvvm-dead-in-swiftui.md)
A few weeks ago, I came across an interesting tweet on Twitter (or perhaps I should say X). The tweet originated from Thomas Ricouard, the creator of IceCubes, the famous Mastodon client for iOS. Here's a screenshot of the tweet.

### [The Complete Guide to JSON Web Tokens (JWT) Authentication in iOS](_posts/2023-11-07-complete-guide-jwt-authentication.md)

Authentication is a cornerstone of application development, safeguarding user interactions in both client and server applications. In a world where JSON reigns as the primary data exchange medium, JSON Web Token (JWT) authentication has emerged as an industry standard, ensuring data security and integrity.

Within this comprehensive guide, we embark on a journey through the intricacies of setting up JWT authentication on the server and seamlessly implementing this security measure on the client side using SwiftUI. This knowledge equips you with the essential skills needed to fortify your application's authentication mechanisms, ensuring reliability and robustness in today's ever-evolving digital landscape.

In this article, the power of SwiftUI is harnessed for the client interface, while ExpressJS serves as the backbone for server-side operations. While this choice may pique the curiosity of iOS developers, it's grounded in my extensive experience with and confidence in ExpressJS, a robust and dependable tool for server development.


### [Convenience Property Wrappers vs Custom Data Access Layer in SwiftUI](_posts/2023-07-15-property-wrappers-vs-data-access-layer.md)

Yesterday, I had the opportunity to speak at [WomenWhoCode Mobile](https://www.womenwhocode.com/network/mobile/) event. It was a remote event and well attended. I spoke about SwiftUI architecture best practices. 

When I was covering Core Data I mentioned that you should use ```@FetchRequest``` property wrappers as they are optimized to work with SwiftUI. Same is true for ```@Query``` property wrapper in SwiftData. 

During this time an interesting question was raised. An attendee asked what if you want to change the data access layer in the future. Currently our views are tightly coupled with either Core Data or SwiftData but what happens if we want to use Realm or GRDB. 

### [The Ultimate Guide to Building SwiftData Applications](/_posts/2023-07-04-the-ultimate-swift-data-guide.md)

SwiftData made its debut at WWDC 2023 as a replacement for the Core Data framework. Serving as a wrapper on top of Core Data, SwiftData enables on-device persistence and seamless syncing to the cloud.

One of the key benefits of utilizing SwiftData lies in its effortless integration with the SwiftUI framework. This article is structured into several sections, each delving into different aspects of the SwiftData framework. First, we will explore the foundational concepts of SwiftData, followed by an examination of its architectural design, relationship management, migration capabilities, and more. By navigating through these sections, you will gain a comprehensive understanding of SwiftData's features and functionalities, empowering them to leverage its full potential in your iOS development endeavors.

### [The What-If Architecture](_posts/2023-04-03-the-what-if-architecture.md)
Lately, I have been thinking about the "What-If Architecture". Most people commonly refer to it as YAGNI (You Aren't Gonna Need It). I was reading some discussion thread, where a developer was creating a Core Data app using SwiftUI and wanted to display information from the database on the screen.

### [Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture](_posts/2023-02-28-building-large-scale-apps-swiftui.md)
Software architecture is always a topic for hot debate, specially when there are so many different choices. For the last 8-12 months, I have been experimenting with MV pattern to build client/server apps and wrote about it in my original article [SwiftUI Architecture - A Complete Guide to MV Pattern Approach](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). In this article, I will discuss how MV pattern can be applied to build large scale client/server applications. 

### [Testing View Model Does NOT Validate the User Interface](_posts/2023-02-16-testing-view-models.md)
Couple of weeks ago, I was having a discussion with another developer, who was mentioning that they test their **user interface** through View Models in SwiftUI. I was not sure what he meant so I checked the source code and found that they had lot of unit tests for their View Models and they were just assuming that if the View Model tests are passing then the user interface will automatically work.

### [Testing is About Confidence](_posts/2023-02-15-testing-is-about-confidence.md)
If you ask 100 people what testing means to them, they will give you 100 different answers. For me personally, testing is all about confidence. Confidence that my code works and confidence that it will work as expected in the future.

### [Resources - Passive Income for Developers](_posts/2023-02-01-resources-passive-income.md)

Here are some resources we discussed during the Twitter Spaces. I hope you find them useful.  

### [Active Record Pattern for Building SwiftUI Apps with Core Data](_posts/2023-01-30-active-record-pattern-swiftui-core-data.md)

Lately, I have been talking a lot about different architectural patterns that can be used to build SwiftUI applications. I discussed [MV pattern](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html) for building client/server applications and [Container pattern](https://azamsharp.com/2023/01/24/introduction-to-container-pattern.html) for building hobby projects or projects where testing is not considered first class citizen. 

Recently, I have been building Core Data applications and I wanted to try out Active Record Pattern and see how it feels. In this post, I will cover my experience with building SwiftUI apps using the Active Record Pattern.

### [A Better Way to Handle Events for SwiftUI Views](_posts/2023-01-24-grouping-events-swiftui-view-using-enums.md.md)

One way to make reusable views in SwiftUI is to expose the events as a closure. This allows the parent to consume the closure and take action. In this post, I will demonstrate how the events from a SwiftUI view can be grouped together into an enum allowing you to reduce code for creating multiple closures per view. 

### [Container Pattern for Building SwiftUI Apps](_posts/2023-01-24-introduction-to-container-pattern.md)
Container pattern is a common pattern used in React community. Since React and SwiftUI are quite similar, this pattern can also be used for building SwiftUI applications. In this article, I will focus on the concepts behind the container pattern and how you can use it to build your SwiftUI applications. 


### [Pragmatic Testing and Avoiding Common Pitfalls ](_posts/2012-12-23-pragmatic-unit-testing.md)

The main purpose of writing tests is to make sure that the software works as expected. Tests also gives you confidence that a change you make in one module is not going to break stuff in the same or other modules. 

Not all applications requires writing tests. If you are building a basic application with a straight forward domain then you can test the complete app using manual testing. Having said that in most professional environments, you are working with a complicated domain with business rules. These business rules form the basis on which company operates and generates revenue. 

In this article, I will discuss different techniques of writing tests and how a developer can write good tests to get the most return on their investment. 

### [Becoming an iOS Developer - The Complete Guide 2023](_posts/2022-11-06-complete-guide-ios-dev-2023.md)

It always excites me to see so many people jumping into iOS development. We have a great community with lot of talented people and it is continuously expanding. 

Recently, I have been talking to a lot of new developers and one of the main challenges they shared with me is that they have trouble getting started and they don't have a clear path on what to learn in order to move forward in their iOS journey.

In this post, I will cover my recommendations on how you can become an iOS developer. Keep in mind that this is not the only path but just one of possible ways you can become an iOS developer. This is the strategy I would have used if I was in their position. 

### [Evolving SwiftUI Architecture for Client/Server Apps](_posts/2022-10-30-evolving-client-server-swiftui.md)

In the last architecture, we discussed in detail about [SwiftUI Architecture using the MV Pattern](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). It is highly recommended that you read the [original post](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). In this post, we will cover how to create SwiftUI client/server applications using patterns and practices learned from ReactJS framework. 

### [Embracing Core Data in SwiftUI](_posts/2022-10-11-embracing-core-data-in-swiftui.md)

Last year I was working on an app which was using the Core Data framework as a persistent medium to the SQLite store. I was reluctant to use any SwiftUI property wrappers for Core Data in my app, because I wanted to structure the app in several layers and those property wrappers were only available inside the View. The app worked but it was a pain to make sure that everything in Core Data was synced with SwiftUI views. 

 SwiftUI team has provided us with APIs to make sure that SwiftUI and Core Data works seamlessly together. In this post, we will be building a small budget app using SwiftUI and Core Data. We will start by discussing our original approach of implementing the app, where we did not use any helpers provided by the SwiftUI frame. Later, we will look into a much simpler implementation, which uses SwiftUI Core Data property wrappers.

### [SwiftUI Architecture - A Complete Guide to MV Pattern Approach](_posts/2022-10-06-practical-mv-pattern-crud.md)

I was listening to an amazing talk by Matias Villaverde and Rens Breur at NSSpain about "Lessons learnt rewriting SoundCloud in SwiftUI". You can watch the complete talk [here](https://vimeo.com/751534042/f1ae29434e). 

This talk really resonated with me because I did similar mistakes when building SwiftUI applications. Instead of embracing the simplicity of the framework, I added unnecessary complexity to please the design pattern Gods. This included creating view models for each view, ignoring ```@FetchRequest``` and ```@SectionFetchRequest``` property wrappers, passing ```@EnvironmentObject``` to the view model instead of accessing it directly in the view and much more.  

After almost two years of driving in the wrong direction, I decided to slam on the brakes and think about my decisions. In this post, I will discuss the SwiftUI architecture that I am using for my apps. 

###  [MV State Pattern - A Better Way of Building SwiftUI Apps ](_posts/2022-08-09-intro-to-mv-state-pattern.md)
I started working with SwiftUI framework in 2019. Like most developers, I also jumped on the MVVM bandwagon. I wrote books on it, gave presentations and even created a lot of videos. I managed to get MVVM working with SwiftUI in almost all of my projects. But it was a constant battle. I always felt that I am fighting SwiftUI framework. Even for small and medium sized projects, I felt that I was writing too much code and adding unnecessary layers. In this post, I will introduce MV pattern. This is not something I invented. This is the same pattern Apple use in their code samples for their SwiftUI apps. Check out the references section at the end of this post. 

###  [SwiftUI View is also a View Model](_posts/2022-07-21-view-is-the-view-model.md)
In this post, I will cover how a SwiftUI View is not only a view but also a ViewModel. We will compare it with WPF framework and see how SwiftUI already has built-in support for ViewModel right within the view. This means that in most cases, you don't need to create an extra layer of View Model per screen. 

### [Why do iOS Developers hates cross platform frameworks? ](_posts/2022-07-25-why-ios-devs-hates-cross-platform.md) 
Most iOS developers have a very strong negative reaction towards cross platform frameworks. Some even advice to not learn cross platform as it will negatively impact your career. In this post, I will discuss my personal experience of working with cross platform frameworks and how I started to appreciate them. 

### [I was wrong! MVVM is NOT a good choice for building SwiftUI applications](_posts/2022-07-17-2022-swiftui-and-mvvm.md)
SwiftUI was introduced at WWDC 2019 and it completely changed how we build our apps for Apple platform. SwiftUI provided a declarative framework, which allowed developers to quickly and easily build user interface as compared to its predecessor UIKit or AppKit. Somewhere along the lines we adopted MVVM (Model View ViewModel) design pattern as the default pattern when building SwitUI applications. In this post, I will cover my experience of using MVVM pattern with SwiftUI framework and how it worked against the SwiftUI framework, making things more complicated. 


### [Displaying App Logo in Navigation Bar for All Views in SwiftUI](_posts/2022-07-06-app-logo-in-navigation-bar.md)
When building iOS applications, sometimes you have a requirement to put application logo in the center of the NavigationBar on all the screens. In this post, you will learn how to acheive that using ZStack. 

### [Configuring Global Routing in SwiftUI Using NavigationStack](_posts/2022-07-02-global-routing-using-navigation-stack.md)
NavigationStack in iOS 16 allows developers to setup global routing for your application. Now, just like React and Flutter we can configure routes for our entire application in a central place. In this post, I will cover how you can setup global routing for your SwiftUI application. 

### [Slicing Global State in SwiftUI Using Multiple EnvironmentObjects](_posts/2022-07-01-slicing-environment-object.md)
@EnvironmentObject in SwiftUI provides a way to configure global state for your application. Updating the global state, allows the views to re-render/refresh. Sometimes we are only interested to update a view when a small part of the global state changes. In this post, I will cover how you can create segments of your global state so your view only updates when that slice is changes. 

### [@EnvironmentObject in Views May Not be a Good Idea But Avoiding Them is Probably Much Worse ](_posts/2022-06-30-environment-object-view-model.md)
In SwiftUI, ```@EnvironmentObject``` allows you to create global state that can be shared and manipulated from any view in your application. We tend to put @EnvironmentObject in our views and directly access the global state. This creates a tight coupling between the view and the ```@EnvironmentObject```, but avoiding this approach opens up a whole new cans of worms. In this article, I will discuss how @EnvironmentObject can be used in a SwiftUI view and how avoiding it can create dependency nightmare. 

### [Building a Rating View in SwiftUI](_posts/2021-08-27-building-swiftui-rating-view.md)
In this post, you will learn how to build a Rating view in SwiftUI. Rating view will allow you to select a star rating and get access to the integer value of the rating.

### [Loading UIKit View into SwiftUI App](_posts/2021-08-26-activity-view-swiftui.md)

In this post, you will learn how to load a UIKit view into SwiftUI application. Consider a scenario that you want to display a loading indicator in a SwiftUI app. UIKit has an `UIActivityIndicatorView` control that can be used to display loading indicators. Let's see how we can load the `UIActivityIndicatorView` in a SwiftUI application. 

### [Introduction to Microservices](_posts/2021-08-26-intro-microservices.md)
Microservices allow you to break down your application into smaller pieces. Each piece, known as a microservice is responsible for handling a particular aspect of your application. In this article, you are going to learn about the concepts of microservices.  

### [Understanding Task Modifier in Swift ](_posts/2021-06-25-Understanding-Task-Modifier-in-Swift.md)
As a developer who has worked in React, Flutter and SwiftUI, it is always nice to see that how many SwiftUI features are inspired from existing platforms. All three major platforms (React, Flutter and SwiftUI) have adopted a declarative approach for building user interfaces. This means you can easily transfer your knowledge between React, Flutter and SwiftUI. 

In iOS 15 a new task modifier has been introduced, which can be used to perform an operation when the view appears and cancelled when the view disappears. In this post, I will talk about the new task modifier and how it can be used to handle dependencies. 


### [Lazy Properties in Swift](_posts/2021-06-21-lazy.md)
Swift language allows you to create several different types of properties, including computed, property observers and even lazy properties. In this article, we will learn how lazy properties can provide performance benefits for time consuming calculations. 

### [Data Modeling Using Enums](_posts/2021-06-17-Data-Modeling-Using-Enums.md)
Application domain objects are the building blocks of any system. The domain represents the entities and the connections betweens entities of the app. The domain is also used to map the real world into our system. 

There are many ways of implementing domain objects. In this article we will learn how to model objects using Swift enum type. 


### [Beginning Async/Await in iOS 15 and Swift 5.5 ](_posts/2021-06-16-Beginning-Async-Await-in-iOS-15-and-Swift-5.5.md)

Asynchronous programming is a common requirement of any iOS application. The ability to perform task on a separate thread and not disturbing or blocking the user interface is always considered a good practice. In iOS 15 and Swift 5.5 Apple introduced async/await feature, which allows developers to easily implement asynchronous tasks with increased clarity and less lines of code. 

In this article, we are going to take a look at how you can use async/await, continuation and actors in your iOS application. 

### [Customizing Buttons in SwiftUI iOS 15](_posts/2021-06-15-Customizing-Buttons-in-SwiftUI.md) 

Buttons are an important part of any iOS application and in iOS 15, SwiftUI introduces several different ways to implement and customize buttons views. In this article we are going to learn about all the new ways you can customize your buttons.

### [Stocks App Using Async/Await, Pull To Refresh and Continuation in SwiftUI for iOS 15](_posts/2021-06-15-stocks-app.md) 

Apple introduced tons of new features in Swift and SwiftUI. This includes Async/Await, Pull to Refresh, Continuation, Text Formatters etc. In this article, we are going to combine all the features together and build a stocks application.

### [Understanding AsyncImage in SwiftUI](_posts/2021-06-15-Understanding-AsyncImage-in-SwiftUI.md) 

At WWDC 2021, Apple unveiled tons of new SwiftUI features that will allow developers to create iOS apps even more fluently. One of the most anticipated features is the ability to display images using the Image view. In previous versions of SwiftUI, this was not possible and had to be implemented using custom code.
In iOS 15 and Xcode 13, Apple introduced AsyncImage, which allows you to download an image using just the URL. In this article, we will look at how to use AsyncImage in our SwiftUI applications.

### [Decoupling SwiftUI Views for Reusability](_posts/2021-06-15-Decouple-SwiftUI-Views-for-Reusability.md) 

When designing SwiftUI applications it is extremely important to make sure that the views are decoupled and reusable. Tightly coupled views are harder to maintain, reuse and can result in future complications.
In this post, I will demonstrate how to implement decoupled views that can be reused in SwiftUI applications.


### [Asynchronous Requests Using SwiftUI and Redux Middlewares](_posts/2021-06-15-async-request-redux-swift.md) 

In the [last post](redux-swiftui), we discussed how to implement Redux design pattern in SwiftUI application. One of the most common operations in an iOS app is to perform asynchronous requests and display the result on the user interface. 

In Redux architecture, middleware are used to fetch data from an API and later dispatch an action updating the store. In this post, you will learn how to implement the middleware flow in Redux. 

___ 

### [Surviving a Coding Bootcamp](_posts/2021-06-15-Surviving-the-Coding-Bootcamp.md) 

After training 100s of developers at multiple bootcamps, I believe it is my responsibility to share my learnings on what makes an excellent student and what steps you can take to survive and thrive in an intensive software development bootcamp.

---

### [Global State Management for SwiftUI Apps Using Redux (Introduction)](_posts/2021-06-15-redux-swiftui.md)

State management is an essential part of any SwiftUI application. SwiftUI provides several built-in ways for managing state, which includes @State, @EnvironmentObject, @Binding and @StateObject.

@EnvironmentObject does manage global state, but it does not provide any structure. In this post, you will learn how to use Redux to organize the flow of your global state. 

___


### [ViewModel Patterns](_posts/2021-06-15-viewmodel-patterns.md)
MVVM Design Pattern allows us to write clean, modular apps where each screen can be controlled by one parent ViewModel. In this post, I will go over several common ViewModel patterns when implementing apps using MVVM pattern. 

---

### [Core Data with SwiftUI Part 1](_posts/2021-06-15-core-data-swift-ui-part-1.md)

In this post, you will learn how to integrate Core Data with your SwiftUI application using MVVM Design Pattern. You will also learn how to decouple your views with models by creating an abstraction for Core Data. 

---

