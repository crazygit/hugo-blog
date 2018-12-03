---
title: Swift入门指南一
slug: swift-tour-001
date: 2018-10-27T09:34:03+08:00
description: Swift入门指南一
categories:
  - iOS开发入门
tags:
  - iOS
  - swift
---

## Hello World

```swift
print("Hello, world!")
```

“Hello world”的写法与`python`的语法, 不需要分号结尾

<!--more-->

## 常量和变量

使用`let`声明常量，使用`var`声明变量。

```swift
// 常量let
let maxNum = 100

// 变量var
var index = 2
var str = "Hello, playground"

// 一次定义多个变量
var x=1, y=2, c=3

// 常量在编译的时候才确定它的值，但是只能赋值一次，不允许再修改
// 如下面的常量c
let a = 1000
let b = 50
let c = a + b

// 指定常量类型
let b: String = "xxx"

// 批量指定变量类型
var j, k, l: Double

let label = "The width is "
let width = 94
// 类型转换
let widthLabel = label + String(width)

// 字符串格式化
let apples = 3
let oranges = 5
let appSumary = "I have \(apples) apples."
let fruitSummary = "I have \(apples + oranges) pieces of fruit"

// 使用三引号（"""）可以起到一次使用跨行声明变量
let a = """
line one
line two
"""
```

### 字符串

Swift中字符串只能使用双引号

```swift
let str: String = "Hello world"
```

## 数组和字典

与`python`不同，Swift数组和字典都是用方括号表示

```swift
// 数组
var a = ["a", "b", "c"]

// 字典
var b = [
"name": "crazygit",
"age": 28
]

// 创建元素类型为String类型的空数组
let emptyArray = [String]()

// 创建key为String，Value为Float类型的空字典
let emptyDictionary = [String: Float]()

// 不明确类型的
shoppingList = []
occupations = [:]

```

## 流程控制

关键字有:

`if`, `switch`, `for-in`,`while`, `repeat-while`

判断条件和循环变量外层的圆括号是可选的。
但是循环体外层的花括号是必须的。


```swift
let individualScore = [75, 43, 103, 87, 12]
var teamScore = 0
for score in individualScore {
    if score > 50 {
        teamScore += 3
    } else {
        teamScore += 1
    }
}
print(teamScore)
```
`if`语句后面必须是`Boolean`表达式, `if score {}` 会直接报语法错误，不是像`python`中暗含把`score`和`0`相比较。

```swift
a.swift:4:8: error: 'Int' is not convertible to 'Bool'
if score {
   ^~~~~
```

变量类型后面如果有`?`号， 表示变量是个可选值

### if 语句

```swift
// 可选值
var optionalString: String? = "Hello"

print(optionalString == nil)

var optionalName: String? = "xx"
var greeting = "hello"

// 如果变量的可选值是nil，条件会判断为false，大括号中的代码会被跳过。
// 如果不是nil，会将值赋给let后面的常量，这样代码块中就可以使用这个值了。
if let name = optionalName {
    print(name)
    greeting = "Hello, \(name)"
}
else{
     print("else")
}

// 使用??可以提供一个默认值，当nickName为nil时，使用fullName
let nickName: String? = nil
let fullName: String = "John Appleseed"
let informalGreeting = "Hi \(nickName ?? fullName)"
```

### switch

switch支持**任意类型**的数据比较操作。

运行switch中匹配到的子句之后，程序会退出switch语句，并不会继续向下运行，所以**不需要**在每个子句结尾写break。

```swift
let vegetable = "red pepper"

switch vegetable {
case "celery":
    print("Add som raisins and make ants on a log.")
case "cucumber", "watercress":
    print("That would make a good tea sandwich.")
case let x where x.hasSuffix("pepper"):
    print("Is is a spicy \(x)?")
default:
    print("Everything tastes good in soup.")
}
```
注意上面`let`的用法

### for-in

在遍历字典字典数据时，遍历结果是无序的

```swift
let interestingNumbers = [
    "Price": [2, 3, 5, 7, 11, 13],
    "Fibonacci": [1, 1, 2, 3, 5, 8],
    "Square": [1, 4, 9, 16, 25],
]

var largest = 0
for (_, numbers) in interestingNumbers {
    for number in numbers {
        if number > largest {
            largest = number
        }
    }
}
print(largest)
```
`for (_, numbers) `的括号不能省略

### while

两种写法，一种先判断条件再实行，一种先执行再判断条件(保证至少运行一次)

```swift
// 先判断再执行（不一定会执行）
var n = 2
while n < 100 {
    n=n * 2
}
print(n)

// 先执行再判断（至少执行一次）
var m = 2
repeat {
    m = m * 2
} while m < 100
print(m)
```

### for 循环

`..<`表示循环索引，不包含最大值

`...`表示循环索引，包含最大值

```swift
var firstForLoop = 0
// 等价于for i=0; i<4; i++
for i in 0..<4 {
    firstForLoop += i
}
print(firstForLoop)

var secondForLoop = 0
// 等价于for i=0; i<=4; i++
for i in 0...4 {
    secondForLoop += i
}
print(secondForLoop)

```
### 函数和闭包

使用 `func` 来声明一个函数, 使用名字和参数来调用函数。使用 `->` 来指定函数返回值的类型。

```swift
// 定义一个函数
func greet(person: String,  day: String) -> String {
    return "Hello \(person),  today is \(day)."
}

//调用函数
greet(person: "Bob", day: "Tuesday")

// 传入第二个参数的day标签不能省略
greet("Bob", day:"Tuesday")
```

默认情况下，函数使用参数名作为参数标签，可以在参数名前写个自定义的标签，或者使用`_`来表示没有标签

```swift
func greet(_ person: String,  on day: String) -> String {
    return "Hello \(person),  today is \(day)."
}

greet("Bob", on: "Tuesday")
```
上面`person`参数没有设置参数标签， `day`的参数标签是`on`


使用元祖可以返回多个值

```swift
func caculateStatistics(scores: [Int]) -> (min: Int, max: Int, sum: Int){
    var min = scores[0]
    var max = scores[0]
    var sum = 0
    for score in scores {
        if score > max {
            max = score
        } else if score < min {
            min = score
        }
        sum += score
    }
    return (min, max, sum)
}

let statistics = caculateStatistics(scores: [5,3,100,3,9])

// 使用名称获取返回的sum
print(statistics.sum)

// 使用下标获取返回的sum
print(statistics.2)
```

函数可以嵌套，内部的函数可以访问外部函数定义的变量

```swift
func returnFifteen() -> Int{
    var y = 10
    func add(){
        y += 5
    }
    add()
    return y
}

returnFifteen()
```

函数的返回值也可以是一个函数

```swift
//返回值是函数，注意返回值的写法
func makeIncrementer() -> ((Int) -> Int){
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}
var increment = makeIncrementer()
print(increment(7))
```

函数的参数也可以是一个函数

```swift
func hasAnyMatch(list: [Int], condtion: (Int) -> Bool) -> Bool {
    for item in list {
        if condtion(item){
            return true
        }
    }
    return false
}

func lessThanTen(number: Int) -> Bool {
    return number < 10
}

var numbers = [20, 19,7,12]
hasAnyMatch(list: numbers, condtion: lessThanTen)
```

闭包的写法，闭包时可以省略掉函数名，用`{}`表示，在函数体内用`in`来区分参数，返回值和函数体

```swift
numbers.map({(number: Int) -> Int in
    let result = 3 * number
    return result
})

// 省略参数类型，返回值类型的写法
let mappedNumbers = numbers.map({number in 3 * number})
print(mappedNumbers)
```

可变参数`...`

```swift
func sumOf(numbers: Int...) -> Int {
    var sum = 0
    for number in numbers {
        sum += number
    }
    return sum
}
print(sumOf())
print(sumOf(numbers: 42, 597, 12))
```
