# HotReloadable
Being able to hot reload views with injection [injectionforxcode.com](injectionforxcode.com) is plain amazing.
The Injection plugin works perfect out of the box but there is still a bit of code we need to write.

When our view file gets saved and injected, we actually need to manually reload the `View` in our `ViewController` in order to see the change on screen. the following is a snippet that makes this process easier 😎


Turn this:

```swift
class ViewController: UIViewController {
    
    var v = View()
    override func loadView() { view = v }
    
    convenience init() {
        self.init(nibName: nil, bundle: nil)
        NotificationCenter.default.addObserver(forName:
            NSNotification.Name(rawValue:"INJECTION_BUNDLE_NOTIFICATION"),
                                               object: nil,
                                               queue: .main) { [weak self] _ in
                                                self?.v = View()
                                                self?.view = self?.v
                                                self?.viewDidLoad()
        }
    }
}
```

into

```swift
class ViewController: UIViewController, HotReloadable {
    
    var v = View()
    override func loadView() { view = v }
    
    convenience init() {
        self.init(nibName: nil, bundle: nil)
        enableHotReload(on: \ViewController.v)
    }
}
```


The magic :

```swift
protocol HotReloadable { }

extension HotReloadable where Self: UIViewController {
    
    func enableHotReload<V: UIView>(on viewProperty: ReferenceWritableKeyPath<Self, V>) {
        let nc = NotificationCenter.default
        nc.addObserver(
            forName:NSNotification.Name(rawValue:"INJECTION_BUNDLE_NOTIFICATION"),
            object: nil,
            queue: .main) { _ in
                self[keyPath: viewProperty] = V()
                self.view = self[keyPath: viewProperty]
                self.viewDidLoad()
        }
    }
}
```
