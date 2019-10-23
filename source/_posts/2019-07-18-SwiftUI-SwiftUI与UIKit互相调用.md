---
title: "SwiftUI:SwiftUI与UIKit互相调用"
categories: SwiftUI
tags:
  - SwiftUI
  - Swift
date: 2019-07-18 00:11:22
---

# 在 SwiftUI 中调用 UIKit

在 SwiftUI 中使用 UIKit 需要用到`UIViewRepresentable`和`UIViewControllerRepresentable`两个协议，分别对应在 SwiftUI 中使用`UIView`和`UIViewController`。

## 在 SwiftUI 中使用 UIView

在 SwiftUI 中使用`UIView`需要实现`UIViewRepresentable`协议，如下代码实现了在 SwiftUI 的`List`控件中使用`UITableViewCell`

```swift
struct TableViewCell: UIViewRepresentable {
    @Binding var text: String
    func makeUIView(context: UIViewRepresentableContext<TableViewCell>) -> UITableViewCell {
        let cell = UITableViewCell(style: .default, reuseIdentifier: "cell")
        cell.textLabel?.text = text
        cell.selectionStyle = .gray
        return cell
    }

    func updateUIView(_ uiView: UITableViewCell, context: Context) {
        uiView.textLabel?.text = text
    }
}

struct ContentView : View {
    @State var text: String = "hello"
    var body: some View {
        List(0...5) { idx in
            TableViewCell(text: self.$text)
        }
    }
}

#if DEBUG
struct ContentView_Previews : PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
#endif
```

## 在 SwiftUI 中使用 UIViewController

在 SwiftUI 中使用`UIViewController`需要实现`UIViewControllerRepresentable`协议，如下示例展示了怎么从一个 SwiftUI 页面跳转到`UIViewController`。

```swift
struct MyViewController: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        let vc = UIViewController()
        vc.view.backgroundColor = UIColor.white
        return vc
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {

    }
}

struct ContentView : View {
    @State var text: String = "hello"
    var body: some View {
        NavigationView {
            NavigationLink(destination:
                MyViewController()
                .navigationBarTitle("My ViewController", displayMode: .inline)
            ) {
                Text("Click to push")
            }
        }
    }
}
```

# 在 UIKit 中调用 SwiftUI

在 UIKit 中使用 SwiftUI 要比在 SwiftUI 中使用 SwiftUI 简单很多，只需要使用`UIHostingController`对 SwiftUI 的控件进行一下包装就可以，代码如下：

```swift
let vc = UIHostingController(rootView: Text("Hello World"))
```

包装出来的结果是一个`UIViewController`，这样就可以在其他`UIView`或`UIViewController`中使用了。可以将这个`UIViewController`作为一个`childViewController`加到其他的`UIView`或`UIViewController`中去
