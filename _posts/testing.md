# Testing Is About Confidence 

If you ask 100 people what testing means to them, they will give you 100 different answers. For me personally, testing is all about confidence. Confidence that my code works and confidence that it will work as expected in the future. 

Confidence is a relative term and a developer's confidence increases with experience. Over the years, I have become very selective about writing tests. 

>> I am not advocating that you should not write tests. Testing is extremely important and you must write good quality tests for your application. 

In the example below, we have a **LoginView** with some basic validation of username and password fields. 

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

Personally, I would not invest the time in writing a unit test for the above code. The main reason is that for me this code is extremely trivial and I can easily and quickly test it using Xcode previews.

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

The LoginView can now utilize the new LoginFormConfig struct as shown below: 

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

In the above test, I have created an expectedOutputs variable which holds all the different usernames, passwords and the validation results.  

>> Hopefully in future, Apple will introduce property wrappers to create parameterized unit test functions. 


















