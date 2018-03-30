# HotReloadable
Helpers to write Hot Reloadable View Controllers


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
