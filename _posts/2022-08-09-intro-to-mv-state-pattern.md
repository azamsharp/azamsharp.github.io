# Introduction to MV State Pattern for SwiftUI Apps



## What is MV State Pattern? 

First of all MV state pattern is not the official name for this pattern. Some people call it state pattern and others call it MVI (Model View Intent). I will simply call it MV pattern of MV state pattern. 

The basic idea of MV pattern is that the user will perform action which changes/mutates the state. This will cause the view to render again. This will be running in a constant loop. Anytime an action changes the state, a new version of view is created and rendered on the screen. This is shown in the following diagram. 

<screenshot>

https://developer.apple.com/videos/play/wwdc2019/226/

The action triggered by the user will change the state. This state can reside in individual properties decorated by ```@State``` property wrappers, model objects and even view models. 

Although MVVM has become a default architecture for SwiftUI apps, it introduces a lot of unnecessary code that is not needed in majority of the cases. The view in SwiftUI has binding capabilities. This means view in SwiftUI is not only a view, but also a view model. You can read my article here. 





