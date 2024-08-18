
# Global Sheets Pattern in SwiftUI   

SwiftUI’s ```sheet``` view modifier provides a straightforward way to present modals, but when your app requires multiple sheets across different screens, managing them can become cumbersome. Typically, developers resort to using multiple sheet modifiers or controlling the display with an enum. However, this approach often leads to redundant code scattered across various views.

What if you could centralize the management of sheets, offering a cleaner, more maintainable solution with a user-friendly API?

In this article, we’ll explore different patterns for displaying and managing sheets in SwiftUI. We'll also look at how adopting ideas from other platforms can enhance your app’s architecture, making sheet management more efficient and scalable.

### Displaying a Basic Sheet  

Displaying a sheet in SwiftUI is quite simple. 


