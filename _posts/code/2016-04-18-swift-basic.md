---
layout: post
title:  Swift基础
date:   2016-04-18 4:51:00 +0800
categories: code
tags: swift
---

## 基础类型

 - Int
 - Double, Float
 - Bool
 - String

#### Tuple

{% highlight swift %}
let someTuple: (Double, Double) = (3.14159, 2.71828)
// http404Error is of type (Int, String), and equals (404, "Not Found")
let http404Error = (404, "Not Found")
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
print("The status code is \(http404Error.0)")
print("The status message is \(http404Error.1)")

// ignore parts of the tuple with an underscore (_)
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")

let http200Status = (statusCode: 200, description: "OK")
print("The status code is \(http200Status.statusCode)")
{% endhighlight %}

#### nil

所有optional变量都可以被设为nil

{% highlight swift %}
var surveyAnswer: String?
// surveyAnswer is automatically set to nil

var serverResponseCode: Int? = 404
serverResponseCode = nil
{% endhighlight %}

----------


## 运算

 - 赋值(=)
 - 算术(+, -, *, /)
 - 取模(%)，支持浮点数
 - Unary Minus/Plus Operator(-, +)
 - +=, -=, *=, /=
 - 比较(==, !=, >, <, >=, <=, ===, !==)
 - 三元运算符(?:)
 - 逻辑`(!, &&, ||)`

#### Nil Coalescing Operator(??)

a ?? b 相当于 (a != nil) ? a! : b

#### Range Operators

 - Closed

{% highlight swift %}
// loops [1, 5]
for i in 1...5 {
    ...
}
{% endhighlight %}

 - Half-Open

{% highlight swift %}
// loops [1, 5)
for i in 1..<5 {
    ...
}
{% endhighlight %}

#### Unwrapping

{% highlight swift %}
// Forced Unwrapping
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // requires an exclamation mark

// Implicitly Unwrapped Optionals
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString // no need for an exclamation mark
{% endhighlight %}

----------


## 变量(var, let; optional)

{% highlight swift %}
var mutableDouble:Double = 1.0
let constantDouble:Double = 1.0
var mutableInferredDouble = 1.0
var optionalDouble:Double? = nil
if let definiteDouble = optionalDouble {
    definiteDouble
}
var x = 0.0, y = 0.0, z = 0.0
let x = 0.0, y = 0.0, z = 0.0
var red, green, blue: Double
{% endhighlight %}

----------


## 函数

{% highlight swift %}
func fnName(name1: Type1, name2: Type2, ...) -> ReturnType {
    ...
}

func sayHello(personName: String) -> String {
    return "Hello, " + personName + "!"
}
print(sayHello("Anna"))
// Prints "Hello, Anna!"
{% endhighlight %}

#### 默认参数

{% highlight swift %}
func someFunction(parameterWithDefault: Int = 12) {
    ...
}
someFunction(6) // parameterWithDefault is 6
someFunction() // parameterWithDefault is 12
{% endhighlight %}

#### 可变参数

{% highlight swift %}
func arithmeticMean(numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// returns 3.0, which is the arithmetic mean of these five numbers
{% endhighlight %}

#### In-Out Parameters

 - 函数的参数默认都是常量，在函数内不可修改
 - 如果需要在函数内修改传入的参数（变量），并且使修改后的变量保持修改后的值，就要使用in-out parameter
 - in-out paramenter只能是变量，不能为常量或字面值(literals)
 - 定义函数时，在参数名的前面加上inout
 - 调用函数时，在变量的前面加上&来标记它可以被修改

{% highlight swift %}
func swapTwoInts(inout a: Int, inout _ b: Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// Prints "someInt is now 107, and anotherInt is now 3"
{% endhighlight %}

#### 函数类型(Function Types)

{% highlight swift %}
// (Int, Int) -> Int
func addTwoInts(a: Int, _ b: Int) -> Int {
    ...
}
func multiplyTwoInts(a: Int, _ b: Int) -> Int {
    ...
}

// () -> Void
func printHelloWorld() {
    ...
}

var mathFunction: (Int, Int) -> Int = addTwoInts
print("Result: \(mathFunction(2, 3))")
mathFunction = multiplyTwoInts
{% endhighlight %}

#### 函数类型作为参数类型

{% highlight swift %}
func printMathResult(mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
    print("Result: \(mathFunction(a, b))")
}
printMathResult(addTwoInts, 3, 5)
// Prints "Result: 8"
{% endhighlight %}

#### 函数类型作为返回值类型

{% highlight swift %}
func chooseOpFunction(add: Bool) -> (Int, Int) -> Int {
    return add ? addTwoInts : multiplyTwoInts
}
{% endhighlight %}

#### Nested Functions

 - 只在函数体内可见

{% highlight swift %}
func chooseOpFunction(add: Bool) -> (Int, Int) -> Int {
    func addTwoInts(a: Int, _ b: Int) -> Int { ... }
    func multiplyTwoInts(a: Int, _ b: Int) -> Int { ... }
    return add ? addTwoInts : multiplyTwoInts
}
{% endhighlight %}


----------


## 闭包(Closures)

闭包可以捕获(capture)并储存定义它所在环境的任何变量  
全局函数和nested function实际上都是特殊情况的闭包

 - 全局函数是一种有名字，但不捕获任何值的闭包
 - nested function是一种有名字，可以捕获其所在函数体内变量的闭包
 - 闭包表达式(Closure expressions)是一种无名字，可捕获其周围变量的闭包

#### 闭包表达式

闭包表达式是一种简化函数定义的形式

{% highlight swift %}
{ (parameters) -> returnType in
    statements
}
{% endhighlight %}

简化过程：

1) 标准形式

{% highlight swift %}
reversed = names.sort({ (s1: String, s2: String) -> Bool in
    return s1 > s2
})
{% endhighlight %}

2) 推断参数、返回值类型

根据sort(_:)函数的定义，可以推断出闭包的类型肯定为(String, String) -> Bool，所以参数和返回值都可以省略，进而->也可以被省略掉

{% highlight swift %}
reversed = names.sort( { s1, s2 in return s1 > s2 } )
{% endhighlight %}

3) Implicit Returns from Single-Expression Closures

根据sort(_:)函数的定义可以推断出，闭包必然返回一个Bool，由于闭包只包含一条语句，且s1 > s2返回的是一个Bool，所以return关键字也可以省略掉

{% highlight swift %}
reversed = names.sort( { s1, s2 in s1 > s2 } )
{% endhighlight %}

4) 参数名简写(Shorthand Argument Names)

Swift自动为inline闭包提供简写的参数名：$0, $1, $2, ...   
如果在闭包表达式中使用了这些简写名，就可以省略掉参数列表，简写参数的数量和类型将自动根据期望的函数类型推断出来

{% highlight swift %}
reversed = names.sort( { $0 > $1 } )
{% endhighlight %}

5) Operator Functions(特殊情况使用)

Swift定义了特殊的操作符(>)，类型为(String, String) -> Bool，可以传递给sort用来对比字符串

{% highlight swift %}
reversed = names.sort(>)
{% endhighlight %}


#### Trailing Closures

 - 如果一个函数的最后一个参数是闭包表达式，就可以将它写成trailing closure
 - trailing closure是将闭包表达式写在参数列表的外面
 - 如果闭包表达式是函数的唯一参数，参数的括号都可以省略

{% highlight swift %}
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}
 
// here's how you call this function without using a trailing closure:
someFunctionThatTakesAClosure({
    // closure's body goes here
})
 
// here's how you call this function with a trailing closure instead:
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}

// omit ()
someFunctionThatTakesAClosure {
    // body
}
{% endhighlight %}

#### Capturing Values

 - closure可以捕获定义时周围的变量
 - closure可以获取、修改捕获的变量，甚至原变量生命结束都不会影响它

{% highlight swift %}
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)
incrementByTen()    // 10
incrementByTen()    // 20
incrementByTen()    // 30

let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven()  // 7

incrementByTen()    // 40

// Closures Are Reference Types
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()    // 50
{% endhighlight %}

#### Nonescaping Closures(@noescape)

 - 只在其定义所在的函数体内被调用
 - 编译器可以对nonescaping closure做更深的优化（因为它知道闭包函数的生命周期）
 - 声明参数中有闭包函数的函数时，可以在参数名的前面加上`@noescape`来表明这个参数是nonescaping
 - `@noescape`的函数中可以省略`self`

{% highlight swift %}
func someFunctionWithNonescapingClosure(@noescape closure: () -> Void) {
    closure()
}
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: () -> Void) {
    completionHandlers.append(completionHandler)
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }  // 省略self
    }
}
 
let instance = SomeClass()
instance.doSomething()
print(instance.x)   // 200
 
completionHandlers.first?()
print(instance.x)   // 100
{% endhighlight %}

#### Autoclosures(@autoclosure)

 - 此特性会让代码难读，在此略过

----------

## 结构体

 - 结构体会自动生成一个构造函数，以其成员名作为变量名
 - 结构体是值类型(value type)
 - 不可继承
 - 不可类型转换
 - String, Array, Dictionary都是结构体

{% highlight swift %}
struct Resolution {
    var width = 0
    var height = 0
}

let someResolution = Resolution()
print(someResolution.width)

// 自动生成的构造函数
let vga = Resolution(width: 640, height: 480)
{% endhighlight %}


----------


## 类

 - 类是引用类型(reference type)
 - 可继承
 - 可类型转换

{% highlight swift %}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}

let someVideoMode = VideoMode()
print(someVideoMode.resolution.width)
someVideoMode.resolution.width = 1280
{% endhighlight %}

#### Identity Operators (===, !==)

由于类是引用类型，有可能有多个变量指向同一个实例，所以Swift提供了如下操作符：

 - Identical to (===)，指向同一个实例
 - Not identical to (!==)


----------


## 枚举(Enumeration)

 - 枚举是值类型(value type)

{% highlight swift %}
enum CompassPoint {
    // enumeration definition goes here
    case North
    case South
    // ...
}

// Multiple cases can appear on a single line, separated by commas
enum Planet {
    case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}

var directionToHead = CompassPoint.West
directionToHead = .East // dot syntax

switch directionToHead {
case .North:
    ...
case .South:
    ...
default:
    ...
}
{% endhighlight %}

#### Associated Values

 - 枚举变量可以关联额外的值
 - 额外值的类型可以相互不同

{% highlight swift %}
enum Barcode {
    case UPCA(Int, Int, Int, Int)   // associated tuple type
    case QRCode(String)
}

var productBarcode = Barcode.UPCA(8, 85909, 51226, 3)
productBarcode = .QRCode("ABCDEFGHIJKLMNOP")

// 提取额外值
switch productBarcode {
case .UPCA(let numberSystem, let manufacturer, let product, let check):
    print("UPC-A: \(numberSystem), \(manufacturer), \(product), \(check).")
case .QRCode(var productCode):
    print("QR code: \(productCode).")
}

// 如果所有的额外值都被当做常量(let)、变量(var)提取，则可以简写，如下：
switch productBarcode {
case let .UPCA(numberSystem, manufacturer, product, check):
    print("UPC-A: \(numberSystem), \(manufacturer), \(product), \(check).")
case var .QRCode(productCode):
    print("QR code: \(productCode).")
}
{% endhighlight %}

#### Raw Values

 - 必须是同一种类型
 - 支持的类型为：字符串，字符，数字(整数、小数)
 - 不可重复

{% highlight swift %}
enum ASCIIControlCharacter: Character {
    case Tab = "\t"
    case LineFeed = "\n"
    case CarriageReturn = "\r"
}
{% endhighlight %}

**Implicitly Assigned Raw Values**

 - Int类型，默认第一个为0，依次递增
 - String类型，默认为变量名

{% highlight swift %}
enum Planet: Int {
    // Venus.rawValue == 2
    case Mercury = 1, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
}
enum CompassPoint: String {
    case North, South, East, West
}

let earthsOrder = Planet.Earth.rawValue // 3
let sunsetDirection = CompassPoint.West.rawValue // "West"
{% endhighlight %}

**Initializing from a Raw Value**

 - 若定义的时候指定了raw value的类型，则枚举会自动生成一个初始化函数，参数为指定的类型

{% highlight swift %}
let possiblePlanet = Planet(rawValue: 7)
// possiblePlanet is of type Planet? and equals Planet.Uranus
{% endhighlight %}

#### Recursive Enumerations (indirect)

 - 递归枚举用自己的实例作为枚举变量的associated value
 - 在枚举变量前加上indirect来标记它为递归枚举

{% highlight swift %}
enum ArithmeticExpression {
    case Number(Int)
    indirect case Addition(ArithmeticExpression, ArithmeticExpression)
    indirect case Multiplication(ArithmeticExpression, ArithmeticExpression)
}
{% endhighlight %}

也可以将indirect放在枚举定义的前面

{% highlight swift %}
indirect enum ArithmeticExpression {
    case Number(Int)
    case Addition(ArithmeticExpression, ArithmeticExpression)
    case Multiplication(ArithmeticExpression, ArithmeticExpression)
}
{% endhighlight %}

递归枚举应用示例

{% highlight swift %}
let five = ArithmeticExpression.Number(5)
let four = ArithmeticExpression.Number(4)
let sum = ArithmeticExpression.Addition(five, four)
let product = ArithmeticExpression.Multiplication(sum, ArithmeticExpression.Number(2))

func evaluate(expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .Number(value):
        return value
    case let .Addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .Multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}
 
print(evaluate(product))    // 18
{% endhighlight %}

----------

## 控制流(Control Flow)

#### for in

{% highlight swift %}
for index in 1...5 {
    ...
}
for _ in 1...5 {
    ...
}

// Array
let names = ["Anna", "Alex", "Brian", "Jack"]
for name in names {
    ...
}

// Dictionary
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
for (animalName, legCount) in numberOfLegs {
    ...
}
{% endhighlight %}

#### while

{% highlight swift %}
while condition {
    ...
}

repeat {
    ...
} while condition
{% endhighlight %}

#### if

{% highlight swift %}
if condition {
    ...
} else if condition2 {
    ...
} else {
    ...
}
{% endhighlight %}

#### switch

不需要break，只要匹配到对应的case，就停止匹配

{% highlight swift %}
switch value {
case value1:
    ...
case value2, value3:
    ...
default:
    ...
}
{% endhighlight %}

Interval Matching

{% highlight swift %}
let approximateCount = 62
var naturalCount: String
switch approximateCount {
case 0:
    naturalCount = "no"
case 1..<5:
    naturalCount = "a few"
case 5..<12:
    naturalCount = "several"
case 12..<100:
    naturalCount = "dozens of"
case 100..<1000:
    naturalCount = "hundreds of"
default:
    naturalCount = "many"
}
print("There are \(naturalCount) moons orbiting Saturn.")
// Prints "There are dozens of moons orbiting Saturn."
{% endhighlight %}

Tuples

{% highlight swift %}
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// Prints "(1, 1) is inside the box"
{% endhighlight %}

Value Bindings

{% highlight swift %}
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// Prints "on the x-axis with an x value of 2"
{% endhighlight %}

Where

{% highlight swift %}
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
// Prints "(1, -1) is on the line x == -y"
{% endhighlight %}

#### Control Transfer Statements

 - continue(loop)
 - break(switch, loop)
 - return
 - fallthrough(switch)
 - throw

----------


## 集合类型(Collection Types)

 - Array
 - Set
 - Dictionary
 
![](/res/img/20160418/3.c.png)
 
#### Array(有序)
 
{% highlight swift %}
var someInts: Array<Int>
var someInts: [Int]    // 简写形式(preferred)
 
var someInts = [Int]()
print("someInts is of type [Int] with \(someInts.count) items.")
someInts.append(3)
someInts = []
// someInts is now an empty array, but is still of type [Int]

var threeDoubles = [Double](count: 3, repeatedValue: 0.0)
// threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
var anotherThreeDoubles = [Double](count: 3, repeatedValue: 2.5)
// anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]
var sixDoubles = threeDoubles + anotherThreeDoubles
// sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]

var shoppingList: [String] = ["Eggs", "Milk"]
var shoppingList = ["Eggs", "Milk"]
if shoppingList.isEmpty {
    ...
}
var firstItem = shoppingList[0]
shoppingList[0] = "Six eggs"

shoppingList.insert("Maple Syrup", atIndex: 0)
let mapleSyrup = shoppingList.removeAtIndex(0)
let apples = shoppingList.removeLast()

for value in shoppingList {
    print(value)
}
for (index, value) in shoppingList.enumerate() {
    print("Item \(index + 1): \(value)")
}
{% endhighlight %}

#### Set(无序，不重复)

当1)顺序不重要；2)确保元素不重复的时候，可以选择Set

{% highlight swift %}
var letters = Set<Character>()
letters.count
letters.isEmpty
letters.insert("a")
letters.remove("a") // return removed value, or nil(not found)
letters.removeAll()
letters.contains("a")   // return Bool
letters = []

// Set类型不可省略，因为和Array的简写形式无法区分
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]

for genre in favoriteGenres {
    ...
}
for genre in favoriteGenres.sort() {
    ...
}

let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]
{% endhighlight %}

##### 运算

![](/res/img/20160418/4.c.png)

{% highlight swift %}
oddDigits.union(evenDigits).sort()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersect(evenDigits).sort()
// []
oddDigits.subtract(singleDigitPrimeNumbers).sort()
// [1, 9]
oddDigits.exclusiveOr(singleDigitPrimeNumbers).sort()
// [1, 2, 9]
{% endhighlight %}

##### 关系

![](/res/img/20160418/5.c.png)

{% highlight swift %}
let houseAnimals: Set = ["a", "b"]
let farmAnimals: Set = ["c", "d", "e", "a", "b"]
let cityAnimals: Set = ["f", "g"]
 
houseAnimals.isSubsetOf(farmAnimals)
// true
farmAnimals.isSupersetOf(houseAnimals)
// true
farmAnimals.isDisjointWith(cityAnimals)
// true
{% endhighlight %}


#### Dictionary(无序，键值对)

{% highlight swift %}
var namesOfIntegers: Dictionary<Int, String> = Dictionary<Int, String>()
var namesOfIntegers = [Int: String]()
namesOfIntegers = [:]

var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]

airports.count
airports.isEmpty

airports["LHR"] = "London"  // add new
airports["LHR"] = "London Heathrow" // change

// updateValue returns old value, or nil(not existed)
if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
    ...
}

if let removedValue = airports.removeValueForKey("DUB") {
    ...
}

for (airportCode, airportName) in airports {
    ...
}
for airportCode in airports.keys {
    ...
}
for airportName in airports.values {
    ...
}

// make Array
let airportCodes = [String](airports.keys)
let airportNames = [String](airports.values)
{% endhighlight %}
