
# Introduction to Microservices  

Microservices allow you to break down your application into smaller pieces. Each piece, known as a microservice is responsible for handling a particular aspect of your application. In this article, you are going to learn about the concepts of microservices.  

---


Monolith applications are self-contained applications. This means that all the layers of the apps, including user interface, domain and data access are combined into a single program. 

Microservices architecture allows the application to be divided into smaller services. Each service will perform a certain task of the application. This means you can have microservices for authentication, fetching ratings, movie details etc. Microservice acts as an independent entity, which means they can be deployed independent of the rest of the system/services. 

Since, microservices are independent it does not matter what technology stack you use to implement it. This provides tremendous flexibility for developers to use the best tools for the job. One service can be using NodeJS with a PostgreSQL backend, while other can be using Vapor with GraphQL and MongoDB database. Although microservices can use any communication medium, REST is most common. 

## Example

In this example we have three separate microservices. 

**Customer Service**: This service is responsible for getting the details about the customer. 

**Credit Score Service**: This service is responsible for fetching the credit score based on the user's id.

**Social Security Service**: This service is responsible for getting social security information based on the user's id. 

Each service is registered with the Gateway API. Gateway API is the main point of contact for the client. The main purpose of Gateway API is to keep track of all the registered services and also provide an access point to the outside world.   

![Microservices](/images/microservices.jpeg)

Each microservice will be an independent deployment and will also maintain an independent database. 

Hopefully, this small post gave you a very basic introduction to the world of microservices. Microservices is an exciting topic and in the future posts we will dive much deeper into the implementation details. 


<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>

