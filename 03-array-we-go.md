## And Array We Go

An Array is a useful tool which every programmer carries in their belt. That construction metaphor is entirely broken but thankfully you can rely on Swift's array implementation to work *every* time. For our purposes, we'll need a specific use of the array. We want to be able to access each location on our game board by its `row` and `column` value. Here's the layout of our game board:

| Row 0 / Col 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|:-------------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|     Row 1     |   |   |   |   |   |   |   |   |   |
|     Row 2     |   |   |   |   |   |   |   |   |   |
|     Row 3     |   |   |   |   |   |   |   |   |   |
|     Row 4     |   |   |   |   |   |   |   |   |   |
|     Row 5     |   |   |   |   |   |   |   |   |   |
|     Row 6     |   |   |   |   |   |   |   |   |   |
|     Row 7     |   |   |   |   |   |   |   |   |   |
|     Row 8     |   |   |   |   |   |   |   |   |   |
|     Row 9     |   |   |   |   |   |   |   |   |   |
|    Row 10     |   |   |   |   |   |   |   |   |   |
|    Row 11     |   |   |   |   |   |   |   |   |   |
|    Row 12     |   |   |   |   |   |   |   |   |   |
|    Row 13     |   |   |   |   |   |   |   |   |   |
|    Row 14     |   |   |   |   |   |   |   |   |   |
|    Row 15     |   |   |   |   |   |   |   |   |   |
|    Row 16     |   |   |   |   |   |   |   |   |   |
|    Row 17     |   |   |   |   |   |   |   |   |   |
|    Row 18     |   |   |   |   |   |   |   |   |   |
|    Row 19     |   |   |   |   |   |   |   |   |   |

20 rows and 10 columns gives us 200 possible locations for a block to occupy. We'll organize these locations into an array and reference them using a custom subscript: `array[column, row]`. Swift arrays by default have a single valued subscript - `array[index]` - but Swift allows us to create our own as well.

Make a brand new file by pressing <key>⌘ + N</key> or **File** > **New** > **File…**, the following window should appear: (Make sure to highlight the "Swiftris" folder, as the image shows below)

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/04-array-we-go-new-swift-file.png)</center>

Choose **Swift** and press **Next**. Name the file, `Array2D` and click **Create**.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/04-array-we-go-save-new-file.png)</center>

The file should automatically open in the editing window. Replace the generated content with the following:

```objc(Array2D.swift)
-import Foundation

// #1
+class Array2D<T> {
+    let columns: Int
+    let rows: Int
// #2
+    var array: Array<T?>

+    init(columns: Int, rows: Int) {
+        self.columns = columns
+        self.rows = rows
// #3
+        array = Array<T?>(count:rows * columns, repeatedValue: nil)
+    }

// #4
+    subscript(column: Int, row: Int) -> T? {
+        get {
+            return array[(row * columns) + column]
+        }
+        set(newValue) {
+            array[(row * columns) + column] = newValue
+        }
+    }
+}
```

Let's briefly discuss what this class does. At **#1** we're defining a class named `Array2D`. Generic arrays in Swift are actually of type `struct`, not `class`, but we need a class in this case such that wherever we use the game board array in the project, we will manipulate the same data.

We may pass class objects by reference (same data) whereas we must pass structures by value (copied data). Our game logic will require a single copy of this data structure to persist across the entire game.

[For a more thorough explanation of Swift classes, check this out.](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/ClassesAndStructures.html)

Notice that in the declaration of this class, we provide a typed parameter: `<T>`. This allows our array to store any data type and remain a general-purpose tool.

At **#2** we declare an actual Swift array; it will be the underlying data structure which maintains references to our objects. It's declared with type `<T?>`. A `?` in Swift symbolizes an *optional value*.

An optional value is just that, optional. Optional variables may or may not contain data, and they may in fact be `nil`, or empty. `nil` locations found on our game board will represent empty spots where no block is present.

During our initialization at **#3**, we instantiate our internal `array` structure with a size of `rows` x `columns`. This guarantees that `Array2D` can store all the objects our game board requires, 200 in our case.

[Go here if you're yearning to learn more about Swift arrays.](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/SwiftStandardLibraryReference/Array.html)

And at **#4** we create a custom subscript for `Array2D`. We mentioned earlier that we wanted to have a subscript capable of supporting `array[column, row]`, this accomplishes that. The getter is self-explanatory. To get the value at a given location we need to multiply the provided `row` by the class variable `columns`, then add the column number to reach the final destination.

[More detail on subscripting.](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Subscripts.html)

The setter is the reverse operation of that; we assign `newValue` to the location determined by the same algorithm found in the custom getter.

**Save** your file and get excited to put your sexy new class to work!
