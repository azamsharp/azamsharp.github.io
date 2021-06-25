
# Understanding Task Modifier in Swift 

As a developer who has worked in React, Flutter and SwiftUI, it is always nice to see that how many SwiftUI features are inspired from existing platforms. All three major platforms (React, Flutter and SwiftUI) have adopted a declarative approach for building user interfaces. This means you can easily transfer your knowledge between React, Flutter and SwiftUI. 

In iOS 15 a new task modifier has been introduced, which can be used to perform an operation when the view appears and cancelled when the view disappears. In this post, I will talk about the new task modifier and how it can be used to handle dependencies. 

> The function onAppear still exists and there are currently no plans to depreciate it. You will learn in this post that task modifier serves a different purpose than onAppear.  

### Implementation

The main purpose of the task modifier is to `await` for result from an `async` operation. This is the reason the task closure is marked with an `async` keyword. Within the task modifiers you can perform a call to an API and populate your view. Here is a small example: 

``` swift

 @State private var taskTitle: String = ""
    
    private func getTodo() async throws -> String  {
        
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/todos/1") else {
            throw NetworkError.badUrl
        }
        
        let (data, _) = try await URLSession.shared.data(from: url)
        let todoItem = try? JSONDecoder().decode(TodoItem.self, from: data)
        return todoItem?.title ?? ""
    }

var body: some View {
        VStack {
            Text(taskTitle)
                .padding()
                .task({
                    do {
                        self.taskTitle = try await getTodo()
                    } catch {
                        print(error)
                    }
                         
                })
        }
    }
```

> It is not recommended that you call the API from the View. You should use a pattern like MVVM, Redux, MVC to layer your app. 

In the task modifier, we are calling the `getTodo` function. The `getTodo` function is an async function, which calls the url, gets the data and returns the task title. Finally, the `taskTitle` property is assigned with the new title and it gets displayed on the screen.

In most cases, you will use the task modifier to perform an initial request and populate your screen. But task modifier can also handle dependencies. In the next section, you will learn how to invoke the task closure, if any of its dependencies change.

## Handling Task Modifier Dependencies 

The concept of handling dependencies in task modifier is almost identical to `useEffect` in React. Take a look at React implementation below: 

``` js
import { useEffect, useState } from 'react'

export default function App() {

  const [count, setCount] = useState(0)

  useEffect(() => {
    // this will get fired when the component is mounted 
    // and then when the count is updated 
  }, [count])
```

You can consider `useEffect` the same as task modifier in Swift language. The body of `useEffect` is fired then the view is loaded for the first time and anytime the dependencies change. This means the body of the `useEffect` is triggered when the value of count changes. 

The same exact concept is available in Swift. Take a look at the following implementation: 

``` swift 
 var body: some View {
        VStack {
            Text("\(count)")
                .padding()
                .task(id: count) {
                    // this is fired when the view is loaded
                    // and also when the count changes
                    _ = try? await getTodo()
                }
            Button("Increment") {
                count += 1
            }
        }
    }
```

In above code, task modifier is fired when the view is loaded and it will be fired in the future whenever the count value is changed. This means if you increment the count by pressing the button, it will call `getTodo` function and make a network call.  

Here is another example, where the task modifier is called repeatedly, since you are setting the state inside the task modifier. This will create an infinite loop and your application will suffer. 

``` swift
    var body: some View {
        VStack {
            Text("\(count)")
                .padding()
                .task(id: count) {
                    // this is fired when the view is loaded
                    // and also when the count changes
                    count += 1
                }
            Button("Increment") {
                count += 1
            }
        }
    }
```

At present we are executing the task modifier whenever count changes. But what if we want an array of dependencies. Take a look at the following code: 

``` swift 

@State private var count: Int = 0
@State private var name: String = ""

 var body: some View {
        VStack {
            Text("\(count)")
                .padding()
                .task(id: [count, name]) {
                    // this is fired when the view is loaded
                    // and also when the count changes
                    count += 1
                    //_ = try? await getTodo()
                }
            Button("Increment") {
                count += 1
            }
        }
    }
```

Unfortunately, this is not going to work and it will result in the following error: 
`Cannot convert value of type 'String' to expected element type 'Int'`. The reason is that an array in Swift language cannot contain  objects of different types, unless you create an array with `Any` type. 

There are several ways to fix this issue, one way is to wrap the actual values behind an enum. This is shown in the implementation below: 

``` swift 
enum Dependency: Equatable {
    case count(Int)
    case name(String)
}

@State private var count: Int = 0
@State private var name: String = "Hello "

   var body: some View {
        VStack {
            Text("\(count) - \(name)")
                .padding()
                .task(id: [Dependency.count(count), Dependency.name(name)]) {
                    // this is fired when the view is loaded
                    // and also when the count changes
                    
                    print("TASK CLOSURE IS FIRED")
                }
            Button("Increment") {
                count += 1
            }
            
            Button("Change Name") {
                name += name
            }
        }
    }

```

Now, when you change either the count or the name it will trigger the task closure. Task dependencies can be really useful if you want to perform the task based on the state changes. But keep in mind to use it with caution. If you are not careful, you can introduce an infinite loop.  

### Conclusion

In this post, you learned about the new task modifier in iOS 15. It is a great place to make an initial and future calls based on the dependencies. Go ahead download Xcode 13 and try it out. 

<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>