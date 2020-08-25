# Integrating Core Data with SwiftUI Part 1

Core Data is an object graph 


---

### Architecture 

The declarative nature of SwiftUI allows it to blend nicely with MVVM Design Pattern. 

### Getting Started 

Create a brand new **Single View Application** and make sure to select **SwiftUI** as the **User Interface** and check the **Use Core Data** checkbox. This will setup basic Core Data setup for your application.  

![Core Data SwiftUI Template](images/core-data-1.png)

If you open the **AppDelegate.swift** file you will see the setup for Core Data.  

Apart from creating the persistentContainer, Xcode will also create an empty data model file. The name of the file will be based on the Xcode project. This means for our app, the data model file will be called **BlogApp.xdatamodeld**. In the next section we will add entities to our Core Data model. 

> Core Data stores the object graph in SQLite database by default but it can be changed to use any other persistent medium. 

### Adding Entities 

Entities define the data persisted to the database. Entities can be standalone or it can also have relationship with other entities. Currently, we are going to only look at standalone entities. 

Click on the **Add Entity** button to add an entity to the data model. Change the name of entity to **Post**. In the right pane you can add attributes/properties to an entity. 

![Core Data Adding Entity](images/core-data-img-2.png)

Apart from adding attributes to an entity, also make sure that your class **Codegen** option is set to **Class Definition**. This means that Xcode will automatically create a class associated with your entity. 

> When you are using the code generation option then make sure that you do not modify the main implementation of the class manually. The reason is that it will be overridden on the next code generation execution. If you do want to add additional behavior to the entity class then consider writing an extension.   

In the next section we will start implementing our CoreDataManager, which will allow us to interact with our database through managed object context. 

### Core Data Manager

The main purpose of **Core Data Manager** is to provide a layer of abstraction between the **View Model** and the **View**.

> SwiftUI consists of a @FetchRequest property wrapper, which can be used directly from the view to access the database through Core Data. Unfortunately, this creates a very tight coupling between the model and view, prevents reusability and creating maintenance issues down the road. 

We will start by creating a single instance of CoreDataManager using the Singleton design pattern. CoreDataManager will have a dependency on NSManagedObjectContext, which will be injected in the initializer.  

```swift
class CoreDataManager {
    
    static let shared = CoreDataManager(moc: NSManagedObjectContext.current)
    
    var moc: NSManagedObjectContext
    
    private init(moc: NSManagedObjectContext) {
        self.moc = moc
    }
}
```

The NSManagedObjectContext.current is a custom extension, which is responsible for returning an instance of NSManagedObjectContext.   

``` swift
extension NSManagedObjectContext {
    
    static var current: NSManagedObjectContext {
        let appDelegate = UIApplication.shared.delegate as! AppDelegate
        return appDelegate.persistentContainer.viewContext
    }
    
}
```

Next, we will create the getAllPosts function in CoreDataManager which will allow us to retrieve all the posts from the database. 

``` swift
 func getAllPosts() -> [Post] {
        
        var posts = [Post]()
        let postRequest: NSFetchRequest<Post> = Post.fetchRequest()
        
        do {
            posts = try self.moc.fetch(postRequest)
        } catch let error as NSError {
            print(error)
        }
        
        return posts
    }
```

Inside the **getAllPosts** function, we create an instance of **NSFetchRequest** and then perform the request on an instance of managed object context. In the end we get an array of posts, which we later returned from the getAllPosts function. 

> At this time, no posts will be returned because database is currently empty. We will manually add some posts from the terminal using SQL commands. 

Next, we need to create our View Models which will use the CoreDataManager to get the posts and then return it to the view. 

### Implementing ViewModels








