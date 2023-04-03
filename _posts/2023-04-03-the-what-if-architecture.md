# The What-If Architecture 

Lately, I have been thinking about the "What-If Architecture". Most people commonly refer to it as YAGNI (You Aren't Gonna Need It). I was reading some discussion thread, where a developer was creating a Core Data app using SwiftUI and wanted to display information from the database on the screen.

Developer's implementation was similar to the code below: 

``` swift 
struct HistoryView: View {
    
    @FetchRequest(sortDescriptors: [NSSortDescriptor(key: "dateCreated", ascending: true)]) private var historyItemResults: FetchedResults<HistoryItem>    
    var body: some View {
        
            List(historyItemResults) { historyItem in
                Text(historyItem.question ?? "")
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .contentShape(Rectangle())
                   
            }.frame(maxWidth: .infinity)            
    }
}
```

The code works and does it job but other developers quickly pointed out that this is not a good approach since now the view ```HistoryView``` is tied up with ```@FetchRequest``` and will only work with Core Data. What if in the future, we want to use Realm or What if we want to use a document database like MongoDB instead of relational database. 

The points raised by other developers are valid, or are they? Should we design our app, which fulfills current business requirements (Core Data + SwiftUI), or should we create abstractions to support future changes, which may or may not ever happen? The part "may not ever happen" is really important here. You really have to think about the cost of what if it does happen vs what if it never happens. Unfortunately, this is not easy to calculate since it depends on many factors, including the cost of change, the cost of carry, and the cost of build, etc. Martin Fowler wrote about it [here](https://martinfowler.com/bliki/Yagni.html), which I highly recommend reading.

> In my own personal experience, when we build software on assumptions rather than business requirements, most of the times our assumptions turned out to be false. 

If we spend time creating those abstractions then we are spending  valuable time away from current business requirements. And in most cases when those future requirements become reality then we find out that they were very different from our initial vision. We also tend to restrict ourselves from not using helper functions/wrappers which can make our development easier and provide results faster. This includes ```@FetchRequest```, ```@SectionedFetchRequest``` in Core Data and ```@ObservedResults``` in Realm. 

I came to realize this during my own experience and also read the same experience from other developers. My advice is to focus on current business needs and stop forcing abstractions, unless they are true abstractions. 

> Genuine abstractions are discovered, not invented” (from Unit Testing Principles, Practices, and Patterns by 
[@vkhorikov](https://twitter.com/vkhorikov))









