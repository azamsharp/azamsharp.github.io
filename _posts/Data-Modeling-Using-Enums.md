# Data Modeling Using Enums 

Application domain is considered as building blocks of the software system. The domain represents the connections betweens different entities of the app. The domain is also used to map the real world into our application. 

There are many ways of implementing domain objects. In this article I will show you how to implement models using enums.   

### Struct as Models 

Let's begin with modeling our data as structs and then we will learn how to take the enum route. Consider a scenario, where we are building an application for a conference. Conference domain can have many models, but we are only focusing on the session. A session represents a speaking engagement at a conference. When using a struct we can represent a model using the following implementation: 

``` swift
struct Session {
    let title: String
    let date: Date
    let speaker: String
    let isRecorded: Bool
    let isKeynote: Bool
    let isWorkshop: Bool
    let isJoint: Bool
    let jointSpeakers: [String]
}
```

As you can see the Session model can be used to represent a lot of different kind of sessions. This includes keynote, workshop, joint session etc. If we want to create a recordable keynote session, we may end up writing the following code. 

``` swift
let keynote = Session(title: "WWDC 2021", date: Date(), speaker: "Tim Cook", isRecorded: true, isKeynote: true, isWorkshop: false, isJoint: false, jointSpeakers: [])
```

We had to pass lot of unnecessary arguments to create the keynote session. Not only that but if we had to display sessions based on their type then we will have to use lot of conditional logic to display the correct view for the model. This is shown in the implementation below: 

``` swift
func displayAllSessions(sessions: [Session]) {
    
    for session in sessions {
        if session.isKeynote {
            // custom display mode for keynote
        } else if session.isWorkshop {
            // display workshop
        } else if session.isJoint {
            // display joint sessions with multiple speakers
        } else {
            // normal session 
        }
    }
    
}
```

Let's see how enums can help us better structure our models. 

### Enum as Models



The implementation is shown below: 

``` swift 
enum Session {
    case keynote(title: String, date: Date, speaker: String, isRecorded: Bool)
    case workshop(title: String, date: Date, speaker: String, isRecorded: Bool)
    case joint(title: String, date: Date, speakers: [String])
    case normal(title: String, date: Date, speaker: String)
}
```

Each enum case allows us to create a particular type of session by passing all the required arguments. Now, we can easily create a session as shown below: 

``` swift 
let workshop = Session.workshop(title: "Introduction to Async/Await", date: Date(), speaker: "Mohammad Azam", isRecorded: true)
```

We can pass an array of sessions to displaySessions function and take action based on their type. 

``` swift
func displaySessions(sessions: [Session]) {
    
    for session in sessions {
        switch session {
        case let .keynote(title: title, date: date, speaker: speaker, isRecorded: isRecorded):
            print("keynote")
        case let .workshop(title: title, date: date, speaker: speaker, isRecorded: isRecorded):
            print("workshop")
        // handle cases for joint and normal, I am just going to use default
        default:
            print("session")
        }
    }
    
}
```
> If you don't want to handle all the cases then you can use if case let syntax to handle a particular case. 

This looks much cleaner as compared to our struct implementation, which was polluted with conditional checks. 

### Enums and Polymorphism

Enums also allows you to practice the art of polymorphism. This means that we can pass session to a function and it can be interpreted in a different form. 

First take a look at the structures associated with our sessions. 

``` swift
struct Keynote {
    let title: String
    let speaker: String
    let date: Date
    let isRecorded: Bool
}

struct Joint {
    let title: String
    let speakers: [String]
    let date: Date
}

struct Workshop {
    let title: String
    let speaker: String
    let date: Date
    let isRecorded: Bool
    let duration: Int
}

enum Session {
    case keynote(Keynote)
    case joint(Joint)
    case workshop(Workshop)
}
```

Next, we can create different kind of sessions as shown below: 

``` swift 
let keynote = Session.keynote(Keynote(title: "WWDC 2021", speaker: "Tim Cook", date: Date(), isRecorded: true))
let workshop = Session.workshop(Workshop(title: "Introduction to Async/Await", speaker: "John Doe", date: Date(), isRecorded: true, duration: 45))
```

Even though we have created different kind of sessions like keynote and workshop, in the end they are all Session enums. This means an array of Session enum types can be easily passed to a function, accepting list of sessions. This is shown below: 

``` swift 
func displaySessions(sessions: [Session]) {
    for session in sessions {
        switch session {
            case .keynote(let keynote):
                print(keynote)
            case .workshop(let workshop):
                print(workshop)
            case .joint(let joint):
                print(joint)
        }
    }
}

displaySessions(sessions: [keynote, workshop])
```

### Conclusion 

Enums are available in almost all modern languages, but in Swift they received special attention, making them more powerful. In iOS 15, enums can also be serialized using the codable protocol, giving them the flexibility to be passed across the network.  