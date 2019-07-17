---
title: "SwiftUI:在SwiftUI中使用UIKit"
categories: SwiftUI
tags:
  - SwiftUI
  - Swift
date: 2019-07-18 00:11:22
---

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
