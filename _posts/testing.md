# Validation Patterns in SwiftUI 

Validation is a crucial component of app development, ensuring that only accurate and meaningful data is processed by your application. As the saying goes in software development, **Garbage in, garbage out.** If your app allows users to submit invalid data, it will inevitably result in unreliable and flawed outputs.

In this article, we will explore various techniques for implementing robust validation in your SwiftUI forms to enhance data integrity and user experience.

<div style="
    background-color: #f0f8ff;
    border-left: 5px solid #0073e6;
    padding: 20px;
    border-radius: 5px;
    font-family: Arial, sans-serif;
    font-size: 1.1rem;
    color: #333;
    margin: 20px 0;
">
    <strong>Want to become a highly valued iOS developer?</strong> 
    Check out AzamSharp School for comprehensive courses and hands-on learning at 
    <a href="https://azamsharp.school" style="color: #0073e6; text-decoration: none; font-weight: bold;">azamsharp.school</a>.
</div>

### Simple Validation

Validation does not have to be complicated. The main idea is to convey to the users that their input is invalid. The conveying part can be performed in many different ways, including enabling or disabling certain actions.

In the following code, we implemented a Login screen and used basic validation to check if username and password are provided by the user. If they are not  then the Login button will be disabled. 

``` swift 
struct LoginScreen: View {
    
    @State private var username: String = ""
    @State private var password: String = ""
    
    private var isFormValid: Bool {
        return !username.isEmptyOrWhitespace && !password.isEmptyOrWhitespace
    }
    
    var body: some View {
        Form {
            TextField("Username", text: $username)
            TextField("Password", text: $password)
            Button("Login") {
                
            }.disabled(!isFormValid)
        }

    }
}
```

In straightforward scenarios like this, error messages aren't even necessary because it's immediately clear to the user that they cannot proceed until they provide their credentials.

Sometimes the use of user interface components also plays an important role in keeping invalid data out. Take a look at the following screenshot. 

![Driver License SignUp Form](../images/driver-lic-1.png)

At first glance, the form appears to be well-designed. However, upon closer inspection, there is a significant risk that users might provide incorrect input for critical fields like **state**, **license type**, and **country**. These fields often play a crucial role in processing and validation, making accurate data essential.

> While similar concerns can be raised for fields like **street**, **city**, and **zip code**, these generally have a lower impact on the overall process in most cases. However, if these fields are critical to the app's functionality—such as for location-based services or accurate delivery—it would be beneficial to integrate the form with Google Location Services or a similar geocoding API. This ensures the data entered is validated and aligned with real-world locations, improving accuracy and reliability.

A straightforward and effective solution would be to replace the text input with a user interface component, such as a dropdown menu or picker, that allows users to select an option from predefined choices. This approach minimizes errors, ensures data consistency, and provides a more user-friendly experience. 

![Driver License SignUp Form](../images/driv-lic-2.png)

The **Driver License Sign-Up Form** can greatly benefit from displaying error messages to the user. Since the form consists of multiple fields, each with specific requirements (e.g., format, type, or length), clear error messages help users understand and correct mistakes. This improves the overall user experience, reduces frustration, and ensures that the submitted data meets the necessary validation criteria.

In the next section, we will explore effective strategies for displaying a validation summary to users, ensuring they can easily identify and correct errors in the form. This approach enhances usability and ensures accurate data entry.

### Validation Summary 

A validation summary is an efficient way to display all form validation errors in a single location, enabling users to quickly review the issues and take appropriate action to correct them. This approach enhances usability by providing clear, consolidated feedback rather than scattering error messages throughout the form.

The simplest way to implement a validation summary is by maintaining a single array to track all validation errors. While you can use a custom type to represent error objects for more flexibility, starting with a straightforward array of strings is often sufficient for basic implementations.

``` swift 
  @State private var validationErrors: [String] = [] // To store validation error messages
```

When the user submits the form by pressing the Submit button, we can trigger validation for the entire form. The validate function below handles this process by checking all form fields for compliance with their requirements. If any validation rules are violated, appropriate error messages are added to the validationErrors array.

``` swift 
 private func validate() -> Bool {
        validationErrors = [] // Clear previous errors

        // Validate each field
        if fullName.trimmingCharacters(in: .whitespaces).isEmpty {
            validationErrors.append("Full Name is required.")
        }
        if street.trimmingCharacters(in: .whitespaces).isEmpty {
            validationErrors.append("Street is required.")
        }
        if city.trimmingCharacters(in: .whitespaces).isEmpty {
            validationErrors.append("City is required.")
        }
        if selectedState.isEmpty {
            validationErrors.append("State is required.")
        }
        if zipCode.trimmingCharacters(in: .whitespaces).isEmpty || !zipCode.isValidZipCode {
            validationErrors.append("Zip Code must be in a valid format (e.g., 12345 or 12345-6789).")
        }
        if country.trimmingCharacters(in: .whitespaces).isEmpty {
            validationErrors.append("Country is required.")
        }
        if selectedLicenseType.isEmpty {
            validationErrors.append("License Type is required.")
        }
        if !agreementAccepted {
            validationErrors.append("You must accept the terms and conditions.")
        }

        return validationErrors.isEmpty
        
    }
```



### Inline Validation Error Messages (Inspired from Flutter)

### Model Validation Using Property Wrappers (Inspiring from ASP.NET)

### Testing Validation Logic 

### Conclusion 