

# The Complete Guide to JSON Web Tokens (JWT) Authentication in iOS 

Authentication is a cornerstone of application development, safeguarding user interactions in both client and server applications. In a world where JSON reigns as the primary data exchange medium, JSON Web Token (JWT) authentication has emerged as an industry standard, ensuring data security and integrity.

Within this comprehensive guide, we embark on a journey through the intricacies of setting up JWT authentication on the server and seamlessly implementing this security measure on the client side using SwiftUI. This knowledge equips you with the essential skills needed to fortify your application's authentication mechanisms, ensuring reliability and robustness in today's ever-evolving digital landscape.

In this article, the power of SwiftUI is harnessed for the client interface, while ExpressJS serves as the backbone for server-side operations. While this choice may pique the curiosity of iOS developers, it's grounded in my extensive experience with and confidence in ExpressJS, a robust and dependable tool for server development.

However, it's crucial to note that Vapor remains a compelling alternative, offering seamless integration with Swift and the iOS ecosystem. For those keen on exploring Vapor's capabilities further, my iOS Vapor course, [Mastering Full Stack iOS Development Using SwiftUI and Vapor](https://www.udemy.com/course/full-stack-ios-development-using-swiftui-and-vapor/?referralCode=5573DBDE821F2E6A80E8), provides a valuable resource. In this course, I delve deep into JWT authentication, providing iOS developers with a comprehensive learning experience to navigate the intricacies of the Vapor framework.

## User Registration (Server)  

As previously mentioned, the server is constructed using the ExpressJS framework. For data storage, PostgreSQL is employed, accompanied by Sequelize, an ORM (Object Relational Mapping) framework. The server's architecture adheres to the MVC (Model View Controller) design pattern, ensuring a structured and organized approach to application development. Below, we outline some of the primary components utilized on the server side:

- **Routers**: These components efficiently manage the application's routes, representing the various endpoints through which the application interacts.  
- **Controllers**: Controllers are responsible for managing requests originating from the routers. They act as intermediaries, facilitating communication with the data access layer to execute operations on the application's models. 
- **Models**: Models define the fundamental entities that the application revolves around, such as User, Movie, Recipe, and others. They serve as the blueprint for the data structures used within the application.
- **Middlewares**: Middleware components are integral parts of the application's execution flow. They execute specific code logic before a request reaches its intended action. In our case, middlewares play a crucial role in enforcing authentication measures on select routes and endpoints, ensuring the security of the application.

Before enabling user authentication, it's imperative to ensure that users can efficiently complete the registration process. To register a new user, the user must supply a valid email and password. The responsibility for managing user registration is delegated to the `authController`. This component is instrumental in overseeing the user registration process and its associated functionalities.

The implementation is shown below: 

```javascript 
exports.register = async (req, res) => {

    const { email, password } = req.body

    const errors = validationResult(req)
    if (!errors.isEmpty()) {
        res.status(400).json({ success: false, message: errors.array().map(error => error.msg).join(' ') })
        return
    }

    // check if the user is already registered 
    const user = await models.User.findOne({
        where: {
            email: email
        }
    })

    if (user) {
        // already exists 
        res.status(409).json({ success: false, message: 'Email is already registered.' })
        return
    }

    try {

        // hash the password 
        const hashedPassword = await bcrypt.hash(password, 10)

        // register new user 
        const newUser = await models.User.create({
            email: email,
            password: hashedPassword 
        })

        // 201 for created and send the new user back 
        res.status(201).json({ success: true })

    } catch (error) {
        console.error(error)
        res.status(500).json({ success: false, message: 'Internal server error' })
    }
}

```

The validation for the body is handled by the validate function which is defined as follows: 

```js 
exports.validate = (method) => {

    switch (method) {
        case 'register': {
            return [
                body('email', 'Email cannot be empty.').exists().isEmail(),
                body('password', 'Password must be at least 6 characters long').exists().isLength({ min: 6 }), 
                body('roleId', 'Role must be provided.').exists() 
            ]
        }
    }
}
```

The ```register``` action uses [express-validator](https://express-validator.github.io/docs/) to validate the incoming data. Once the body is validated we check if the email is unique or not. If email is already registered then ```409``` status code is returned. If email is unique then we hash the password and persist it in the database. Password is hashed using the package called [bcryptjs](https://www.npmjs.com/package/bcryptjs). 

The ```authController``` needs to be registered by the router. This will ensure that all the designated requests are forwarded to the ```authController```. This registration is performed inside the ```index``` router. The implementation is shown below: 

```js 
const express = require('express')
const router = express.Router() 
const authController = require('../controllers/authController') 

router.post('/register', authController.validate('register'), authController.register)

module.exports = router 
```

And finally, ```index``` router needs to be set as a middleware in the main ```app.js``` file. 

``` js 
const express = require('express')
const app = express() 
const cors = require('cors')
const indexRouter = require('./routes/index') 

app.use(cors())
app.use(express.json())

app.use('/api', indexRouter)

app.listen(8080, () => {
    console.log('Server is running...')
})
```

Our user registration action is completed. Now we need to focus on the SwiftUI client to perform registration. 

> It is important that you make sure that your register action is working as expected. This can be performed writing test on the server or integration tests on the client. At minimum use [Postman](https://www.postman.com/) to test your endpoints. 


### User Registration (Client)

After validating the functionality of our registration endpoint, our next step will be to shift our focus to the client side. Rather than creating distinct network functions for various requests, such as login, registration, fetching courses etc we'll streamline our approach by introducing a JSON HTTPClient. This HTTPClient wll take on the responsibility of handling network requests and delivering the model data to the caller. You can find the implementation details of the HTTPClient below:

``` swift 
struct HTTPClient {
    
    static let shared = HTTPClient()
    private let session: URLSession
    
    private init() {
        let configuration = URLSessionConfiguration.default
         // add the default header
        configuration.httpAdditionalHeaders = ["Content-Type": "application/json"]
        self.session = URLSession(configuration: configuration)
    }
    
    func load<T: Codable>(_ resource: Resource<T>) async throws -> T {
        
        var request = URLRequest(url: resource.url)
        
        switch resource.method {
            case .get(let queryItems):
                var components = URLComponents(url: resource.url, resolvingAgainstBaseURL: false)
                components?.queryItems = queryItems
                guard let url = components?.url else {
                        throw NetworkError.badRequest
                }
                
                request = URLRequest(url: url)
                
            case .post(let data):
                request.httpMethod = resource.method.name
                request.httpBody = data
            
            case .delete:
                request.httpMethod = resource.method.name
        }
        
        let (data, response) = try await session.data(for: request)
        
        if let httpResponse = response as? HTTPURLResponse {
            
            switch httpResponse.statusCode {
                case 401:
                    throw NetworkError.unauthorized
                // handle other cases here ...
                default: break
            }
            
        }
        
        do {
            let result = try JSONDecoder().decode(resource.modelType, from: data)
            return result
        } catch {
            throw NetworkError.decodingError(error)
        }
    }
}
```

> You can find the complete implementation of the HTTPClient [here](https://gist.github.com/azamsharp/91919badd2c70c75732ad715b1c3e01e). 

You might be contemplating breaking down the ```load``` function into smaller sub-functions, perhaps with the expectation of reusing these components across your codebase. However, I'd recommend abstaining from such abstraction at this point. The primary rationale behind this recommendation is that the logic within the ```load``` function is presently exclusive to that function alone. There is currently no demand or necessity to employ isolated segments of the ```load``` function independently. If such a requirement arises in the future, we can always extract and modularize specific functions based on the existing ```load``` function's implementation.   

> Code that goes together, stays together 

There are various ways of invoking HTTPClient layer from the view. You can expose HTTPClient as an environment value and directly access it from within the view or you can introduce an ObservableObject like ```Account```. 

Each approach has its own benefits. Environment value approach is much simpler and allows you to directly accessing the network layer in the view. This is ideal of scenarios, where the Web API is read-only and does not allow the view to mutate the data. 

> In SwiftUI, view is the view model. If you want to read more about SwiftUI architecture then check out my article [Building Large Apps Using SwiftUI](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html). 

The ObservableObject approach is used in scenarios where you may want to reuse the results from the request in other views and also perform some logical operation on the data.    

For this article, I am going to use the ObservableObject approach as it will allow us to persist state in the future when we are dealing with login and authentication. 

The ```Account``` implementation is shown below: 

``` swift 

@Observable
class Account {
    
    private var httpClient: HTTPClient
    
    init(httpClient: HTTPClient) {
        self.httpClient = httpClient
    }
    
    func register(email: String, password: String) async throws -> RegistrationResponse {
        
        let params = [JSON.Keys.email: email, JSON.Keys.password: password]
        let body = try JSONEncoder().encode(params)
        
        let resource = Resource(url: API.endpointURL(for: .register), method: .post(body), modelType: RegistrationResponse.self)
        let response = try await httpClient.load(resource)
        return response
    }
    
}

```

```Account``` has a dependency on HTTPClient. One thing to note is that we are not using an abstraction for HTTPClient but using the concrete implementation directly. The main reason is that HTTPClient is a managed dependency. Managed dependencies are dependencies that are completely in your control. They are not accessible by the outside world. This includes filesystem, databases etc. When writing tests for behaviors that interact with managed dependencies, it is recommended to use the real implementation instead of mocks. Mocks should be used for un-managed dependencies. Common examples of un-managed dependency are payment gateways, local exchange services etc.   

> One important thing to keep in mind when using concrete implementations is to always clean up the data after the test. This means that if you inserted a user into a database during a test then make sure to remove the user after the test is completed. This is usually performed in a ```teardown``` function in ```XCTest``` framework. 


```Account``` is injected as an environment object to the root view of the application. The implementation is shown below: 

``` swift 
@main
struct ExamPrepApp: App {
    
    @State private var account = Account(httpClient: HTTPClient.shared)
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                ContentView()
                    .environment(account)
            }
        }
    }
}
```

> As mentioned before, since HTTPClient is a managed dependency we are using the concrete implementation instead of a mock. 

The view can access the ```Account``` instance using the ```@EnvironmentObject``` property wrapper. Once the view has access to the account, it can use the register function as shown below: 

``` swift 
 private func register() async {
        do {
            let response = try await account.register(email: email, password: password)
            if response.success {
                // navigate to the login screen
                navigate(.login)
            } else {
                message = response.message ?? ""
            }
        } catch {
            message = error.localizedDescription 
        }
    }
```

One thing to note about our interaction with the server is that the server is returning messages to the client. These messages indicate the reasoning about the failure that occurred on the server.  

If you are interested in learning how to build a navigation system in SwiftUI and handling messages globally then check out the following resources. 

- [Navigation](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#navigation)
- [Displaying Errors (Technique #2)](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#displaying-errors)

After successful registration, user will be redirected to the login screen. At this point we need to go back to the server and implement the login endpoint. 

### Login (Server)

One of the most advantageous aspects of becoming proficient in JSON Web Token (JWT) authentication is that once you successfully implement it in one framework and programming language, you'll find it relatively straightforward to apply in any other framework. Personally, I've employed JWT in various contexts, including React, Flutter, and iOS, and discovered that, aside from language-specific syntax, the core principles remain consistent.

The JWT authentication login process comprises the following steps:

1. Validate the parameters provided in the request body.
2. Retrieve the user based on their username or email.
3. Verify the submitted password against the hashed password stored in the database.
4. Upon successful verification, generate a JSON Web Token, sign it with the user's ID, and set an expiration date.
5. Provide the user with the token in the response.

Here is the implementation of the ```login``` route. 

``` swift 
exports.login = async (req, res) => {

    const { email, password } = req.body
     
    // perform validation on email and password  

    // check if the user exists 
    const user = await models.User.findOne({
        where: {
            email: email
        }
    })

    if (user) {
        // check the password 
        let result = await bcrypt.compare(password, user.password)
        if (result) {
            // generate the expiration time = 1 hour 
            const expirationTime = Math.floor(Date.now() / 1000) + 60 * 60 * 24 * 60; // 60 days in seconds.. you can change that :) 
            // generate the jwt token 
            const token = jwt.sign({ userId: user.id, exp: expirationTime }, process.env.JWT_PRIVATE_KEY)
            res.json({ success: true, token: token, exp: expirationTime, roleId: user.roleId })

        } else {
            res.status(400).json({ success: false, message: 'Incorrect password' })
        }

    } else {
        res.status(400).json({ success: false, message: 'User not found' })
        return
    }

}
```

> It's crucial to avoid storing the JWT private key directly within your code. One key reason is to prevent your private key from becoming visible in source control systems such as GitHub. A more secure approach involves using environment variables via a `.env` file and ensuring it remains confidential by adding the file to your `.gitignore` to keep it out of version control.

Once more, it's imperative to thoroughly test your `login` action. You have the option to write tests for your controllers, and you can also create integration tests that encompass both your client (SwiftUI) and your server (ExpressJS). In the context of integration tests, it's critical to assess the longest happy paths and examine edge cases to ensure robust and comprehensive testing.

> Longest happy path means successful execution of a business scenario. 

Next step is to implement the client. This will include invoking the ```login``` action to authenticate the user. 

### Login (Client)

The initial task involves creating a DTO object responsible for mapping the server's response. In addition to the expected properties like "success" and "message," the server also provides a "token" that must be stored on the client side. Below, you can find the implementation of the "LoginResponse" object:

``` swift 
struct LoginResponse: Codable {
    let success: Bool
    let message: String?
    let token: String?
}
```

> ```LoginResponse``` does not contain a property for ```userId```. The main reason is that ```userId``` is already signed into the token itself so server will be able to extract it out when needed.  

The next step is to implement the ```login``` function in out ```Account``` class. 

``` swift 
  func login(email: String, password: String) async throws {
        
        let params = [JSON.Keys.email: email, JSON.Keys.password: password]
        let body = try JSONEncoder().encode(params)
        
        let resource = Resource(url: API.endpointURL(for: .login), method: .post(body), modelType: LoginResponse.self)
        let response = try await httpClient.load(resource)
        
        // Check if the login was successful
        guard response.success, let token = response.token else {
            throw LoginError.loginFailed
        }
        
        // Store the JWT token in the Keychain
        guard Keychain.set(token, forKey: "jwttoken") else {
            throw LoginError.keychainError
        }
        
        isLoggedIn = true
    }
```

The critical aspect of the `login` function lies in its utilization of Keychain for token storage. Keychain offers a highly secure storage solution specifically designed for safeguarding sensitive data, including passwords, tokens, and cryptographic keys. Its encryption and protective measures render it an ideal choice for safeguarding confidential information.

> UserDefaults presents an alternative for storing key/value pairs; however, it's important to note that UserDefaults lacks encryption and should not be employed for persisting sensitive or secure data.

Once issue with the above implementation is that when the user launches the app again, the ```isLoggedIn``` property is set to false even though the user has successfully logged in. We can fix that by setting ```isLoggedIn``` value based on the token stored in keychain. This is implemented below: 

```swift
 init(httpClient: HTTPClient) {
        self.httpClient = httpClient
        // set the isLoggedIn to be the value from the keychain
        isLoggedIn = Keychain<String>.get("jwttoken") != nil
    }
```

Now, when you launch the app from scratch the ```isLoggedIn``` property will be initialized based on the JWT token. 

Now, you can use the ```Account``` class in ```LoginScreen``` as shown below: 

``` swift 
struct LoginScreen: View {
    
    @Environment(Account.self) private var account
    
    @State private var email: String = ""
    @State private var password: String = ""
    
    @State private var authenticating: Bool = false
    
    private func login() async {
        do {
            try await account.login(email: email, password: password)
            // navigate to the dashboard screen
            navigate(.dashboard)
        } catch {
            print(error)
        }
    }
    
    var body: some View {
        Form {
            TextField("Email", text: $email)
                .textInputAutocapitalization(.never)
            SecureField("Password", text: $password)
            Button("Login") {
                authenticating = true
            }.task(id: authenticating) {
                if authenticating {
                    await login()
                    authenticating = false
                }
            }
        }.navigationTitle("Login")
    }
}
```

On successful login, user is taken to the dashboard screen. 

> If you are interested in learning about how to configure navigation in SwiftUI then check out my article [here](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#navigation). 

In the next section, you are going to learn how to create protected routes on the server, which requires the user to be logged in. 

### Implementing Protecting Routes on the Server 

The term protected routes means that a user needs to be authenticated in order to be able to access the requested resources.  

In ```facultyController``` we implement the following function to fetch all the courses offered by the faculty member. 

``` swift 
exports.getCourses = async (req, res) => {
    
    // fetch the courses from the database

    // return courses 
    res.json(courses)
}
```

The "getCourses" function mentioned above serves as a protected resource, and one of the prerequisites for accessing it is user authentication. While one approach is to embed the authentication logic directly within the "getCourses" function, it's advisable to implement this functionality as middleware. This way, we can centralize the authentication logic and make it readily available for use in other actions and routes, enhancing reusability and maintainability.

The ```authenticate``` function is implemented in ```authMiddleware.js``` as shown below: 

``` swift 
// authenticate middleware 
exports.authenticate = async (req, res, next) => {

    const authHeader = req.headers['authorization']
    if(authHeader) {
        // get the token out of the header 
        const token = authHeader.split(' ')[1]
        console.log(token)
        if(token) {
            // decode the token 
            try {
                const decodedToken = jwt.verify(token, process.env.JWT_PRIVATE_KEY)
                const user = await models.User.findByPk(decodedToken.userId)
                if(!user) {
                    res.status(401).json({success: false, message: 'User not found'})
                }
                // store the userId in the request
                req.userId = user.id 
                next() 
            } catch(error) {
                if(error instanceof jwt.JsonWebTokenError) {
                    res.status(401).json({success: false, message: 'Token expired'})
                } else {
                    res.status(500).json({success: false, message: 'Internal server error'})
                }
            }
        } else {
            res.status(401).json({success: false, message: 'Bearer token is missing'})
        }

    } else {
        res.status(401).json({success: false, message: 'Required headers are missing'})
    }
}
```

Our initial step involves obtaining the authorization header from the request. After obtaining the authorization header, we proceed to extract the token contained within it. This token is then subjected to verification using the "verify" function, a component of the "jsonwebtoken" package. Upon successful verification, we assign the extracted userId to the request, ensuring its availability for use in future requests.

> The `jsonwebtoken` package additionally handles token expiration checks. When a token has expired, the server will throw a `jwt.JsonWebTokenError`.

Following the implementation of the `authMiddleware`, the next step is to register it for the routes that require protection. This registration process is carried out in the `app.js` file as shown below:

``` js
app.use('/api/faculty', authenticate, facultyRouter)
app.use('/api/students', authenticate, studentRouter)
``` 

> Routes like ```login``` and ```register``` do not need to be protected. You cannot add a precondition to login to be already logged in. 

In the next section, we will move our attention to the client side and learn how to extract the token and make it available to the protected requests. 

### Accessing Protected Routes (Client)

For a client to access a protected route on the server, it must send the token in the request headers. This operation can be implemented directly in the HTTPClient as shown below: 

``` swift 
struct HTTPClient {

    static let shared = HTTPClient() 
    private let session: URLSession
    
    private init() {
        let configuration = URLSessionConfiguration.default
         // add the default header
        configuration.httpAdditionalHeaders = ["Content-Type": "application/json"]
        // get the token from the Keychain
        let token: String? = Keychain.get("jwttoken")
        
        if let token {
 configuration.httpAdditionalHeaders?["Authorization"] = "Bearer \(token)"        }
        
        self.session = URLSession(configuration: configuration)
    }

}

// load function is implemented here. 

```

When the HTTPClient is initialized, we check the keychain for the persisted JSON web token. If the token is present then we add the ```Authorization``` header to the request. This makes sure that the token is part of every request sent by the client.   

Now, when the user launches the app you can check the status of ```isLoggedIn``` property in the **Account** class and redirect to the user to the correct screen, based on their login status. 

``` swift 
@main
struct ExamPrepApp: App {
    
    @State private var account = Account(httpClient: HTTPClient.shared)
    
    var body: some Scene {
        WindowGroup {
            NavigationStack {
                if account.isLoggedIn {
                    DashboardScreen() 
                } else {
                    LoginScreen()
                        .environment(account)
                }
            }
        }
    }
}
```

In a similar scenario, if the user requests a resource and the token has expired then you can redirect the user to the login screen. This is shown in the implementation below: 

``` swift 
struct DashboardScreen: View {
    
    @Environment(Faculty.self) private var faculty
    
    var body: some View {
        VStack {
            Text("Dashboard")
        }.task {
            do {
                try await faculty.loadCourses()
            } catch NetworkError.unauthorized {
                 navigate(.login)
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

> This article does NOT cover refresh tokens. Maybe this is something that I can add in the future. The primary purpose of refresh tokens in JSON Web Tokens (JWTs) is to extend the lifetime of access tokens while maintaining a secure authentication mechanism. Access tokens, which are typically short-lived, are used to verify a user's identity for specific actions within an application. However, frequent re-authentications due to expiring access tokens can disrupt the user experience and hinder the application's usability.

### Source Code 

You can download the code from [here](https://github.com/azamsharp/ExamPrepCode). 

### Conclusion: 

In conclusion, this comprehensive guide has provided an in-depth understanding of JSON Web Token (JWT) authentication in the context of iOS development. Authentication is a crucial aspect of application security, and JWT is widely recognized as an industry standard for securing client/server applications, especially when JSON is the primary mode of communication.

Throughout this article, you've learned how to set up JSON Web Token authentication on the server and perform authentication from the client side using a SwiftUI application. While the specific tools and frameworks used in this guide are SwiftUI for the client and ExpressJS for the server, the principles discussed can be applied across various platforms.

The server-side implementation covered user registration and login, emphasizing the importance of security measures such as hashing passwords. The server also protected certain routes by implementing middleware for user authentication.

On the client side, you learned how to create an HTTPClient for handling network requests and securely storing and retrieving the JWT token from the Keychain. The client-side code showcased how to interact with the server, handle successful login, and navigate to different views in your SwiftUI application.

Overall, JWT authentication is a valuable skill for developers working on iOS and beyond. Whether you choose to follow the specific tools and practices outlined in this guide or adapt them to your preferred framework, the knowledge gained here can be applied to enhance the security and functionality of your applications.

If you liked this article and want to learn about building Full Stack applications using SwiftUI and Vapor then checkout my course [Mastering Full Stack iOS Development Using SwiftUI and Vapor](https://www.udemy.com/course/full-stack-ios-development-using-swiftui-and-vapor/?referralCode=5573DBDE821F2E6A80E8). 