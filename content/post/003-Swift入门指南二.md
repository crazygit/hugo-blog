---
title: Swift入门指南二
slug: swift-tour-002
date: 2018-10-27T22:55:22+08:00
description: Swift入门指南二
categories:
  - ios开发入门
tags:
  - iOS
---

## 对象和类

创建类

```swift
class Shape {
    var numberOfSides = 0
    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
}

var shape = Shape()
shape.numberOfSides = 7
var shapeDescription = shape.simpleDescription()
```

<!--more-->

`init`构造函数，`deinit`析构函数

```swift
class NamedShape {
    var numberOfSides: Int = 0
    var name: String

    init(name: String) {
        self.name = name
    }

    func simpleDescription() -> String{
        return "A shape whit \(numberOfSides) sides"
    }

}
```
类属性必须要设置一个值，要么在声明属性的时候指定，要么在`init`函数中指定。


继承类, 使用`:`， 覆写父类的方法**必须**使用`override`

```swift
class Square: NamedShape {
    var sideLength: Double

    init(sideLength: Double, name: String){
        self.sideLength = sideLength
        // 调用父类的方法
        super.init(name: name)
        numberOfSides = 4
    }

    func area() -> Double {
        return sideLength * sideLength
    }

    // 覆写父类的方法
    override func simpleDescription() -> String {
        return "A square with sides of length \(sideLength)."
    }
}
```

属性的`getter`和`setter`方法，`set`没有指定参数名时，默认用`newValue`

```swift

class EquilateralTriangle: NamedShape {
    var sideLength: Double = 0.0

    init(sideLength: Double, name: String){
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 3
    }

    var perimeter: Double {
        get {
            return 3.0 * sideLength
        }

        set {
            sideLength = newValue / 3.0
        }
    }

    //var perimeter: Double {
    //    get {
    //        return 3.0 * sideLength
    //    }

    //    set(newPerimeter) {
    //        sideLength = newPerimeter / 3.0
    //    }
    //}

    override func simpleDescription ()->String{
        return "An equilateral triangle with sides of length \(sideLength)"
    }
}
```

`willSet`和`didSet`实现属性观察

```swift
class TriangleAndSquare {
    var triangle: EquilateralTriangle {
        willSet {
            square.sideLength = newValue.sideLength
        }
    }

    var square: Square {
        willSet {
            triangle.sideLength = newValue.sideLength

        }
    }

    init(size: Double, name: String){
        square = Square(sideLength: size, name: name)
        triangle = EquilateralTriangle(sideLength: size, name: name)
    }
}

var triganleAndSquare = TriangleAndSquare(size: 10, name: "another test shape")

// 10.0
print(triganleAndSquare.square.sideLength)

// 10.0
print(triganleAndSquare.triangle.sideLength)
triganleAndSquare.square = Square(sideLength: 50, name: "Larger square")

// 50.0
print(triganleAndSquare.triangle.sideLength)
```

当使用可选值时，可以写个`?`在运行符前面，如果`?`前面的值是`nil`, 所有在`?`后面的值都是`nil`,

```swift
let optionalSquare: Square? = Square(sideLength: 2.5, name: "optional square")
let sideLength = optionalSquare?.sideLength
```

## 枚举`enum`

枚举`enum`类型跟类`class`一样也可以有方法

```swift
enum Rank: Int {
    case ace = 1
    case two, three, four, five, sixe, seven, eight, nine, ten
    case jack, queen, king
    func simpleDescription() -> String {
        switch self {
        case .ace:
            return "ace"
        case .jack:
            return "jack"
        case .queen:
            return "queen"
        default:
            return String(self.rawValue)
        }
    }
}

let ace = Rank.ace

// 1
let aceRawValue = ace.rawValue

// 2
print(Rank.two.rawValue)

print(ace.simpleDescription())
```

默认情况下，枚举类型的原始值是从0开始的，然后依次加1，但是可以通过明确赋值，可以改变这个行为。
比如上面的例子，`ace`就是从1开始，后面的值也依次加1，同时，也可以给枚举类型赋值为字符串或浮点数。

使用`init?(rawValue:)`初始化一个枚举类型，当这个值是enum类型的值时，就返回枚举类型，否则返回`nil`

```swift

// 13刚好对应king
if let convertedRank = Rank(rawValue: 13) {
    let threeDescription = convertedRank.simpleDescription() // 13
}


// convertedRank 为nil, if语句不会执行
if let convertedRank = Rank(rawValue: 14) {
    let threeDescription = convertedRank.simpleDescription() // 1
}
```

如果结构体有原始值的话，可以在初始化时决定它的类型

```swift
enum ServerResponse {
    case result(String, String)
    case failure(String)
}

let success = ServerResponse.result("6:00 am", "8:09 pm")
let failure = ServerResponse.failure("Out of cheese.")
print(success)

switch success {
case let .result(sunrise, sunset):
    print("Sun rise at \(sunrise), Sun set at \(sunset)")
case let .failure(message):
    print("Failure \(message)")
}

```

## 结构体`struct`

`struct`和类一样，支持初始化和方法，不同之处在于:

struct在代码里总是复制一份(值传参)，而类是被引用的(引用传参)

```swift
struct Card {
    var rank: Rank
    var suit: Suit

    func simpleDescription() -> String{
        return "The \(rank.simpleDescription()) of \(suit.simpleDescription())"
    }

}

//枚举传参数
let threeOfSpades = Card(rank: .three, suit: .spades)
print(threeOfSpades.simpleDescription())
```

## Protocols和Extensions

有点类似`java`的`interface`，但是也有[不同之处](https://stackoverflow.com/questions/30859334/compare-protocol-in-swift-vs-interface-in-java).

```swift
protocol ExampleProtocol {
    var simpleDescription: String { get }
    // 注意关键字mutating
    mutating func adjust()
}

```

类，枚举和结构体都可以适配协议

```swift
class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "A very simple class."

    var anotherProperty: Int = 69105

    func adjust(){
        simpleDescription += " Now 100% adjusted."
    }
}

var a = SimpleClass()
a.adjust()
print(a.simpleDescription)


struct SimpleStruct: ExampleProtocol {
    var simpleDescription: String = "A very simple struct."

    //注意 mutating 关键字
    mutating func adjust(){
        simpleDescription += " (adjusted)"
    }
}

var b = SimpleStruct()
b.adjust()
print(b.simpleDescription)
```
注意上面在结构体中声明`adjust()`方法时，使用了关键字`mutating`,而在类中并没有使用， 是因为在类中，方法是可以直接修改的。


使用`extension`可以往已经存在的类型添加方法和计算属性，哪怕是Swift内置的类

```swift
extension Int: ExampleProtocol{
    var simpleDescription: String {
        return "The number \(self)"
    }
    mutating func adjust(){
        self += 42
    }
}

// 修改内置Int类的行为
print(7.simpleDescription)
```

我们也可以使用protocol名字来访问实现了这个`protocol`的方法，但是不在`protocol`里定义的就不能访问。

```swift
let protocolValue: ExampleProtocol = SimpleClass()
print(protocolValue.simpleDescription)
//anotherProperty属性虽然在SimpleClass里有，但是它不属于ExampleProtocol, protocolValue访问依然会报错
//print(protocolValue.anotherProperty)
```

## 异常处理

可以通过适配`Error`的protocol来定义错误， 通过`throw`抛出异常，通过`throws`标记可能产生异常的方法。

```swift
enum PrinterError: Error {
    case outOfPaper
    case noToner
    case onFire
    case unKown
}


func send(job: Int, toPrinter printerName: String) throws -> String {
    if printerName == "Never Has Toner" {
        throw PrinterError.noToner
    } else if printerName == "outOfPaper" {
        throw PrinterError.outOfPaper
    } else if printerName == "OnFire" {
        throw PrinterError.onFire
    } else if printerName == "UnKown" {
        throw PrinterError.unKown
    }

    return "job sent"
}
```

有几种方式处理异常，一种是使用`do-catch`, `try`写在可能产生异常的代码前面，`catch`代码块里，如果不指定名字的话，默认用`error`代表错误。

```swift
do {
    let printerResponse = try send(job:1040, toPrinter: "Never Has Toner")
    print(printerResponse)
} catch {
    print(error)
}


do {
    let printerResponse = try send(job:1440, toPrinter: "UnKown")
    print(printerResponse)
} catch PrinterError.onFire {
    print("I'll just put this over here, with the rest of the fire.")
} catch let printerError as PrinterError {
    print("Printer error: \(printerError)")
} catch {
    print(error)
}
```

另一种处理异常的方式是使用`try?`。把结果转变为可选值，当有异常出现的时候，异常会被直接忽略，并把结果赋值为`nil`.

```swift
let printerSuccess = try? send(job: 1884, toPrinter: "Mergenthaler")
let printerFailure = try? send(job: 1885, toPrinter: "Never Has Toner")
```

使用`defer`，可以在函数返回前，执行一段代码，无论函数是正常执行或抛出异常，可以在多个函数之间使用`defer`写一些初始化或清理工作的代码。

```swift
var fridgeIsOpen = false
let fridgeContent = ["milk", "eggs", "leftovers"]

func fridgeContains(_ food: String) -> Bool {
    fridgeIsOpen = true
    defer {
        print("call defer")
        fridgeIsOpen = false
    }

    let result = fridgeContent.contains(food)
    return result
}

fridgeContains("banana")
print(fridgeIsOpen)
```

## 泛型

```swift
func makeArray<Item>(repeating item: Item, numberOfTimes: Int) -> [Item] {
    var result = [Item]()
    for _ in 0..<numberOfTimes {
        result.append(item)
    }
    return result
}

print(makeArray(repeating: "knock", numberOfTimes: 4))
```
泛型不光可以用于函数和方法，还可以用在类，枚举，结构体上面，下面是利用泛型实现Swift标准库里的可选类型

```swift
enum OptionalValue<Wrapped>{
    case none
    case some(Wrapped)
}

var possibleInteger: OptionalValue<Int> = .none
possibleInteger = .some(100)
```

使用`where`关键字在语句前可以限定一些条件

```swift
func anyCommonElements<T: Sequence, U: Sequence>(_ lhs: T, _ rhs: U) -> Bool
where T.Element: Equatable, T.Element == U.Element {
    for lhsItem in lhs {
        for thsItem in rhs {
            if lhsItem == thsItem {
                return true
            }
        }
    }
    return false
}

anyCommonElements([1, 2, 3], [3])
```
