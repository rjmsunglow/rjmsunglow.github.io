---
layout: post
title: "Swift的?和！"
date: 2014-08-26 08:49
comments: true
categories: 
---
* #####Swift语言使用var定义变量，但和别的语言不同，Swift里不会自动给变量赋初始值，也就是说变量不会有默认值，所以要求使用变量之前必须要对其初始化。如果在使用变量之前不进行初始化就会报错。
* #####Optional其实是个enum，里面有None和Some两种类型。其实所谓的nil就是Optional.None, 非nil就是Optional.Some, 然后会通过Some(T)包装（wrap）原始值，这也是为什么在使用Optional的时候要拆包（从enum里取出来原始值）的原因, 也是PlayGround会把Optional值显示为类似{Some "hello world"}的原因.
这里是enum Optional的定义：

```Swift
	enum Optional<T> : LogicValue, Reflectable {
    	case None
    	case Some(T)
    	init()
    	init(_ some: T)

    	/// Allow use in a Boolean context.
    	func getLogicValue() -> Bool

    	/// Haskell's fmap, which was mis-named
    	func map<U>(f: (T) -> U) -> U?
    	func getMirror() -> Mirror
	}
```

* 声明为Optional只需要在类型后面紧跟一个?即可。

```Swift
	var strValue: String?   //?相当于下面这种写法
	var strValue: Optional<String>
```
上面这个Optional的声明，意思不是”我声明了一个Optional的String值”, 而是”我声明了一个Optional类型值，它可能包含一个String值，也可能什么都不包含”，也就是说实际上我们声明的是Optional类型，而不是声明了一个String类型，这一点需要铭记在心。一旦声明为Optional的，如果不显式的赋值就会有个默认值nil。

* 另外，?还可以用在安全地调用protocol类型方法上:
*
```Swift
@objc protocol Downloadable {
    @optional func download(toPath: String) -> Bool;
}

@objc class Content: Downloadable {
    //download method not be implemented
}

var delegate: Downloadable = Downloadable()
delegate.download?("some path")
```
因为上面的delegate是Downloadable类型的，它的download方法是optional，所以它的具体实现有没有download方法是不确定的。Swift提供了一种在参数括号前加上一个?的方式来安全地调用protocol的optional方法。

* ?还有一种用法是用在向下转型上(Downcast)，可能会用到 as?：
```Swift
if let dataSource = object as? UITableViewDataSource {
    let rowsInFirstSection  = dataSource.tableView(tableView, numberOfRowsInSection: 0)
}
```
#####也就是说？用在以下场景：
* 1.声明Option的值变量
* 2.用在对Optional值操作中，用来判断是否能响应后面的操作
* 3.用于安全调用protocol的optional方法
* 4.使用 as? 向下转型(Downcast)

#####----下面来说一下！的用法

* 第一种用法表示我这个变量是确定非nil的，你可以尽情使用和调用它的方法

```Swift
let hashValue = strValue!.hashValue
```
如果你确定一个变量是肯定非nil的，就能直接加上!，强制拆包(unwrap)并执行后面的操作。

考虑下这一种情况，我们有一个自定义的MyViewController类，类中有一个属性是myLabel，myLabel是在viewDidLoad中进行初始化。因为是在viewDidLoad中初始化，所以不能直接声明为普通值：var myLabel : UILabel，因为非Optional的变量必须在声明时或者构造器中进行初始化，但我们是想在viewDidLoad中初始化，所以就只能声明为Optional：var myLabel: UILabel?, 虽然我们确定在viewDidLoad中会初始化，并且在ViewController的生命周期内不会置为nil，但是在对myLabel操作时，每次依然要加上!来强制拆包
```Swift
myLabel!.text = "text"
myLabel!.frame = CGRectMake(0, 0, 10, 10)
```

对于这种类型的值，我们可以直接这么声明：var myLabel: UILabel!,这种是特殊的Optional，称为Implicitly Unwrapped Optionals, 直译就是隐式拆包的Optional，就等于说你每次对这种类型的值操作时，都会自动在操作前补上一个!进行拆包，然后在执行后面的操作，当然如果该值是nil，也一样会报错crash掉。
```Swift
var myLabel: UILabel!  //!相当于下面这种写法
var myLabel: ImplicitlyUnwrappedOptional<UILabel>
```

#####！将会用在以下场景
* 1.强制对Optional值进行拆包(unwrap)
* 2.声明Implicitly Unwrapped Optionals值，一般用于类中的属性
























