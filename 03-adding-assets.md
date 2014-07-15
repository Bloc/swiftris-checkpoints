## Adding Assets

Don't get me wrong, *Spin-The-Bottle: Space Edition* was a great game. However, you didn't start this Bloc book to make that. At least I hope not, if so, please stop now because your quest is over. For those of you still interested in building *Swiftris,* we must unceremoniously delete every unnecessary file provided to us by Xcode.

Open Project Navigator by either clicking the designated icon or pressing <key>⌘ + 1</key>:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-project-navigator-button.png)</center>

Right-click `GameScene.sks` and choose the **Delete** option:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-delete-gamescene.png)</center>

When asked to confirm, make sure to choose **Move to trash**:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-delete-gamescene-confirm.png)</center>

To get rid of the aimless space ship once and for all, click the `Images.xcassets` folder and highlight the `Spaceship` entry, press <key>backspace</key> to delete that sucker.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-delete-spaceship.png)</center>

### Trimming The Fat

Having slaughtered those files which are of no use, we must now purge our project of any and all *code* which we do not require. There's no need to have lingering source code designed to support inept space pilots. Delete all of the lines marked in red within their corresponding files:

```ruby(GameScene.swift)
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

```ruby(GameViewController.swift)
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

In order to experience *Swiftris* in all its visual and auditory glory, we're going to need images and sounds, respectively. [Download the necessary assets]() to your `Desktop` or `Downloads` folder, anywhere other than the Swiftris project directory. Unzip the archive, perform a drag-and-drop action of the `Sounds` folder right above `Supporting Files` in Xcode's Project Navigator. The following window should appear:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/03-adding-assets-copy-if-necessary.png)</center>

Make sure to check the **Copy items if necessary** option. This will place a copy of the directory and all of the sound files within it into your Swiftris project and directory. Repeat this task with the `Sprites.atlas` folder. Next, select all of the images found within the `Images` directory and drag them into the `Supporting Files` folder found in Project Navigator, again make sure the items get copied.