---
title: SwiftNotes
date: 2020-01-15 21:29:12
abstract: "The Learning Notes of \"Swift Programming Language\"."
tags: ["Swift", "Notes"]
categories: "Programming"
---

# A Swift Tour

> For the _**P(number)**_ in soucre code, means the page of counterpart details in the book _**"The Swift Programming Language (Swift 5)"**_ under _**Apple Books**_ verson. If need to say, I have to tell you I read the book with iPad, whose model is _**iPad Air (Generation 3, 10.5inch)**_.


## <font color='blue'>Start with _**"Hello, World!"**_</font>
```
// P5
print("Hello, World!")
```

## <font color='blue'>Simple Values</font>
```
// P5
var myVariable = 42
myVariable = 50
let myConstant = 42

// P6
let implicitInteger = 70
let implicitDouble = 70.0
let explicitDouble: Double = 70

// EXPERIMENT:
//   Create a constant with an explicit type of `Float` and a value of `4`.
let explicitFloat: Float = 4

let label = "The width is "
let width = 94
let widthLabel = label + String(width)

// EXPERIMENT:
//   Try removing the conversion to `String` from the last line. What error
// do you get?
//
// ANSWER:
//   Error: "Binary operator '+' cannot be applied to operands of type 'String' 
// and 'Int'"

let apples = 3
let oranges = 5
let appleSummary = "I have \(apples) apples."
let fruitSummary = "I have \(apples + oranges) pieces of fruit."

// EXPERIMENT:
//   Use `\()` to include a floating-point calculation in a string and to
// include someone's name in a greeting.
let math_pi: Float = 3.14159
let math_e: Float = 2.71828
let author = "Modnar"
let expStr = "Sum of PI and e is \(math_pi + math_e)."
let expGreeting = "Hello, \(author)!"

// P7
let quotation = """
I said "I have \(apples) apples."
And then I said "I have \(apples + oranges) pieces of fruit."
"""

var shoppingList = ["catfish", "water", "tulips"]
shoppingList[1] = "bottle of water"

var occupations = [
    "Malcolm": "Captain",
    "Kaylee": "Mechanic",
]
occupations["Jayne"] = "Public Relations"

shoppingList.append("blue paint")
print(shoppingList)

let emptyArray = [String]()
let emptyDictionary = [String: Float]()

shoppingList = []
occupations = [:]
```

## <font color='blue'>Control Flow</font>
```
// P8
let individualScores = [75, 43, 103, 87, 12]
var teamScore = 0
for score in individualScores {
    if score > 50 {
        teamScore += 3
    } else {
        teamScore += 1
    }
}
print(teamScore)

var optionalString: String? = "Hello"
print(optionalString == nil)

var optionalName: String? = "John Appleseed"
// var optionalName: String? = nil // For the EXPERIMENT.
var greeting = "Hello!"
if let name = optionalName {
    greeting = "Hello, \(name)"
} else { // For the EXPERIMENT.
    greeting = "Hello, Modnar!"
}
print(greeting)

// EXPERIMENT:
//   Change `optionalName` to `nil`. What greeting do you get? Add an `else`
// clause that sets a different greeting if `optionalName` is `nil`.
//
// ANSWER:
//   Prints "Hello!"
//   For the `else` clause, see the source code.

// P9
let nickName: String? = nil
let fullName: String = "John Appleseed"
let informalGreeting = "Hi \(nickName ?? fullName)"

let vegetable = "red pepper"
switch vegetable {
case "celery":
    print("Add some raisins and make ants on a log.")
case "cucumber", "watercress":
    print("That would make a good tea sandwich.")
case let x where x.hasSuffix("pepper"):
    print("Is it a spicy \(x)?")
default:
    print("Everything tastes good in soup.")
}

// EXPERIMENT:
//   Try removing the default case. What error do you get?
//
// ANSWER:
//   Error: "Switch must be exhaustive".

// P10
let interestingNumbers = [
    "Prime": [2, 3, 5, 7, 11, 13],
    "Fibnoacci": [1, 1, 2, 3, 5, 8],
    "Square": [1, 4, 9, 16, 25],
]
var largest = 0
var kindLabel = ""
for (kind, numbers) in interestingNumbers {
    for number in numbers {
        if number > largest {
            // EXPERIMENT:
            //   Add another variable to keep track of which kind of number was
            // the largest, as well as what that largest number was.
            kindLabel = kind
            largest = number
        }
    }
}
print(largest)

// P11
var n = 2
while n < 100 {
    n *= 2
}
print(n)

var m = 2
repeat {
    m *= 2
} while m < 100
print(m)

var total = 0
for i in 0..<4 {
    total += i
}
print(total)
```

## <font color='blue'>Functions and Closures</font>
```
// P12
func greet(person: String, day: String) -> String {
    return "Hello \(person), today is \(day)."
}
greet(person: "Bob", day: "Tuesday")

// EXPEIMENT:
//   Remove the `day` parameter. Add a parameter to include today's lunch 
// special in the greeting.
func greet(person: String, lunch: String) -> String {
    return "Hello \(person), today's lunch is \(lunch)!"
}

func greet(_ person: String, on day: String) -> String {
    return "Hello \(person), today is \(day)."
}
greet("John", on: "Wednesday")

// P13
func calculateStatistics(scores: [Int]) -> (min: Int, max: Int, sum: Int) {
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
let statistics = calculateStatistics(scores: [5, 3, 100, 3, 9])
print(statistics.sum)
print(statistics.2)

func returnFifteen() -> Int {
    var y = 10
    func add() {
        y += 5
    }
    add()
    return y
}
returnFifteen()

// P14
func makeIncrementer() -> ((Int) -> Int) {
    func addOne(number: Int) -> Int {
        return 1 + number
    }
    return addOne
}
var increment = makeIncrementer()
increment(7)

func hasAnyMaches(list: [Int], condition: (Int) -> Bool) -> Bool {
    for item in list {
        if condition(item) {
            return true
        }
    }
    return false
}
func lessThanTen(number: Int) -> Bool {
    return number < 10
}
var numbers = [20, 19, 7, 12]
hasAnyMaches(list: numbers, condition: lessThanTen)

// P15
numbers.map({ (number: Int) -> Int in
    let result = 3 * number
    return result
})

// EXPERIMENT:
//   Rewrite the closure to return zero for all odd numbers.
numbers.map({ (number: Int) -> Int in
    let result = (number % 2 == 0) ? number : 0
    return result
})

let mappedNumbers = numbers.map({ number in 3 * number })
print(mappedNumbers)

let sortedNumbers = numbers.sorted { $0 > $1 }
print(sortedNumbers)
```

## <font color='blue'>Objects and Classes</font>
```
// P16
class Shape {
    var numberOfSides = 0
    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
    // EXPERIMENT:
    //   Add a constant property with `let`, and add another method that takes an 
    // argument.
    let constVal = 0
    func printMessage(message: String) -> String {
        return "Message details: \(message)"
    }
}

var shape = Shape()
shape.numberOfSides = 7
var shapeDescription = shape.simpleDescription()

class NamedShape {
    var numberOfSides = 0
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
}

// P17
class Square: NamedShape {
    var sideLength: Double
    
    init(sideLength: Double, name: String) {
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 4
    }
    
    func area() -> Double {
        return sideLength * sideLength
    }
    
    override func simpleDescription() -> String {
        return "A square with sides of length \(sideLength)."
    }
}
let test = Square(sideLength: 5.2, name: "my test square")
test.area()
test.simpleDescription()

// P18
class EquilateraTriangle: NamedShape {
    var sideLength: Double = 0.0
    
    init(sideLength: Double, name: String) {
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
    
    override func simpleDescription() -> String {
        return "An equilateral triangle with sides of length \(sideLength)."
    }
}

// EXPERIMENT:
//   Make another subclass of `NamedShape` called `Circle` that takes a radius and a
// name as arguments to its initializer. Implement an `area()` and a
// `simpleDescription()` method on the `Circle` class.
class Circle: NamedShape {
    var radius: Double = 0.0
    
    init(radius: Double, name: String) {
        self.radius = radius
        super.init(name: name)
    }
    
    func area() -> Double {
        return 3.14 * radius * radius
    }
    
    override func simpleDescription() -> String {
        return "A circle with radius \(radius)."
    }
}

var triangle = EquilateraTriangle(sideLength: 3.1, name: "a triangle")
print(triangle.perimeter)
triangle.perimeter = 9.9
print(triangle.sideLength)

// P20
class TriangleAndSquare {
    var triangle: EquilateraTriangle {
        willSet {
            square.sideLength = newValue.sideLength
        }
    }
    var square: Square {
        willSet {
            triangle.sideLength = newValue.sideLength
        }
    }
    init(size: Double, name: String) {
        square = Square(sideLength: size, name: name)
        triangle = EquilateraTriangle(sideLength: size, name: name)
    }
}
var triangleAndSquare = TriangleAndSquare(size: 10, name: "another test shape")
print(triangleAndSquare.square.sideLength)
print(triangleAndSquare.triangle.sideLength)
triangleAndSquare.square = Square(sideLength: 50, name: "larger square")
print(triangleAndSquare.triangle.sideLength)

let optionalSquare: Square? = Square(sideLength: 2.5, name: "optional square")
let sideLength = optionalSquare?.sideLength
```

## <font color='blue'>Enumerations and Structures</font>
```
// P21
enum Rank: Int {
    case ace = 1
    case two, three, four, five, six, seven, eight, nine, ten
    case jack, queen, king
    
    func simpleDescription() -> String {
        switch self {
        case .ace:
            return "ace"
        case .jack:
            return "jack"
        case .queen:
            return "queen"
        case .king:
            return "king"
        default:
            return String(self.rawValue)
        }
    }
}
let ace = Rank.ace
let aceRawValue = ace.rawValue

// EXPERIMENT:
//   Write a function that compares two `Rank` values by comparing their raw values.
func compareRankByRawValue(v1: Rank, v2: Rank) -> Int {
    return v1.rawValue - v2.rawValue
}
print(compareRankByRawValue(v1: Rank.ace, v2: Rank.queen))

// P22
if let convertedRank = Rank(rawValue: 3) {
    // let threeDescription = convertedRank.simpleDescription()
    let _ = convertedRank.simpleDescription()
}

enum Suit {
    case spades, hearts, diamonds, clubs
    
    func simpleDescription() -> String {
        switch self {
        case .spades:
            return "spades"
        case .hearts:
            return "hearts"
        case .diamonds:
            return "diamonds"
        case .clubs:
            return "clubs"
        }
    }
    
    // EXPERIMENT
    //   Add a `color()` method to `Suit` that returns "black" for spades and clubs,
    // and returns "red" for hearts and diamonds.
    func color() -> String {
        switch self {
        case .spades, .clubs:
            return "black"
        case .hearts, .diamonds:
            return "heart"
        }
    }
}
let hearts = Suit.hearts
let heartsDescription = hearts.simpleDescription()

// P23
enum ServerResponse {
    case result(String, String)
    case failure(String)
    // EXPERIMENT:
    //   Add a third case to `ServerResponse` and to the switch.
    case offline(Void)
}

let success = ServerResponse.result("6:00 am", "8:09 pm")
let failure = ServerResponse.failure("Out of cheese.")

switch success {
case let .result(sunrise, sunset):
    print("Sunrise is at \(sunrise) and sunset is at \(sunset)")
case let .failure(message):
    print("Failure... \(message)")
case .offline():
    // Not use `case let .offline():` :
    // 'let' pattern has no effect; sub-pattern didn't bind any variables
    print("Your device is offline...")
}

// P24
struct Card {
    var rank: Rank
    var suit: Suit
    func simpleDescription() -> String {
        return "The \(rank.simpleDescription()) of \(suit.simpleDescription())"
    }
}
let threeOfSpades = Card(rank: .three, suit: .spades)
let threeOfSpadesDescription = threeOfSpades.simpleDescription()

// EXPERIMENT:
//   Write a function that returns an array containing a full deck of cards, with one
// card of each combaination of rank and suit.
func getPokerCards() -> [Card] {
    var cards = [Card]()
    for rank in 1...13 {
        cards.append(Card(rank: Rank(rawValue: rank) ?? .ace, suit: .spades))
        cards.append(Card(rank: Rank(rawValue: rank) ?? .ace, suit: .hearts))
        cards.append(Card(rank: Rank(rawValue: rank) ?? .ace, suit: .clubs))
        cards.append(Card(rank: Rank(rawValue: rank) ?? .ace, suit: .diamonds))
    }
    for card in cards {
        print(card.simpleDescription())
    }
    return cards
}
```

## <font color='blue'>Protocols and Extensions</font>
```
protocol ExampleProtocol {
    var simpleDescription: String { get }
    mutating func adjust()
    // mutating func anotherFunc() // For later Experiment.
}

// P25
class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "A very simple class."
    var anathorProperty: Int = 69105
    func adjust() {
        simpleDescription += " Now 100% adjusted."
    }
}
var a = SimpleClass()
a.adjust()
let aDescription = a.simpleDescription

struct SimpleStructure: ExampleProtocol {
    var simpleDescription: String = "A simple structure"
    mutating func adjust() {
        simpleDescription += " (adjusted)"
    }
}
var b = SimpleStructure()
b.adjust()
let bDescription = b.simpleDescription

// EXPERIMENT:
//   Add another requirement to `ExampleProtocol`. What changes do you need to make to
// `SimpleClass` and `SimpleStructure` so that they still conform to the protocol?
//
// ANSWER:
//   Like the function `adjust()`, the `anotherFunc()` also need the same
// implementation under `class` and `struct`.

// P26
extension Int: ExampleProtocol {
    var simpleDescription: String {
        return "The number \(self)"
    }
    mutating func adjust() {
        self += 42
    }
}
print(7.simpleDescription)

// EXPERIMENT:
//   Write an extension for the `Double` type that adds an `absoluteValue` property.
extension Double {
    var absoluteValue: Double {
        return self < 0 ? -self : self
    }
}
print((-10.0).absoluteValue)
print((82.3).absoluteValue)

let protocolValue: ExampleProtocol = a
print(protocolValue.simpleDescription)
```

## <font color='blue'>Error Handling</font>
```
// P27
enum PrinterError: Error {
    case outOfPaper
    case noToner
    case onFire
}

func send(job: Int, toPrinter printerName: String) throws -> String {
    if printerName == "Never Has Toner" {
        throw PrinterError.noToner
    }
    return "Job sent"
}

do {
    let printerResponse = try send(job: 1040, toPrinter: "Bi Sheng")
    // let printerResponse = try send(job: 1040, toPrinter: "Never Has Toner")
    print(printerResponse)
} catch {
    print(error)
}

// EXPERIMENT:
//   Change the printer name to "Never Has Toner", so that the `send(job:toPrinter:)`
// function throws an error.
//
// ANSWER:
//   See the comment in source code.

// P28
do {
    let printerResponse = try send(job: 1040, toPrinter: "Gutenberg")
    print(printerResponse)
} catch PrinterError.onFire {
    print("I'll just put this over here, with the rest of the fire.")
} catch let printerError as PrinterError {
    print("Printer error: \(printerError).")
} catch {
    print(error)
}

// EXPERIMENT:
//   Add code to throw an error inside the `do` block. What kind of error do you 
// need to throw so that the error is handled by the first `catch` block? What 
// about the second and third blocks?
//
// ANSWER:
//   For the first `catch` block, we should define the action to throw
// `PrinterError.onFire` in the function `send` body, and call the counterpart
// function to get the Error.
//   In the same way, for the second block, we can use the same way to get the
// `PrinterError` except the specific PrinterError type which has been caught.
//   Besides, for the third block, we should make the code throw the error whose
// type is not in `PrinterError`.

let printerSuccess = try? send(job: 1884, toPrinter: "Mergenthaler")
let printerFailure = try? send(job: 1885, toPrinter: "Never Has Toner")

// P29
var fridgeIsOpen = false
let fridgeContent = ["milk", "eggs", "leftovers"]

func fridgeContains(_ food: String) -> Bool {
    fridgeIsOpen = true
    defer {
        fridgeIsOpen = false
    }
    
    let result = fridgeContent.contains(food)
    return result
}
fridgeContains("banana")
print(fridgeIsOpen)
```

## <font color='blue'>Generics</font>
```
func makeArray<Item>(repeating item: Item, numberOfTimes: Int) -> [Item] {
    var result = [Item]()
    for _ in 0..<numberOfTimes {
        result.append(item)
    }
    return result
}
makeArray(repeating: "knock", numberOfTimes: 4)

// P30
enum OptionalValue<Wrapped> {
    case none
    case some(Wrapped)
}
var possibleInteger: OptionalValue<Int> = .none
possibleInteger = .some(100)

func anyCommonElements<T: Sequence, U: Sequence>(_ lhs: T, _ rhs: U) -> Bool
    where T.Element: Equatable, T.Element == U.Element
{
    for lhsItem in lhs {
        for rhsItem in rhs {
            if lhsItem == rhsItem {
                return true
            }
        }
    }
    return false
}
anyCommonElements([1, 2, 3], [3])

// EXPERIMENT:
//   Modify the anyCommonElements(_:_:) function to make a function that returns an
// array of the elements that any two sequences have in common.
func commonElements<T: Sequence, U: Sequence>(_ lhs: T, _ rhs: U) -> [T.Element]
    where T.Element: Equatable, T.Element == U.Element
{
    var ans = [T.Element]()
    for lhsItem in lhs {
        for rhsItem in rhs {
            if lhsItem == rhsItem {
                ans.append(lhsItem)
            }
        }
    }
    return ans
}
commonElements([1, 2, 3, 4], [2, 4, 5])
```
