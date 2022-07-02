# Articles 

[Home](https://azamsharp.github.io)
[Courses](/courses)
[Books](/books)
[Articles](/articles)
[YouTube](https://www.youtube.com/channel/UCKvDySsrOVgUgRLhWHeyHJA?view_as=subscriber)
[Speaking](/speaking)
[Contact](/contact)

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

