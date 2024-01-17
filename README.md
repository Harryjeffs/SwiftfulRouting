# SwiftfulRouting  🤙

SwiftfulRouting is a native, declarative framework that enabled programmatic navigation in SwiftUI applications. 

SwiftUI is a declarative framework, and therefore, a SwiftUI router must be declarative by nature. Routers based on programatic code do not declare the view heirarchy in advance, but rather at the time of execution. The solution herein is to declare modifiers to support all possible routing in advance. The result is a Router struct that is fully decoupled from the View and added into the Environment on each screen.

**Setup time:** 36.9 seconds

**Sample project:** https://github.com/SwiftfulThinking/SwiftfulRoutingExample

<br>

## Setup

<details>
<summary> Details (Click to expand) </summary>
<br>
Add the package to your Xcode project.

```
https://github.com/SwiftfulThinking/SwiftfulRouting.git
```

Import the package

```swift
import SwiftfulRouting
```

Add a `RouterView` at the top of your view heirarchy. A `RouterView` will embed your view into a Navigation heirarchy and add modifiers to support all potential segues.

```swift
struct ContentView: View {
    var body: some View {
        RouterView { _ in
            MyView()
        }
    }
}
```

All child views have access to a `Router` in the `Environment`.

```swift
@Environment(\.router) var router
    
var body: some View {
     Text("Hello, world!")
          .onTapGesture {
               router.showScreen(.push) { _ in
                    Text("Another screen!")
               }
          }
     }
}
```

Instead of relying on the `Environment`, you may also pass the `Router` directly into the child views. This allows the `Router` to be fully decoupled from the View (for more complex app architectures).

```swift
RouterView { router in
     ContentView(router: router)
          .onTapGesture {
               router.showScreen(.push) { router2 in
                    Text("View2")
                         .onTapGesture {
                              router2.showScreen(.push) { router3 in
                                   Text("View3")
                              }
                         }
               }
          }
}
```

Each `Router` object can simultaneously support 1 active Segue, 1 active Alert, and 1 active Modal. A new Router is created and added to the view heirarchy after each Segue. Refer to `AnyRouter.swift` to see all accessible methods.


```swift
struct MyView: View {

    let router: AnyRouter
    
    var body: some View {
        VStack {
            Text("Segue")
                .onTapGesture {
                    router.showScreen(.push) { router in
                        ThirdView(router: router)
                    }
                }
            
            Text("Alert")
                .onTapGesture {
                    router.showAlert(.alert, title: "Title") {
                        Button("OK") {
                            
                        }
                        Button("Cancel") {
                            
                        }
                    }
                }
            
            Text("Modal")
                .onTapGesture {
                    router.showModal {
                        ChildView()
                    }
                }
        }
    }
}
```

</details>

## Setup (existing projects) 

<details>
<summary> Details (Click to expand) </summary>
<br>
    
In order to enter the framework's view heirarchy, you must wrap your content in a RouterView. By default, your view will be wrapped in with navigation stack (iOS 16+ uses a NavigationStack, iOS 15 and below uses NavigationView). 
- If your view is already within a navigation heirarchy, set `addNavigationView` to `FALSE`. 
- If your view is already within a NavigationStack, use `screens` to bind to the existing stack path.
- The framework uses the native SwiftUI navigation bar, so all related modifiers will still work.

```swift
RouterView(addNavigationView: false, screens: $existingStack) { router in
   MyView(router: router)
        .navigationBarHidden(true)
        .toolbar {
        }
}
```

</details>

## Show Screens

<details>
<summary> Details (Click to expand) </summary>
<br>

Router supports all native SwiftUI segues.

```swift
// NavigationLink
router.showScreen(.push) { _ in
     Text("View2")
}

// Sheet
router.showScreen(.sheet) { _ in
     Text("View2")
}

// FullScreenCover
router.showScreen(.fullScreenCover) { _ in
     Text("View2")
}
```

Segue methods also accept `AnyRoute` as a convenience, which make it easy to pass the `Route` around your code.

```swift
let route = AnyRoute(.push, destination: { router in
     Text("Hello, world!")
})
                        
router.showScreen(route)
```

iOS 16+ uses NavigationStack, which supports pushing multiple screens at once.

```swift
let route1 = PushRoute(destination: { router in
     Text("View1")
})
let route2 = PushRoute(destination: { router in
     Text("View2")
})
let route3 = PushRoute(destination: { router in
     Text("View3")
})
                        
router.pushScreenStack(destinations: [route1, route2, route3])
```

iOS 16+ also supports resizable sheets.

```swift
router.showResizableSheet(sheetDetents: [.medium, .large], selection: nil, showDragIndicator: true) { _ in
     Text("Hello, world!)
}
```

Additional convenience methods:
```swift
router.showSafari {
     URL(string: "https://www.apple.com")
}
```

</details>

## Show Flows

<details>
<summary> Details (Click to expand) </summary>
<br>

WORKING

## Dismiss Screens

. Note that `popToRoot` purposely dismisses all views pushed onto the NavigationStack, but does not dismiss `.sheet` or `.fullScreenCover`.
There is a `dismissScreen` method. You can also dismiss the screen using native SwiftUI code, including swipe-back gestures & `presentationMode`. 

```swift
router.dismissScreen()
```

## Alerts 🚨

Router supports native SwiftUI alerts, including `.alert` and `.confirmationDialog`.

```swift
router.showAlert(.alert, title: String, subtitle: String?, alert: () -> View)
router.showAlert(.confirmationDialog, title: String, subtitle: String?, alert: () -> View)
router.dismissAlert()
```

Additional convenience methods:

```swift
router.showBasicAlert(text: String, action: (() -> Void)?)
```

## Modals 🪧

Router also supports any modal transition, which displays above the current content. Customize transition, animation, background color/blur, etc.

```swift
router.showModal(destination: () -> View)
router.showModal(
  transition: AnyTransition, 
  animation: Animation, 
  alignment: Alignment, 
  backgroundColor: Color?,
  backgroundEffect: BackgroundEffect?,
  useDeviceBounds: Bool, 
  destination: () -> View)
router.dismissModal()
```

Additional convenience methods:

```swift
router.showBasicModal(destination: () -> View)
```



## Under the hood ⚙️

As you segue to a new screen, the framework adds a set ViewModifers to the root of the destination View that will support all potential navigation routes. Currently, the framework can simultaneously support 1 active Segue, 1 active Alert, and 1 active Modal on each View in the heirarchy. The ViewModifiers are based on generic and/or type-erased destinations, which maintains a declarative view heirarchy while allowing the developer to still determine the destination at the time of execution. 

See sample project for example implementations, UI Tests and sample MVC, MVVM and VIPER design patterns.




## Contribute 🤓

Community contributions are encouraged! Please ensure that your code adheres to the project's existing coding style and structure. Most new features are likely to be derivatives of existing features, so many of the existing ViewModifiers and Bindings should be reused.

- [Open an issue](https://github.com/SwiftfulThinking/SwiftfulRouting/issues) for issues with the existing codebase.
- [Open a discussion](https://github.com/SwiftfulThinking/SwiftfulRouting/discussions) for new feature requests.
- [Submit a pull request](https://github.com/SwiftfulThinking/SwiftfulRouting/pulls) when the feature is ready.
