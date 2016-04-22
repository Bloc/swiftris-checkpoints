<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-its-only-logical-final.gif)</center>

## Playing by the Rules

Even a game as seemingly quaint as Swiftris abides by a complex set of rules. Start considering the laws we must account for in our game: blocks may not collide, blocks must not fall further than the game board or exceed their column boundaries, players earn points for each line formed, remaining blocks must fall after lines beneath them fall, the list goes on.

This checkpoint will establish the rules and mechanics of Swiftris. We'll begin by creating a custom protocol. As we've discussed earlier, `Hashable` and `CustomStringConvertible` are protocols which some of our classes conform to. We need to create a protocol specifically designed to receive updates from the `Swiftris` class:

```swift(Swiftris.swift)
let PreviewRow = 1

+protocol SwiftrisDelegate {
+    // Invoked when the current round of Swiftris ends
+    func gameDidEnd(swiftris: Swiftris)

+    // Invoked after a new game has begun
+    func gameDidBegin(swiftris: Swiftris)

+    // Invoked when the falling shape has become part of the game board
+    func gameShapeDidLand(swiftris: Swiftris)

+    // Invoked when the falling shape has changed its location
+    func gameShapeDidMove(swiftris: Swiftris)

+    // Invoked when the falling shape has changed its location after being dropped
+    func gameShapeDidDrop(swiftris: Swiftris)

+    // Invoked when the game has reached a new level
+    func gameDidLevelUp(swiftris: Swiftris)
+}

class Swiftris {
    var blockArray:Array2D<Block>
    var nextShape:Shape?
    var fallingShape:Shape?
+   var delegate:SwiftrisDelegate?
```

`Swiftris` notifies the `delegate` of events throughout the course of the game. In our case, `GameViewController` will attach itself as the `delegate` to update the user interface and react to game state changes whenever something occurs inside of the `Swiftris` class.

Swiftris will work on a trial-and-error basis. The user interface, `GameViewController`, will ask `Swiftris` to move its falling shape either down, left, or right. `Swiftris` will accept this request, move the shape and then detect whether its new position is legal. If so, the shape will remain, otherwise it will revert to its original location.

Let's add methods to detect when a shape breaks the rules:

```swift(Swiftris.swift)
    func beginGame() {
        if (nextShape == nil) {
            nextShape = Shape.random(PreviewColumn, startingRow: PreviewRow)
        }
+       delegate?.gameDidBegin(self)
    }

    func newShape() -> (fallingShape:Shape?, nextShape:Shape?) {
        fallingShape = nextShape
        nextShape = Shape.random(PreviewColumn, startingRow: PreviewRow)
        fallingShape?.moveTo(StartingColumn, row: StartingRow)
// #1
+        guard detectIllegalPlacement() == false else {
+            nextShape = fallingShape
+            nextShape!.moveTo(PreviewColumn, row: PreviewRow)
+            endGame()
+            return (nil, nil)
+        }
         return (fallingShape, nextShape)
     }

// #2
+    func detectIllegalPlacement() -> Bool {
+        guard let shape = fallingShape else {
+            return false
+        }
+        for block in shape.blocks {
+            if block.column < 0 || block.column >= NumColumns
+                || block.row < 0 || block.row >= NumRows {
+                return true
+            } else if blockArray[block.column, block.row] != nil {
+                return true
+            }
+        }
+    return false
+}
```

At **#1** we added some logic to `newShape()` which may now detect the ending of a Switris game. The game ends when a new shape located at the designated starting location collides with existing blocks. This is the case where the player no longer has room to move the new shape, and we must destroy their tower of terror.

At **#2** we added a function for checking both block boundary conditions. This first determines whether a block exceeds the legal size of the game board. The second determines whether a block's current location overlaps with an existing block. Remember, Swiftris will function by *trial-and-error*. We'll send our shapes to all sorts of bizarre places before we check whether they are legally allowed to be there.

Before proceeding, let's add some convenient helper functions to `Shape.swift` which will aide in `Switris`' ability to move and rotate each shape at will:

```swift(Shape.swift)
    final func rotateBlocks(orientation: Orientation) {
        guard let blockRowColumnTranslation:Array<(columnDiff: Int, rowDiff: Int)> = blockRowColumnPositions[orientation] else {
            return
        }
        for (idx, diff) in blockRowColumnTranslation.enumerate() {
            blocks[idx].column = column + diff.columnDiff
            blocks[idx].row = row + diff.rowDiff
        }
    }

// #3
+    final func rotateClockwise() {
+        let newOrientation = Orientation.rotate(orientation, clockwise: true)
+        rotateBlocks(newOrientation)
+        orientation = newOrientation
+    }

+    final func rotateCounterClockwise() {
+        let newOrientation = Orientation.rotate(orientation, clockwise: false)
+        rotateBlocks(newOrientation)
+        orientation = newOrientation
+    }

    final func lowerShapeByOneRow() {
        shiftBy(0, rows:1)
    }

+    final func raiseShapeByOneRow() {
+        shiftBy(0, rows:-1)
+    }

+    final func shiftRightByOneColumn() {
+        shiftBy(1, rows:0)
+    }

+    final func shiftLeftByOneColumn() {
+        shiftBy(-1, rows:0)
+    }

    final func shiftBy(columns: Int, rows: Int) {
        self.column += columns
        self.row += rows
        for block in blocks {
            block.column += columns
            block.row += rows
        }
    }
```

These new functions should be self explanatory. At **#3** we created a couple methods for rotating a shape one turn clockwise or counterclockwise. These will come in handy when testing a potential rotation and reverting it if necessary. Below that we've added convenience functions which allow us to move our shapes incrementally in any direction.

Let's put those new functions to good use by giving the user interface access to shape manipulation:

```swift(Swiftris.swift)
    func detectIllegalPlacement() -> Bool {
        guard let shape = fallingShape else {
            return false
        }
        for block in shape.blocks {
            if block.column < 0 || block.column >= NumColumns
                || block.row < 0 || block.row >= NumRows {
                return true
            } else if blockArray[block.column, block.row] != nil {
                return true
            }
        }
        return false
    }

// #4
+    func dropShape() {
+        guard let shape = fallingShape else {
+            return
+        }
+        while detectIllegalPlacement() == false {
+            shape.lowerShapeByOneRow()
+        }
+        shape.raiseShapeByOneRow()
+        delegate?.gameShapeDidDrop(self)
+    }

// #5
+    func letShapeFall() {
+        guard let shape = fallingShape else {
+            return
+        }
+        shape.lowerShapeByOneRow()
+        if detectIllegalPlacement() {
+            shape.raiseShapeByOneRow()
+            if detectIllegalPlacement() {
+                endGame()
+            } else {
+                settleShape()
+            }
+        } else {
+            delegate?.gameShapeDidMove(self)
+            if detectTouch() {
+                settleShape()
+            }
+        }
+    }

// #6
+    func rotateShape() {
+        guard let shape = fallingShape else {
+            return
+        }
+        shape.rotateClockwise()
+        guard detectIllegalPlacement() == false else {
+            shape.rotateCounterClockwise()
+            return
+        }
+        delegate?.gameShapeDidMove(self)
+    }

// #7
+    func moveShapeLeft() {
+        guard let shape = fallingShape else {
+            return
+        }
+        shape.shiftLeftByOneColumn()
+        guard detectIllegalPlacement() == false else {
+            shape.shiftRightByOneColumn()
+            return
+        }
+        delegate?.gameShapeDidMove(self)
+    }

+    func moveShapeRight() {
+        guard let shape = fallingShape else {
+            return
+        }
+        shape.shiftRightByOneColumn()
+        guard detectIllegalPlacement() == false else {
+            shape.shiftLeftByOneColumn()
+            return
+        }
+        delegate?.gameShapeDidMove(self)
+    }
```

Dropping a shape is the act of sending it plummeting towards the bottom of the game board. The user will elect to do this when their patience for the slow-moving Tetromino wears thin. **#4** provides a convenient function to achieve this. It will continue dropping the shape by a single row until it detects an illegal placement state, at which point it will raise it and then notify the delegate that a drop has occurred.

These functions use conditional assignments before taking action, this guarantees that regardless of what state the user interface is in, `Swiftris` will never operate on invalid shapes.

At **#5** we've defined a function to call once every tick. This attempts to lower the shape by one row and ends the game if it fails to do so without finding legal placement for it. Don't worry about the missing functions here, we'll create them soon.

Our user interface will allow the player to rotate the shape clockwise as it falls and the function at **#6** implements that behavior. `Swiftris` attempts to rotate the shape clockwise. If its new block positions violate the boundaries of the game or overlap with settled blocks, we revert the rotation and return. Otherwise, we let the delegate know that the shape has moved.

Lastly, the player will enjoy the privilege of moving the shape either leftwards or rightwards. The functions written at **#7** permit such behavior and follow the same pattern found in `rotateShape`.

I bet you're tired of looking at those errors, let's fix them by adding the missing functions:

```swift(Swiftris.swift)
// #8
+    func settleShape() {
+        guard let shape = fallingShape else {
+            return
+        }
+        for block in shape.blocks {
+            blockArray[block.column, block.row] = block
+        }
+        fallingShape = nil
+        delegate?.gameShapeDidLand(self)
+    }

// #9
+    func detectTouch() -> Bool {
+        guard let shape = fallingShape else {
+            return false
+        }
+        for bottomBlock in shape.bottomBlocks {
+            if bottomBlock.row == NumRows - 1
+                || blockArray[bottomBlock.column, bottomBlock.row + 1] != nil {
+                    return true
+            }
+        }
+        return false
+    }

+    func endGame() {
+        delegate?.gameDidEnd(self)
+    }

    func dropShape() {
        guard let shape = fallingShape else {
            return
        }
        while detectIllegalPlacement() == false {
            shape.lowerShapeByOneRow()
        }
        shape.raiseShapeByOneRow()
        delegate?.gameShapeDidDrop(self)
    }
```

At **#8**, `settleShape()` adds the falling shape to the collection of blocks maintained by `Swiftris`. Once the falling shape's blocks are part of the game board, we nullify `fallingShape` and notify the delegate of a new shape settling onto the game board.

`Swiftris` needs to be able to tell when a shape should settle. This happens under two conditions: when one of the shapes' bottom blocks touches a block on the game board or when one of those same blocks has reached the bottom of the game board. The function at **#9** properly detects this occurrence and returns `true` when detected.

### Scoring

Most, if not all games involve some sort of scoring mechanism; a means by which to garner points. Yes, even a game as sophisticated and well-traveled as Swiftris must admit that its players feed off of small psychological rewards, meaningless as they may be. Let's have the `Swiftris` class track the player's current score and level them up as they reach digital milestones:

```swift(Swiftris.swift)
 let PreviewColumn = 12
 let PreviewRow = 1

+let PointsPerLine = 10
+let LevelThreshold = 500

 protocol SwiftrisDelegate {
```

```swift(Swiftris.swift)
    var blockArray:Array2D<Block>
    var nextShape:Shape?
    var fallingShape:Shape?
    var delegate:SwiftrisDelegate?

+   var score = 0
+   var level = 1

    init() {
        fallingShape = nil
        nextShape = nil
        blockArray = Array2D<Block>(columns: NumColumns, rows: NumRows)
    }
```

```swift(Switris.swift)
    func endGame() {
+        score = 0
+        level = 1
        delegate?.gameDidEnd(self)
    }
```

We added a couple variables to help us keep track of the player's progress: `score` and `level`. Score represents their cumulative point total. Level represents which level of Swiftris they're playing on. We'll need a function capable of deducing when the player has formed a solid horizontal line; that's how a player earns points in `Swiftris`:

```swift(Swiftris.swift)
     func endGame() {
        score = 0
        level = 1
        delegate?.gameDidEnd(self)
     }

// #10
+    func removeCompletedLines() -> (linesRemoved: Array<Array<Block>>, fallenBlocks: Array<Array<Block>>) {
+        var removedLines = Array<Array<Block>>()
+        for row in (1..<NumRows).reverse() {
+            var rowOfBlocks = Array<Block>()
// #11
+            for column in 0..<NumColumns {
+                guard let block = blockArray[column, row] else {
+                    continue
+                }
+                rowOfBlocks.append(block)
+            }
+            if rowOfBlocks.count == NumColumns {
+                removedLines.append(rowOfBlocks)
+                for block in rowOfBlocks {
+                    blockArray[block.column, block.row] = nil
+                }
+            }
+        }

// #12
+        if removedLines.count == 0 {
+            return ([], [])
+        }
// #13
+        let pointsEarned = removedLines.count * PointsPerLine * level
+        score += pointsEarned
+        if score >= level * LevelThreshold {
+            level += 1
+            delegate?.gameDidLevelUp(self)
+        }

+        var fallenBlocks = Array<Array<Block>>()
+        for column in 0..<NumColumns {
+            var fallenBlocksArray = Array<Block>()
// #14
+            for row in (1..<removedLines[0][0].row).reverse() {
+                guard let block = blockArray[column, row] else {
+                    continue
+                }
+                var newRow = row
+                while (newRow < NumRows - 1 && blockArray[column, newRow + 1] == nil) {
+                    newRow += 1
+                }
+                block.row = newRow
+                blockArray[column, row] = nil
+                blockArray[column, newRow] = block
+                fallenBlocksArray.append(block)
+            }
+            if fallenBlocksArray.count > 0 {
+                fallenBlocks.append(fallenBlocksArray)
+            }
+        }
+        return (removedLines, fallenBlocks)
+    }
```

This looks like a long and unwieldy function, so let's walk through it like we would through tall grass; holding hands and wearing overalls. At **#10**, we defined a function which returns yet another tuple. This time it's composed of two arrays: `linesRemoved` and `fallenBlocks`. `linesRemoved` maintains each row of blocks which the user has filled in.

At **#11** we use a `for` loop  which iterates from `0` all the way up to, but not including `NumColumns`, `0` to `9`. This `for` loop adds every block in a given row to a local array variable named `rowOfBlocks`. If it ends up with a full set, `10` blocks in total, it counts that as a removed line and adds it to the return variable.

At **#12** we check and see if we recovered any lines at all, if not, we return empty arrays.

At **#13**, we add points to the player's score based on the number of lines they've created and their level. If their points exceed their level times 1000, they level up and we inform the delegate.

At **#14** we do something a bit murky-looking. Starting in the left-most column and above the bottom-most removed line, we count upwards towards the top of the game board. As we do so, we take each remaining block we find on the game board and lower it as far as possible. `fallenBlocks` is an array of arrays, we've filled each sub-array with blocks that fell to a new position as a result of the user clearing lines beneath them.

Woo, that was a doozy… Let's take a break by enjoying some cuteness:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-its-only-logical-puppy.gif)</center>

### One Last Thing

`Swiftris` is almost ready, it just needs one small function which will allow the user interface to remove the blocks and send them straight into the digital abyss:

```swift(Swiftris.swift)
+    func removeAllBlocks() -> Array<Array<Block>> {
+        var allBlocks = Array<Array<Block>>()
+        for row in 0..<NumRows {
+            var rowOfBlocks = Array<Block>()
+            for column in 0..<NumColumns {
+                guard let block = blockArray[column, row] else {
+                    continue
+                }
+                rowOfBlocks.append(block)
+                blockArray[column, row] = nil
+            }
+            allBlocks.append(rowOfBlocks)
+        }
+        return allBlocks
+    }
```

This function loops through and creates rows of blocks in order for the game scene to animate them off the game board. Meanwhile, it nullifies each location in the block array to empty it entirely, preparing it for a new game.

### Bringing It Together

It's time to have `GameViewController` implement `SwiftrisDelegate` and begin reacting to changes in `Swiftris`' state:

```swift(GameViewController.swift)
-class GameViewController: UIViewController {
+class GameViewController: UIViewController, SwiftrisDelegate {
```

```swift(GameViewController.swift)
    override func viewDidLoad() {
        super.viewDidLoad()

        // Configure the view.
        let skView = view as! SKView
        skView.multipleTouchEnabled = false

        // Create and configure the scene.
        scene = GameScene(size: skView.bounds.size)
        scene.scaleMode = .AspectFill
        scene.tick = didTick

        swiftris = Swiftris()
+       swiftris.delegate = self
        swiftris.beginGame()

        // Present the scene.
        skView.presentScene(scene)

-        scene.addPreviewShapeToScene(swiftris.nextShape!) {
-            self.swiftris.nextShape?.moveTo(StartingColumn, row: StartingRow)
-            self.scene.movePreviewShape(self.swiftris.nextShape!) {
-                let nextShapes = self.swiftris.newShape()
-                self.scene.startTicking()
-                self.scene.addPreviewShapeToScene(nextShapes.nextShape!) {}
-            }
-        }
    }
```

```swift(GameViewController.swift)
     func didTick() {
// #15
+        swiftris.letShapeFall()
-        swiftris.fallingShape?.lowerShapeByOneRow()
-        scene.redrawShape(swiftris.fallingShape!, completion: {})
     }

+    func nextShape() {
+        let newShapes = swiftris.newShape()
+        guard let fallingShape = newShapes.fallingShape else {
+            return
+        }
+        self.scene.addPreviewShapeToScene(newShapes.nextShape!) {}
+        self.scene.movePreviewShape(fallingShape) {
// #16
+            self.view.userInteractionEnabled = true
+            self.scene.startTicking()
+        }
+    }

+    func gameDidBegin(swiftris: Swiftris) {
+        // The following is false when restarting a new game
+        if swiftris.nextShape != nil && swiftris.nextShape!.blocks[0].sprite == nil {
+            scene.addPreviewShapeToScene(swiftris.nextShape!) {
+                self.nextShape()
+            }
+        } else {
+            nextShape()
+        }
+    }

+    func gameDidEnd(swiftris: Swiftris) {
+        view.userInteractionEnabled = false
+        scene.stopTicking()
+    }

+    func gameDidLevelUp(swiftris: Swiftris) {
+
+    }

+    func gameShapeDidDrop(swiftris: Swiftris) {
+
+    }

+    func gameShapeDidLand(swiftris: Swiftris) {
+        scene.stopTicking()
+        nextShape()
+    }

// #17
+    func gameShapeDidMove(swiftris: Swiftris) {
+        scene.redrawShape(swiftris.fallingShape!) {}
+    }
```

At **#15** we substituted our previous efforts with `Swiftris`' `letShapeFall()` function, precisely what we need at each tick.

At **#16** we introduced a boolean which allows us to shut down interaction with the view. Regardless of what the user does to the device at this point, they will not be able to manipulate Switris in any way. This is useful during intermediate states when we animate or shift blocks, and perform calculations. Otherwise, a well-timed user interaction may cause an unpredictable game state to occur.

Lastly, at **#17** all that is necessary to do after a shape has moved is to redraw its representative sprites at their new locations.

Run Swiftris to discover that your shapes are aware of one another as well as the floor! It may not be overly exciting but at least Swiftris obeys the laws of physics.
