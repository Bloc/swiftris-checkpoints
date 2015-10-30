## Introduction to Swiftris

Today you will begin *putting the pieces together* for a brand new game – see what we did there? To most, Swiftris resembles in name and in nearly every other respect a game written in the 1980s that people from all around the world still play to this day. Rest assured, Bloc is certain that any semblance to that game is a pure coincidence.

In all seriousness, this is a Tetris clone written in Swift for the iOS platform. This Bloc Book is strictly for educational purposes and we do not recommend releasing your version of Swiftris to the App Store. If you do release Swiftris anyway, hope that you never cross paths with [Alexey Pajitnov](http://en.wikipedia.org/wiki/Alexey_Pajitnov). As you can see, he is a dangerous man.

<center>![Alexey Pajitnov](http://bloc-global-assets.s3.amazonaws.com/screencaps/alexey.jpg)</center>

Before we start playing with blocks, you should know the tools we'll be using: Swift, SpriteKit and Xcode.

### Swift

Swift is Apple's latest programming language. In time it will replace Objective-C as the primary language for iOS and Mac apps. We wrote Swiftris entirely in Swift and this book will present a wide variety of the language's capabilities.

If you are not a programmer, *do not worry*. Regardless of skill level, you will have your own copy of Swiftris after completing this guide. This book does not intend to teach you the language in its entirety. We cover most aspects of it in brief and supplement with external resources.

### SpriteKit

SpriteKit is a set of APIs provided by the iOS SDK (software development kit) which allow native 2D game development from within Xcode. SpriteKit powers Swiftris, so this great game does not require other libraries or 3rd party tools.

### Xcode

[Download Xcode 7](https://developer.apple.com/xcode/downloads/) before continuing.

While it's not required for this book, you may want to consider signing up for the [iOS Developer Program](https://developer.apple.com/programs/ios/). We think the $99 annual fee is wholly worthwhile: it provides access to yet-to-be public software updates and allows you to publish apps to your iPhone and the App Store.

Once you have Xcode downloaded and installed, you are ready to move to the next chapter.

>**We wrote Swiftris for Xcode 7.1.** Attempting to follow this tutorial with any version of Xcode other than 7.1 may result in syntax or interface errors.
