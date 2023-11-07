

# The Complete Guide to JSON Web Tokens (JWT) Authentication in iOS 

Authentication plays a pivotal role in all applications. In the context of client/server applications, especially when JSON is the predominant mode of communication, JWT authentication is widely recognized as a fundamental industry standard.


> In this article, I've employed SwiftUI for the client and ExpressJS for the server. If you're an iOS developer, you might be curious about my choice to not opt for Vapor and stay within the iOS/Swift ecosystem. The reason is my extensive experience and comfort with ExpressJS. However, if you prefer to use Vapor, it's also an excellent option. You also might be interested in checking out my iOS Vapor course [Mastering Full Stack iOS Development Using SwiftUI and Vapor](https://www.udemy.com/course/full-stack-ios-development-using-swiftui-and-vapor/?referralCode=5573DBDE821F2E6A80E8). 

## User Registration (Server)  

As mentioned previously, server is implemented using ExpressJS framework. PostgreSQL is used as the database using Sequelize as an ORM (Object Relational Mapping) framework. Server implementation follows the MVC (Model View Controller) design pattern. Some of the main components used on the server side are defined below: 

- **Routers**: Manages the routes for the application, represented by the endpoints used in the application.  
- **Controllers**: Handles the request from the routers and communicate with the data access layer to perform operations on the models.  
- **Models**: Defines the entities used in the application (User, Movie, Recipe) etc.   
- **Middlewares**: Middleware are pieces of code that is executed before the request reaches the action. We will be using middleware to perform authentication on certain routes/endpoints.


Before a user can be authenticated, we need to make sure that a user can successfully register. In order to register a new user, a valid email and password needs to be provided. User registration is handled by the ```authController```. The implementation is shown below: 

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

The ```register``` action uses [express-validator](https://express-validator.github.io/docs/) to validate the incoming data. Once the body is validated we check if the email is unique or not. If email is already registered then ```409``` status code is returned. If the email is unique then we hash the password and persist it in the database. Password is hashed using the package called [bcryptjs](https://www.npmjs.com/package/bcryptjs). 

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

> It is important that you make sure that your register action is working as expected. This can be performed writing test on the server or integration tests on the client. If you are not writing tests then you can manually confirm the expected response by using a networking tool like [Postman](https://www.postman.com/). 


### User Registration (Client)

After validating the functionality of our registration endpoint, our next step will be to shift our focus to the client side. Rather than creating distinct network functions for various requests, such as login, registration, fetching courses etc we'll streamline our approach by introducing a JSON HTTPClient. This HTTPClient wll take on the responsibility of handling network requests and delivering the model data to the caller. You can find the implementation details of the HTTPClient below:

``` swift 
struct HTTPClient {
    
    static let shared = HTTPClient()
    private let session: URLSession
    
    private init() {
        let configuration = URLSessionConfiguration.default
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
        
        guard let _ = response as? HTTPURLResponse else {
                throw NetworkError.invalidResponse
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

You might be contemplating breaking down the ```load``` function into smaller sub-functions, perhaps with the expectation of reusing these components across your codebase. However, I'd recommend abstaining from such abstraction at this point. The primary rationale behind this recommendation is that the logic within the "load" function is presently exclusive to that function alone. There is currently no demand or necessity to employ isolated segments of the "load" function independently. If such a requirement arises in the future, we can always extract and modularize specific functions based on the existing "load" function's implementation.   

> Code that goes together, stays together 


There are various ways of invoking HTTPClient layer from the view. You can expose HTTPClient as an environment value and directly access it from within the view or you can introduce an ObservableObject like ```Account```. 

Each approach has its own benefits. Environment value approach is much simpler and allows you to directly accessing the network layer in the view. This is ideal of scenarios, where the Web API is read-only and does not allow the view to mutate the data. 

> In SwiftUI, view is the view model. If you want to read more about SwiftUI architecture then check out my article "Building Large Apps Using SwiftUI". 

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

```Account``` has a dependency on HTTPClient. One thing to note is that we are not using an abstraction for HTTPClient but using the concrete implementation directly. The main reason is that HTTPClient is a managed dependency. Managed dependencies are dependencies that are completely in your control. They are not accessible by the outside world. This includes filesystem, databases etc.  

When writing tests for behaviors that interact with managed dependencies, it is recommended to use the real implementation instead of mocks. Mocks should be used for un-managed dependencies. A common example of un-managed dependency is Payment Gateway. 

> One important thing to keep in mind when using concrete implementations is to always clean up the data after the test. This means that if you inserted a user into a database during a test then make sure to remove the user after the test is completed. This is usually performed in a ```teardown``` function in ```XCTest``` framework. 


```Account``` is injected as an environment object to the root view of the application. The implementation is shown below: 

``` swift 
@main
struct ExamPrepApp: App {
    
    @State private var account = Account(httpClient: HTTPClient())
    
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

One thing to note about our interaction with the server is that the server is returning the messages to the client. These messages indicate the reasoning about the failure that occurred on the server.  

If you are interested in learning how to build a navigation system in SwiftUI and handling messages globally then check out the following resources. 

- [Navigation](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#navigation)
- [Displaying Errors (Technique #2)](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#displaying-errors)

> If you are interested in learning how to build a navigation system in SwiftUI then check [this](https://azamsharp.com/2023/02/28/building-large-scale-apps-swiftui.html#navigation) article. 

After successful registration, user will be redirected to the login screen. At this point we need to go back to the server and implement the login endpoint. 

### Login (Server)

The login process for JWT authentication is composed for the following steps. 

1. Validate parameters passed in the request body. 
2. Fetch user based on username or email. 
3. Compare password with the stored hashed password in the database. 
4. If successful, create a JSON Web Token and sign it with userId and provide an expiration date. 
5. Return the response with the token to the user. 

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

> It is important that you don't store JWT private key directly in the code itself. One of the reasons is that you don't want your private key to be visible on source control systems like GitHub. A better approach is to use environment variables through ```.env``` file and making sure that it is ignored by adding it to the ```.gitignore``` file. 

Once again it is important that you test your ```login``` action. You can write  tests for your controllers, you can also write integration tests between your client (SwiftUI) and the server (ExpressJS). When writing integration tests, it is important to test longest happy paths and edge cases. 

> Longest happy path means successful execution of a business scenario. 

Next step is to implement the client. This will include invoking the ```login``` action to authenticate the user. 

### Login (Client)

The first step is to implement the DTO object that will map the response from the server. Apart from ```success``` and ```message``` properties, server is also returning ```token```, which needs to be persisted on the client side. The implementation of ```LoginResponse``` is shown below: 

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

The most important part of the ```login``` function is the use of Keychain to store the token. Keychain provides a secure storage option designed for storing sensitive data, such as passwords, tokens, or cryptographic keys. It's encrypted and protected, making it suitable for confidential information.

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

The above ```getCourses``` function is a protected resource. One of the requirement to access the resource is that a user must be authenticated. One option is to implement the authentication logic right inside the ```getCourses``` function. But we will need the authentication logic in other actions too, so it is a good idea to implement this feature in the middleware. By putting it in the middleware we can easily reuse it for other routes. 

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

We start by accessing the authorization header from the request. Once the authorization header is retrieved, we extract it to access the token. The token is verified using the verify function (verify function is part of the jsonwebtoken package). Once, the token is verified we assign the extracted userId to the request. This makes sure that the userId is available to the future requests. 

After implementing the ```authMiddleware``` we need to register it for the routes we need to protect. This is implemented in the ```app.js``` file below: 

``` js
app.use('/api/faculty', authenticate, facultyRouter)
app.use('/api/students', authenticate, studentRouter)
``` 

> Routes like ```login``` and ```register``` do not need to be protected. You cannot add a precondition to login to be already logged in. 

In the next section, we will move our attention to the client side and learn how to extract the token and make it available to the protected requests. 

### Accessing Protected Routes (Client)

