# Convenience Property Wrappers vs Custom Data Access Layer in SwiftUI

Yesterday, I had the opportunity to speak at [WomenWhoCode Mobile](https://www.womenwhocode.com/network/mobile/) event. It was a remote event and well attended. I spoke about SwiftUI architecture best practices. 

When I was covering Core Data I mentioned that you should use ```@FetchRequest``` property wrappers as they are optimized to work with SwiftUI. Same is true for ```@Query``` property wrapper in SwiftData. 

During this time an interesting question was raised. An attendee asked what if you want to change the data access layer in the future. Currently our views are tightly coupled with either Core Data or SwiftData but what happens if we want to use Realm or GRDB. 

I have thought about this question for a very long time. This is one of those questions, which does not have a straight forward answer or a single answer to be exact. The answer is opinion based so let's take a look at both sides of the equation. I will use SwiftData in my examples.  

### Property Wrappers

Consider a scenario that we are building a simple TodoList application. We add our model ```TodoItem```, which consists of a single property ```title```. We add a ```TodoListView```, which uses the ```@Query``` property wrapper to fetch and also monitor the changes in the model context of the application. In just a few lines of code we are able to display todo items on our screen. The code is listed below: 

``` swift 
@Model
class TodoItem {
    
    var title: String
    
    init(title: String) {
        self.title = title
    }
}

struct TodoItemView: View {
    
    let todoItem: TodoItem
    
    var body: some View {
        Text(todoItem.title)
    }
}

struct TodoListView: View {
    
    @Query private var todoItems: [TodoItem]
    
    var body: some View {
        List(todoItems) { todoItem in
           TodoItemView(todoItem: todoItem)
        }
    }
    
}
```

But what if few weeks later you decided that you want to replace SwiftData with Realm. And we are also assuming that you are interested in using Realm SwiftUI property wrappers in your application. This means all those ```@Query``` property wrappers will not work. Realm uses ```@ObservedResults``` property wrapper instead of ```@Query``` property wrapper. Also Realm models are not decorated with ```@Model``` macro so you need to remove that too. 

> If your intention is to utilize Realm, it's crucial to note that all of your models will necessitate updating, irrespective of the approach you choose.

The ```TodoItemView``` also needs to be updated as Realm uses ```@ObservedRealmObject``` or similar instead of ```[TodoItem]```. 

Ultimately, it becomes apparent that altering the data persistence framework will have a ripple effect throughout a significant portion of our application.

### Custom Data Access Layer 

Now, let's see the what happens when we implement our own custom data access layer. 

We will start with implementing the protocol. 

``` swift
protocol TodoServiceProtocol {
    func saveTodoItem(_ todoItem: TodoItem)
    func getTodoItems() throws -> [TodoItem]
}
```

Next, we will implement the concrete implementation of our data access service: 

``` swift
class SwiftDataTodoService: TodoServiceProtocol {
    
    private var context: ModelContext
    
    init(context: ModelContext) {
        self.context = context
    }
    
    func saveTodoItem(_ todoItem: TodoItem) {
        context.insert(todoItem)
    }
    
    func getTodoItems() throws -> [TodoItem] {
        return try context.fetch(FetchDescriptor<TodoItem>())
    }
}
```

And finally, you can use the ```SwiftDataTodoService``` in your application. 

``` swift
struct ContentView: View {
    
    private var service: TodoServiceProtocol
    @State private var todoItems: [TodoItem] = []
    
    init(service: TodoServiceProtocol) {
        self.service = service
    }
    
    private func loadTodoItems() {
        // get all the items
        do {
            todoItems = try service.getTodoItems()
        } catch {
            print(error.localizedDescription)
        }
    }
    
    var body: some View {
        VStack {
            Button("Insert Todo Item") {
                service.saveTodoItem(TodoItem(title: "Feed the Rabbit"))
                // load the todo items from db
                loadTodoItems()
            }
            
            List(todoItems) { todoItem in
                Text(todoItem.title)
            }.task {
                loadTodoItems()
            }
        }
        .padding()
    }
}
```
This is definitely a lot more code as compared to our previous approach. We pass the ```TodoService``` as a dependency to our ContentView and then uses the ```saveTodoItem``` and ```getTodoItems``` functions to perform appropriate actions. By not using the built-in property wrappers like ```@FetchRequest``` or ```@Query``` we lost the ability of tracking changes but gained the flexibility of swapping out the data access layers when and if needed. 

### Final Words 

When determining which option is superior, the answer is **it depends**. If convenience is your priority, then SwiftUI property wrappers are recommended. However, if you value flexibility to accommodate future changes, implementing a custom data access layer would be more suitable.

I pose a question for you to contemplate: When was the last instance in which you entirely swapped out your data access layers?