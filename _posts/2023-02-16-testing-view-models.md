# Testing View Model Does NOT Validate the User Interface 

Couple of weeks ago, I was having a discussion with another developer, who was mentioning that they test their **user interface** through View Models in SwiftUI. I was not sure what he meant so I checked the source code and found that they had lot of unit tests for their View Models and they were just assuming that if the View Model tests are passing then the user interface will automatically work.

In this post, I will cover how writing unit tests for View Models is different then testing the user interface. 

> Please keep in mind that I am not suggesting that you should not write unit tests for your View Models. I am simply saying that your View Model unit tests does not validate that the user interface is working as expected. 

Let's take a very simple example of building a counter application. 


``` swift 
class CounterViewModel: ObservableObject {
    
    @Published var count: Int = 0
    
    func increment() {
        count += 1
    }
}

struct ContentView: View {
    
    @StateObject private var counterVM = CounterViewModel()
    
    var body: some View {
        VStack {
            Text("\(counterVM.count)")
            Button("Increment") {
                counterVM.increment()
            }
        }
    }
}
```

When the increment button is pressed, we call the increment function on the CounterViewModel instance and increment the count. Since count property is decorated with @Published property wrapper, it notifies the view to reevaluate and eventually rerender. 

In order to test that the count is incremented and displayed on the screen, the following unit test was written. 

``` swift 
import XCTest
@testable import Learn

final class LearnTests: XCTestCase {

    func test_user_updated_count() {
        let vm = CounterViewModel()
        vm.increment()
        XCTAssertEqual(1, vm.count)
    }

}
```

This is a perfectly **valid** unit test but it does not verify that the count has been updated and displayed on the screen. Let me repeat it again. **A View Model unit test does not verify that the count is successfully displayed on the screen. This is a unit test not a UI test.**

To prove that a View Model unit test does not verify user interface elements, simply remove the Button view or even the Text view from the ContentView. The unit test will still pass. This can give you false confidence that your interface is working. 

A better way to verify that a user interface is working as expected is to implement a UI test. Take a look at the following implementation. 

``` swift 
final class LearnUITests: XCTestCase {

    func testExample() throws {
        // UI tests must launch the application that they test.
        let app = XCUIApplication()
        app.launch()

        app.buttons["incrementButton"].tap()
        XCTAssertEqual("1", app.staticTexts["countLabel"].label)
    }
}
```

This test will launch the app in a simulator and verify that when the button is pressed, label is updated correctly. 

> Depending on the complexity of the behavior you are testing, you may not even need to write a user interface test. I have found that most of the user interfaces can be tested quickly using Xcode Previews. 

So what is the right balance? How many unit tests should you have for your View Model as compared to UI tests. 

The answer is **it depends**. If you have complicated logic in your View Model then unit test can help. UITest (E2E) tests provide the best defense against regression. For each story, you can write couple of long happy path user interface tests and couple of edge cases. Once again, this really depends on the story and the complexities associated with the story. 

In the end [testing is all about **confidence**](https://azamsharp.com/2023/02/15/testing-is-about-confidence.html). Sometimes you can gain confidence by writing fewer or no tests and other times you have to write more tests to achieve the level of confidence. 

If you like this post and want to support my work then check out my [Udemy courses](https://azamsharp.com/courses). 





