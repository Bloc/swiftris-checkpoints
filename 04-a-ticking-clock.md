## A Ticking Clock

If you've played any version of Tetris before, you expect that a piece will drop by one row periodically at a given time interval. Pieces perilously lower themselves towards your doom, unstoppable by all that is right in the world… So! Swiftris will be no different, our game will mimic this behavior.

A class which extends `SKScene` inherits the `update(currentTime: CFTimeInterval)` function. iOS invokes `update` every *frame*. A frame is a single image presented to the user. Like a frame found in a cat video, it's a small time-slice of cat content compared to the whole cat-sperience. Smooth-running games have higher frame rates; about 60 frames per second or more. Slower games typically plummet below a dismal 30 fps.

A game looks slow when our eyes begin to perceive each individual frame; this is because of discrete motion. Here's an example of identical content playing at different frame rates:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/05-a-ticking-clock-frame-rate-comparison.gif)</center>

Let's take advantage of the `update` method to discover if a time interval has passed:

```swift(GameScene.swift)
import SpriteKit

// #1
+let TickLengthLevelOne = NSTimeInterval(600)

class GameScene: SKScene {

// #2
+    var tick:(() -> ())?
+    var tickLengthMillis = TickLengthLevelOne
+    var lastTick:NSDate?

    required init(coder aDecoder: NSCoder) {
        fatalError("NSCoder not supported")
    }

    override init(size: CGSize) {
        super.init(size: size)

        anchorPoint = CGPoint(x: 0, y: 1.0)

        let background = SKSpriteNode(imageNamed: "background")
        background.position = CGPoint(x: 0, y: 0)
        background.anchorPoint = CGPoint(x: 0, y: 1.0)
        addChild(background)
    }

    override func update(currentTime: CFTimeInterval) {
        /* Called before rendering each frame */
// #3
+        guard let lastTick = lastTick else {
+            return
+        }
+        let timePassed = lastTick.timeIntervalSinceNow * -1000.0
+        if timePassed > tickLengthMillis {
+            self.lastTick = NSDate()
+            tick?()
+        }
    }

// #4
+    func startTicking() {
+        lastTick = NSDate()
+    }

+    func stopTicking() {
+        lastTick = nil
+    }
}
```

First, we define a new constant at **#1**, `TickLengthLevelOne`. This variable will represent the slowest speed at which our shapes will travel. We've set it to `600` milliseconds, which means that every 6/10<super>ths</super> of a second, our shape should descend by one row.

At **#2** you can see we've defined some variables. `tickLengthMillis` and `lastTick` look like declarations we've seen before: one being the `GameScene`'s current tick length, set to `TickLengthLevelOne` by default, and the other will track the last time we experienced a tick, an `NSDate` object.

`tick:(() -> ())?` looks horrifying… `tick` is what's known as a *closure* in Swift. A closure is essentially a block of code that performs a function, and Swift refers to functions as closures. In defining `tick`, its type is `(() -> ())?` which means that it's a closure which takes no parameters and returns nothing. Its question mark indicates that it's optional and may be `nil`.

[Learn more about Swift closures.](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Closures.html)

At **#3** we'll put our new member variables to work. Swift's `guard` statement checks the conditions which follow it, `let lastTick = lastTick` in our case. If the conditions fail, `guard` executes the `else` block. If `lastTick` is missing, the game is in a paused state and not reporting elapsed ticks, so we return.

But if `lastTick` is present, we recover the time passed since the last execution of `update` by invoking `timeIntervalSinceNow` on our `lastTick` object. We multiply the result by `-1000` to calculate a positive millisecond value. We invoke functions on objects using *dot syntax* in Swift.

[Read more about dot-syntax.](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html)

We then check if the time passed has exceeded our `tickLengthMillis` variable. If enough time has elapsed, we must report a tick. We do so by first updating our last known tick time to the present and then invoking our closure.

By placing a `?` after the variable name, we are asking Swift to first check if `tick` exists and if so, invoke it with no parameters. It's shorthand for the following statements:

```swift
if tick != nil {
    tick!()
}
```

Lastly, at **#4** we provide accessor methods to let external classes stop and start the ticking process, something we'll make use of later to keep pieces from falling at key moments.
