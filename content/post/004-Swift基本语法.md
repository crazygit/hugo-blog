---
title: Swift基本语法
slug: swfit-the-basics
date: 2018-10-30T11:58:55+08:00
description: Swift基本语法
categories:
  - iOS开发入门
tags:
  - iOS
---

### 常量和变量

一次声明多个变量

```swift
var x = 0.0, y = 0.0, z = 0.0

var red, green, blue: Double
```

<!--more-->

### 变量名

Swift的变量名**可以是**任何的Unicode字符, 如: 中文, Emoji表情等都可以是变量名。

常量与变量名不能包含空格符,数学符号,箭头,保留的(或者非法的)Unicode 码位,连线与制表符。也不能以数字开头, 但是可以在常量与变量名的其他地方包含数字。

```swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow”
```

### print函数

```swift
let x=1 ,y = 2, z=3
let name = "John"
print(x, y, z) // 可以一次传入多个参数
print(x, y, z, separator: ":")  // 指定分隔符
print(x, y, z, separator: ":", terminator: ";") //指定行尾截止符号，默认是回车
print("My name is \(name)")  // 变量替换
```

### 注释

```swift
// 单行注释

/*
多行注释
*/
```
### 分号

swift语法中，每行代码结尾的分号是可以省略的，但是如果一行包含多个语句时，分号则不能省略

```swift
let cat = 'cat'; print(cat)
```

### 整型

一些整型常量

```swift
// 有符号数最大值和最小值
Int.max
Int.min
// 无符号数最大值和最小值
UInt.max
Uint.min

// 指定机器位数
Int8.max
Int8.min

// 二级制表示17
let binaryInt: Int = 0b10001
// 八进制表示17
let octalInt: Int = 0o21
// 十六进制表示17
let hexInt: Int = 0x11

// 小技巧: 整型赋值时的小技巧，可以用下划线分隔，方便阅读
let a = 1_000_1000 // 一百万

let decimalInteger = 17

// 二进制的17 “0b”
let binaryInteger = 0b10001

// 八进制的17  "0o"
let octalInteger = 0o21

// 十六进制的17 "0x"
let hexadecimalInteger = 0x11
```

一般建议直接使用`Int`类型，它的位数默认与当前使用的操作平台相关:

* 在32位的平台上，等效于`Int32`
* 在64位的平台上，等效于`Int64`


### 浮点型

```swift
// 单精度和双精度浮点数
var a: Float = 3.1415926
var b: Double = 3.1415926
```

### 类型别名

可以使用 `typealias` 关键字来定义类型别名

```swift
typealias AudioSample = UInt16
//AudioSample.min 实际上是 UInt16.min ,所以会给 maxAmplitudeFound 赋一个初值 0
var maxAmplitudeFound = AudioSample.min
```

### 布尔类型和if语句

```swift
// bool值都是小写
let imTrue: Bool = true
let imFalse: Bool = false

if imTrue {
    print("I am True")
}
else if 3+4 == 7{
    print("3+4 == 7")
}
else{
    print("I am false")
}

// 不能用非0值作为为真的条件，如下面的语句会报语法错误
if 1 {
    print("I am true")
}
```

### 元组

* 将多个不同的值集合成一个数据
* 可以有任意多个值
* 不同的值可以是不同的类型

```swift
var point = (5,2)
var httpResponse = (404, "Not Found")


var point2: (Int, Int, Int) = (1,2,3)

// 元组赋值
var point = (5,2)

// 与pythn中不一样，赋值时等号左右的圆括号不能省略
let (x, y) = point
let (status_code, status_message) = http404Error
print(x)
print(y)

// 通过下标访问元组元素, 与python中使用下标的方式不太一样
print(point.0)
print(point.1)

// 通过元素名称访问，有点python字典的感觉
var point3 = (x: 1, y: 2)
point3.x
point3.y

// 声明时指定元素名称
var point4: (x: Int, y: Int) = (2,3)
point4.x
point4.y
// 使用"_"忽略一些元素
var logingResult = (true, "xxxxx")
let (isLoginSuccess, _) = logingResult
isLoginSuccess
```

### 可选值

当可以确定一个可选值包含有值时，可以使用`!`来强制使用值

```swift
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)

if convertedNumber != nil {
    print("convertedNumber contains some integer value.")
}

if convertedNumber != nil {
    print("convertedNumber has an integer value of \(convertedNumber!).")
}
```

`if`语句中可以同时绑定多个可选值（用`,`分隔）。只有当所有的值都真实有值时，`if`条件才成立。

下面两个if语句是等效的

```swift
if let firstNumber = Int("4"), let secondNumber = Int("42"),
    firstNumber < secondNumber && secondNumber < 100 {
        print("\(firstNumber) < \(secondNumber) < 100")
    }

//等效
if let firstNumber = Int("4") {
    if let secondNumber = Int("42") {
        if firstNumber < secondNumber && secondNumber < 100 {
            print("\(firstNumber) < \(secondNumber) < 100")
        }
    }
}
```
if语句绑定的可选值有效范围仅在if语句内部

### 断言(Assertions)和先决条件(Preconditons)

Assertions和Preconditions的区别在于Assertions只在debug build中生效，
而Precondtions在debug build和production build中都生效。

```swift

let age = -3
assert(age >= 0, "A person's age can't be less than zero.")

//省略断言消息
assert(age >= 0)

//使用assertionFailure直接抛出断言异常
if age > 10 {
    print("You can ride the rollercoaster or the ferris wheel.")
} else if age > 0{
    print("You can ride the ferris wheel.")
}
else {
    assertionFailure("A person's age can't be less than zero.")
}

```


