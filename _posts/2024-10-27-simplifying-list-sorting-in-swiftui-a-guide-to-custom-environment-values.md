
# Simplifying List Sorting in SwiftUI: A Guide to Custom Environment Values 

In React, **hooks** are special functions that enable you to tap into React's state and lifecycle features within function components. Additionally, you can create **custom hooks** to encapsulate and reuse logic across your components. Here are a few common examples of custom hooks:

- **`useFetch`**: A custom hook that manages data fetching from an API, returning the data, loading state, and any errors.
- **`useLocalStorage`**: This hook helps manage state that persists in the browser’s localStorage.
- **`useSorting`**: This hook provides sorting functionality for lists based on a specific property or criteria.

> In React, components are similar to **views** in SwiftUI.

While SwiftUI doesn’t have a direct equivalent to React hooks, you can achieve similar functionality using **custom Environment values**. You may already be familiar with built-in environment values like `dismiss`, `authorizationController`, `locale`, and `colorScheme`, which offer global access to shared values across your app.

In this article, I’ll show you how to create a custom Environment value in SwiftUI that mimics the behavior of a custom hook, providing reusable sorting logic for your application.

### What Problem Are We Solving?

Sorting is a common requirement when displaying lists of items on the screen. Depending on the model’s properties, sorting criteria can vary. We aim to create a solution that makes it easy to sort lists inside your views, regardless of the data type.

The goal is to provide a **clear and easy-to-use API**. Here’s what we want to achieve by the end of this guide:

```swift
struct MovieListView: View {
    @Environment(\.sort) var sort
    let movies: [Movie]
    
    private var sortedMovies: [Movie] {
        sort(movies, by: \.title, .asc)
    }
}
```

This solution will simplify sorting and make your code cleaner and more reusable, similar to the power of custom hooks in React.

### Solution:

The provided code offers a concise and efficient solution for sorting data in SwiftUI using custom environment values. This approach makes it easier to handle sorting logic within SwiftUI views, by centralizing the functionality in a reusable and declarative manner.

#### 1. **`SortOrder` Enum:**
The `SortOrder` enum defines two possible sorting options: ascending (`asc`) and descending (`desc`). This simple enum helps in specifying the order in which items should be sorted.

```swift
enum SortOrder {
    case asc, desc
}
```

By providing this enum, we standardize sorting behavior and make it more explicit, allowing views or other parts of the code to easily choose the appropriate sorting order.

#### 2. **`Sort` Struct:**
The `Sort` struct encapsulates the sorting logic. It defines a property `sortOrder` to store the current order (ascending by default) and uses a `callAsFunction` method to perform the actual sorting.

```swift
struct Sort {
    
    var sortOrder: SortOrder = .asc
    
    func callAsFunction<T: Comparable, U>(_ array: [U], by keyPath: KeyPath<U, T>, _ order: SortOrder = .asc) -> [U] {
        switch order {
            case .asc:
                return array.sorted { $0[keyPath: keyPath] < $1[keyPath: keyPath] }
            case .desc:
                return array.sorted { $0[keyPath: keyPath] > $1[keyPath: keyPath] }
        }
    }
}

```swift
extension EnvironmentValues {
    @Entry var sort = Sort()
}
```

- **`callAsFunction` Method**: This method allows the `Sort` struct to behave like a function. You can invoke it directly on an array, passing a key path for sorting. Depending on the provided `SortOrder`, it sorts the array in ascending or descending order.
  
- **KeyPath-based Sorting**: The sorting logic leverages Swift's powerful `KeyPath` feature, allowing the sorting to be done based on a specific property of each element in the array. This makes the solution versatile and applicable to any data model with comparable properties.

#### 3. **`EnvironmentValues` Extension:**
The extension on `EnvironmentValues` allows the sorting logic to be injected into SwiftUI’s environment, making it globally accessible in any view without needing to pass it explicitly as a parameter.

- **`@Entry` Macro**: The `@Entry` macro simplifies the creation of environment values by eliminating boilerplate code, making the process more concise and streamlined.

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

### How the Solution Works:

This approach allows sorting logic to be centralized and easily reusable across multiple views by placing it into the environment. The use of SwiftUI's environment system lets you access sorting functionality anywhere in your view hierarchy without needing to manually pass sorting functions down through props.

### Usage Example:
``` swift 
struct MovieListView: View {
    @Environment(\.sort) var sort
    @State private var sortOrder: SortOrder = .asc // State to track current sort order
    let movies: [Movie]
    
    private var sortedMovies: [Movie] {
        sort(movies, by: \.title, sortOrder)
    }
    
    var body: some View {
        VStack {
            Text("Movies (List)")
            
            // Button to toggle the sort order
            Button(action: {
                // Toggle the sort order between ascending and descending
                sortOrder = sortOrder == .asc ? .desc : .asc
            }) {
                Text("Sort by Title (\(sortOrder == .asc ? "Asc" : "Desc"))")
            }
            .padding()
            
            // Sorting the movies based on the current sortOrder
            ForEach(sortedMovies) { movie in
                Text("\(movie.title) - \(movie.rating)")
            }
        }
    }
}
```

### Key Benefits of This Solution:

1. **Declarative and Simple API**: Using the `callAsFunction` method makes the sorting functionality intuitive. You can use `sort` like a function to sort arrays based on a key path in a clean and readable manner.

2. **Reusability**: By embedding the sorting logic in the environment, the same `Sort` struct can be reused across multiple views, promoting DRY (Don't Repeat Yourself) principles and avoiding redundant code.

3. **Flexibility**: The `Sort` struct and `SortOrder` enum allow easy switching between ascending and descending sorting orders, and the use of `KeyPath` enables sorting on any comparable property of a data model.

4. **Global Access via Environment**: By extending `EnvironmentValues`, the sorting functionality becomes globally accessible throughout the SwiftUI view hierarchy, reducing the need for explicitly passing it down as a parameter.

### Source code: 

[Download](https://gist.github.com/azamsharpschool/1317c5d249a5c3052ebd4edd63b1c265)

### Summary:

This solution efficiently leverages SwiftUI’s environment system to provide a centralized and reusable sorting mechanism. By defining the sorting logic in the `Sort` struct and placing it into the environment, you enable flexible, easy-to-use sorting capabilities across your entire SwiftUI application. The declarative API, combined with the power of `KeyPath`, ensures that your views stay clean and maintainable while benefiting from consistent sorting behavior.









