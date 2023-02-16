# Testing Is About Confidence 

If you ask 100 people what testing means to them, they will give you 100 different answers. For me personally, testing is all about confidence. Confidence that my code works and confidence that it will work as expected in the future. 

Confidence is a relative term and developer's confidence increases with experience. Over the years, I have become selective about writing tests. 

>> I am not advocating that you should not write tests. Testing is extremely important and you must write good quality tests for your application. But remember that bad tests are even worse than having no tests. 

Let's take a look at an example. 

In this example below, we have a **LoginView** with some basic validation of username and password fields. 

``` swift 
struct LoginView: View {
    
    @State private var username: String = ""
    @State private var password: String = ""
    
    private var isFormValid: Bool {
        !username.isEmptyOrWhiteSpace && !password.isEmptyOrWhiteSpace
    }
    
    var body: some View {
        
        VStack {
            Form {
                TextField("User name", text: $username)
                TextField("Password", text: $password)
                Button("Login") {
                    
                }.disabled(!isFormValid)
            }
        }
    }
}
```

>> I don't use MVVM pattern for building SwiftUI applications. You can read my detail post [here](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html). 

So, how do you unit test the above code? 

Personally, I would not invest time in writing a unit test for the above code. The main reason is that for me this code is extremely trivial and I can easily and quickly test it using Xcode previews.

>> Testing views using Xcode previews basically means manual testing. Having said that Xcode previews provides a fast feedback cycle, which can be very useful for testing simple user interfaces. 

But I want to write unit tests, what are my options? 

There are few options to unit test the validation for your views. You can extract out the form inputs and the validation into a separate ```struct``` and then write tests for the ```struct```. This is shown in the listing below: 

``` swift 
struct LoginFormConfig {
    
    var username: String = ""
    var password: String = ""
    
    var isFormValid: Bool {
        !username.isEmptyOrWhiteSpace && !password.isEmptyOrWhiteSpace
    }
}
```

The LoginView can now utilize the new LoginFormConfig ```struct``` as shown below: 

``` swift 
struct LoginView: View {
    
    @State private var loginFormConfig = LoginFormConfig()
    
    var body: some View {
        
        VStack {
            Form {
                TextField("User name", text: $loginFormConfig.username)
                TextField("Password", text: $loginFormConfig.password)
                Button("Login") {
                    
                }.disabled(!loginFormConfig.isFormValid)
            }
        }
    }
}
```

Now, you can write a unit test for LoginFormConfig. 

``` swift 
final class LearnTests: XCTestCase {

    func test_login_form_validates_successfully() {
        
        let expectedOutputs: [[String: Any]] = [
            ["username": "johndoe", "password": "password", "isFormValid": true],
            ["username": "", "password": "password", "isFormValid": false],
            ["username": "johndoe", "password": " ", "isFormValid": false],
            ["username": "", "password": " ", "isFormValid": false],
            ["username": "   ", "password": "password", "isFormValid": false],
            ["username": " johndoe", "password": " password", "isFormValid": true]
        ]
        
        for expectedOutput in expectedOutputs {
            let username = expectedOutput["username"] as! String
            let password = expectedOutput["password"] as! String
            let isFormValid = expectedOutput["isFormValid"] as! Bool
            
            let loginFormConfig = LoginFormConfig(username: username, password: password)
            XCTAssertEqual(loginFormConfig.isFormValid, isFormValid)
        }
    }
}
```

In the above test, I have created an ```expectedOutputs``` variable which holds all the different usernames, passwords and the validation results. I use a ```for``` loop to go through all the expectations and verify it through ```LoginFormConfig```.    

>> Hopefully in future, Apple will introduce property wrappers to create parameterized unit test functions. 

Apart from manual testing and unit testing you can also write an end-to-end test for a particular behavior. 

>> End-to-end tests provides the best defense against regression. E2E tests are slow because they test all layers of the application. My advice is to use end-to-end tests to cover couple of longer happy paths and few edge cases per story.  

In one of the apps I was working on, I had to to display list of hotels based on a keyword matching the hotel's name. I created a local JSON file and populated it with list of hotels from the JSON response. I placed the file in project's "Preview Content" folder so it is only used for previews and not for release builds. I quickly implemented the algorithm and tested it using Xcode previews. 

For this scenario, I did not write any tests. My main reason was that I was quite familiar with the implementation since I had already written similar code for previous projects. 

Later, the client introduced new requirements. These requirements includes searching the hotels based on name, wifi, parking, meals and even price. This is definitely more complicated than the original story. There are too many variables that can affect the outcome and I am certainly not confident to implement it without the support of tests. This time I implemented the tests and made sure all the different variations are working as expected. 

As you can see that for me personally, testing is all about confidence. When I am feeling confident about a certain task, I only use manual testing using Xcode previews. When the task feels complicated and I am not confident, I take help from tests. 

>> Implementing tests for application flow automation is another good scenario where testing can be really helpful.  

This reasoning has allowed my test code to not spiral out of control. I value my code based on the business requirements. Not all code is same. Some code is more important than others and is responsible for driving the business aka generating revenue. That code needs to be tested thoroughly. 

>> Next time you are writing a test, validate the situation and evaluate its benefits to the business. 

**This post described how I evaluate and write tests. This may not reflect your needs and criteria in your application. Talk to your team and choose the best path based on your needs.**
