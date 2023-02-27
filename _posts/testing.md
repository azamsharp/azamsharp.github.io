# Building Large-Scale Apps with SwiftUI: A Guide to Modular Architecture

In one of my previous articles, I mentioned that how I am using MV Pattern for building my client/server SwiftUI applications. 

## Modular Architecture 

## Screens vs Views 

When I was working with Flutter, I observed a common pattern for organizing the widgets. Flutter developers were separating the widgets based on whether the widgets represents an entire screen of just a reusable control. For example

Since React, Flutter and SwiftUI are extremely similar in nature we can apply the same principles when building SwiftUI applications. 

For example when displaying details of a movie, instead of calling that view MovieDetailView, you can call it MovieDetailScreen. This will make it clear that the detail view is an actual screen and not some reusable child view. Here are few more examples. 

**Screens** 
- MovieDetailView 
- HomeScreen 
- LoginScreen
- RegisterScreen 
- SettingsScreen 

**Views** 
- RatingsView 
- MessageView 
- ReminderListView
- ReminderCellView 

I find that it is always a good idea to keep a close eye on our friendly neighbors React and Flutter. You never know what ideas you will bring from other declarative frameworks into SwiftUI. 

## Understanding MV Pattern 

The main idea behind the MV Pattern is to allow views directly talk to the model. This eliminate the need for creating unnecessary layer of view models for each view, which simply contribute to the size of the project instead of providing any benefits.  

> In SwiftUI, view is similar to Component in React or Widget in Flutter. This means, it is not only used for displaying data but can also handle binding capabilities. **The View in SwiftUI is also a View Model.**. Keep in mind that if you plan to put all the logic in your views then check out Container Pattern. MV Pattern is not the same as Container Pattern. 

Apple has shown MV pattern in several places with different flavors. This includes Fruta, FoodTruck and ScrumDinger applications. My focus for this article is on client/server applications as they are one of the most common types of iOS applications. 

In WWDC 2020 talk titled "Data Essentials in SwiftUI" Apple presented the following diagram. 

![ObservableObject as the data dependency surface](/images/single-source.png)

The main idea is to provide view access to a single layer that serves as source of truth and allows access to all the entities in the application.

Apple demonstrated this approach in their talk titled
**"Use Xcode for server-side development"**. The screenshot from the talk is shown below. 

![Use Xcode for server-side development](/images/xcode-server.png)

[Use Xcode for server-side development](https://developer.apple.com/videos/play/wwdc2022/110360/)

Apart from the WWDC video, Apple also demonstrated this technique in their Fruta and FoodTruck applications, although in their sample apps they did not use a network layer.  

Here is the updated diagram to supporting networking layer.  

![Aggregate Root](/images/aggregate-model-updated.001.jpeg)

> I know what you are thinking. Are we going to take advice based on Apple's code samples blindly? No! Never take any advice blindly. Always invest time and research the advantages and disadvantages of each approach. I have evaluated many different techniques and patterns and found this to be the best and simplest option when building client/server apps using SwiftUI. **Do your research!**  

Based on Apple's recommendation in their WWDC videos and code samples, I have been implementing a single aggregate model, which holds the entire state of the application. For small and medium sized apps, a single aggregate model might be enough. For complicated apps, you can have multiple aggregate models which will group related entities together. Multiple aggregate models are discussed later in the article. 

> Keep in mind that this article is about client/server apps. If you are using Core Data or anything else then you will have to do your research. For purely Core Data apps, I have been experimenting with Active Record Pattern. You can read about it [here](https://azamsharp.com/2023/01/30/active-record-pattern-swiftui-core-data.html). 

The implementation of my aggregate model is shown below: 

``` swift 
class StoreModel: ObservableObject {
    
    private var storeHTTPClient: StoreHTTPClient
    
    init(storeHTTPClient: StoreHTTPClient) {
        self.storeHTTPClient = storeHTTPClient
    }
    
    @Published var products: [Product] = []
    @Published var categories: [Category] = []
    
    func addProduct(_ product: Product) {
        // storeHTTPClient.addProduct(product)
    }
    
    func populateProducts() async throws {
        self.products = try await storeHTTPClient.loadProducts()
    }
}
```

> In Domain-Driven Design (DDD), an aggregate is a cluster of related objects that are treated as a single unit of work for the purpose of data consistency and transactional boundaries. An aggregate model, then, is a representation of an aggregate in code, typically as a class or group of classes.

```StoreModel``` is an aggregate model that centralizes all the data for the application. Views communicate directly with the StoreModel to perform queries and persistence operations. StoreModel also utilizes ```StoreHTTPClient```, which is used to perform network operations. StoreHTTPClient is a stateless network layer. This means it can be used in other parts of the application that are not SwiftUI, meaning UIKit. 

StoreModel can be used in views in a variety of different ways. You can StoreModel as a @StateObject to if you only want the data available to a particular view. But quite often I find myself adding StoreModel to @EnvironmentObject so that it can be available in the injected view and all its sub views.   

``` swift 
@main
struct StoreAppApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(StoreModel(client: StoreHTTPClient()))
            
        }
    }
}
```

Now, you can use the ```StoreModel``` as shown in the implementation below. 

``` swift 
struct ContentView: View {

    @EnvironmentObject private var model: StoreModel
    
    var body: some View {
        ProductListView(products: model.products)
            .task {
                do {
                    try await model.populateProducts()
                } catch {
                    print(error.localizedDescription)
                }
            }
    }
}
```

> You might be tempted to use @EnvironmentObject inside all views. Although, it will work as expected but for larger applications you need to make presentation views independent of any dependencies. Presentation views are usually child views that are created for the purpose of reusability. 

Apart from fetching and persistence, StoreModel can also provide sorting, filtering, searching and other operations etc. 

A single StoreModel is ideal for small or even medium sized apps. But for larger apps it will be a good idea to introduce multiple aggregate models based on the bounded context of the application. In the next section, we will cover how to introduce multiple aggregate root models and .  

## Multiple Aggregate Models 

As you learned in the previous section, the purpose of an aggregate model is to expose data to your view. As Luca explained in [Data Essentials in SwiftUI WWDC 2020 (11:30)](https://developer.apple.com/videos/play/wwdc2020/10040/) "The aggregate model is an ```ObservableObject```, which acts as your data dependency surface. This allows us to model the data using value type and manage its life cycle and side effects with a reference type."  

As your business grows, a single aggregate model might not be enough to maintain the life cycle and side effects of an entire application. This is where we will introduce multiple aggregate models. These aggregate models are based on the bounded context of the application. Bounded context refers to a specific area of the system that has clear boundaries and is designed to serve a particular business purpose or domain. 

In an e-commerce application, we have have several bounded contexts including checkout process, inventory management system, catalog, fulfillment, shipment, ordering, marketing and customer management module. 

Defining bounded context is important in software development and it helps to break down the application into small manageable pieces. This also allows teams to work on different parts of the system without interfering with each other. 

Developers are usually not good in finding bounded context of software applications. The main reason is that their technical knowledge does not directly map into domain knowledge. Domain knowledge requires different set of skills and a domain expert is a better fit for this kind of role. A domain expert is a person, who may not be tech savvy but understands how the business or a particular domain works. In large projects, you may have multiple domain experts each handling a different business domain. This is why it is extremely important for developers to communicate with domain experts and understand the domain before starting any development.  

Once, you have identified different bounded contexts associated with your application you can represent them in the form of aggregate models. This is shown in the diagram below. 

![Multiple Aggregate Root](/images/aggregate-model-updated.002.jpeg)

The network layer can also be divided into multiple HTTP clients or you can use a single generic network layer for your entire application. This is shown in the diagram below.  

![Multiple Aggregate Root](/images/aggregate-model-updated.003.jpeg)

The Catalog aggregate model will be responsible for providing the view with all the entities associated with Catalog. This can include but not limited to: 

- Product 
- Category 
- Brand 
- Review 

The Ordering aggregate model will be responsible for providing view with all the ordering related entities. This can include but not limited to:  

- Order 
- OrderLineItem 
- OrderStatus
- ShippingMethod 
- Discount 

The ```Catalog``` and ```Ordering``` aggregate models are be reference types conforming to ObservableObject. And all the entities they provide will be value types. 

The outline of ```Catalog``` aggregate model and ```Product``` entity is shown below: 

``` swift 

struct Product: Codable {
    let productId: Int
    let name: String
    let category: Category
    let price: Double
    let description: String
    let reviews: [Review]?
}

@MainActor 
class Catalog: ObservableObject {
    
    // designated or generic HTTP client 
    let storeHTTPClient: StoreHTTPClient
    
    @Published var products: [Product]
    @Published var categories: [Category]
    
    init(storeHTTPClient: StoreHTTPClient) {
        self.storeHTTPClient = storeHTTPClient
    }
    
    func loadProducts() {
         products = storeHTTPClient.loadProducts
    }
    
    func getProductById(_ productId: Int) -> Product? {
        // fetch product by id 
    }
    
    func getProductsByCategory(_ categoryId: Int) -> [Product] {
       // get products by category
    }
    
    func getCategories() -> [Category] {
        categories = storeHTTPClient.loadCategories()
    }
}
```

Catalog and Ordering aggregate models are injected into the application as an environment object. You can inject them directly in your application root view or the root view of each bounded context. The later is shown below: 

``` swift 
@main
struct StoreAppApp: App {
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(Catalog(client: CatalogHTTPClient()))
                .environmentObject(Ordering(client: OrderingHTTPClient()))
            
        }
    }
}
```

Now, inside a view you can access Catalog or Order by accessing it through ```@EnvironmentObject```. This is shown below: 

``` swift 
struct CatalogListScreen: View {
    
    @EnvironmentObject private var catalog: Catalog
    
    var body: some View {
        List(catalog.products) { product in
            Text(product.name)
        }.task {
            do {
                try await catalog.loadProducts()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

If your CatalogListScreen needs to access Order information then it can utilize the Ordering aggregate model. 

``` swift 
struct AdminDashboardScreen: View {
    
    @EnvironmentObject private var catalog: Catalog
    @EnvironmentObject private var ordering: Ordering
    
    var body: some View {
        VStack {
            List(catalog.products) { product in
                Text(product.name)
            }
            List(ordering.allOrders) { order in
                Text(order.status)
            }
        }.task {
            do {
                try await catalog.loadProducts()
                try await ordering.loadOrders()
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

## Testing 

> This section of the article is taken from my post [Pragmatic Testing and Avoiding Common Pitfalls](https://azamsharp.com/2012/12/23/pragmatic-unit-testing.html)

The main purpose of writing tests is to make sure that the software works as expected. Tests also gives you confidence that a change you make in one module is not going to break stuff in the same or other modules. 

Not all applications requires writing tests. If you are building a basic application with a straight forward domain then you can test the complete app using manual testing. Having said that in most professional environments, you are working with a complicated domain with business rules. These business rules form the basis on which company operates and generates revenue. 

In this article, I will discuss different techniques of writing tests and how a developer can write good tests to get the most return on their investment. 

## Not all tests are created equal 

Consider a scenario that you are writing an application for a bank. One of the business rules is to charge overdraft fees in case of insufficient funds. Banks generate [billions of dollars income by just fees](https://www.depositaccounts.com/blog/banks-income-fees.html) alone. As a developer, you must write good quality tests to make sure that overdraft fee calculation works as expected.

In the same bank app, you may have features like rendering templates for emails or logging certain interactions. These features are important but may not produce the same return on investment as compared to charging overdraft fees. This means if the email template is not in the correct format then the banks are not going to loose millions of dollars and you will not receive a call in the middle of the night. If the logging is meant for developers then in most cases you don't even need to write tests for it. It is just an implementation detail. 

>> If you are building a logging framework then it is essential that you thoroughly test the public API exposed by your framework.  

Next time you are writing a test, ask yourself how important this feature is for the business. If it is an integral part of the business then make sure to test it thoroughly and go for high code coverage.  

## Test behavior not implementation 

One of the biggest mistakes developers make is to focus on writing tests against the implementation details instead of the behavior of the application.

>> A trigger to add a new test is the requirement, not a class or a function. 

Just because you added a new class or a function does not mean that you will start writing tests. That is just an implementation detail which can change overtime. Your tests should target the business requirements and not the implementation details. 

Here are few examples of behaviors, derived from business requirements: 

1. When a customer withdraw amount and has insufficient funds then charge an overdraft fee.  
2. The number of stocks specified by customer are submitted for trade at a specified price, once the limit has reached. 

The behavior stems from the requirement of the project. Tests that checks the implementation details instead of the behavior tends to be very brittle and can easily break when the implementation changes even though the behavior remains the same.  

Let's consider a scenario, where you are building an application to display a list of products on the screen. The products are fetched from a JSON API and rendered using SwiftUI framework, following the principles of MVVM design pattern.

>> I personally don't use MVVM pattern when implementing SwiftUI application. I have written a lot about it and you can read my article [SwiftUI Architecture - A Complete Guide to MV Pattern Approach](https://azamsharp.com/2022/10/06/practical-mv-pattern-crud.html)

First we will look at a common way of testing the above scenario that is adopted by most developers and then later we will implement tests in a more **pragmatic** way. 

The complete app might look like the implementation below: 

```swift 
class Webservice {
    
    func fetchProducts() async throws -> [Product] {
        // ignore the hard-coded URL. We can inject the URL from using test configuration. 
        let url = URL(string: "https://api.escuelajs.co/api/v1/products")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Product].self, from: data)
    }
    
}

class ProductListViewModel: ObservableObject {
    
    @Published var products: [ProductViewModel] = []
    
    func populateProducts() async {
        do {
            let products = try await Webservice().fetchProducts()
            self.products = products.map(ProductViewModel.init)
        } catch {
            print(error)
        }
    }
    
}

struct ProductViewModel: Identifiable {
    
    private let product: Product
    
    init(product: Product) {
        self.product = product
    }
    
    var id: Int {
        product.id
    }
    
    var title: String {
        product.title
    }
}


struct ProductListScreen: View {
    
    @StateObject private var vm = ProductListViewModel()
    
    var body: some View {
        List(vm.products) { product in
            Text(product.title)
        }.task {
            await vm.populateProducts()
        }
    }
}
```

The above application works as expected and produces the expected result. Instead of testing the concrete implementation of the ```Webservice```, we will introduce an interface/contract/protocol just so that we can inject a mock. The sole purpose of creating the protocol is to satisfy the tests, even though there is only one concrete implementation that conforms to that protocol/interface.  

>> This is called [**Test Induced Damage**](https://dhh.dk/2014/test-induced-design-damage.html). The tests are dictating that we should add dependencies so you can mock out the service. The only purpose of introducing a protocol/contract/interface is so you can eventually mock it. Keep in mind there is nothing wrong with using protocols/contracts in your application. They do serve a very important purpose to hide the implementation details from the user and providing abstraction, but just to add contracts to satisfy testing goals in not a good practice as it complicates the implementation and your tests are directed away from testing the actual behavior of the app. 

In the code below we have introduced a WebserviceProtocol. Both Webservice and the newly created MockedWebservice conforms to the WebserviceProtocol as shown below: 

```swift 
protocol WebserviceProtocol {
    func fetchProducts() async throws -> [Product]
}

class Webservice: WebserviceProtocol {
    
    func fetchProducts() async throws -> [Product] {
        
        let url = URL(string: "https://api.escuelajs.co/api/v1/products")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Product].self, from: data)
    }
}

class MockedWebService: WebserviceProtocol {
    func fetchProducts() async throws -> [Product] {
        return [Product(id: 1, title: "Product 1"), Product(id: 2, title: "Product 2")]
    }
}
```

>> You should probably use a better name, instead of calling it WebserviceProtocol. The main reason, I am calling it WebserviceProtocol is just for the sake of simplicity and convenience.   

The webservice is now injected as a dependency to our ProductListViewModel. This is shown below: 

```swift 
class ProductListViewModel: ObservableObject {
    
    private let webservice: WebserviceProtocol
    @Published var products: [ProductViewModel] = []
    
    init(webservice: WebserviceProtocol) {
        self.webservice = webservice
    }
    
    func populateProducts() async {
        do {
            let products = try await Webservice().fetchProducts()
            self.products = products.map(ProductViewModel.init)
        } catch {
            print(error)
        }
    }
    
}
```

The view, ProductListScreen is also updated to reflect the change. 

```swift 
struct ProductListScreen: View {
    
    @StateObject private var vm = ProductListViewModel(webservice: WebserviceFactory.create())
    
    var body: some View {
        List(vm.products) { product in
            Text(product.title)
        }.task {
            await vm.populateProducts()
        }
    }
}
```

>> WebserviceFactory is responsible for either returning the Webservice or MockedWebservice, depending on the application environment. 

Now, let's go ahead and check out the test. 

```swift 
final class ProductsTests: XCTestCase {
    
    func test_populate_products() async throws {
        
        let mockedWebService = MockedWebService()
        let productListVM = ProductListViewModel(webservice: mockedWebService)
        
        await productListVM.populateProducts()
        
        // This line is verifying the implementation detail.
        // Implementation details can change
        // fetchProducts can change to getProducts and the test will fail. 
        verify(mockedWebService.fetchProducts()).wasCalled()
        
        XCTAssertEqual(2, productListVM.products.count)
    }
}
```

We created an instance of ```MockedWebservice``` inside our test and pass it to the ```ProductListViewModel```. Next, we invoke the ```populateProducts``` function on the view model and then check to make sure that the ```fetchProducts``` on the mockedWebservice instance was called. Finally, the test checks the products property of the ````ProductListViewModel``` instance to make sure that is is populated correctly. 

I have seen hundreds of these kind of tests implemented in large projects. There is a lot of things wrong with the above test. First, we are testing the view model. There is no need to test a view model through a unit test, since it does not contain any logic or behavior. There are no business rules implemented in view models. 

>> You will have a much better return on your investment, if you write an end to end test for your view models instead of unit testing them.   

Another problem with the above test is that it is not testing the behavior but the implementation. The following line of code is an implementation detail. 

```swift
verify(mockedWebService.fetchProducts()).wasCalled()
```

This means if you decide to refactor your code and rename the function ```fetchProducts``` to ```getProducts``` then your test will fail. These kind of tests are often known as brittle tests as they break when the internal implementation changes even though the functionality/behavior provided by the API remains the same. This is also the main reason that your test should validate the behavior instead of the implementation.  

>> The code that you write is a liability, including tests. When writing tests, focus on the quality of the tests instead of the quantity. Remember, you are not only responsible for writing tests but also maintaining them. 

In the next section, you will learn how to write tests that validates the behavior of the application instead of the implementation details. 

## End to End Testing 

In the previous section, you learned that testing view models and mocking does not provide the return on your investment. Tests written for view models that use mocking validates the implementation details instead of the behavior. Implementation can change due to refactoring, breaking all the dependent tests even though the behavior remained the same. 

Human psychology also plays an important role when writing tests. As software developers we want fast feedback with small amounts of dopamine hit along the way. There is nothing wrong with receiving fast feedback. Fast feedback is one of the important characteristics of a unit test. Unfortunately, sometimes we are going too fast to realize that we were on the wrong path. We start behaving like a test addict, who wants to see green checkmarks alongside the tests instantly.  

As explained earlier testing view models may add more tests to your test suite but does not provide any benefit. It may even work against you in the long run since now you will be responsible for maintaining those test cases and anytime the implementation detail changes, all your test will break even though the functionality remained the same. 

In these scenarios, end to end testing is a much better choice. A good end to end will test one complete story/behavior. Below you can find the implementation of an end to end test. 

```swift 
final class ProductTests: XCTestCase {
    
    private var webservice: Webservice!
    // products
    let products = [Product(id: 1, title: "Handmade Fresh Table"),Product(id: 2, title: "Durable Water Bottle")]
    
    override func setUp() {
        // make sure the Webservice is using the TEST server endpoints and not PRODUCTION
        webservice = Webservice()
        
        // add few products // seeding the database
        for product in products {
            await webservice.addProduct(product: product)
        }
    }
    
    func test_display_list_of_all_products() async {
        
        let app = XCUIApplication()
        app.launch()
        
        let productList = app.tables["productList"]
        
        // check if the item numbers is correct
        XCTAssertEqual(productList.tables.cells.count, 2)
        
        // check if the correct items are displayed
        for(index, product) in products.enumerated() {
            let cell = productList.cells.element(boundBy: index)
            XCTAssertEqual(cell.staticTexts["productTitle"].label, product.title)
        }
        
    }
    
    override func tearDown() async throws {
        // make sure to delete ALL records from the database so future test results are not influenced
        await webservice.deleteProductById(productId: 1)
        await webservice.deleteProductById(productId: 2)
    }
    
}
```
  
>> Developers can run E2E tests locally on their development machine. This will require initial setup such as testing framework, test environment, dependencies (database, services). E2E tests can be time-consuming, as a result developers may choose to run E2E tests less frequently than unit tests or other types of tests.  

E2E tests are slower than the previous tests discussed earlier in the section but the main reason they are slower is because they tests all layers of the application. E2E tests are complete test and targets a particular behavior of the app. 

End to end tests also requires some initial setup that will allow your test to run database migrations, insert seed data, simulate user interface events and then rolling back changes after the tests are completed. 

End to end tests are NOT replacement of your domain model tests. You MUST write tests against your domain models, specially if your app is domain heavy and consists of lots of business rules. 

>> You will have to find the right balance as to how often to run end to end tests. If you run it with each code check-in then your continuous integration server will always be running 100% of the time. If you run it once every couple of days then you will be notified of failures much later than expected. Keep in mind that you can run E2E tests locally on your machine. Once you get the desired outcome, the CI server can run all the tests during the code check-in process.   

## What about Integration Testing 

Integration tests are performed to make sure that two different systems can work together. These systems can be external dependencies like database or API but it can also be different modules within the same system. 

Dependencies can be classified as managed and unmanaged dependencies. A managed dependency includes database, file systems etc. For managed dependencies, it is important that you use real instance and not a mock. Unmanaged dependencies include SMTP server, payment gateway etc. For unmanaged dependencies use mocks to verify their behavior. 

Let's check out a sample integration test for a network service for user login operation. 

```swift 
// This test is generated by ChatGPT AI 
import XCTest

class IntegrationTests: XCTestCase {
    func testLogin() {
        // Set up the URL for the login endpoint
        let url = URL(string: "https://api.example.com/login")!

        // Create a URL request
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")

        // Set the body of the request to a JSON object with the login credentials
        let body = ["username": "user123", "password": "password"]
        request.httpBody = try! JSONSerialization.data(withJSONObject: body)

        // Create a URLSession and send the request
        let session = URLSession.shared
        let task = session.dataTask(with: request) { data, response, error in
            // Make sure there is no error
            XCTAssertNil(error)

            // Check the response status code
            let httpResponse = response as! HTTPURLResponse
            XCTAssertEqual(httpResponse.statusCode, 200)

            // Check the response data
            XCTAssertNotNil(data)
            let responseBody = try! JSONSerialization.jsonObject(with: data!, options: []) as! [String: Any]
            XCTAssertEqual(responseBody["status"], "success")
        }
        task.resume()
    }
}

```

The above integration test makes sure that the HTTP client layer is working as expected. The integration is between the network client and the server. The client is making sure that the response is correct and valid for a successful login operation.  

Unmanaged dependencies like payment gateway, SMTP clients etc can be mocked out during integration tests. For managed dependencies, use the concrete implementations. 

## Code Coverage 

Code coverage is a metric that calculates how much of your code is covered under test. Let's take a very simple example. In the code below we have a ```BankAccount``` class, which consists of ```deposit``` and ```withdraw``` functions. 

>> Keep in mind that in real world scenario, a bank account is not implemented as a calculator. A bank account is recorded in a ledger, where all financial transactions are persisted. 

```swift 
class BankAccount {
    
    private(set) var balance: Double
    
    init(balance: Double) {
        self.balance = balance
    }
    
    func deposit(_ amount: Double) {
        self.balance += amount
    }
    
    func withdraw(_ amount: Double) {
        self.balance -= amount
    }
    
}
```

One possible test for the BankAccount may check if the account is successfully deposited. 

```swift 
final class BankAccountTests: XCTestCase {
    
    func test_deposit_amount() {
        
        let bankAccount = BankAccount(balance: 0)
        bankAccount.deposit(100)
        XCTAssertEqual(100, bankAccount.balance)
        
    }
}
```

If this is the only test we have in our test suite then our code coverage is not 100%. This means not all paths/functions are under test. This is true because we never implemented the test for ```withraw``` function. 

You may be wondering that should you always have 100% code coverage. The simple answer is NO. But it also depends on the apps that you are working on. If you are writing code for NASA, where it will be responsible for landing rover on Mars then you better make sure that every single line is tested and your code coverage is 100%. 

If you are implementing an app for a pace maker device that helps to regulate the heartbeat then you better make sure that your code coverage is 100%. One line of missed and untested code can result in someones life... literally. 

So, what is the ideal code coverage number. It really depends on the app but any number above 70% is considered a decent code coverage. 

>> When calculating code coverage make sure to ignore the third party libraries/frameworks as their code coverage is not your responsibility. 

## Unit Testing, Data Access and File Access 

Most developers that I have talked to believe that a unit test cannot access a database or a file system. **That is incorrect and plain wrong**. A unit test CAN access a database or a file system. 

It is very important to understand that ```Unit test is the isolation, not the thing under test```. This is so important that I am going to repeat it again. 

>> Unit test is the isolation, not the thing under test 

One of the valid reasons of not accessing a database or a file system during unit tests is that a test may leave data behind which may cause other tests to behave in unexpected manner. The solution is to make sure that the database is always reverted to an initial state after each test is completed so that future tests gets a clean database without any side effects.  

Some frameworks also allows you to construct in-memory databases. Core Data for instance uses SQLite by default but it can be configured to use an in-memory database as shown below: 

```swift 
storeDescription.type = NSInMemoryStoreType
```

In-memory database provides several benefits including: 

- No removal of test data 
- Run faster 
- Can be initialized before each test run 

Even thought these benefits looks appealing, I personally do not recommend using in-memory database for testing purposes. The main reason is that in-memory databases does not represent an actual production environment. This means you may not encounter the same issues during tests, which you may witness when using an actual database. 

>> It is always a good idea to to make sure that your test environment and production environment are nearly identical in nature. 

## Testing View Model Does NOT Validate the User Interface  

Couple of weeks ago, I was having a discussion with another developer, who was mentioning that they test their **user interface** through View Models in SwiftUI. I was not sure what he meant so I checked the source code and found that they had lot of unit tests for their View Models and they were just assuming that if the View Model tests are passing then the user interface will automatically work.

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

## The Ideal test 

We talked about several different types of tests. You may be wondering what is the best kind of test to write. What is the ideal test? 

Unfortunately, there is no ideal test. It all depends on your project and requirements. If your project is domain heavy then you should have more domain level tests. If your project is UI heavy then your should have end to end tests. Finally, if your project integrates with managed and unmanaged dependencies then integration tests will be more suitable in those scenarios.

Remember to test the public API exposed by the module and not the implementation details. This way you can write useful quality tests, which will also help you to catch errors.  

Don't create protocols/interfaces/contracts with the sole purpose of mocking. If a protocol consists of a single concrete implementation then use the concrete implementation and remove the interface/contract. Your architecture should be based on current business needs and not on what if scenarios that may never happen. Remember YAGNI (You aren't going to need it). Less code is better than more code. 


Caching 

Navigation 

Events 

Conclusion 

