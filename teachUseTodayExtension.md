#学习使用extension 

##从iOS8以来 Apple新加了不少extension 日程安排 通知之心 分享模式 但是没多少app使用它们 一个是程序猿的能力不行 第二个是不少人并不关心这些 在铁蛋看来，这些都是提高app刘春率的东西 很有研究价值
* 先看today Extension 

* 从target 中新建 today

* 这里不得不说一点 关于代码复用 可能一样的逻辑在mainApp中会使用 而extension中同样会得到使用 copy paste ？ 不符合Do not repeat yourself 的原则

  怎么解决呢？使用静态库即可 制作一个静态库 大家一起用 岂不乐哉

1. 铁蛋开始是用label作为显示控件。添加首饰没有起作用 不知哪里有问题 而后使用了**uibutton** 
2. 
```
    preferredContentSize = CGSizeMake(0, 100)

    let button = UIButton(frame: CGRectMake(0, 50, 50, 63))
    button.setTitle("Open", forState: UIControlState.Normal)
    button.addTarget(self, action: "buttonPressed:", forControlEvents: UIControlEvents.TouchUpInside)

    view.addSubview(button)
```
3. 打开主app
```
@objc private func buttonPressed(sender: AnyObject!) {
    extensionContext.openURL(NSURL(string: "simpleTimer://finished"), completionHandler: nil)
}
```
4. 这里不得不说关于URLtype  设置identified 会有schemes example：identified：test，schemes：go； 则对应的url则应该是go://test

5. 模版代码 
```
func widgetPerformUpdateWithCompletionHandler(completionHandler: ((NCUpdateResult) -> Void)!) {
    // Perform any setup necessary in order to update the view.

    // If an error is encoutered, use NCUpdateResult.Failed
    // If there's no update required, use NCUpdateResult.NoData
    // If there's an update, use NCUpdateResult.NewData

    completionHandler(NCUpdateResult.NewData)
}
```
**这个是类似主app的后台机制，在这里可以进行api请求之类的事情 实时刷新也是可以的**

__必须要conform ncwidgetproviding__
