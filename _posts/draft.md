# Building the Complete Registration Flow in SwiftUI 


### The Backend

Nowadays, there are inifinite options to implement your backend. I have personally, implemented backends in ASP.NET, Java SpringBoot, Flask, Vapor, Hummingbird, Node/ExpressJS and several more. I prefer ExpressJS, because I taught Full Stack Web Development for a bootcamp and that is what we covered and that is what I am comfortable with. Instead of showing you the backend implementation, let me show you the register route and what does the request and response looks like: 

``` javascript  
Endpoint: http://localhost:8080/api/auth/register
Request: 
{
    "firstName": "John", 
    "lastName": "Doe", 
    "email": "jdoe@gmail.com", 
    "password": "password", 
    "roleId": 1
}
Response: 
{
    "user": {
        "id": 26,
        "firstName": "John",
        "lastName": "Doe",
        "email": "jdoe@gmail.com",
        "roleId": 1
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjI2LCJpYXQiOjE3ODgwMTAyODEsImV4cCI6MTc4ODAxMTE4MX0.5nNe2UDbz1_Zn4_UxYvJEjnM_ND0znGhiFnHqsB2B5o",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjI2LCJpYXQiOjE3ODgwMTAyODEsImV4cCI6MTc4ODYxNTA4MX0.QodXAZI2Df0NpTKJ4IH_DmicZRJk5We-QGAYM6QSCuA"
}
``` 

As you can see when the user is registered, apart from returning the user's information (minus the password), we also return the accessToken and refreshToken. This allows the user to automatically login, on successful registration. 

If the user is already registered with the same email address then the server will return the following response: 

``` javascript 
{
    "message": "Email is already associated with an account."
}
```

If the user does not provide the required information like first name, last name, email etc then the following error is returned back to the client: 

``` swift 
{
    "errors": [
        {
            "type": "field",
            "value": "",
            "msg": "First name is required",
            "path": "firstName",
            "location": "body"
        },
        {
            "type": "field",
            "value": "",
            "msg": "Last name is required",
            "path": "lastName",
            "location": "body"
        },
        {
            "type": "field",
            "value": "@.cm",
            "msg": "Invalid email",
            "path": "email",
            "location": "body"
        }
    ]
}
```