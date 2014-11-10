> “The object of art is to give life shape“<br>
> -- William Shakespeare

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-tetronimoes.png)</center>

## Shaping Up

Without shape, our blocks are aimless colored squares longing for purpose in their short-lived binary lives. Let's put them to use by creating a `Shape` class. `Shape` will define the recognizable [Tetromino](http://en.wikipedia.org/wiki/Tetromino) pieces we all know and love. You know the drill; create a new file named `Shape.swift` and replace its contents with the following:

```objc(Shape.swift)
-import Foundation
+import SpriteKit

+let NumOrientations: UInt32 = 4

+enum Orientation: Int, Printable {
+    case Zero = 0, Ninety, OneEighty, TwoSeventy

+    var description: String {
+        switch self {
+            case .Zero:
+                return "0"
+            case .Ninety:
+                return "90"
+            case .OneEighty:
+                return "180"
+            case .TwoSeventy:
+                return "270"
+        }
+    }

+    static func random() -> Orientation {
+        return Orientation(rawValue:Int(arc4random_uniform(NumOrientations)))!
+    }

// #1
+    static func rotate(orientation:Orientation, clockwise: Bool) -> Orientation {
+        var rotated = orientation.rawValue + (clockwise ? 1 : -1)
+        if rotated > Orientation.TwoSeventy.rawValue {
+            rotated = Orientation.Zero.rawValue
+        } else if rotated < 0 {
+            rotated = Orientation.TwoSeventy.rawValue
+        }
+        return Orientation(rawValue:rotated)!
+    }
+}
```

This first piece of code should appear familiar. Once again we created an enumeration helper which will define the shape's orientation. A Tetromino can face one of four directions at any given point, we shall refer to them as `0`, `90`, `180` and `270`. Imagine a circle whose degrees begin at the top and continue clockwise.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-rotation-degrees.png)</center>

At 0˚ the piece is at origin and as it rotates clockwise its degree advances along the circumference. That's how we're going to define a shape's orientation. At **#1**, we provided a method capable of returning the next orientation when traveling either clockwise or counterclockwise.

Let's write the shape class itself, shall we?

```objc(Shape.swift)
+// The number of total shape varieties
+let NumShapeTypes: UInt32 = 7

+// Shape indexes
+let FirstBlockIdx: Int = 0
+let SecondBlockIdx: Int = 1
+let ThirdBlockIdx: Int = 2
+let FourthBlockIdx: Int = 3

+class Shape: Hashable, Printable {
+    // The color of the shape
+    let color:BlockColor

+    // The blocks comprising the shape
+    var blocks = Array<Block>()
+    // The current orientation of the shape
+    var orientation: Orientation
+    // The column and row representing the shape's anchor point
+    var column, row:Int

+    // Required Overrides
// #1
+    // Subclasses must override this property
+    var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
+        return [:]
+    }
// #2
+    // Subclasses must override this property
+    var bottomBlocksForOrientations: [Orientation: Array<Block>] {
+        return [:]
+    }
// #3
+    var bottomBlocks:Array<Block> {
+        if let bottomBlocks = bottomBlocksForOrientations[orientation] {
+            return bottomBlocks
+        }
+        return []
+    }

+    // Hashable
+    var hashValue:Int {
// #4
+        return reduce(blocks, 0) { $0.hashValue ^ $1.hashValue }
+    }

+    // Printable
+    var description:String {
+        return "\(color) block facing \(orientation): \(blocks[FirstBlockIdx]), \(blocks[SecondBlockIdx]), \(blocks[ThirdBlockIdx]), \(blocks[FourthBlockIdx])"
+    }

+    init(column:Int, row:Int, color: BlockColor, orientation:Orientation) {
+        self.color = color
+        self.column = column
+        self.row = row
+        self.orientation = orientation
+        initializeBlocks()
+    }

// #5
+    convenience init(column:Int, row:Int) {
+        self.init(column:column, row:row, color:BlockColor.random(), orientation:Orientation.random())
+    }
+}

+func ==(lhs: Shape, rhs: Shape) -> Bool {
+    return lhs.row == rhs.row && lhs.column == rhs.column
+}
```

Your project won't compile at the moment and you will certainly see some errors, but we'll get it fixed soon. Both **#1** and **#2** introduce some Swift tools that you'll certainly be interested in. First and foremost we have written these two computed properties and left their results empty. This was done on purpose such that our actual shape classes will `override` them in their respective implementations. You'll see what we mean shortly.

**#1**, `blockRowColumnPositions` defines a computed **Dictionary**. A dictionary is defined with square braces – `[…]` – and maps one type of object to another. The first object type listed defines the **key** and the second, a **value**. Keys map one-to-one with values and multiple copies of a single key may not exist.

[Peruse Swift Dictionary details here.](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/SwiftStandardLibraryReference/Dictionary.html)

We access dictionary values similarly to those of an array by employing square braces. However, our subscripts are now **keys**, and therefore in the case of `blockRowColumnPositions`, they are `Orientation` objects. The values found in this dictionary are peculiar as well, `Array<(columnDiff: Int, rowDiff: Int)>`. What the heck is that?

It's a regular Swift array, its type is a **tuple**, pronounced *too-pūll*. A tuple is perfect for passing or returning multiple variables without defining a custom struct. Our tuple has two pieces of data but the number allowed is indefinite. Both pieces of data are of type `Int`, the first is named `columnDiff` and the second is `rowDiff`. Here's a sample accessor statement for this dictionary:

```objc
let arrayOfDiffs = blockRowColumnPositions[Orientation.0]!
let columnDifference = arrayOfDiffs[0].columnDiff
```

Elements found within a dictionary are optional by default, therefore we must unwrap them using the `!` symbol. To access the first element's `columnDiff` value, we index the array at `0` to recover the first tuple and use dot syntax to retrieve our desired variable.

[Can't get enough of tuples? We know that feeling.](https://developer.apple.com/library/prerelease/mac/documentation/Swift/Conceptual/Swift_Programming_Language/Types.html)

Both **#1** and **#2** return empty values, they're meant for subclasses to provide meaningful data later. At **#3** we wrote a complete computed property which is designed to return the bottom blocks of the shape at its current orientation. This will be useful later when our blocks get physical and start contacting walls and each other.

At **#4** we use the `reduce<S : Sequence, U>(sequence: S, initial: U, combine: (U, S.GeneratorType.Element) -> U) -> U` method to iterate through our entire `blocks` array. We exclusively-or each block's `hashValue` together to create a single `hashValue` for the `Shape` they comprise.

At **#5** we introduce a special type of initializer. A `convenience` initializer must call down to a standard initializer or otherwise your class will fail to compile. We've placed this one here in order to simplify the initialization process for users of the `Shape` class. It assigns the given `row` and `column` values while generating a random color and a random orientation.

[We get it, it'd be *convenient* to read more about those.](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html)

Your `Shape` class has some build errors, let's fix them.

```objc(Shape.swift)
    convenience init(column:Int, row:Int) {
        self.init(column:column, row:row, color:BlockColor.random(), orientation:Orientation.random())
    }

// #1
+    final func initializeBlocks() {
// #2
+        if let blockRowColumnTranslations = blockRowColumnPositions[orientation] {
+            for i in 0..<blockRowColumnTranslations.count {
+                let blockRow = row + blockRowColumnTranslations[i].rowDiff
+                let blockColumn = column + blockRowColumnTranslations[i].columnDiff
+                let newBlock = Block(column: blockColumn, row: blockRow, color: color)
+                blocks.append(newBlock)
+            }
+        }
+    }
}

func ==(lhs: Shape, rhs: Shape) -> Bool {
    return lhs.row == rhs.row && lhs.column == rhs.column
}
```

At **#1** we defined a `final` function which means it cannot be overridden by subclasses. This implementation of `initializeBlocks()` is the only one allowed by `Shape` and its subclasses.

At **#2** we introduced conditional assignments. This `if` conditional first attempts to assign an array into `blockRowColumnTranslations` after extracting it from the computed dictionary property. If one is *not* found, the `if` block is not executed.

The following code sample is equivalent to what we've written:

```objc
let blockRowColumnTranslations = blockRowColumnPositions[orientation]
if blockRowColumnTranslations != nil {
    // Code…
}
```

### Subclasses

You've written a solid `Shape` class, yet it hasn't truly defined any possible Tetrominoes for us to play with. `Shape` is merely a generic tool meant to support an infinite number of shapes. Let's define the seven shapes which Swiftris will allow. *Create* and *code* the following subclass of `Shape`, `SquareShape`:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-square.png)</center>

```objc(SquareShape.swift)
-import Foundation
+class SquareShape:Shape {
+    /*
// #1
+        | 0•| 1 |
+        | 2 | 3 |

+    • marks the row/column indicator for the shape

+    */

+    // The square shape will not rotate

// #2
+    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
+        return [
+            Orientation.Zero: [(0, 0), (1, 0), (0, 1), (1, 1)],
+            Orientation.OneEighty: [(0, 0), (1, 0), (0, 1), (1, 1)],
+            Orientation.Ninety: [(0, 0), (1, 0), (0, 1), (1, 1)],
+            Orientation.TwoSeventy: [(0, 0), (1, 0), (0, 1), (1, 1)]
+        ]
+    }

// #3
+    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
+        return [
+            Orientation.Zero:       [blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
+            Orientation.OneEighty:  [blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
+            Orientation.Ninety:     [blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
+            Orientation.TwoSeventy: [blocks[ThirdBlockIdx], blocks[FourthBlockIdx]]
+        ]
+    }
+}
```

Thanks to the `Shape` class, defining Tetrominoes with a subclass is fairly trivial. Subclasses must simply provide the distance of each block from the shape's `row` and `column` location with respect to each possible orientation. A square shape is the easiest, it will not rotate at all since its shape is identical at every orientation. Consequently, its bottom blocks will always be the third and fourth block as described by the comments at **#1**.

At **#2** we've overridden the `blockRowColumnPositions` computed property to provide a full dictionary of tuple arrays. Each index of the arrays represents one of the four blocks ordered from block `0` to block `3`. For example, the top-left block location – block `0` – of a square is exactly identical to its `row` and `column` location. Therefore the tuple is `(0, 0)`, `0` column difference and `0` row difference. The second block is always `1` column to the right of the shape's given `column` value, therefore its tuple is always `(1, 0)`.

Finally, at **#3** we perform a similar `override` by providing a dictionary of bottom block arrays. As was stated earlier, a square shape does not rotate, therefore its bottom-most blocks are consistently the third and fourth blocks as indicated by the comments at **#1**.

You have the opportunity to write the remaining shapes yourself, they are: `TShape`, `LineShape`, `SShape`, `ZShape`, `LShape` and `JShape`. Or… you can read on and copy the remaining shapes into your project, try it yourself for a fun challenge. You can always come back and use our versions if you like.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-t.png)</center>

```objc(TShape.swift)
class TShape:Shape {
    /*
    Orientation 0

      • | 0 |
    | 1 | 2 | 3 |

    Orientation 90

      • | 1 |
        | 2 | 0 |
        | 3 |

    Orientation 180

      •
    | 1 | 2 | 3 |
        | 0 |

    Orientation 270

      • | 1 |
    | 0 | 2 |
        | 3 |

    • marks the row/column indicator for the shape

    */

    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
        return [
            Orientation.Zero:       [(1, 0), (0, 1), (1, 1), (2, 1)],
            Orientation.Ninety:     [(2, 1), (1, 0), (1, 1), (1, 2)],
            Orientation.OneEighty:  [(1, 2), (0, 1), (1, 1), (2, 1)],
            Orientation.TwoSeventy: [(0, 1), (1, 0), (1, 1), (1, 2)]
        ]
    }

    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
        return [
            Orientation.Zero:       [blocks[SecondBlockIdx], blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
            Orientation.Ninety:     [blocks[FirstBlockIdx], blocks[FourthBlockIdx]],
            Orientation.OneEighty:  [blocks[FirstBlockIdx], blocks[SecondBlockIdx], blocks[FourthBlockIdx]],
            Orientation.TwoSeventy: [blocks[FirstBlockIdx], blocks[FourthBlockIdx]]
        ]
    }
}
```

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-line.png)</center>

```objc(LineShape.swift)
class LineShape:Shape {
    /*
        Orientations 0 and 180:

            | 0•|
            | 1 |
            | 2 |
            | 3 |

        Orientations 90 and 270:

        | 0 | 1•| 2 | 3 |

    • marks the row/column indicator for the shape

    */

    // Hinges about the second block

    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
        return [
            Orientation.Zero:       [(0, 0), (0, 1), (0, 2), (0, 3)],
            Orientation.Ninety:     [(-1,0), (0, 0), (1, 0), (2, 0)],
            Orientation.OneEighty:  [(0, 0), (0, 1), (0, 2), (0, 3)],
            Orientation.TwoSeventy: [(-1,0), (0, 0), (1, 0), (2, 0)]
        ]
    }

    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
        return [
            Orientation.Zero:       [blocks[FourthBlockIdx]],
            Orientation.Ninety:     blocks,
            Orientation.OneEighty:  [blocks[FourthBlockIdx]],
            Orientation.TwoSeventy: blocks
        ]
    }
}
```

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-l.png)</center>

```objc(LShape.swift)
class LShape:Shape {
    /*

    Orientation 0

        | 0•|
        | 1 |
        | 2 | 3 |

    Orientation 90

          •
    | 2 | 1 | 0 |
    | 3 |

    Orientation 180

    | 3 | 2•|
        | 1 |
        | 0 |

    Orientation 270

          • | 3 |
    | 0 | 1 | 2 |

    • marks the row/column indicator for the shape

    Pivots about `1`

    */

    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
        return [
            Orientation.Zero:       [ (0, 0), (0, 1),  (0, 2),  (1, 2)],
            Orientation.Ninety:     [ (1, 1), (0, 1),  (-1,1), (-1, 2)],
            Orientation.OneEighty:  [ (0, 2), (0, 1),  (0, 0),  (-1,0)],
            Orientation.TwoSeventy: [ (-1,1), (0, 1),  (1, 1),   (1,0)]
        ]
    }

    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
        return [
            Orientation.Zero:       [blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
            Orientation.Ninety:     [blocks[FirstBlockIdx], blocks[SecondBlockIdx], blocks[FourthBlockIdx]],
            Orientation.OneEighty:  [blocks[FirstBlockIdx], blocks[FourthBlockIdx]],
            Orientation.TwoSeventy: [blocks[FirstBlockIdx], blocks[SecondBlockIdx], blocks[ThirdBlockIdx]]
        ]
    }
}
```

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-j.png)</center>

```objc(JShape.swift)
class JShape:Shape {
    /*

    Orientation 0

      • | 0 |
        | 1 |
    | 3 | 2 |

    Orientation 90

    | 3•|
    | 2 | 1 | 0 |

    Orientation 180

    | 2•| 3 |
    | 1 |
    | 0 |

    Orientation 270

    | 0•| 1 | 2 |
            | 3 |

    • marks the row/column indicator for the shape

    Pivots about `1`

    */

    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
        return [
            Orientation.Zero:       [(1, 0), (1, 1),  (1, 2),  (0, 2)],
            Orientation.Ninety:     [(2, 1), (1, 1),  (0, 1),  (0, 0)],
            Orientation.OneEighty:  [(0, 2), (0, 1),  (0, 0),  (1, 0)],
            Orientation.TwoSeventy: [(0, 0), (1, 0),  (2, 0),  (2, 1)]
        ]
    }

    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
        return [
            Orientation.Zero:       [blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
            Orientation.Ninety:     [blocks[FirstBlockIdx], blocks[SecondBlockIdx], blocks[ThirdBlockIdx]],
            Orientation.OneEighty:  [blocks[FirstBlockIdx], blocks[FourthBlockIdx]],
            Orientation.TwoSeventy: [blocks[FirstBlockIdx], blocks[SecondBlockIdx], blocks[FourthBlockIdx]]
        ]
    }
}
```

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-s.png)</center>

```objc(SShape.swift)
class SShape:Shape {
    /*

    Orientation 0

    | 0•|
    | 1 | 2 |
        | 3 |

    Orientation 90

      • | 1 | 0 |
    | 3 | 2 |

    Orientation 180

    | 0•|
    | 1 | 2 |
        | 3 |

    Orientation 270

      • | 1 | 0 |
    | 3 | 2 |

    • marks the row/column indicator for the shape

    */

    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
        return [
            Orientation.Zero:       [(0, 0), (0, 1), (1, 1), (1, 2)],
            Orientation.Ninety:     [(2, 0), (1, 0), (1, 1), (0, 1)],
            Orientation.OneEighty:  [(0, 0), (0, 1), (1, 1), (1, 2)],
            Orientation.TwoSeventy: [(2, 0), (1, 0), (1, 1), (0, 1)]
        ]
    }

    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
        return [
            Orientation.Zero:       [blocks[SecondBlockIdx], blocks[FourthBlockIdx]],
            Orientation.Ninety:     [blocks[FirstBlockIdx], blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
            Orientation.OneEighty:  [blocks[SecondBlockIdx], blocks[FourthBlockIdx]],
            Orientation.TwoSeventy: [blocks[FirstBlockIdx], blocks[ThirdBlockIdx], blocks[FourthBlockIdx]]
        ]
    }
}
```

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/07-giving-shape-z.png)</center>

```objc(ZShape.swift)
class ZShape:Shape {
    /*

    Orientation 0

      • | 0 |
    | 2 | 1 |
    | 3 |

    Orientation 90

    | 0 | 1•|
        | 2 | 3 |

    Orientation 180

      • | 0 |
    | 2 | 1 |
    | 3 |

    Orientation 270

    | 0 | 1•|
        | 2 | 3 |


    • marks the row/column indicator for the shape

    */

    override var blockRowColumnPositions: [Orientation: Array<(columnDiff: Int, rowDiff: Int)>] {
        return [
            Orientation.Zero:       [(1, 0), (1, 1), (0, 1), (0, 2)],
            Orientation.Ninety:     [(-1,0), (0, 0), (0, 1), (1, 1)],
            Orientation.OneEighty:  [(1, 0), (1, 1), (0, 1), (0, 2)],
            Orientation.TwoSeventy: [(-1,0), (0, 0), (0, 1), (1, 1)]
        ]
    }

    override var bottomBlocksForOrientations: [Orientation: Array<Block>] {
        return [
            Orientation.Zero:       [blocks[SecondBlockIdx], blocks[FourthBlockIdx]],
            Orientation.Ninety:     [blocks[FirstBlockIdx], blocks[ThirdBlockIdx], blocks[FourthBlockIdx]],
            Orientation.OneEighty:  [blocks[SecondBlockIdx], blocks[FourthBlockIdx]],
            Orientation.TwoSeventy: [blocks[FirstBlockIdx], blocks[ThirdBlockIdx], blocks[FourthBlockIdx]]
        ]
    }
}
```
