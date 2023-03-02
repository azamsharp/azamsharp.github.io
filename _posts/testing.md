
## View Specific Logic 

Sometimes, our views will require presentational logic, formatting, sorting, filtering etc. Most of the time, this logic can be directly implemented right inside a view. This is shown in the implementation below: 

``` swift 
struct SalesHistoryView: View {
    @ObservedObject var model: FoodTruckModel
    
    var annualHistoryIsUnlocked: Bool {
        storeController.isEntitled
    }
    
    var hideChartContent: Bool {
        timeframe != .week && !annualHistoryIsUnlocked
    }
     
    var totalSales: Int {
        sales.flatMap(\.entries).reduce(into: 0) { (partialResult, entry) in
            partialResult += entry.sales
        }
    }
```

As you can see the properties ```annualHistoryIsUnlocked```, ```hideChartContent``` and ```totalSales``` are contained inside the view instead of in a separate layer. The view takes the data from the Model and then format it in a way it needs to be displayed. 

> Sorting and filtering depends on your app needs. Sometimes, it makes sense to perform sorting inside the view and other times you may need to put sorting code in a model. I have a video called [SwiftUI List - Searching and Sorting - JSON API](https://youtu.be/hMZdRduyA_4), which shows how to perform sorting and searching inside the view. **Keep in mind that this is not the only solution and it really depends on a particular scenario.**  

To prevent your views from getting massive, you will need to compose them in a correct way. If your view is getting too big then you need to evaluate the responsibilities of the view. If it is doing too much then it is better to divide the view into multiple views. 

Last week, I was watching [React.js: The Documentary](https://youtu.be/8pDqJVdNa44) and one of the things mentioned in the documentary was the idea of separation of concerns. React team was talking about how separation of concerns meant putting JS files, css files and HTML files separate but now with modern architecture, we have to think of separation of concerns in a different way. With React, Flutter and even SwiftUI separation of concern is the behavior exposed by the component. This means a component is a reusable piece, which can be easily plugged into the application. A component will be responsible for UI logic, styling etc. 

> This does not mean, you should start putting actual domain logic in the views :) 

Definitely, check out React.js documentary mentioned above. React and SwiftUI are very similar, so I am sure you will learn new things. 