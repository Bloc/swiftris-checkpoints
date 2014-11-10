## Block Party

It's time to get a little abstract. In programming we use classes to represent real-world objects. In Swiftris, our objects are shapes and each shape is composed of four independent parts; building *blocks* if you will. Let's represent these independent blocks as objects. We'll do so by creating a brand new class named `Block`.

> Follow the same steps found in the *Array We Go* checkpoint to create a brand new Swift file named `Block`.

Replace its generated content with the following:

```objc(Block.swift)
-import Foundation
+import SpriteKit

// #1
+let NumberOfColors: UInt32 = 6

// #2
+enum BlockColor: Int, Printable {

// #3  
+    case Blue = 0, Orange, Purple, Red, Teal, Yellow

// #4
+    var spriteName: String {
+        switch self {
+        case .Blue:
+            return "blue"
+        case .Orange:
+          return "orange"
+        case .Purple:
+          return "purple"
+        case .Red:
+            return "red"
+        case .Teal:
+          return "teal"
+        case .Yellow:
+            return "yellow"
+        }
+    }

// #5
+    var description: String {
+        return self.spriteName
+    }

// #6
+    static func random() -> BlockColor {
+        return BlockColor(rawValue:Int(arc4random_uniform(NumberOfColors)))!
+    }
+}
```

As you may have guessed, this isn't everything you'll need to represent a block. The first part of this file defines an *enumeration*: `BlockColor`. Swiftris supports six different colors in total.

At **#1** we define precisely how many colors are available in Swiftris, `6`.

At **#2** we declare the enumeration. It is of type `Int` and it *implements* the `Printable` protocol.

[Learn more about Protocols in Swift](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html)

Classes, structures and enums which implement `Printable` are capable of generating human-readable strings when debugging or printing their value to the console.

At **#3** we provide the full list of enumerable options, one for each color beginning with `Blue` at `0` and ending at `5` with `Yellow`.

[Read about Swift's Enumerations](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html)

At **#4** we define a *computed property*, `spriteName`. A computed property is one that behaves like a typical variable, but when accessing it, a code block is invoked to generate its value each time. We could have put this inside of a function named, `getSpriteName()` but a computed property  is the better design choice.

`spriteName` returns the correct filename for the given color, we use a simple `switch…case` statement to accomplish this.

At **#5** we declare yet another computed property, `description`. This property is required if we are to adhere to the `Printable` protocol. Without it, our code will fail to compile. It simply returns the `spriteName` of the color which is more than enough to describe the object.

[Interested in reading more about computed properties? Try here.](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html)

Finally at **#6**, we declare a static function named `random()`. As you may have guessed, this function returns a random choice among the colors found in `BlockColor`. It creates a BlockColor using the `rawValue:Int` initializer to setup an enumeration which assigned to the numerical value passed into it, in our case numbers `0` through `5`.

Now that we've mastered the colors, let's do what we came here to do:

```objc(Block.swift)
    static func random() -> BlockColor {
        return BlockColor(rawValue:Int(arc4random_uniform(NumberOfColors)))!
    }
}

// #1
+class Block: Hashable, Printable {
// #2
+    // Constants
+    let color: BlockColor

// #3
+    // Properties
+    var column: Int
+    var row: Int
+    var sprite: SKSpriteNode?

// #4
+    var spriteName: String {
+        return color.spriteName
+    }

// #5
+    var hashValue: Int {
+        return self.column ^ self.row
+    }

// #6
+    var description: String {
+        return "\(color): [\(column), \(row)]"
+    }

+    init(column:Int, row:Int, color:BlockColor) {
+        self.column = column
+        self.row = row
+        self.color = color
+    }
+}

// #7
+func ==(lhs: Block, rhs: Block) -> Bool {
+    return lhs.column == rhs.column && lhs.row == rhs.row && lhs.color.rawValue == rhs.color.rawValue
+}
```

Really quick, we promise. At **#1**, `Block` is declared as a class which implements both the `Printable` and `Hashable` protocols. `Hashable` allows `Block` to be stored in `Array2D`.

At **#2** we define our `color` property as `let`, meaning once we assign it, it can no longer be re-assigned. A block should not be able to change colors mid-game unless you decide to make *Swiftris: Epileptic Adventures*.

[Dive deeper into `var` vs `let`](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html)

At **#3** we declare a `column` and `row`. These properties represent the location of the `Block` on our game board. The `SKSpriteNode` will represent the visual element of the `Block` to be used by `GameScene` when rendering and animating each `Block`.

At **#4** we provide a convenient shortcut for recovering the file name of the sprite to be used when displaying this `Block`. It effectively shortened our code from `block.color.spriteName` to `block.spriteName`.

At **#5**, we implement the `hashValue` calculated property, which is required in order to support the `Hashable` protocol. We return the exclusive-or of our `row` and `column` properties to generate a unique integer for each `Block`.

At **#6**, we implement `description` as we must do in order to comply with the `Printable` protocol. `Printable` object types can be placed in the middle of a string by surrounding them with `\(` and `)`. For a blue block at row 3, column 8, printing that `Block` will result in: "blue: [8, 3]".

[Dig into the `Printable` protocol](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/SwiftStandardLibraryReference/Printable.html)

Finally, at **#7** we create a custom operator- `==` - when comparing one `Block` with another. It returns `true` if and only if both `Block`s are in the same location and of the same color. This operator is required in order to support the `Hashable` protocol.

[Read more about custom operators in Swift](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AdvancedOperators.html)

Congratulations! You will likely *never* have to touch this file again. We managed to cram a lot of Swift techniques all into one tiny place and we're quite proud of that at Bloc – no 'k'.
