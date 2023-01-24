# Container Pattern in SwiftUI 

>> Updated 01/24/2023 - Added E2E and ViewInspector example

Container pattern is a common pattern used in React community. Since React and SwiftUI are quite similar, this pattern can also be used for building SwiftUI applications. In this article, I will focus on the concepts behind the container pattern and how you can use it to build your SwiftUI applications. 

>> Please keep in mind that I am not recommending any pattern in this article. I am just explaining what Container Pattern is and how it can be used to build SwiftUI applications. I suggest you do your own research and choose the best architecture pattern that fits your project. 


## What is Container Pattern? 

The main idea behind container pattern revolves around two different kind of views namely container and presenter/presentation views. The container view is responsible for fetching the data, sorting, filtering and other operations and then passing it down to presentation view for display. 

>> Container view takes help from the stateless network layer to retrieve the data from the server. 

In other words, container is a smart view and presenter is a dumb view. The only job of the presenter view is to display data on the screen. Data is always passed down from the container view to the presenter view. 

Here is a small example, where the container view uses network layer to fetch the data and then passes it down to the presenter view for display. 

``` swift 
// Container
struct ContentView: View {
    
    @State private var products: [Product] = []
    
    var body: some View {
        ProductListView(products: products)
            .task {
                do {
                    products = try await Webservice().getProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
    }
}

// Presenter
struct ProductListView: View {
    
    let products: [Product]
    
    var body: some View {
        List(products) { product in
            Text(product.title)
        }
    }
}
```

Notice how the presenter view, ```ProductListView``` is only responsible for displaying the data. It has no idea, where and how the data was fetched. A single container view can also have multiple presenter views as nested views. In those cases, container view will be responsible for fetching the data for both the views and then passing it down for display.    

>> Sometimes developers get carried away with Container Pattern and try to introduce containers for every single view. This increases software complexity and adds needs redirection that would have been avoided. Dan Abramov talks about this in his post "[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)". 

## Testing 

One of the biggest complaints about Container Pattern is that it is hard to unit test your application. Since, there are no view models you cannot simply write a test for it to verify the user interface behavior due to an event. 

Having said that writing a unit test for a view model to verify the user interface is not a good practice. These tests are extremely brittle since they are not testing the behavior but the implementation details. Check out my article "[Pragmatic Testing and Avoiding Common Pitfalls](https://azamsharp.com/2012/12/23/pragmatic-unit-testing.html)". 

>> If you have logic in view models then you should definitely write tests for it. This logic can include sorting, filtering operations that can be tested in isolation. Just make sure that you are not using view models to test the interface of your app. For those scenarios you can use manual testing using Xcode previews, view testing using ViewInspector or End-to-End testing. 

>> For E2E testing, you should be more focused on writing tests with longer happy paths and few edge cases rather than testing individual rule etc. Keep in mind that E2E tests takes longer to run but they also test a lot more and are considered best for regression and finding bugs. 

Here is an example of E2E test for checking that the order is placed successfully. Keep in mind that since this is an E2E test, it will invoke an actual server (TEST) to place an order. 

``` swift 
final class When_user_adds_a_new_valid_order: XCTestCase {
    
    override func setUp() {
        // run script to setup database
        // create the tables etc
    }
    
    func test_should_display_new_order_successfully() throws {
       
        let app = XCUIApplication()
        app.launch()

        // click on the Add New Order Button
        app.buttons["addOrderButton"].tap()
        
        // fill out the order
        app.textFields["nameTextField"].tap()
        app.textFields["nameTextField"].typeText("John Doe")
        
        app.textFields["coffeeNameTextField"].tap()
        app.textFields["coffeeNameTextField"].typeText("Hot Coffee")
        
        app.textFields["priceTextField"].tap()
        app.textFields["priceTextField"].typeText("4.50")
        
        app.buttons["placeOrderButton"].tap()
        
        let orderList = app.collectionViews["orderList"]
        
        let nameLabel = orderList.children(matching: .cell).element(boundBy: 0).staticTexts["John Doe"].label
        
        XCTAssertTrue(orderList.cells.count == 1)
        XCTAssertEqual(nameLabel, "John Doe")
        
    }
    
    override func tearDown() {
        // drop or clear the tables for the next test
    }

}
```

You can also use [ViewInspector](https://github.com/nalexn/ViewInspector) to just test the view itself. ViewInspector reminds me of [Widget testing](https://docs.flutter.dev/cookbook/testing/widget/introduction) in Flutter. 

>> ViewInspector example is for article purposes. Please consult actual documentation to learn more about ViewInspector. 

``` swift 
 func test_invalid_state_button_is_disabled() throws {
        let sut = AddNewOrderView().environmentObject(Model())
        let isDisabled = try sut.inspect().find(button: "Done").isDisabled()
        XCTAssertTrue(isDisabled)
    }
    
    func test_form_valid_button_is_enabled() throws {
        let sut = AddNewOrderView().environmentObject(Model())
        try sut.inspect().find(viewWithAccessibilityIdentifier: "nameTextField").textField().setInput("Mary Doe")
        try sut.inspect().find(viewWithAccessibilityIdentifier: "coffeeNameTextField").textField().setInput("Hot Coffee")
        try sut.inspect().find(viewWithAccessibilityIdentifier: "priceTextField").textField().setInput("5.75")
        try sut.inspect().find(viewWithAccessibilityIdentifier: "coffeeSizePicker").picker().select(value: CoffeeSize.large)
        let isDisabled = try sut.inspect().find(button: "Done").isDisabled()
        XCTAssertFalse(isDisabled)
    } 
```

>> Usually for such a simple/trivial scenario I just use Xcode Previews to manually test the validation rules instead of writing any unit test. If the scenario is complicated like If I have to filter orders based on several different options then I would unit test that algorithm.  

## References: 

1. [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) 
2. [Exploring SwiftUI Architectures](https://youtu.be/HEDSVXyq2fw)
3. [SwiftUI Architecture - A Complete Guide to MV Approach](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html)
4. [Why I don't do MVVM Anymore](https://youtu.be/kWEyKdZ4G38)

## Conclusion: 

Depending on the complexity of the app, you can use Container Pattern to build SwiftUI applications. As always, evaluate the needs for your application and then choose the best architecture. There is no one size fits all solution. Research and choose what works best for you.  




