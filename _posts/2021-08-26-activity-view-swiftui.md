
# Loading UIKit View into SwiftUI App

In this post, you will learn how to load a UIKit view into SwiftUI application. Consider a scenario that you want to display a loading indicator in a SwiftUI app. UIKit has an `UIActivityIndicatorView` control that can be used to display loading indicators. Let's see how we can load the `UIActivityIndicatorView` in a SwiftUI application.  

> For the sake of this article, we are assuming that SwiftUI does not have a built-in loading indicator. 

In order to represent a UIView in a SwiftUI app, we will use the `UIViewRepresentable` protocol. `UIViewRepresentable` protocol consists of two required functions. 

**makeUIView** - This function is used to construct and return the UIView. 

**updateUIView** - This function is used to update the uiView. 

The implementation for LoadingIndicator is shown below: 

``` swift 
import Foundation
import SwiftUI
import UIKit

struct LoadingIndicator: UIViewRepresentable {
    
    typealias UIViewType = UIActivityIndicatorView
    
    func makeUIView(context: Context) -> UIActivityIndicatorView {
        let activityIndicatorView = UIActivityIndicatorView()
        activityIndicatorView.color = UIColor.gray
        return activityIndicatorView
    }
    
    func updateUIView(_ uiView: UIActivityIndicatorView, context: Context) {
        uiView.startAnimating()
    }
}
```
`LoadingIndicator` is our SwiftUI representation of the UIActivityIndicatorView. The `makeUIView` function simply constructs the UIActivityIndicatorView, sets the color and returns it. The `updateUIView` function starts the animation. 

Now, you can jump into the SwiftU view and use your new `LoadingIndicator`. This is shown below: 

``` swift 
struct ContentView: View {
    
    var body: some View {
        VStack {            
            LoadingIndicator()
        }
    }
}
```

If you run the app, you will see the loading indicator being displayed on the screen. In order to control the visibility of the loading indicator, we need to pass the loading state. This means we can dictate, when the loading indicator is animating and when it stops animating. 

In the code below, we are passing a `loading` variable to the LoadingIndicator view. The button is changing the state of the `loading` variable from `true` to `false` after 2 seconds. 

``` swift

import SwiftUI

struct ContentView: View {
    
    @State private var loading: Bool = false
    
    var body: some View {
        VStack {
            
            LoadingIndicator(loading: loading)
            
            Button("Get Data") {
                loading = true
                
                DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
                    loading = false 
                }
            }
            
        }
    }
}
```

Now, let's update the `LoadingIndicator` to utilize the new `loading` argument.  

``` swift 
import Foundation
import SwiftUI
import UIKit

struct LoadingIndicator: UIViewRepresentable {
    
    var loading: Bool
    
    typealias UIViewType = UIActivityIndicatorView
    
    func makeUIView(context: Context) -> UIActivityIndicatorView {
        let activityIndicatorView = UIActivityIndicatorView()
        activityIndicatorView.color = UIColor.gray
        return activityIndicatorView
    }
    
    func updateUIView(_ uiView: UIActivityIndicatorView, context: Context) {
        
        if loading {
            uiView.startAnimating()
        } else {
            uiView.stopAnimating()
        }
        
    }
}
```

And that's it! Now when you press the button, the `LoadingIndicator` will show the spinning image and after two seconds, it will stop animating. 

I hope you liked the article. If you want to support my work then please check out my Udemy courses below: 

<center>
<a href = "http://www.azamsharp.com/courses">
<img src="https://raw.githubusercontent.com/azamsharp/azamsharp.github.io/master/_posts/images/banner.png"> 
</a>
</center>