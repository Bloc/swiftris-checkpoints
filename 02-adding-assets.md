## Adding Assets

Don't get me wrong, *Spin-The-Bottle: Space Edition* was a great game. However, you didn't start this Bloc Book to make that. At least I hope not, if so, please stop now because your quest is over. For those of you still interested in building Swiftris, we must unceremoniously delete every unnecessary file provided to us by Xcode.

Open Project Navigator by either clicking the designated icon or pressing <key>⌘ + 1</key>:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-project-navigator-button.png)</center>

Right-click `GameScene.sks` and choose the **Delete** option:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-delete-gamescene.png)</center>

When asked to confirm, make sure to choose **Move to trash**:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-delete-gamescene-confirm.png)</center>

To get rid of the aimless space ship once and for all, click the `Images.xcassets` folder and highlight the `Spaceship` entry, press the **delete** key to delete that sucker.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-delete-spaceship.png)</center>

### Trimming The Fat

Having slaughtered those files which are of no use, we must now purge our project of any and all *code* which we do not require. There's no need to have lingering source code designed to support inept space pilots. Delete all of the lines marked in red within their corresponding files:

```objc(GameScene.swift)
import SpriteKit

class GameScene: SKScene {
-    override func didMoveToView(view: SKView) {
-        /* Setup your scene here */
-        let myLabel = SKLabelNode(fontNamed:"Chalkduster")
-        myLabel.text = "Hello, World!";
-        myLabel.fontSize = 65;
-        myLabel.position = CGPoint(x:CGRectGetMidX(self.frame), y:CGRectGetMidY(self.frame));
-
-        self.addChild(myLabel)
-    }

-    override func touchesBegan(touches: NSSet, withEvent event: UIEvent) {
-        /* Called when a touch begins */
-
-        for touch: AnyObject in touches {
-            let location = touch.locationInNode(self)
-
-            let sprite = SKSpriteNode(imageNamed:"Spaceship")
-
-            sprite.xScale = 0.5
-            sprite.yScale = 0.5
-            sprite.position = location
-
-            let action = SKAction.rotateByAngle(CGFloat(M_PI), duration:1)
-
-            sprite.runAction(SKAction.repeatActionForever(action))
-
-            self.addChild(sprite)
-        }
-    }

    override func update(currentTime: CFTimeInterval) {
        /* Called before each frame is rendered */
    }
}
```

That was a lot, but there's more:

```objc(GameViewController.swift)
import UIKit
import SpriteKit

-extension SKNode {
-   class func unarchiveFromFile(file : NSString) -> SKNode? {
-
-        let path = NSBundle.mainBundle().pathForResource(file, ofType: "sks")
-
-        var sceneData = NSData.dataWithContentsOfFile(path, options: .DataReadingMappedIfSafe, error: nil)
-        var archiver = NSKeyedUnarchiver(forReadingWithData: sceneData)
-
-        archiver.setClass(self.classForKeyedUnarchiver(), forClassName: "SKScene")
-        let scene = archiver.decodeObjectForKey(NSKeyedArchiveRootObjectKey) as GameScene
-        archiver.finishDecoding()
-        return scene
-    }
-}

class GameViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

-        if let scene = GameScene.unarchiveFromFile("GameScene") as? GameScene {
-            // Configure the view.
-            let skView = self.view as SKView
-            skView.showsFPS = true
-            skView.showsNodeCount = true

-            /* Sprite Kit applies additional optimizations to improve rendering performance */
-            skView.ignoresSiblingOrder = true

-            /* Set the scale mode to scale to fit the window */
-            scene.scaleMode = .AspectFill

-            skView.presentScene(scene)
-        }
    }

-    override func shouldAutorotate() -> Bool {
-        return true
-    }

-    override func supportedInterfaceOrientations() -> Int {
-        if UIDevice.currentDevice().userInterfaceIdiom == .Phone {
-            return Int(UIInterfaceOrientationMask.AllButUpsideDown.toRaw())
-        } else {
-            return Int(UIInterfaceOrientationMask.All.toRaw())
-        }
-    }

-    override func didReceiveMemoryWarning() {
-        super.didReceiveMemoryWarning()
-        // Release any cached data, images, etc that aren't in use.
-    }

    override func prefersStatusBarHidden() -> Bool {
        return true
    }
}
```

### The Sights And Sounds Of Swiftris

In order to experience Swiftris in all its visual and auditory glory, we're going to need images and sounds, respectively. [Download the necessary assets](http://bloc-books.s3.amazonaws.com/swiftris/swiftris-assets.zip) to your `Desktop` or `Downloads` folder, anywhere other than the Swiftris project directory. Unzip the archive and perform a drag-and-drop of the `Sounds` folder into the Project Navigator immediately above the `Supporting Files` directory. The following window should appear:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-copy-if-necessary.png)</center>

Make sure to check the **Copy items if necessary** option. This will place a copy of the directory and all of the sound files within it into your Swiftris project and directory. Click **Finish**. Repeat this task with the `Sprites.atlas` folder. Next, select all of the images found within the `Images` directory and drag them into the `Supporting Files` folder found in Project Navigator. Once again, make sure that the **Copy items if needed** checkbox is checked. Finally, click on `Images.xcassets` to open the window and highlight `AppIcon`. Drag and drop the appropriate `icon` file from the downloaded "Blocs" folder into its respective slot: 29pt, 40pt and 60pt.

All this dragging and dropping has my clickin' hand beat, let's just code already…

### Start At The Back

Let's put those new background images to work. We'll begin by establishing `GameScene` inside of `GameViewController`. `GameScene` will be responsible for displaying everything for Swiftris - it will render the tetrominos on screen, the background, and the game board. Furthermore, `GameScene` will be responsible for playing the sounds and keeping track of the time.

`GameViewController`, on the other hand, will be responsible for handling user input and communicating between `GameScene` and a game logic class you'll write soon.

> If you're working with Swift for the first time, we highly encourage you to type each line by hand in order to get a feel for the language in your fingers… it sounds dirty but it's good for you.


```objc(GameScene.swift)
+required init(coder aDecoder: NSCoder!) {
+    fatalError("NSCoder not supported")
+}

+override init(size: CGSize) {
+    super.init(size: size)

+    anchorPoint = CGPoint(x: 0, y: 1.0)

+    let background = SKSpriteNode(imageNamed: "background")
+    background.position = CGPoint(x: 0, y: 0)
+    background.anchorPoint = CGPoint(x: 0, y: 1.0)
+    addChild(background)
+}
```

SpriteKit is based on OpenGL and therefore its coordinate system is opposite to iOS' native cocoa coordinates. `0, 0` in SpriteKit is the bottom-left corner. Swiftris will be drawn from the top down so therefore we anchor our game in the top-left corner of the screen: `0, 1.0`. We then create an `SKSpriteNode` capable of representing our background image and we add it to the scene.

>`background` is the variable's name, its type is inferred to be that of `SKSpriteNode` and the keyword `let` indicates that it can not be re-assigned. `let` is akin to Java's `final`.

```objc(GameViewController.swift)
+var scene: GameScene!

override func viewDidLoad() {
    super.viewDidLoad()

+    // Configure the view.
+    let skView = view as SKView
+    skView.multipleTouchEnabled = false

+    // Create and configure the scene.
+    scene = GameScene(size: skView.bounds.size)
+    scene.scaleMode = .AspectFill

+    // Present the scene.
+    skView.presentScene(scene)
}
```

In `GameViewController` we've added a member variable, `scene`. Its declaration: `var scene: GameScene!` lets us know that it is a variable, its name is `scene`, its type is `GameScene` and it is a non-optional value which will *eventually* be instantiated. Swift typically enforces instantiation either in-line where you declare the variable or during the initializer, `init…`. In order to circumvent this requirement we've added an `!` after the type.

In `viewDidLoad()` we assign `scene` as promised, using the initializer we had just written moments ago. We tell it to fill the screen and then ask our view to present that scene to the user. Run Swiftris and you should see a super cool background appear. Not titillating enough for you? Read on to continue the fun.
