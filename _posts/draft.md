# Understanding @MainActor in Swift 


Let's start with a simple example. In the code below we are incrementing a counter value. 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    var body: some View {
        VStack {
            Button("Increment") {
               value += 1
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```

If we even use the following it will work as expected because Task is going to use the context provided by the parent. 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task {
                    value += 1
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```

The following will ignore the current context or the context provided by the parent and use background context. 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    value += 1
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```

You will receive the warning: 

` Main actor-isolated property 'value' can not be mutated from a nonisolated context`

@State properties are main actor isolated.  

Fix is to run the value in the MainActor closure: 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    await MainActor.run {
                        value += 1
                    }
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```

This also works 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    private func increment() async {
        value += 1
    }
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    await increment()
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```

this also works 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    private func increment() async {
        //await MainActor.run {
            value += 1
        //}
    }
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    await increment()
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```


this also works 

``` swift 
struct MainActorDemo: View {
   
    @State private var value: Int = 0
    
    nonisolated private func increment() async {
        await MainActor.run {
            value += 1
        }
    }
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    await increment()
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(value)")
                .font(.largeTitle)
        }
    }
}
```


``` swift 

@Observable
class CounterStore {
    
    var value: Int = 0
    
    func increment() {
        value += 1
    }
    
}

struct MainActorDemo: View {
   
    @State private var value: Int = 0
    @State private var counterStore = CounterStore()
    
    nonisolated private func increment() async {
        await MainActor.run {
            value += 1
        }
    }
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    await counterStore.increment()
                }
            }.buttonStyle(.borderedProminent)
            
            Text("\(counterStore.value)")
                .font(.largeTitle)
        }
    }
}

```

The default actor isolated in Xcode 26 is MainActor. If you switch to non isolation then you will have to use MainActor. 

``` swift 
@Observable
@MainActor
class CounterStore {
    
    var value: Int = 0
    
    func increment() {
        value += 1
    }
}

struct MainActorDemo: View {
   
    @State private var value: Int = 0
    @State private var counterStore = CounterStore()
    
    nonisolated private func increment() async {
        await MainActor.run {
            value += 1
        }
    }
    
    var body: some View {
        VStack {
            Button("Increment") {
                Task.detached {
                    await counterStore.increment()
                }
            }.buttonStyle(.borderedProminent)
            
            //Text("\(counterStore.value)")
              //  .font(.largeTitle)
        }
    }
}
```

default actor isolation is non isolated 

``` swift 
@Observable
class CounterStore {
    
    var value: Int = 0
    
    func increment() {
        value += 1
    }
}

struct MainActorDemo: View {
   
    @State private var value: Int = 0
    @State private var counterStore = CounterStore()
    
    nonisolated private func increment() async {
        await MainActor.run {
            value += 1
        }
    }
    
    var body: some View {
        VStack {
            Button("Increment") {
                counterStore.increment()
                /*
                Task.detached {
                    await counterStore.increment()
                } */
            }.buttonStyle(.borderedProminent)
            
            Text("\(counterStore.value)")
                .font(.largeTitle)
        }
    }
}
```