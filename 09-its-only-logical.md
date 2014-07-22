<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-its-only-logical-teaser.gif)</center>

## It's Only Logical…

Even a game as seemingly quaint as Swiftris abides by a complex set of rules. Start considering all of the possibilities which must be accounted for by our game: pieces may not collide, pieces must not fall further than the game board or exceed its column boundaries, points are earned for each line formed, remaining pieces must fall after lines beneath them have been removed, the list goes on.

This checkpoint will establish the rules and mechanics by which Swiftris is played, let's begin by first creating a custom protocol. As we've discussed earlier, `Hashable` and `Printable` or both protocols which to this point, several of our classes have adhered to. We need a protocol specifically designed to receive updates from the `Swiftris` class:

```objc(Swiftris.swift)
let PreviewRow = 1

+protocol SwiftrisDelegate {
+    func gameDidEnd(swiftris: Swiftris)
+    func gameDidBegin(swiftris: Swiftris)
+    func gamePieceDidLand(swiftris: Swiftris)
+    func gamePieceDidMove(swiftris: Swiftris)
+    func gamePieceDidDrop(swiftris: Swiftris)
+    func gameDidLevelUp(swiftris: Swiftris)
+}

class Swiftris {
    var blockArray:Array2D<Block>
    var nextShape:Shape?
    var fallingShape:Shape?
+    var delegate:SwiftrisDelegate?
```

`SwiftrisDelegate` will be notified of several events throughout the course of the game. In our case, `GameViewController` will implement and attach itself as `delegate` in order to update the user interface and react to game state changes whenever something occurs inside of `Swiftris.swift`. Let's continue 