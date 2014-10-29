## A Ticking Clock

If you've played any version of Tetris before, you expect that a piece will drop by one row periodically at a given time interval. Pieces perilously lower themselves towards your doom, unstoppable by all that is right in the world… So! Swiftris will be no different, our game will mimic this behavior.

A class which extends `SKScene` inherits the `update(currentTime: CFTimeInterval)` function. `update` is invoked every *frame.* A frame can be thought of as a single image presented to the user. Like a frame found in a cat video, it is a small time-slice of cat content with respect to the whole cat-sperience. Smooth-running games have higher frame rates; about 60 frames per second or more. Slower games typically plummet below a dismal 30 fps.

A game looks slow when our eyes begin to perceive each individual frame; this is because of a concept known as discrete motion. Here's an example of identical content playing at various frame rates:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/05-a-ticking-clock-frame-rate-comparison.gif)</center>

Let's take advantage of the `update` method to discover if and when a time interval has passed:

```objc(GameScene.swift)
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
        /* Called before each frame is rendered */
// #3
+        if lastTick == nil {
+            return
+        }
+        var timePassed = lastTick!.timeIntervalSinceNow * -1000.0
+        if timePassed > tickLengthMillis {
+            lastTick = NSDate()
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

First, we define a new constant at `#1`, `TickLengthLevelOne`. This variable will represent the slowest speed at which our shapes will travel. We've set it to `600` milliseconds, which means that every 6/10<super>ths</super> of a second, our shape should descend by one row. At `#2` you can see we've defined a few variables. `tickLengthMillis` and `lastTick` look similar to declarations we've seen before: one being the `GameScene`'s current tick length – set to `TickLengthLevelOne` by default – and the other will track the last time we experienced a tick, an `NSDate` object.

However, `tick:(() -> ())?` looks horrifying… `tick` is what's known as a *closure* in Swift. A closure is essentially a block of code that performs a function, and Swift refers to functions as  closures. In defining `tick`, its type is `(() -> ())?` which means that it's a closure which takes no parameters and returns nothing. Its question mark indicates that it is optional and therefore may be `nil`.

[Learn more about Swift closures](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/Closures.html)

At `#3` we'll put our new member variables to work. If `lastTick` is missing, we are in a paused state, not reporting elapsed ticks to anyone, therefore we simply return. However, if `lastTick` is present we recover the time passed since the last execution of `update` by invoking `timeIntervalSinceNow` on our `lastTick` object. Functions on objects are invoked using *dot syntax* in Swift.

[Read more about dot-syntax](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html)

What's special about our invocation here is the exclamation mark, `!`. This symbol is required if the object in question is an optional type. Before we can access it, we must de-reference the optional by placing an exclamation mark immediately after its name. We multiply the result by `-1000` in order to get a positive millisecond value.

We then check if the time passed has exceeded our `tickLengthMillis` variable. If enough time has elapsed, we must report a tick. We do so by first updating our last known tick time to the present and then invoking our closure. The syntax we use is conditioned on whether or not `tick` is present. By placing a `?` after the variable name, we are asking Swift to first check if `tick` exists and if so, invoke it with no parameters. It is shorthand for the following statement:

```objc
if tick != nil {
    tick!()
}
```

Lastly, at `#4` we provide accessor methods to let external classes stop and start the ticking process, something we'll make use of later in order to keep pieces from falling at key moments.
