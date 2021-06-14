
# The Complete Guide to Async/Await in iOS 15 and Swift 5.5 

Asynchronous programming is a common requirement of any iOS application. The ability to perform task on a separate thread and not disturbing or blocking the user interface is always considered a good practice. In iOS 15 and Swift 5.5 Apple introduced async/await feature, which allows developers to easily implement asynchronous tasks with increased clarity and less lines of code. 

In this article, we are going to take a look at how you can use async/await, continuation and actors in your iOS application. 

---

### Asynchronous Programming Using Callbacks 

Before we jump into the new async/await features, we need to see how things are handled currently. It is by looking at the the past, we can appreciate the future. 

For this example we are going to use [JSONPlacefolder API](https://jsonplaceholder.typicode.com/posts) to fetch all posts. The implementation of our client is shown below: 

``` swift
class Webservice {
    
    func getAllPosts(completion: @escaping (Result<[Post], NetworkError>) -> Void) {
        
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
            completion(.failure(.badURL))
            return
        }
        
        URLSession.shared.dataTask(with: url) { data, _, error in
            
            guard let data = data, error == nil else {
                completion(.failure(.badRequest))
                return
            }
            
            let posts = try? JSONDecoder().decode([Post].self, from: data)
            completion(.success(posts ?? []))
            
        }.resume()
        
    }
    
}
```

We all have implemented similar code in our application. The main issue with the above code is that you need to remember to return from the guard clauses. The other issue is the overall length of the code. For an operation like this, it turns out to be 10-15 lines of code. 

In the next section, we are going to implement the same code but using the new async/await features. 

### Understanding Async/await 

Async and await is not something new, several programming languages including C#, and JavaScript have been using async/await for a very long time.

It is definitely a welcome change in Swift language and it will allow developers to write better and clear concurrency code. Now, let's check the async/await implementation for the `getAllPosts` function as implemented below.  


``` swift
 func getAllPosts() async throws -> [Post] {
        
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
            throw NetworkError.badURL
        }
        
        let (data, _)  = try await URLSession.shared.data(from: url)
        return (try? JSONDecoder().decode([Post].self, from: data)) ?? []
        
    }
```

The async/await version is not only fewer lines of code, but it is also much easier to understand. The function `getAllPosts` is also decorated with throws, which indicates that the async function can throw an error. This is the reason we called the `URLSession.shared.data` with `try await` instead of just await. 

> You can only decorate the function with await, if it is marked with async.   

The URLSession now exposes a new async function, which allows us to pass the request URL. The await keyword is required since the URLSession data function is marked with async. Whenever Swift compiler see async with a function it suspends the operation. This means the thread is now free to be used for something else. When the function finally returns after fetching data, Swift compiler picks up where it left off and returns the result to the user. 

> Apart from URLSession, async/await is also available in several other frameworks including HealthKit, CoreData, Notifications etc. 

If you are using MVVM pattern then your view model will be responsible for calling the Webservice. Since, `getAllPosts` is marked with async we need to await for the result in the view model function as shown below: 

``` swift 
class PostListViewModel: ObservableObject {
    
    @Published var posts: [PostViewModel] = []
    
    func fetchPosts() async {
        
        do {
            let posts = try await Webservice().getAllPosts()
            DispatchQueue.main.async {
                self.posts = posts.map(PostViewModel.init)
            }
        } catch {
            print(error)
        }
    }
    
}
```

Once we get the result, we assign it to the posts property which is marked with `@Published` property wrapper. The `Dispatch.main.async` is making sure that the `@Published` property is set on the main thread.  
 
In iOS 15, there is an easier way to make sure that properties and functions are called on the main thread. This is accomplished by using the `@MainActor` property wrapper as shown below: 

``` swift 
@MainActor
class PostListViewModel: ObservableObject {
    
    @Published var posts: [PostViewModel] = []
    
    func fetchPosts() async {
        
        do {
            let posts = try await Webservice().getAllPosts()
            self.posts = posts.map(PostViewModel.init)
        } catch {
            print(error)
        }
    }
    
}
```

The `@MainActor` is part of the new actor API introduced in Swift language. The `@MainActor` makes sure that all properties and functions in PostListViewModel are called on the main thread. This means you don't have to worry about using `DispatchQueue.main.async` anymore.  

> Actors will be covered in more detail in the future articles. 

Now, you can call `fetchPosts` from inside your view as shown in the implementation below: 

``` swift 
struct ContentView: View {
    
    @StateObject private var postListVM = PostListViewModel()
    
    var body: some View {
        List(postListVM.posts, id: \.id) { post in
            Text(post.title)
        }.task {
            await postListVM.fetchPosts()
        }
    }
}
```

One thing you will notice is the use of `task` modifier. The task modifier is introduced in iOS 15 and it used to perform a task when the view is loaded. The task modifier is cancelled when the view disappears. The task modifier is an async closure, which means you can call and await inside the body of the task modifier.  

> At this point you might be wondering that what will happen to the onAppear function of the view. According to the conversations with Apple engineers, currently there are no plans to depreciate onAppear.    


The async keyword is not only for functions, but it can also be decorated on properties and even closures. Take a look at the following code, which calls `fetchPosts` on a button click event. 

``` swift 
 Button {
        await postListVM.fetchPosts()
            } label: {
                Text("Fetch Posts")
            }
```

The above code is not going to compile. Can you see the problem? 

The problem is that we are using await to call the `fetchPosts` function but the context is not marked with async. This means we cannot use await, unless we make the context async. But how can we decorate the button click function with async? The answer is to use async closures. This is shown below: 

``` swift 
  Button {
        async {
        await postListVM.fetchPosts()
            }
        } label: {
                Text("Fetch Posts")
            }
```

The async closure is executed immediately and it allows us to satisfy the requirement of performing an await inside the async context. This is really useful as it allows you to call async functions from non-async contexts. 


# Continuation 

Async and await provides a great way to perform asynchronous tasks but what about your legacy callback implementations. You can use the new `continuation` feature in iOS 15 to invoke your callbacks using async/await. 

At the start of this article we implemented `getAllPosts` using callbacks. If we want to use that function as async/await we could wrap it in a continuation closure as shown below: 

``` swift 
 func getAllPosts() async throws -> [Post] {
        
        return try await withCheckedThrowingContinuation { continuation in
            getAllPosts { result in
                switch result {
                    case .success(let posts):
                        continuation.resume(returning: posts)
                    case .failure(let error):
                        continuation.resume(throwing: error)
                }
            }
        }
    }
```

You can see that we are wrapping the call to `getAllPosts` inside the `withCheckedThrowingContinuation` closure. This means that once your `getAllPosts` call completes, continuation can either resume or throw an error. 

The continuation API comes in different flavors as shown below: 

* withUnsafeContinuation
* withUnsafeThrowingContinuation
* withCheckedContinuation
* withCheckedThrowingContinuation

You can read more about continuation on Swift evolution repository [here](https://github.com/apple/swift-evolution/blob/main/proposals/0300-continuation.md). 

> The continuation concept is similar to Promises and Futures. I hope that Apple make the continuation API more intuitive by giving it a developer friendly name.  

Continuation is a great way to wrap your existing functions and give them the ability to be called using async/await syntax. In JavaScript, you can also use the `Promise.all` and `Promise.any` syntax to make sure that the Promise is only resumed if all or any of the tasks are completed. This functionality is currently not available in Swift language but I hope they add it in the future versions. 

# Understanding AsyncSequence

iOS 15 also introduces the support for asynchronous sequences using `AsyncSequence` protocol. `AsyncSequence` is useful when you are interested in functions that return many values over times. 

`AsyncSequence` is already available in several Apple APIs including `URL`, `URLSession` and `Notification` etc. 

In order to iterate an `AsyncSequence`, you can use the `for-await in` syntax. In the code below we are iterating through the `bytes.lines` sequence using the new `await in` keyword in Swift. 

``` swift
 let request = URLRequest(url: url)
        
        let (bytes, _) = try await URLSession.shared.bytes(for: request)
        
        for try await line in bytes.lines {
            print(line)
        }
```

`AsyncSequence` will allow us to wait on each element, instead of the getting the entire result. 

Here is another example of `AsyncSequence` using `NotificationCenter`. 

``` swift 
for await note in NotificationCenter.default.notifications(named: .itemCreated) {
            // Use item.
        }
```

The above code will iterate through all the `.itemCreated` notifications asynchronously and process them. 

### Conclusion 

Async/await is one of the more anticipated new feature of the Swift language. It will drastically change how we write concurrent code and it will allow developers to implement asynchronous code in a clear concise manner. 

### Resources 

- [Video - Async/Await in Swift](https://youtu.be/U1H72vSgZl0)
- [WWDC Video - Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)
- [Video - Stocks App in iOS 15 - Pull to Refresh, async/await, actors](https://youtu.be/c3lvJSkniB8)


If you liked this article and want to support my work then check out courses on Udemy. 

<center>
<a href = "http://www.azamsharp.com/courses">
<img src="images/banner.png"> 
</a>
</center>

