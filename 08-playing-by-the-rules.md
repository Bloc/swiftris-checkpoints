<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-its-only-logical-final.gif)</center>

## Playing by the Rules

Even a game as seemingly quaint as Swiftris abides by a complex set of rules. Start considering all of the possibilities which must be accounted for by our game: blocks may not collide, blocks must not fall further than the game board or exceed its column boundaries, points are earned for each line formed, remaining blocks must fall after lines beneath them have been removed, the list goes on.

This checkpoint will establish the rules and mechanics by which Swiftris is played. We'll begin by  creating a custom protocol. As we've discussed earlier, `Hashable` and `Printable` are both protocols which several of our classes adhere to. We need a protocol specifically designed to receive updates from the `Swiftris` class:

```objc(Swiftris.swift)
let PreviewRow = 1

+protocol SwiftrisDelegate {
+    // Invoked when the current round of Swiftris ends
+    func gameDidEnd(swiftris: Swiftris)

+    // Invoked immediately after a new game has begun
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

The `delegate` will be notified of several events throughout the course of the game. In our case, `GameViewController` will implement and attach itself as `delegate` in order to update the user interface and react to game state changes whenever something occurs inside of `Swiftris.swift`.

Swiftris will work on a trial-and-error basis. The user interface - `GameViewController` - will ask `Swiftris` to move its falling shape either down, left, or right. `Swiftris` will accept this request, move the shape and then detect whether or not its new position is legal. If so, the shape will remain, otherwise it will revert to its original location.

Let's add a few methods to detect if and when a shape is breaking the rules:

```objc(Swiftris.swift)
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
+        if detectIllegalPlacement() {
+            nextShape = fallingShape
+            nextShape!.moveTo(PreviewColumn, row: PreviewRow)
+            endGame()
+            return (nil, nil)
+        }
         return (fallingShape, nextShape)
     }

// #2
+    func detectIllegalPlacement() -> Bool {
+        if let shape = fallingShape {
+            for block in shape.blocks {
+                if block.column < 0 || block.column >= NumColumns
+                    || block.row < 0 || block.row >= NumRows {
+                    return true
+                } else if blockArray[block.column, block.row] != nil {
+                    return true
+                }
+            }
+        }
+        return false
+    }
```

At **#1** we added some logic to `newShape()` which may now detect the ending of a Switris game. The game ends when a new shape located at the designated starting location collides with existing blocks. This is the case where the player no longer has enough room to move the new shape, and therefore, we must terminate their tower of terror.

At **#2** we added a function for checking both block boundary conditions. This first determines whether or not a block exceeds the legal size of the game board. The second determines whether or not a block's current location overlaps with an existing block. Remember, Swiftris will function by *trial-and-error*. We'll send our shapes to all sorts of bizarre places before we check whether or not they are legally allowed to be there.

Before proceeding, let's add some convenient helper functions to `Shape.swift` which will aide in `Switris`' ability to move and rotate each shape at will:

```objc(Shape.swift)
    final func rotateBlocks(orientation: Orientation) {
        if let blockRowColumnTranslation:Array<(columnDiff: Int, rowDiff: Int)> = blockRowColumnPositions[orientation] {
            for (idx, diff) in enumerate(blockRowColumnTranslation) {
                blocks[idx].column = column + diff.columnDiff
                blocks[idx].row = row + diff.rowDiff
            }
        }
    }

// #1
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

These new functions should be self explanatory. At **#1** we created a couple methods for quickly rotating a shape one turn clockwise or counterclockwise, this will come in handy when testing a potential rotation and reverting it if it breaks the rules. Below that we've added convenience functions which allow us to move our shapes incrementally in any direction.

Let's put those new functions to good use by giving the user interface access to shape manipulation:

```objc(Swiftris.swift)
    func detectIllegalPlacement() -> Bool {
        if let shape = fallingShape {
            for block in shape.blocks {
                if block.column < 0 || block.column >= NumColumns
                    || block.row < 0 || block.row >= NumRows {
                    return true
                } else if blockArray[block.column, block.row] != nil {
                    return true
                }
            }
        }
        return false
    }

// #1
+    func dropShape() {
+        if let shape = fallingShape {
+            while detectIllegalPlacement() == false {
+                shape.lowerShapeByOneRow()
+            }
+            shape.raiseShapeByOneRow()
+            delegate?.gameShapeDidDrop(self)
+        }
+    }

// #2
+    func letShapeFall() {
+        if let shape = fallingShape {
+            shape.lowerShapeByOneRow()
+            if detectIllegalPlacement() {
+                shape.raiseShapeByOneRow()
+                if detectIllegalPlacement() {
+                    endGame()
+                } else {
+                    settleShape()
+                }
+            } else {
+                delegate?.gameShapeDidMove(self)
+                if detectTouch() {
+                    settleShape()
+                }
+            }
+        }
+    }

// #3
+    func rotateShape() {
+        if let shape = fallingShape {
+            shape.rotateClockwise()
+            if detectIllegalPlacement() {
+                shape.rotateCounterClockwise()
+            } else {
+                delegate?.gameShapeDidMove(self)
+            }
+        }
+    }

// #4
+    func moveShapeLeft() {
+        if let shape = fallingShape {
+            shape.shiftLeftByOneColumn()
+            if detectIllegalPlacement() {
+                shape.shiftRightByOneColumn()
+                return
+            }
+            delegate?.gameShapeDidMove(self)
+        }
+    }

+    func moveShapeRight() {
+        if let shape = fallingShape {
+            shape.shiftRightByOneColumn()
+            if detectIllegalPlacement() {
+                shape.shiftLeftByOneColumn()
+                return
+            }
+            delegate?.gameShapeDidMove(self)
+        }
+    }
```

Dropping a shape is the act of sending it plummeting towards the bottom of the game board. The user will elect to do this when their patience for the slow-moving Tetromino wears thin. **#1** provides a convenient function to accomplish this. It will continue dropping the shape by a single row until an illegal placement state is reached, at which point it will raise it and then notify the delegate that a drop has occurred.

All of these functions use conditional assignments before taking action, this guarantees that regardless of what state the user interface is in, `Swiftris` will never operate on invalid shapes.

At **#2** we've defined a function to be called once every tick. This attempts to lower the shape by one row and ends the game if it fails to do so without finding legal placement for it. Don't worry about the missing functions here, we'll create them soon.

Our user interface will allow the player to rotate the shape clockwise as it falls and the function at **#3** implements that behavior. `Swiftris` attempts to rotate the shape clockwise. If its new block positions violate the boundaries of the game or overlap with settled blocks, we revert the rotation and return. Otherwise, we let the delegate know that the shape has moved.

Lastly, the player will enjoy the privilege of moving the shape either leftwards or rightwards. The functions written at **#4** permit such behavior and follow the same pattern found in `rotateShape`.

I bet you're tired of looking at those errors, let's fix them by adding a few missing functions:

```objc(Swiftris.swift)
// #1
+    func settleShape() {
+        if let shape = fallingShape {
+            for block in shape.blocks {
+                blockArray[block.column, block.row] = block
+            }
+            fallingShape = nil
+            delegate?.gameShapeDidLand(self)
+        }
+    }

// #2
+    func detectTouch() -> Bool {
+        if let shape = fallingShape {
+            for bottomBlock in shape.bottomBlocks {
+                if bottomBlock.row == NumRows - 1 ||
+                    blockArray[bottomBlock.column, bottomBlock.row + 1] != nil {
+                        return true
+                }
+            }
+        }
+        return false
+    }

+    func endGame() {
+        delegate?.gameDidEnd(self)
+    }

    func dropShape() {
        if let shape = fallingShape {
            while detectIllegalPlacement() == false {
                shape.lowerShapeByOneRow()
            }
            shape.raiseShapeByOneRow()
            delegate?.gameShapeDidDrop(self)
        }
    }
```

At **#1**, `settleShape()` adds the falling shape to the collection of blocks maintained by `Swiftris`. Once the falling shape's blocks are part of the game board, `fallingShape` is nullified and the delegate is notified of a new shape settling into the game board.

`Swiftris` needs to be able to tell when a shape should settle. This happens under two conditions: when one of the shapes' bottom blocks is located immediately above a block on the game board or when one of those same blocks has reached the bottom of the game board. The function at **#2** properly detects this occurrence and returns `true` when detected.

### Scoring

Most, if not all games involve some sort of scoring mechanism; a means by which to garner points. Yes, even a game as sophisticated and worldly as Swiftris must admit that its players feed off of small psychological rewards, meaningless as they may be. Let's have the `Swiftris` class track the player's current score and level them up as they reach digital milestones:

```objc(Swiftris.swift)
 let PreviewColumn = 12
 let PreviewRow = 1

+let PointsPerLine = 10
+let LevelThreshold = 1000

 protocol SwiftrisDelegate {
```

```objc(Swiftris.swift)
    var blockArray:Array2D<Block>
    var nextShape:Shape?
    var fallingShape:Shape?
    var delegate:SwiftrisDelegate?

+   var score:Int
+   var level:Int

    init() {
+       score = 0
+       level = 1
        fallingShape = nil
        nextShape = nil
        blockArray = Array2D<Block>(columns: NumColumns, rows: NumRows)
    }
```

```objc(Switris.swift)
    func endGame() {
+        score = 0
+        level = 1
        delegate?.gameDidEnd(self)
    }
```

We added a couple variables to help us keep track of the player's progress: `score` and `level`. Score represents their cumulative point total. Level represents which level of Swiftris they're currently playing on. We'll need a function capable of deducing if and when a solid horizontal line has been formed; that's the only way a player can earn points in `Swiftris`:

```objc(Swiftris.swift)
     func endGame() {
        score = 0
        level = 1
        delegate?.gameDidEnd(self)
     }

// #1
+    func removeCompletedLines() -> (linesRemoved: Array<Array<Block>>, fallenBlocks: Array<Array<Block>>) {
+        var removedLines = Array<Array<Block>>()
+        for var row = NumRows - 1; row > 0; row-- {
+            var rowOfBlocks = Array<Block>()
// #2
+            for column in 0..<NumColumns {
+                if let block = blockArray[column, row] {
+                    rowOfBlocks.append(block)
+                }
+            }
+            if rowOfBlocks.count == NumColumns {
+                removedLines.append(rowOfBlocks)
+                for block in rowOfBlocks {
+                    blockArray[block.column, block.row] = nil
+                }
+            }
+        }

// #3
+        if removedLines.count == 0 {
+            return ([], [])
+        }
// #4
+        let pointsEarned = removedLines.count * PointsPerLine * level
+        score += pointsEarned
+        if score >= level * LevelThreshold {
+            level += 1
+            delegate?.gameDidLevelUp(self)
+        }

+        var fallenBlocks = Array<Array<Block>>()
+        for column in 0..<NumColumns {
+            var fallenBlocksArray = Array<Block>()
// #5
+            for var row = removedLines[0][0].row - 1; row > 0; row-- {
+                if let block = blockArray[column, row] {
+                    var newRow = row
+                    while (newRow < NumRows - 1 && blockArray[column, newRow + 1] == nil) {
+                        newRow++
+                    }
+                    block.row = newRow
+                    blockArray[column, row] = nil
+                    blockArray[column, newRow] = block
+                    fallenBlocksArray.append(block)
+                }
+            }
+            if fallenBlocksArray.count > 0 {
+                fallenBlocks.append(fallenBlocksArray)
+            }
+        }
+        return (removedLines, fallenBlocks)
+    }
```

This looks like a long and unwieldy function, so let's walk through it like we would through tall grass; holding hands and wearing overalls. At **#1**, we defined a function which returns yet another tuple. This time it's composed of two arrays: `linesRemoved` and `fallenBlocks`. `linesRemoved` maintains each row of blocks which the user has filled in completely. 

At **#2** we use a `for` loop  which iterates from `0` all the way up to, but not including `NumColumns`; therefore `0` to `9`. This `for` loop adds every block in a given row to a local array variable named `rowOfBlocks`. If it ends up with a full set - `10` blocks in total - it counts that as a removed line and adds it to the return variable.

At **#3** we check and see if we recovered any lines at all, if not, we return empty arrays immediately.

At **#4**, we add points to the player's score based on the number of lines they've created and their level. If their points exceed their level times 1000, they level up and our delegate is informed.

At **#5** we do something a bit murky-looking. Starting in the left-most column and immediately above the bottom-most removed line, we count upwards towards the top of the game board. As we do so, we take each remaining block we find on the game board and lower it as far as possible. `fallenBlocks` is an array of arrays, each sub-array is filled with blocks that fell to a new position as a result of the user clearing lines beneath them.

Woo, that was a doozy… Let's take a break by enjoying some cuteness:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-its-only-logical-puppy.gif)</center>

#### One Last Thing

`Swiftris` is almost ready, it just needs one small function which will allow the user interface to remove all of the blocks and send them straight into the digital abyss:

```objc(Swiftris.swift)
+    func removeAllBlocks() -> Array<Array<Block>> {
+        var allBlocks = Array<Array<Block>>()
+        for row in 0..<NumRows {
+            var rowOfBlocks = Array<Block>()
+            for column in 0..<NumColumns {
+                if let block = blockArray[column, row] {
+                    rowOfBlocks.append(block)
+                    blockArray[column, row] = nil
+                }
+            }
+            allBlocks.append(rowOfBlocks)
+        }
+        return allBlocks
+    }
```

This function loops through and creates rows of blocks in order for the game scene to animate them off the game board. Meanwhile, it nullifies each location in the block array to empty it entirely, preparing it for a new game.

### Bringing It Together

It's time to have `GameViewController` implement `SwiftrisDelegate` and begin reacting to changes in `Swiftris`' state:

```objc(GameViewController.swift)
-class GameViewController: UIViewController {
+class GameViewController: UIViewController, SwiftrisDelegate {
```

```objc(GameViewController.swift)
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

```objc(GameViewController.swift)
     func didTick() {
// #1
+        swiftris.letShapeFall()
-        swiftris.fallingShape?.lowerShapeByOneRow()
-        scene.redrawShape(swiftris.fallingShape!, completion: {})
     }

+    func nextShape() {
+        let newShapes = swiftris.newShape()
+        if let fallingShape = newShapes.fallingShape {
+            self.scene.addPreviewShapeToScene(newShapes.nextShape!) {}
+            self.scene.movePreviewShape(fallingShape) {
// #2
+                self.view.userInteractionEnabled = true
+                self.scene.startTicking()
+            }
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

// #3
+    func gameShapeDidMove(swiftris: Swiftris) {
+        scene.redrawShape(swiftris.fallingShape!) {}
+    }
```

At **#1** we substituted our previous efforts with `Swiftris`' `letShapeFall()` function, precisely what we need at each tick.

At **#2** we introduced a boolean which allows us to shut down interaction with the view. Regardless of what the user does to the device at this point, they will not be able to manipulate Switris in any way. This is useful during intermediate states when blocks are being animated, shifted around or calculated. Otherwise, a well-timed user interaction may cause an unpredictable game state to occur.

Lastly, at **#3** all that is necessary to do after a shape has moved is to redraw its representative sprites at their new locations.

Run Swiftris to discover that your shapes are aware of one another as well as the floor! It may not be overly exciting but at least the laws of physics are being obeyed.
