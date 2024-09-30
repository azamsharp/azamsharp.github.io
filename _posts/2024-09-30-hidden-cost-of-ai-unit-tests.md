
# The Hidden Cost of AI-Generated Unit Tests: Sacrificing Domain Knowledge

> The article's main point isn't about mistrusting AI-generated code but about the missed opportunity to learn domain rules. Manually writing domain tests forces developers to engage with the business logic, deepening their understanding, which is essential for building robust software.

Over the past 20 years, I have worked with a diverse range of large organizations across various industries, including retail, healthcare, oil and gas, and energy. The biggest challenge I encountered wasn't the programming languages, technologies, or platforms—it was the domain knowledge. To build high-quality software, it's crucial to fully grasp the business rules and understand the intricate processes that drive operations behind the scenes.

The second-best way to gain domain understanding is through writing unit tests. Unit tests force you to break down the business rules into smaller, manageable components, helping you to clarify and validate the core logic while ensuring your software aligns with the domain's specific requirements.

> I still believe that the most effective way to learn a new domain is to engage with a domain expert, ideally in a casual setting like over a meal. Preparing your questions ahead of time allows you to make the most of the conversation, asking as many insightful questions as possible while your expert is relaxed and enjoying good food. Oh, and don’t forget—you’re picking up the bill! 😉

In this post, I’ll explain that while leveraging AI to write your domain-level unit tests can help you move faster, it also causes you to miss a crucial opportunity to learn and deeply understand the domain. Writing these tests manually forces you to engage with the business rules, gaining valuable insights that are essential for building effective and accurate software.

### Domain Unit Tests 

The domain is the core of any application, housing all the critical business rules and logic. It dictates how the application functions and ensures alignment with the real-world operations and processes of the business, making it essential for delivering software that meets business objectives.

You can still be an effective developer without deep domain knowledge, but gaining that understanding can increase your value exponentially. By mastering the domain, you become more than just a coder—you become a key contributor to the success of the business, as you can better design solutions that align with the business’s unique needs and goals. This makes you a more valuable asset to any team or organization.

As I mentioned before, one of the best way to understand an **unfamiliar** domain is to write unit tests for it. Let's take an example of a user story from medical insurance industry. 

#### User Story: Managing Complex Insurance Plan Coverage
Title: Process Patient Claim Based on Specific Insurance Plan Rules

As a claims processing system,
I want to accurately apply the specific rules of each patient's insurance plan during 
claim adjudication,
so that I can correctly calculate coverage, co-pays, deductibles, co-insurance, and 
patient responsibility.

A developer working on this large insurance project may have little to no experience with the medical insurance domain. As they navigate the project, they might encounter the following code:

``` swift 
struct InsurancePlan {
    let deductible: Double
    let coInsurance: Double
    let coPayForGP: Double?
    let outOfPocketMax: Double
    let exclusions: [String]
    var paidTowardsDeductible: Double = 0
    var totalOutOfPocket: Double = 0
    
    func isServiceExcluded(_ service: String) -> Bool {
        return exclusions.contains(service)
    }
    
    mutating func processVisit(for visit: Visit) -> (insurancePays: Double, patientPays: Double) {
        var patientResponsibility = 0.0
        var insurancePayment = 0.0
        
        // If this is a GP visit and there's a co-pay
        if visit.isGeneralPractitioner, let coPay = coPayForGP {
            patientResponsibility += coPay
            insurancePayment = max(0, visit.cost - coPay)
            return (insurancePayment, patientResponsibility)
        }
        
        // Deductible logic
        let remainingDeductible = deductible - paidTowardsDeductible
        
        if remainingDeductible > 0 {
            if visit.cost > remainingDeductible {
                // Apply the rest of the deductible
                patientResponsibility += remainingDeductible
                paidTowardsDeductible = deductible
                
                // Apply co-insurance after deductible is met
                let remainingCostAfterDeductible = visit.cost - remainingDeductible
                patientResponsibility += remainingCostAfterDeductible * coInsurance
                insurancePayment = remainingCostAfterDeductible * (1 - coInsurance)
            } else {
                // Entire cost goes toward the deductible
                patientResponsibility += visit.cost
                paidTowardsDeductible += visit.cost
            }
        } else {
            // Deductible has already been met, apply co-insurance directly
            patientResponsibility += visit.cost * coInsurance
            insurancePayment = visit.cost * (1 - coInsurance)
        }
        
        // Check if out-of-pocket max is exceeded
        totalOutOfPocket += patientResponsibility
        if totalOutOfPocket > outOfPocketMax {
            let overage = totalOutOfPocket - outOfPocketMax
            patientResponsibility -= overage
            totalOutOfPocket = outOfPocketMax
            insurancePayment += overage
        }
        
        return (insurancePayment, patientResponsibility)
    }
}

```

This code is critical because it handles the financial processing of a patient’s visit. Accurate processing is essential to ensure that patients are billed correctly, healthcare providers are reimbursed properly, and the system complies with healthcare regulations. Any errors could lead to financial discrepancies, patient dissatisfaction, or even legal issues.

> One way to better understand a function is by providing the code to an AI and having it explain the logic. However, keep in mind that AI makes assumptions based on the visible code and may not fully understand the implementation details of any smaller or dependent functions that aren't provided.

The developer must ensure that this code is well-protected by unit tests. Proper testing guarantees that the complex business rules—such as deductibles, co-pays, and exclusions—are applied accurately, reducing the risk of errors that could impact both patients and healthcare providers. Comprehensive unit tests act as a safety net, ensuring the code behaves as expected in all scenarios.

One approach to writing tests is to leverage AI by feeding all the dependent code into a system like ChatGPT, allowing it to generate test cases automatically. Below are some tests that were implemented by ChatGPT. These tests ensure the correctness of complex business logic, such as applying deductibles, co-pays, exclusions, and out-of-pocket maximums, for various patient scenarios in the medical insurance domain.


``` swift 
import XCTest

class InsurancePlanTests: XCTestCase {

    func testSpecialistVisitWithDeductibleNotMet() {
        var insurancePlan = InsurancePlan(
            deductible: 1000,
            coInsurance: 0.20, // 20%
            coPayForGP: nil,
            outOfPocketMax: 5000,
            exclusions: []
        )
        
        let patient = Patient(name: "Sarah", insurancePlan: insurancePlan)
        let specialistVisit = Visit(cost: 350, isGeneralPractitioner: false, serviceType: "cardiology")
        
        let (insurancePays, patientPays) = insurancePlan.processVisit(for: specialistVisit)
        
        XCTAssertEqual(insurancePays, 120.0, "Insurance should pay $120 for the specialist visit after the deductible")
        XCTAssertEqual(patientPays, 230.0, "Patient should pay $230 towards the deductible and co-insurance")
    }
    
    func testGeneralPractitionerVisitWithCoPay() {
        var insurancePlan = InsurancePlan(
            deductible: 1000,
            coInsurance: 0.20,
            coPayForGP: 40,  // $40 co-pay for GP visits
            outOfPocketMax: 5000,
            exclusions: []
        )
        
        let patient = Patient(name: "John", insurancePlan: insurancePlan)
        let gpVisit = Visit(cost: 100, isGeneralPractitioner: true, serviceType: "general practitioner")
        
        let (insurancePays, patientPays) = insurancePlan.processVisit(for: gpVisit)
        
        XCTAssertEqual(insurancePays, 60.0, "Insurance should pay $60 for a GP visit with a $40 co-pay")
        XCTAssertEqual(patientPays, 40.0, "Patient should pay the $40 co-pay for the GP visit")
    }

    func testSpecialistVisitWhenDeductibleIsMet() {
        var insurancePlan = InsurancePlan(
            deductible: 1000,
            coInsurance: 0.20,
            coPayForGP: nil,
            outOfPocketMax: 5000,
            exclusions: []
        )
        
        // Deductible has been met
        insurancePlan.paidTowardsDeductible = 1000
        
        let specialistVisit = Visit(cost: 500, isGeneralPractitioner: false, serviceType: "cardiology")
        let (insurancePays, patientPays) = insurancePlan.processVisit(for: specialistVisit)
        
        XCTAssertEqual(insurancePays, 400.0, "Insurance should pay 80% ($400) after the deductible is met")
        XCTAssertEqual(patientPays, 100.0, "Patient should pay 20% ($100) after the deductible is met")
    }

    func testVisitThatExceedsOutOfPocketMax() {
        var insurancePlan = InsurancePlan(
            deductible: 1000,
            coInsurance: 0.20,
            coPayForGP: nil,
            outOfPocketMax: 2000,
            exclusions: []
        )
        
        // Total out-of-pocket already near the limit
        insurancePlan.totalOutOfPocket = 1950
        
        let specialistVisit = Visit(cost: 200, isGeneralPractitioner: false, serviceType: "cardiology")
        let (insurancePays, patientPays) = insurancePlan.processVisit(for: specialistVisit)
        
        XCTAssertEqual(insurancePays, 150.0, "Insurance should cover the rest of the out-of-pocket limit ($150)")
        XCTAssertEqual(patientPays, 50.0, "Patient should only pay $50 to reach the out-of-pocket maximum")
    }

    func testExclusionForPhysicalTherapy() {
        var insurancePlan = InsurancePlan(
            deductible: 1000,
            coInsurance: 0.20,
            coPayForGP: nil,
            outOfPocketMax: 5000,
            exclusions: ["physical therapy"]
        )
        
        let physicalTherapyVisit = Visit(cost: 300, isGeneralPractitioner: false, serviceType: "physical therapy")
        
        XCTAssertTrue(insurancePlan.isServiceExcluded(physicalTherapyVisit.serviceType), "Physical therapy should be excluded from coverage")
    }

    func testSpecialistVisitWithPartialDeductible() {
        var insurancePlan = InsurancePlan(
            deductible: 1000,
            coInsurance: 0.20, // 20%
            coPayForGP: nil,
            outOfPocketMax: 5000,
            exclusions: []
        )
        
        // Deductible partially met
        insurancePlan.paidTowardsDeductible = 600
        
        let specialistVisit = Visit(cost: 350, isGeneralPractitioner: false, serviceType: "cardiology")
        let (insurancePays, patientPays) = insurancePlan.processVisit(for: specialistVisit)
        
        XCTAssertEqual(insurancePays, 120.0, "Insurance should pay $120 for the specialist visit")
        XCTAssertEqual(patientPays, 230.0, "Patient should pay $230 (remaining deductible and co-insurance)")
    }
}

```

However, by allowing ChatGPT or AI to write the unit tests, the developer may miss a crucial opportunity to deeply engage with and learn the domain. Writing these tests manually forces the developer to break down and understand the business rules, ensuring they fully grasp how the system operates within the context of healthcare. This hands-on process is essential for building domain expertise, which ultimately makes the developer more valuable and effective in implementing and maintaining the software.

You may argue that AI-generated unit tests provide a great starting point for the developer, offering a quick way to ensure the code works as expected. However, I would counter that it’s not the starting point but rather the ending point. Once the developer sees the reassuring green checkmark next to each test, they are unlikely to revisit the code with the same depth of understanding. By relying solely on AI-generated tests, the developer risks missing the opportunity to truly engage with the domain and refine their understanding of the business rules that drive the software. This lack of involvement can result in a superficial grasp of the domain, reducing the developer's ability to identify deeper issues or **improve and modify the system over time**.

I'm not saying all developers are like that—of course not! Many developers strive to understand the domain thoroughly. However, sometimes circumstances beyond their control, like a fast-approaching deadline, force them to prioritize speed over depth. In such situations, the convenience of AI-generated unit tests can be tempting.

> The best way to avoid donuts on your way home is to take a different route altogether. By changing your path, you're not relying solely on willpower but instead removing the temptation entirely, making it easier to stick to your goal.

Using AI for testing can be highly beneficial in certain scenarios where it enhances productivity and complements the developer’s workflow. Here are some situations when using AI for testing is particularly useful:

### 1. Automating Repetitive Test Creation
AI is excellent at automating the creation of repetitive tests, especially in cases where multiple similar methods need to be tested with various inputs. This frees up the developer to focus on more complex, higher-value tasks, like designing tests for edge cases or performance issues.

### 2. Exploring Edge Cases
AI can help discover edge cases that might be overlooked by humans. By generating a wide range of test inputs, AI can test the system with unusual or extreme cases, potentially identifying bugs that would otherwise go unnoticed.

### 3. Complementing Manual Testing Efforts
AI-generated tests can serve as a complement to manually written tests. While the developer focuses on writing domain-specific, behavior-driven tests, AI can handle the more mechanical aspects of test generation, ensuring broader test coverage with less effort.

### 4. Providing a Baseline for New Projects
In a new project, AI can generate basic unit tests that provide an immediate safety net for developers. These tests act as a starting point, ensuring initial code coverage while developers gradually add more domain-specific tests as they gain understanding of the business rules.

### Conclusion

While using AI to generate unit tests can speed up development, reduce repetitive work, and help explore edge cases, it also comes with significant trade-offs. The primary concern isn’t about mistrusting the code generated by AI, but about missing the opportunity for developers to truly learn the domain. Manually writing tests forces developers to deeply engage with business rules and better understand the intricate processes that drive the software. This understanding is invaluable in ensuring long-term code quality and creating more robust, domain-aligned solutions. 

Relying solely on AI-generated tests may lead to developers treating it as the end point of their engagement, missing the chance to enhance their expertise and refine their approach. While AI has its place in automating repetitive tasks and providing quick wins, the real value of testing lies in the developer’s active involvement with the domain, which ultimately leads to better software and deeper knowledge. Balancing the use of AI with manual efforts ensures that both speed and understanding are achieved.





