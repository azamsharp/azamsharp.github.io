

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

After validating the functionality of our registration endpoint, our next step will be to shift our focus to the client side. Rather than creating distinct network functions for various requests, such as login, registration, and fetching courses, we'll streamline our approach by introducing a JSON HTTPClient. This HTTPClient will take on the responsibility of handling network requests and delivering the model data to the caller. You can find the implementation details of the HTTPClient below:

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

You might be contemplating breaking down the "load" function into smaller sub-functions, perhaps with the expectation of reusing these components across your codebase. However, I'd recommend abstaining from such abstraction at this point. The primary rationale behind this recommendation is that the logic within the "load" function is presently exclusive to that function alone. There is currently no demand or necessity to employ isolated segments of the "load" function independently. If such a requirement arises in the future, we can always extract and modularize specific functions based on the existing "load" function's implementation.   

> Code that goes together, stays together 




