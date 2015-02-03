
[Source](http://www.raywenderlich.com/87232/make-game-like-mega-jump-sprite-kit-swift-part-2 "Permalink to How to Make a Game Like Mega Jump With Sprite Kit and Swift: Part 2/2")

# How to Make a Game Like Mega Jump With Sprite Kit and Swift: Part 2/2

_Update note:_ This tutorial was updated for Swift and iOS 8 by Michael Briscoe. [Original post][4] by tutorial team member [Toby Stephens][5].

Welcome to the second part of the tutorial series that walks you through using Sprite Kit and Swift to create a game like _Mega Jump_.

In the [first part][6] of the tutorial, you created a new Sprite Kit game called "Uber Jump." You added graphics, a player sprite and some gameplay elements.

In this second part, you'll use that firm foundation to build an entire level for Uber Jump, including a scoring system. You'll also add accelerometer support so that your Uber Jumper can move from side to side as well as up and down. When you're done, you'll have a completely playable game that you could expand in many different ways.

As with [Part One][6], be sure you are familiar with [the basics of Sprite Kit][7] before continuing.

Your level awaits; so let's jump to it!

## Getting Started

If you don't have it already, [grab a copy][8] of the complete project from Part One.

Your level will contain many stars and platforms. Rather than arrange them manually, download [this][9] level configuration file, and drag _Level01.plist_ into your Xcode project. Make sure that _"Destination: Copy items if needed"_ is checked and that your _UberJump_ target is selected.

Open _Level01.plist_ and examine its contents. At the root, it has three elements:

* _EndY_ specifies the height the player must reach to finish the level.
* _Stars_ defines the positions of all the stars in the level.
* _Platforms_ defines the positions of all the platforms in the level.

The _Stars_ and _Platforms_ elements each contain two sub-elements:

* _Patterns_ contains a number of reusable patterns of stars or platforms.
* _Positions_ specifies where to place the patterns of stars or platforms throughout the level.

![jm_level_plist][10]

To better understand the file format, take a look at _Stars/Positions/Item 0_. This contains three elements telling the game to place stars in a cross pattern positioned at (160, 240).

![jm_level_plist1][11]

Now look at _Patterns/Cross_ and you'll see this pattern is made up of five items, including (x, y) coordinates relative to the position given in _Stars/Positions_ and the type of star, where `Normal` = 0 or `Special` = 1.

![jm_level_plist2][12]

This is simply a convenient way of reusing patterns of stars and platforms without having to code the position of every individual object.

![rrr][13]

## Loading the Level Data

To add support for loading the level from _Level01.plist_, open _GameScene.swift_ and add the following property to the class:

| ----- |
|

    // Height at which level ends
    let endLevelY = 0

 |

`endLevelY` will store the height, or y-value, that the player must reach to finish the level.

Insert the following code into `init(size:)`, just before the lines that instantiate and add a `platform`:

| ----- |
|

    // Load the level
    let levelPlist = NSBundle.mainBundle().pathForResource("Level01", ofType: "plist")
    let levelData = NSDictionary(contentsOfFile: levelPlist!)!
    &nbsp;
    // Height at which the player ends the level
    endLevelY = levelData["EndY"]!.integerValue!

 |

This loads the data from the property list into a dictionary named `levelData` and stores the property list's `EndY` value in `endLevelY`.

Now for the stars and platforms. Begin with the platforms. In `init(size:)`, replace the following lines:

| ----- |
|

    // Add a platform
    let platform = createPlatformAtPosition(CGPoint(x: 160, y: 320), ofType: .Normal)
    foregroundNode.addChild(platform)

 |

With this code:

| ----- |
|

    // Add the platforms
    let platforms = levelData["Platforms"] as NSDictionary
    let platformPatterns = platforms["Patterns"] as NSDictionary
    let platformPositions = platforms["Positions"] as [NSDictionary]
    &nbsp;
    for platformPosition in platformPositions {
      let patternX = platformPosition["x"]?.floatValue
      let patternY = platformPosition["y"]?.floatValue
      let pattern = platformPosition["pattern"] as NSString
    &nbsp;
      // Look up the pattern
      let platformPattern = platformPatterns[pattern] as [NSDictionary]
      for platformPoint in platformPattern {
        let x = platformPoint["x"]?.floatValue
        let y = platformPoint["y"]?.floatValue
        let type = PlatformType(rawValue: platformPoint["type"]!.integerValue)
        let positionX = CGFloat(x! + patternX!)
        let positionY = CGFloat(y! + patternY!)
        let platformNode = createPlatformAtPosition(CGPoint(x: positionX, y: positionY), ofType: type!)
        foregroundNode.addChild(platformNode)
      }
    }

 |

There's a lot going on here, but it's simple stuff. You load the `Platforms` dictionary from `levelData` and then loop through its `Positions` array. For each item in the array, you load the relevant pattern and instantiate a `PlatformNode` of the correct type at the specified (x, y) positions. You add all the platform nodes to the foreground node, where all the game objects belong.

Build and run. You'll see a set of three platforms aligned in the scene, which is the "Triple" pattern described in _Level01.plist_.

![16-Level01Platforms][14]

Now do the same for the stars. Inside _GameScene.swift_, replace the following line in `init(size:)`:

| ----- |
|

    // Add a star
    let star = createStarAtPosition(CGPoint(x: 160, y: 220), ofType: .Special)
    foregroundNode.addChild(star)

 |

With this code:

| ----- |
|

    // Add the stars
    let stars = levelData["Stars"] as NSDictionary
    let starPatterns = stars["Patterns"] as NSDictionary
    let starPositions = stars["Positions"] as [NSDictionary]
    &nbsp;
    for starPosition in starPositions {
      let patternX = starPosition["x"]?.floatValue
      let patternY = starPosition["y"]?.floatValue
      let pattern = starPosition["pattern"] as NSString
    &nbsp;
      // Look up the pattern
      let starPattern = starPatterns[pattern] as [NSDictionary]
      for starPoint in starPattern {
        let x = starPattern["x"]?.floatValue
        let y = starPattern["y"]?.floatValue
        let type = StarType(rawValue: starPattern["type"]!.integerValue)
        let positionX = CGFloat(x! + patternX!)
        let positionY = CGFloat(y! + patternY!)
        let starNode = createStarAtPosition(CGPoint(x: positionX, y: positionY), ofType: type!)
        foregroundNode.addChild(starNode)
      }
    }

 |

This is exactly what you did to create the platforms, but this time you create stars for the items in the `Stars` dictionary.

Build and run. This is starting to look like a real game!

![17-Level01Stars][15]

## The Midground Layer

Graphically, there's just one more thing to add to give the game a greater illusion of depth, and that's the midground layer. This is the node that's going to contain decorative graphics to bring the game to life.

Add the following method to _GameScene.swift_:

| ----- |
|

    func createMidgroundNode() -&gt; SKNode {
      // Create the node
      let theMidgroundNode = SKNode()
      var anchor: CGPoint!
      var xPosition: CGFloat!
    &nbsp;
      // 1
      // Add some branches to the midground
      for index in 0...9 {
        var spriteName: String
        // 2
        let r = arc4random() % 2
        if r &gt; 0 {
          spriteName = "BranchRight"
          anchor = CGPoint(x: 1.0, y: 0.5)
          xPosition = self.size.width
        } else {
          spriteName = "BranchLeft"
          anchor = CGPoint(x: 0.0, y: 0.5)
          xPosition = 0.0
        }
        // 3
        let branchNode = SKSpriteNode(imageNamed: spriteName)
        branchNode.anchorPoint = anchor
        branchNode.position = CGPoint(x: xPosition, y: 500.0 * CGFloat(index))
        theMidgroundNode.addChild(branchNode)
      }
    &nbsp;
      // Return the completed midground node
      return theMidgroundNode
    }

 |

Take a closer look at this code:

1. You add ten branches to `midgroundNode`, spaced evenly throughout the level.
2. There are two different branch images, one showing branches coming in from the left of the screen and the other from the right. Here you grab one randomly.
3. You space the branches at 500-point intervals on the y-axis of the midground node.

Now add the midground node to your scene by inserting the following code into `init(size:)`, immediately after the line that adds the background node:

| ----- |
|

    // Midground
    midgroundNode = createMidgroundNode()
    addChild(midgroundNode)

 |

Build and run. Look! It's a branch (of sorts) and maybe some pink butterflies!

![18-MidgroundNode][16]

_Note:_ The pink butterflies will only appear if the randomly-chosen branch is the one coming from the right side of the screen. The left branch image does not include the butterflies.

Tap to start the game and you will see the player sprite shoot up the screen. However, even as the Uber Jumper ascends, the game world remains still.

![19-MidgroundNodeRun][17]

The background, midground and foreground layers should all move with the player node to keep the player sprite in the center of the screen. You're going to sort that out next.

## Parallaxalization

No, that's not a word! ;]

To give your game the parallax effect, you're going to move the background, midground and foreground nodes at different rates as the player moves up and down the scene. Sprite Kit calls `update()` on your scene every frame, so that's the place to implement this logic to produce smooth animation.

Open _GameScene.swift_ and add the following method:

| ----- |
|

    override func update(currentTime: NSTimeInterval) {
      // Calculate player y offset
      if player.position.y &gt; 200.0 {
        backgroundNode.position = CGPoint(x: 0.0, y: -((player.position.y - 200.0)/10))
        midgroundNode.position = CGPoint(x: 0.0, y: -((player.position.y - 200.0)/4))
        foregroundNode.position = CGPoint(x: 0.0, y: -(player.position.y - 200.0))
      }
    }

 |

You check to make sure the player node has moved up the screen at least 200 points, because otherwise you don't want to move the background. If so, you move the three nodes down at different speeds to produce a parallax effect:

* You move the foreground node at the same rate as the player node, effectively keeping the player from moving any higher on the screen.
* You move the midground node at 25% of the player node's speed so that it appears to be farther away from the viewer.
* You move the background node at 10% of the player node's speed so that it appears even farther away.

Build and run, and tap to start the game. You'll see the layers all move with the player, and the different speeds of the background and midground nodes will produce a very pleasing parallax effect.

![20-Parallax][18]

Great work! But don't rest on your laurels yet. To get any higher in this world, you need to get your tilt on.

## Moving With the Accelerometer

It's now time to consider the accelerometer. You've got movement along the vertical axis working well, but what about movement along the horizontal axis? Just like in _Mega Jump_, the player will steer their Uber Jumper using the accelerometer.

_Note:_ To test accelerometer, you need to run your game on a device. The iPhone Simulator does not simulate accelerometer inputs.

The accelerometer inputs are part of the Core Motion library, so at the top of _GameScene.swift_, add the following import:

Then add the following properties to the `GameScene` class:

| ----- |
|

    // Motion manager for accelerometer
    let motionManager: CMMotionManager = CMMotionManager()
    &nbsp;
    // Acceleration value from accelerometer
    var xAcceleration: CGFloat = 0.0

 |

You're going to use `motionManager` to access the device's accelerometer data, and you'll store the most recently calculated acceleration value in `xAcceleration`, which you'll use later when you set the player node's velocity along the x-axis.

To instantiate your `CMMotionManager`, add the following code to `init(size:)`, just after the line that adds `tapToStartNode` to the HUD:

| ----- |
|

    // CoreMotion
    // 1
    motionManager.accelerometerUpdateInterval = 0.2
    // 2
    motionManager.startAccelerometerUpdatesToQueue(NSOperationQueue.currentQueue(), withHandler: {
      (accelerometerData: CMAccelerometerData!, error: NSError!) in
      // 3
      let acceleration = accelerometerData.acceleration
      // 4
      self.xAcceleration = (CGFloat(acceleration.x) * 0.75) + (self.xAcceleration * 0.25)
    })

 |

There's a lot going on here, so take a deeper dive:

1. `accelerometerUpdateInterval` defines the number of seconds between updates from the accelerometer. A value of 0.2 produces a smooth update rate for accelerometer changes.
2. You enable the accelerometer and provide a block of code to execute upon every accelerometer update.
3. Inside the block, you get the acceleration details from the latest accelerometer data passed into the block.
4. Here you calculate the player node's x-axis acceleration. You could use the x-value directly from the accelerometer data, but you'll get much smoother movement using a value derived from three quarters of the accelerometer's x-axis acceleration (say that three times fast!) and one quarter of the current x-axis acceleration.

Now that you have an x-axis acceleration value, you need to use that value to set the player node's velocity along the x-axis.

As you are directly manipulating the velocity of the player node, it's important to let Sprite Kit handle the physics first. Sprite Kit provides a method for you to implement to get that timing correct called `didSimulatePhysics`. Sprite Kit will also call this method once per frame, after the physics have been calculated and performed.

Add the following method to _GameScene.swift_:

| ----- |
|

    override func didSimulatePhysics() {
      // 1
      // Set velocity based on x-axis acceleration
      player.physicsBody?.velocity = CGVector(dx: xAcceleration * 400.0, dy: player.physicsBody!.velocity.dy)
      // 2
      // Check x bounds
      if player.position.x &lt; -20.0 {
        player.position = CGPoint(x: 340.0, y: player.position.y)
      } else if (player.position.x &gt; 340.0) {
        player.position = CGPoint(x: -20.0, y: player.position.y)
      }
    }

 |

A couple of things are happening here:

1. You change the x-axis portion of the player node's velocity using the `xAcceleration` value. You multiply it by 400.0 because the accelerometer scale does not match the physics world's scale and so increasing it produces a more satisfying result. You leave the velocity's y-axis value alone because the accelerometer has no effect on it.
2. In _Mega Jump_, when the player leaves the screen from the left or right, they come back onscreen from the other side. You replicate that behavior here by checking the bounds of the screen and leaving a 20-point border at the edges.

Build and run on your device, and use the accelerometer to steer the player sprite as high as you can!

![21-AccelerometerRun][19]

## The Scoring System

Your complete Uber Jump will have three pieces of information pertinent to the player:

* _The Current Score._ The score will start at zero. The higher the player gets, the more points you'll award toward the score. You'll also add points for each star the player collects.
* _The High Score._ Every run through the game will result in a final score. Uber Jump will save the highest of these in the app's user defaults so that the player always knows the score to beat.
* _The Stars._ While the current score will reset at the beginning of each game, the player's stars will accumulate from game to game. In a future version of Uber Jump, you may decide to make stars the in-game currency with which users can buy upgrades and boosts. You won't be doing that as part of this tutorial, but you'll have the currency set up in case you'd like to do it on your own.

You're going to store the current score, the high score and the number of the collected stars in a singleton class called `GameState`. This class will also be responsible for saving the high score and number of stars to the device so that the values persist between game launches.

Create a new _iOS/Source/Swift File_ called _GameState_. Add the following class definition and properties to _GameState.swift_:

| ----- |
|

    class GameState {
      var score: Int
      var highScore: Int
      var stars: Int
    &nbsp;
      class var sharedInstance: GameState {
        struct Singleton {
          static let instance = GameState()
        }
    &nbsp;
        return Singleton.instance
      }
    }

 |

These three properties will provide access to the current score, the high score and the star count. The class variable `sharedInstance` will provide access to the singleton instance of `GameState`.

You need to provide an initialization method for `GameState` that sets the current score to zero and loads any existing high score and star count from the user defaults.

Add the following `init` method to _GameState.swift_:

| ----- |
|

    init() {
      // Init
      score = 0
      highScore = 0
      stars = 0
    &nbsp;
      // Load game state
      let defaults = NSUserDefaults.standardUserDefaults()
    &nbsp;
      highScore = defaults.integerForKey("highScore")
      stars = defaults.integerForKey("stars")
    }

 |

`NSUserDefaults` is a simple way to persist small bits of data on the device. It's intended for user preferences, but in this example it works well to store the high score and star count. In a real app, you would want to use something more secure than `NSUserDefaults` as someone could easily tamper with the data stored there and give themselves more stars than they earned!

To store these values, you need a method in `GameState`. Add the following method to _GameState.swift_:

| ----- |
|

    func saveState() {
      // Update highScore if the current score is greater
      highScore = max(score, highScore)
    &nbsp;
      // Store in user defaults
      let defaults = NSUserDefaults.standardUserDefaults()
      defaults.setInteger(highScore, forKey: "highScore")
      defaults.setInteger(stars, forKey: "stars")
      NSUserDefaults.standardUserDefaults().synchronize()
    }

 |

You now have a `GameState` class that synchronizes with storage on the device. What good is a score if nobody knows what it is? You've got to show me the money!

## Building the HUD

Before you start awarding points, you're going to build a simple heads-up display (HUD) so that the player can see their score and star count.

Your HUD will show the total number of collected stars on the top-left of the scene and the current score on the top-right of the scene. For this, you need to create two `SKLabelNodes`.

Add the following class properties to the file _GameScene.swift_:

| ----- |
|

    // Labels for score and stars
    var lblScore: SKLabelNode!
    var lblStars: SKLabelNode!

 |

To build the HUD, add the following code to `init(size:)` in _GameScene.swift_, just before the line that initializes `motionManager`:

| ----- |
|

    // Build the HUD
    &nbsp;
    // Stars
    // 1
    let star = SKSpriteNode(imageNamed: "Star")
    star.position = CGPoint(x: 25, y: self.size.height-30)
    hudNode.addChild(star)
    &nbsp;
    // 2
    lblStars = SKLabelNode(fontNamed: "ChalkboardSE-Bold")
    lblStars.fontSize = 30
    lblStars.fontColor = SKColor.whiteColor()
    lblStars.position = CGPoint(x: 50, y: self.size.height-40)
    lblStars.horizontalAlignmentMode = SKLabelHorizontalAlignmentMode.Left
    &nbsp;
    // 3
    lblStars.text = String(format: "X %d", GameState.sharedInstance.stars)
    hudNode.addChild(lblStars)
    &nbsp;
    // Score
    // 4
    lblScore = SKLabelNode(fontNamed: "ChalkboardSE-Bold")
    lblScore.fontSize = 30
    lblScore.fontColor = SKColor.whiteColor()
    lblScore.position = CGPoint(x: self.size.width-20, y: self.size.height-40)
    lblScore.horizontalAlignmentMode = SKLabelHorizontalAlignmentMode.Right
    &nbsp;
    // 5
    lblScore.text = "0"
    hudNode.addChild(lblScore)

 |

Take a closer look at this section of code:

1. First you add a star graphic in the top-left corner of the scene to tell the player that the following number is the collected star count.
2. Next to the star, you place a left-aligned `SKLabelNode`.
3. You initialize the label with the number of stars from `GameState`.
4. You add a right-aligned `SKLabelNode` in the top-right corner of the scene.
5. You initialize that label to zero, as there is no score currently.

Build and run. You'll see the two labels at the top of the screen.

![23-HUD][20]

The last layer of your game is complete! Now let's have some points already!

## Awarding Points

It's finally time to award points for the player's hard work. There are two ways to score points in Uber Jump: climbing up the scene and collecting stars.

To award points for collecting stars, open _GameObjectNode.swift_ and simply add the following code to the bottom of `collisionWithPlayer` in `StarNode`, just before the `return` statement:

| ----- |
|

    // Award score
    GameState.sharedInstance.score! += (starType == .Normal ? 20 : 100)

 |

That's it! You add 20 points to the score for a normal star and 100 points for the special type.

To show this updated score, go back to `didBeginContact` in _GameScene.swift_. Recall from Part One of this tutorial that this method sets a flag named `updateHUD` to `true` when it determines that the values displayed in the HUD need to change.

Add the following two lines of code to `didBeginContact` inside the `if updateHUD {...}` condition curly braces. There should be a comment there that reads `TODO: Update HUD in Part 2`:

| ----- |
|

    lblStars.text = String(format: "X %d", GameState.sharedInstance.stars)
    lblScore.text = String(format: "%d", GameState.sharedInstance.score)

 |

Build and run, and tap to start the game. As you collect stars, watch your score increase.

![24-HUDPointsFromStars][21]

To work out when to award points for traveling up the screen, you need to store the highest point along the y-axis that the player has reached during this play-through. You'll use this data to increase the score only when the player reaches a new high point, rather than award points constantly as the player bobs up and down the screen.

Add the following class property to the file _GameScene.swift_:

| ----- |
|

    // Max y reached by player
    var maxPlayerY: Int

 |

The player node initially starts at a y-coordinate of 80.0, so you need to initialize `maxPlayerY` to 80 if you don't want to give the player 80 points just for starting the game. ;]

Add the following line to `init(size:)` in _GameScene.swift_, just after the line that sets the background color:

To increase the score when the player travels up the screen, go to the top of `update:`, and add the following lines:

| ----- |
|

    // New max height ?
    // 1
    if Int(player.position.y) &gt; maxPlayerY! {
      // 2
      GameState.sharedInstance.score! += Int(player.position.y) - maxPlayerY!
      // 3
      maxPlayerY = Int(player.position.y)
      // 4
      lblScore.text = String(format: "%d", GameState.sharedInstance.score)
    }

 |

This deserves closer inspection:

1. First, you check whether the player node has travelled higher than it has yet travelled in this play-through.
2. If so, you add to the score the difference between the player node's current y-coordinate and the max y-value.
3. You set the new max y-value.
4. Finally, you update the score label with the new score.

Build and run, and tap to start. As you play, your score will go up as you move higher through the level.

![25-HUDPointsFromY][22]

Now consider the star count. You need to increment it every time the player node collides with a star, so open _GameObjectNode.swift_, and add the following code to `collisionWithPlayer` within `StarNode`, just after the line that awards the points:

| ----- |
|

    // Award stars
    GameState.sharedInstance.stars! += (starType == .Normal ? 1 : 5)

 |

That's all that needs doing! Build and run, and tap to start. Watch the star count increase as you collect them.

![26-HUDStars][23]

As you play, you may notice that when you fall, all the game objects you passed are still in the game. "Hey!" you are surely thinking "I spent a considerable amount of time with Mega Jump, and I am sure that's not how it was in that game!" Yes, but that's easy to fix.

Recall that you added a method named `checkNodeRemoval` to `GameObjectNode` that checks whether or not to remove a node. It's now time to call that method every frame.

Add the following code to `update` in _GameScene.swift_, just before the line that checks if the player node's position is greater than 200:

| ----- |
|

    // Remove game objects that have passed by
    foregroundNode.enumerateChildNodesWithName("NODE_PLATFORM", usingBlock: {
      (node, stop) in
      let platform = node as PlatformNode
      platform.checkNodeRemoval(self.player.position.y)
    })
    &nbsp;
    foregroundNode.enumerateChildNodesWithName("NODE_STAR", usingBlock: {
      (node, stop) in
      let star = node as StarNode
      star.checkNodeRemoval(self.player.position.y)
    })

 |

Here you enumerate through the platforms in the foreground node and call `checkNodeRemoval` for each one. You then do the same for each of the stars.

Build and run, and tap to start. Now when you fall, it's nothing but empty sky!

![27-RemoveOldGameNodes][24]

## Game Over!

At the end of the game, when the player falls off the bottom of the scene or climbs to the top of the level, you want to show the final score and the current high score. You'll do it by transitioning to an end game scene.

Create a new _Cocoa Touch Class_ called _EndGameScene_ and make it a subclass of _SKScene_.

![NewClassEndGameScene][25]

`EndGameScene` will be a simple screen that shows the player's score, their star count, and the current high score. You add all the nodes in `init(size:):`, so open _EndGameScene.swift_ and replace the entire file contents with the following:

| ----- |
|

    import SpriteKit
    &nbsp;
    class EndGameScene: SKScene {
    &nbsp;
      required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
      }
    &nbsp;
      override init(size: CGSize) {
        super.init(size: size)
    &nbsp;
        // Stars
        let star = SKSpriteNode(imageNamed: "Star")
        star.position = CGPoint(x: 25, y: self.size.height-30)
        addChild(star)
    &nbsp;
        let lblStars = SKLabelNode(fontNamed: "ChalkboardSE-Bold")
        lblStars.fontSize = 30
        lblStars.fontColor = SKColor.whiteColor()
        lblStars.position = CGPoint(x: 50, y: self.size.height-40)
        lblStars.horizontalAlignmentMode = SKLabelHorizontalAlignmentMode.Left
        lblStars.text = String(format: "X %d", GameState.sharedInstance.stars)
        addChild(lblStars)
    &nbsp;
        // Score
        let lblScore = SKLabelNode(fontNamed: "ChalkboardSE-Bold")
        lblScore.fontSize = 60
        lblScore.fontColor = SKColor.whiteColor()
        lblScore.position = CGPoint(x: self.size.width / 2, y: 300)
        lblScore.horizontalAlignmentMode = SKLabelHorizontalAlignmentMode.Center
        lblScore.text = String(format: "%d", GameState.sharedInstance.score)
        addChild(lblScore)
    &nbsp;
        // High Score
        let lblHighScore = SKLabelNode(fontNamed: "ChalkboardSE-Bold")
        lblHighScore.fontSize = 30
        lblHighScore.fontColor = SKColor.cyanColor()
        lblHighScore.position = CGPoint(x: self.size.width / 2, y: 150)
        lblHighScore.horizontalAlignmentMode = SKLabelHorizontalAlignmentMode.Center
        lblHighScore.text = String(format: "High Score: %d", GameState.sharedInstance.highScore)
        addChild(lblHighScore)
    &nbsp;
        // Try again
        let lblTryAgain = SKLabelNode(fontNamed: "ChalkboardSE-Bold")
        lblTryAgain.fontSize = 30
        lblTryAgain.fontColor = SKColor.whiteColor()
        lblTryAgain.position = CGPoint(x: self.size.width / 2, y: 50)
        lblTryAgain.horizontalAlignmentMode = SKLabelHorizontalAlignmentMode.Center
        lblTryAgain.text = "Tap To Try Again"
        addChild(lblTryAgain)
      }
    }

 |

That's a lot of code, but by now, after all you've done in this tutorial, you've got pwnage.

In brief, you create three labels: one each to show the star count, the final score and the high score. You populate these labels with values from the `GameState` singleton. You also add a label explaining to the player that they can tap the screen to play again.

To track whether or not the game is over, add the following `Bool` to the class properties in _GameScene.swift_:

| ----- |
|

    // Game over dude!
    var gameOver = false

 |

At the end of the game, the `GameState` singleton needs to save the current state and transition to the new scene. Add the following method to _GameScene.swift_:

| ----- |
|

    func endGame() {
      // 1
      gameOver = true
    &nbsp;
      // 2
      // Save stars and high score
      GameState.sharedInstance.saveState()
    &nbsp;
      // 3
      let reveal = SKTransition.fadeWithDuration(0.5)
      let endGameScene = EndGameScene(size: self.size)
      self.view!.presentScene(endGameScene, transition: reveal)
    }

 |

Look at this method in detail:

1. First you set `gameOver` to `true`.
2. Then you instruct the `GameState` singleton to save the game state to the app's user defaults.
3. Finally, you instantiate an `EndGameScene` and transition to it by fading over a period of 0.5 seconds.

The game needs to call `endGame` when the player node either falls off the bottom of the screen or reaches the maximum height for the level. You'll test for both of these triggers in `update` in the main scene.

Still in _GameScene.swift_, add the following code to the end of `update:`:

| ----- |
|

    // 1
    // Check if we've finished the level
    if Int(player.position.y) &gt; endLevelY {
      endGame()
    }
    &nbsp;
    // 2
    // Check if we've fallen too far
    if Int(player.position.y) &lt; maxPlayerY - 800 {
      endGame()
    }

 |

Take a look at these checks:

1. Remember, you loaded `endLevelY` from the level's property list; it's the y-value at which the player has finished the level.
2. If the player node falls by more than 800 points below the max height they've reached, then it's game over.

Before you can run the game to see your end game scene, you need to make sure the game doesn't try to call `endGame` more than once, which might happen if `update()` runs again for another frame.

Add the following line to the start of `update()` in _GameScene.swift_:

Now, when the game calls `update()`, the method checks to see if the game is already over before progressing.

Build and run. Tap to start and then play the game, allowing the Uber Jumper to fall at some point. The game will transition to the end game scene.

![UberJump-EndGameScene][26]

## Restarting

To restart the game from the end game scene, you need to modify `EndGameScene` so it transitions back to the main game scene upon a touch event.

Open _EndGameScene.swift_ and add the following method to the class to handle touch events:

| ----- |
|

    override func touchesBegan(touches: NSSet, withEvent event: UIEvent) {
      // Transition back to the Game
      let reveal = SKTransition.fadeWithDuration(0.5)
      let gameScene = GameScene(size: self.size)
      self.view!.presentScene(gameScene, transition: reveal)
    }

 |

This code simply transitions to a new `GameScene` in much the same way you transition to this scene when the app first starts.

One more bit of code, and you're all done! Open _GameScene.swift_. In `init(size:)`, where you reset `maxPlayerY`, add the following code:

| ----- |
|

    GameState.sharedInstance.score = 0
    gameOver = false

 |

This resets the score in the `GameState` singleton and also resets the `gameOver` flag so the game can start fresh.

Build and run. Tap to start and play the game. Now go and beat that high score! ;]

## Where to Go From Here?

Congratulations, you've made a game like _Mega Jump_!

Here's the [finished project][27] with all of the code from this tutorial series.

In the 2 parts of this series you've covered the whole process of creating a physics based game. You learned much about how to setup collisions and how to build your game logic around detecting contacts in the game.

But you can do much more for your Uber Jump game! Open up the plist level data and think of new levels. What new shapes could you implement? You can just copy the plist source code and create any number of levels by adjusting what's inside the level file.

Play some _Mega Jump_ and get inspiration! Can you implement some better game features compared to the original game?

And last but not least if you want to learn more about Sprite Kit, check out the [iOS Games by Tutorials][28] book, where you'll learn how to make five complete games—from zombie action to car racing to shootouts in space!

In the meantime, if you have any questions or comments, please join the forum discussion below.

[1]: http://www.raywenderlich.com/feed/
[2]: http://twitter.com/rwenderlich
[3]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/01/FeatureImage.png
[4]: http://www.raywenderlich.com/63578/make-game-like-mega-jump-spritekit-part-22
[5]: http://www.raywenderlich.com/u/tobystephens
[6]: /?p=87231 "How to Make a Game Like Mega Jump With Sprite Kit and Swift: Part 1/2"
[7]: /?p=84434 "Sprite Kit Tutorial for Beginners"
[8]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/11/UberJump-Part-1.zip
[9]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/11/Level01.zip
[10]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/02/jm_level_plist.png
[11]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/02/jm_level_plist1.png
[12]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/02/jm_level_plist2.png
[13]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/02/rrr.png
[14]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/16-Level01Platforms-284x500.png
[15]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/17-Level01Stars-281x500.png
[16]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/18-MidgroundNode-283x500.png
[17]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/19-MidgroundNodeRun-282x500.png
[18]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/20-Parallax-284x500.png
[19]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/21-AccelerometerRun-281x500.jpeg
[20]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/23-HUD-283x500.png
[21]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/24-HUDPointsFromStars-283x500.png
[22]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/01/25-HUDPointsFromY-282x500.png
[23]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/26-HUDStars-283x500.png
[24]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/27-RemoveOldGameNodes-281x500.jpeg
[25]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/11/NewClassEndGameScene-700x413.png
[26]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/02/UberJump-EndGameScene-281x500.png
[27]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/11/UberJump-Part-2.zip
[28]: /?page_id=48022 "iOS Games by Tutorials"
  