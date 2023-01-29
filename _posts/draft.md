# Active Record Pattern for Building SwiftUI Apps with Core Data  

Lately, I have been talking a lot about different architectural patterns that can be used to build SwiftUI applications. I discussed [MV pattern](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html) for building client/server applications and [Container pattern](https://azamsharp.com/2023/01/24/introduction-to-container-pattern.html), when testing is not high priority.  

Recently, I have been building Core Data applications and I wanted to try out Active Record Pattern and see how it feels. In this post, I will cover my experience with building SwiftUI apps using Active Record Pattern.

## What is Active Record Pattern? 

The Active Record pattern is a design pattern used in software development for connecting a database to an object-oriented programming language. It is a pattern in which an object that represents a database table or view is essentially a direct mapping to a single row of the table or view. Active Record objects typically have methods for retrieving and manipulating the data stored in the database, and they can also include validation logic for enforcing business rules on the data. The Active Record pattern is often used in web application development frameworks such as Ruby on Rails and Laravel.

Diagram of active record pattern 

## Why Active Record Pattern?

Active Record Pattern is ideal for building apps that exhibit an ORM (Object Relational Mapping) behavior. iOS apps that uses Core Data reflects that behavior and are good candidates to use the Active Record Pattern. 

I have past experience with several different ORM frameworks including NHibernate, Light Speed, Entity Framework, Sequelize etc and I think Core Data will fit nicely with the Active Record Pattern.  

>> Core Data is not an ORM, it is a framework that provides an object-oriented way to manage data in a persistent store, such as a SQLite database. 

In this article, we will look at how to implement a simple Reminders App using SwiftUI and Core Data. The Reminders App will allow the user to add a list as well as add reminders to an existing list. 

## Implementation

The first thing we need to implement is a common base class or a protocol that will contain all the common functions, properties used by all our Core Data models. 

The implementation is shown below: 

```swift 
import Foundation
import CoreData

protocol Model {
   func save() throws
}

extension Model where Self: NSManagedObject {
   
    func save() throws {
        try CoreDataProvider.shared.viewContext.save()
    }
    
    func delete() throws {
        CoreDataProvider.shared.viewContext.delete(self)
        try save() 
    }
    
    static var all: NSFetchRequest<Self> {
        let request = NSFetchRequest<Self>(entityName: String(describing: self))
        request.sortDescriptors = []
        return request
    }
}
```




