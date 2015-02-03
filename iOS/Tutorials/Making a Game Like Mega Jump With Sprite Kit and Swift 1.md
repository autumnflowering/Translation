
[Source](http://www.raywenderlich.com/87231/make-game-like-mega-jump-sprite-kit-swift-part-1 "Permalink to How to Make a Game Like Mega Jump With Sprite Kit and Swift: Part 1/2")

# How to Make a Game Like Mega Jump With Sprite Kit and Swift: Part 1/2

![Create a game like Mega Jump using Sprite Kit][1]

Make A Game Like Mega Jump Using Sprite Kit

_Update note:_ This tutorial was updated for Swift and iOS 8 by Michael Briscoe. [Original post][2] by tutorial team member [Toby Stephens][3].

In this two-part tutorial series, you will use Sprite Kit and Swift to create a game in the style of _Mega Jump_. If you've somehow missed trying _Mega Jump_ from the App Store, it's quite a popular vertical jumping game with really nice graphics and addictive gameplay. As you build your game, you'll learn how to use a number of Sprite Kit's capabilities, including the physics engine, collision detection, accelerometer control and scene transitions.

If you are new to Sprite Kit, it would be best to work your way through Ray's [Sprite Kit Tutorial for Beginners][4] before starting this tutorial series.

But if you're comfortable with the basics of Sprite Kit, then jump right in!

_Note:_ The game you'll build will eventually use the accelerometer to control horizontal movement. That means you'll need a paid developer account and a device to test on to get the most out of this tutorial.

## Getting Started

You are going to create a game similar to _Mega Jump_. Yours will obviously be a truly amazing game and so it needs a truly amazing title. As everyone knows, "uber" is better than "mega," so let's call your game "Uber Jump." ;]

At its heart, a game like _Mega Jump_ is a physics game. The game hurls the player character up the screen and from that point the player must fight against gravity to get as high as they can. Collecting coins boosts the player upward while platforms provide temporary respite along the way.

Uber Jump will take the same model, initially thrusting your player character upward and continuously applying gravity to pull them down. Collecting stars along the way will propel the player further on their journey upward, taking care of the up and down (y-axis) portion of the player's movement (miss too many stars and you fall down though!). The accelerometer will handle the player's left and right movement (x-axis).

The result will be a complex, hybrid physics manipulation, as you both apply forces to the player node and also directly change its velocity. It'll also be lots of fun!

![][5]

To begin, you need a brand new Sprite Kit Xcode project. Fire up Xcode, select _FileNewProject_, choose the _iOSApplicationSpriteKit Game_ template and click _Next_.

![NewProject][6]

Enter _UberJump_ for the Product Name, set the Language to _Swift_, and _iPhone_ for Devices and then click _Next_.

![NewProjectName][7]

Choose somewhere to save the project and click _Create_.

Before getting down to business, you'll do some preliminary setup. Locate the _GameScene.sks_ file in the _Project Navigator_. You won't be using this, so select it and press the _delete_ key. When the warning alert shows, choose _Move to Trash_.

![DeleteGameScene][8]

Now open _GameViewController.swift_ and delete the `unarchiveFromFile` SKNode extension at the top of the file, as you won't be needing this extraneous code. Then replace `viewDidLoad` with the following:

| ----- |
|

    override func viewDidLoad() {
      super.viewDidLoad()
    &nbsp;
      let skView = self.view as SKView
    &nbsp;
      skView.showsFPS = true
      skView.showsNodeCount = true
    &nbsp;
      let scene = GameScene(size: skView.bounds.size)
      scene.scaleMode = .AspectFit
    &nbsp;
      skView.presentScene(scene)
    }

 |

Build and run to make sure everything is working so far.

![FirstRun][9]

Don't worry about the _"Hello, World"_ text label clipping. You'll be replacing what's in the _GameScene.swift_ file in just a moment, but first you have one last thing to setup. Since your game is going to use the accelerometer, you need to make sure that tilting the device doesn't flip the game into landscape mode.

With your project selected in the _Project Navigator_ in Xcode, select the _UberJump_ target, go to the _General_ settings tab and look in the _Deployment Info_ section. Ensure that the only available device orientation is _Portrait_.

![NewProjectPortrait][10]

Now you are ready to start adding your game's components. First up: the pretty pictures.

## Importing the Art

Before you get down to adding Sprite Kit nodes to your game, you need some art.

Download the [graphics resources for this project][11], drag the contents into your Xcode project. Make sure that _"Destination: Copy items if needed"_ is checked and that your _UberJump_ target is selected.

![uj_asset_files][12]

I've split the artwork into _background tiles_ and _game assets_.

Background tiles are large tiles that scroll up and down the screen as the player moves:

![uj_bg_thumb][13]

The game assets are all of the sprites, such as the player character, platforms and stars:

![Player@2x][14]

The game assets are stored in a texture atlas for efficiency. You can learn more about texture atlases in Sprite Kit [in this tutorial][15].

_Note:_ I pieced together the graphics for Uber Jump using the amazing artwork provided by [lostgarden.com][16]. This website provides high-quality graphics for devs to use in prototyping their games and is well worth a visit. You can check their [exact licensing policy][17].

## Building the Scene

In your project, you can see that the Sprite Kit template has already created the `GameScene` class for you. This is the Sprite Kit scene that is currently showing the "Hello, World!" message when you run the game. Most of Uber Jump's action will take place here, so open _GameScene.swift_ and replace the contents with the following:

| ----- |
|

    import SpriteKit
    &nbsp;
    class GameScene: SKScene {
    &nbsp;
      required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
      }
    &nbsp;
      override init(size: CGSize) {
        super.init(size: size)
        backgroundColor = SKColor.whiteColor()
      }
    }

 |

Build and run. `GameScene` now displays a simple white screen. This is the blank canvas onto which you'll add your game nodes.

_Mega Jump_ uses parallaxed layers to produce a neat visual representation of speed. (Eg. things in front move faster than things in the background.) In Uber Jump, you're going to produce the same effect by creating the following layers in your scene:

* _Background:_ a slow-moving layer that shows the distant landscape.
* _Midground:_ faster-moving scenery made up of tree branches.
* _Foreground:_ the fastest layer, containing the player character, stars and platforms that make up the core of the gameplay.
* _HUD:_ the top layer that does not move and displays the score labels.

First, add the background node. Open _GameScene.swift_ and add the following method:

| ----- |
|

    func createBackgroundNode() -&gt; SKNode {
      // 1
      // Create the node
      let backgroundNode = SKNode()
      let ySpacing = 64.0 * scaleFactor
    &nbsp;
      // 2
      // Go through images until the entire background is built
      for index in 0...19 {
        // 3
        let node = SKSpriteNode(imageNamed:String(format: "Background%02d", index + 1))
        // 4
        node.setScale(scaleFactor)
        node.anchorPoint = CGPoint(x: 0.5, y: 0.0)
        node.position = CGPoint(x: self.size.width / 2, y: ySpacing * CGFloat(index))
        //5
        backgroundNode.addChild(node)
      }
    &nbsp;
      // 6
      // Return the completed background node
      return backgroundNode
    }

 |

Let's take a closer look at what you're doing here:

1. First, you create a new `SKNode`. SKNode's have no visual content, but do have a position in the scene. This means you can move the node around and its child nodes will move with it.
2. You have 20 background images to stack to complete the background.
3. Each child node is made up of an `SKSpriteNode` with the sequential background image loaded from your resources.
4. Changing each node's anchor point to its bottom center makes it easy to stack in sections.
5. You add each child node to the background node.
6. Finally, you return the background node.

Now you can add the background node to your scene. Still in _GameScene.swift_, add the following class properties:

| ----- |
|

    // Layered Nodes
    let backgroundNode = SKNode()
    let midgroundNode = SKNode()
    let foregroundNode = SKNode()
    let hudNode = SKNode()
    &nbsp;
    // To Accommodate iPhone 6
    let scaleFactor: CGFloat = 0.0

 |

You are adding properties for each of the nodes you need for the game. You only need the background node for the moment, but it doesn't hurt to add the other node declarations now. You are also adding the _scaleFactor_ property. This ensures that your graphics are scaled and positioned properly across all iPhone models.

To add the background to the scene, insert the following into `init(size:)` in _GameScene.swift_, just after the line that sets the background color:

| ----- |
|

    scaleFactor = self.size.width / 320.0
    &nbsp;
    // Create the game nodes
    // Background
    backgroundNode = createBackgroundNode()
    addChild(backgroundNode)

 |

The graphics are sized for the standard 320-point width of most iPhone models, so the scale factor here will help with the conversion on other screen sizes.

All you need to do after initializing the background node is to add it as a child to the current scene.

Build and run to see your background node displayed in the scene, as shown below:

![05-BackgroundNode][18]

_Note:_ Although you added 20 nodes to the background node, you'll see that the node count in the scene is only nine nodes if you're running on a 4″ display or eight nodes if you're running on a 3.5″ display. Sprite Kit is clever enough to include only the nodes that are actually visible at any given moment in the game.

## Adding the Player Node

It's time for your Uber Jumper to enter the scene. In _GameScene.swift_, add the following method:

| ----- |
|

    func createPlayer() -&gt; SKNode {
      let playerNode = SKNode()
      playerNode.position = CGPoint(x: self.size.width / 2, y: 80.0)
    &nbsp;
      let sprite = SKSpriteNode(imageNamed: "Player")
      playerNode.addChild(sprite)
    &nbsp;
      return playerNode
    }

 |

As with the background node you added earlier, you create a new `SKNode` and add the `SKSpriteNode` containing the player sprite to it as a child. You position the player node so that it is horizontally centered and just above the bottom of the scene.

To add the player node to the scene, you first need to create a foreground node. As discussed above, the foreground node will contain the player, the stars and the platforms. That means when you move the foreground node, all the game elements it contains will move together.

Add the following to `init(size:)`, just after the line that adds `backgroundNode` to the scene:

| ----- |
|

    // Foreground
    foregroundNode = SKNode()
    addChild(foregroundNode)

 |

Now that you have your foreground node for the gameplay elements, you can add the player node to it.

At the top of _GameScene.swift_, add the following property along with the others you added to hold the layer nodes:

| ----- |
|

    // Player
    let player = SKNode()

 |

Now add the player node to the scene by inserting the following into `init(size:)`, just after the line that adds `foregroundNode` to the scene:

| ----- |
|

    // Add the player
    player = createPlayer()
    foregroundNode.addChild(player)

 |

Build and run to see your Uber Jumper ready to start a new adventure:

![06-ForegroundNode][19]

## Adding Gravity and a Physics Body

The Uber Jumper looks a little too comfortable sitting there, so you're going bring some physics into play and see what happens.

First, your game can't have physics without gravitation. In _GameScene.swift_, add the following line to `init(size:)`, after the line that sets the background color:

| ----- |
|

    // Add some gravity
    physicsWorld.gravity = CGVector(dx: 0.0, dy: -2.0)

 |

You add a suitable amount of gravity to the physics world, based on the gameplay you want to achieve. Gravity has no influence along the x-axis, but produces a downward force along the y-axis.

Build and run to see how this gravity affects the player node.

![06-ForegroundNode][19]

Hmm… Nothing happened. Why isn't the gravity affecting the player node? See if you can figure it out yourself before clicking below to see the answer.

| ----- |
| Solution Inside: Solution |  [Select][20][Show&gt;][20] |
|

The force produced by the gravity in the physics world only affects dynamic physics bodies in the scene, but SKNode objects like your player node do not have physics bodies defined by default. You need to define a physics body for your player node.

 |  |

In `createPlayer`, add the following just before the `return` statement at the end:

| ----- |
|

    // 1
    playerNode.physicsBody = SKPhysicsBody(circleOfRadius: sprite.size.width / 2)
    // 2
    playerNode.physicsBody?.dynamic = true
    // 3
    playerNode.physicsBody?.allowsRotation = false
    // 4
    playerNode.physicsBody?.restitution = 1.0
    playerNode.physicsBody?.friction = 0.0
    playerNode.physicsBody?.angularDamping = 0.0
    playerNode.physicsBody?.linearDamping = 0.0

 |

The above code defines the player node's physics body. Take a look at it in detail:

1. Each physics body needs a shape that the physics engine can use to test for collisions. The most efficient body shape to use in collision detection is a circle (easier to detect if overlaps another circle you see), and fortunately a circle fits your player node very well. The radius of the circle is half the width of the sprite.
2. Physics bodies can be static or dynamic. Dynamic bodies are influenced by the physics engine and are thus affected by forces and impulses. Static bodies are not, but you can still use them in collision detection. A static body such as a wall or a solid platform will never move, but things can bump into it. Since you want your player node to be affected by gravity, you set its `dynamic` property to `true`.
3. You want your player node to remain upright at all times and so you disable rotation of the node.
4. Since you're handling collisions yourself in this game, you adjust the settings on the player node's physics body so that it has no friction or damping. However, you set its `restitution` to 1, which means the physics body will not lose any of its momentum during collisions.

Build and run to see your player sprite fall off the bottom of the screen as gravity draws it relentlessly toward the Earth's core.

![05-BackgroundNode][18]

Perhaps that sounded a little melodramatic, but how is an Uber Jumper supposed to get anywhere in this world?

## Tap To Start

Now that gravity is working, you're going to give the player sprite a fighting chance and make its physics body static until the user actually decides to start playing.

Still inside _GameScene.swift_, find the line in `createPlayer` that sets the `dynamic` property on `playerNode`'s physics body. Change it to `false`, as shown here:

| ----- |
|

    playerNode.physicsBody?.dynamic = false

 |

You are going to write code to allow the player to start the game by tapping the screen. The player needs to know this, though, so first you need to put some instructions on the screen.

Since the instructions will go in your HUD node, create that layer first. In `init(size:)`, just after where you add the foreground node to the scene, insert the following code:

| ----- |
|

    // HUD
    hudNode = SKNode()
    addChild(hudNode)

 |

The graphic resources include an image that tells the player to tap to start the game, so add the following variable to the properties at the top of _GameScene.swift_:

| ----- |
|

    // Tap To Start node
    let tapToStartNode = SKSpriteNode(imageNamed: "TapToStart")

 |

To show the node, add the following code to `init(size:)`, just after the line that adds the player node:

| ----- |
|

    // Tap to Start
    tapToStartNode.position = CGPoint(x: self.size.width / 2, y: 180.0)
    hudNode.addChild(tapToStartNode)

 |

Build and run, and you will see the instruction to "Tap to Start" above the player sprite:

![07-TapToStart][21]

To respond to touch events and start the game, add the following method to _GameScene.swift_:

| ----- |
|

    override func touchesBegan(touches: NSSet, withEvent event: UIEvent) {
      // 1
      // If we're already playing, ignore touches
      if player.physicsBody!.dynamic {
        return
      }
    &nbsp;
      // 2
      // Remove the Tap to Start node
      tapToStartNode.removeFromParent()
    &nbsp;
      // 3
      // Start the player by putting them into the physics simulation
      player.physicsBody?.dynamic = true
    &nbsp;
      // 4
      player.physicsBody?.applyImpulse(CGVector(dx: 0.0, dy: 20.0))
    }

 |

Take a look at the method in detail:

1. Check to see if the player node is already dynamic. If so, you ignore the touch event and return.
2. Remove the Tap to Start node.
3. Change the player node's physics body to dynamic so that the physics engine can influence it.
4. Give the player node an initial upward impulse to get them started.

Build and run. When you tap the screen, the game removes the Tap to Start node and thrusts the player sprite upward, albeit briefly, before gravity takes over.

## Game Objects: Reach for the Stars!

To give your player something to do—as well as some upward momentum—it's "high" time for you to add stars to your game. Stars have a key role in Uber Jump: They are what the player needs to grab to move higher into the level.

Initially, you will add one star to the game and implement it fully. Then in part two of this tutorial, you'll complete the level.

In Uber Jump, as in _Mega Jump_, when the player sprite goes a certain distance higher than a star, platform or any other game object, the game will remove that object from the scene. Since star and platform nodes share this functionality, it makes sense to create a subclass of `SKNode` for all game objects.

Create a new _Cocoa Touch Class_ called _GameObjectNode_ and make it a subclass of _SKNode_. Be sure that the _Langauge:_ is set to _Swift_.

![NewClassGameObjectNode][22]

`GameObjectNode` will provide the following functionality:

* It will remove the game object from the scene if the player node has passed it by more than a set distance.
* It will handle collisions between the player node and the object. This method will return a `Bool` that informs the scene whether the collision with the game object has resulted in a need to update the HUD—for example, if the player has scored points.

Replace the code in _GameObjectNode.swift_ with the following:

| ----- |
|

    import SpriteKit
    &nbsp;
    class GameObjectNode: SKNode {
      func collisionWithPlayer(player: SKNode) -&gt; Bool {
        return false
      }
    &nbsp;
      func checkNodeRemoval(playerY: CGFloat) {
        if playerY &gt; self.position.y + 300.0 {
          self.removeFromParent()
        }
      }
    }

 |

You'll call `collisionWithPlayer` whenever the player node collides with this object, and you'll call `checkNodeRemoval` every frame to give the node a chance to remove itself.

In the `GameObjectNode` class, `collisionWithPlayer` is simply a stub. You will define the full method in each of your game object subclasses.

`checkNodeRemoval` checks to see if the player node has traveled more than 300 points beyond this node. If so, then the method removes the node from its parent node and thus, removes it from the scene.

### The Star Class

Now that you have a base class for the interactive game nodes, you can create a subclass for your stars. To keep things simple, you'll add all your `GameObjectNode` subclasses within the _GameObjectNode.swift_ file. Add the following code after your `GameObjectNode` class:

| ----- |
|

    class StarNode: GameObjectNode {
      override func collisionWithPlayer(player: SKNode) -&gt; Bool {
        // Boost the player up
        player.physicsBody?.velocity = CGVector(dx: player.physicsBody!.velocity.dx, dy: 400.0)
    &nbsp;
        // Remove this Star
        self.removeFromParent()
    &nbsp;
        // The HUD needs updating to show the new stars and score
        return true
      }
    }

 |

Collision with a star boosts the player node up the y-axis. You may be thinking, _"Why am I not using a force or impulse in the physics engine to do this?"_

If you were to apply the star boost as a force or impulse, it wouldn't always have the same effect. For example, if the player node were moving down the screen when it collided with the star, then the force would have a much weaker effect on the player than if the player were already moving up the screen.

The following diagram shows a very simplified visualization of this:

![10-ForcesAndBounce][23]

A solution to this problem is to change the player node's velocity directly. The player node's velocity is obviously made up of an x-axis speed and a y-axis speed.

The x-axis speed needs to stay the same, since it is only affected by the accelerometer, which you'll implement later. In the method above, you set the y-axis velocity to 400 on collision — a fixed amount so that the collision has the same effect no matter what the player node is doing when it collides with the star.

Open _GameScene.swift_ and add the following method:

| ----- |
|

    func createStarAtPosition(position: CGPoint) -&gt; StarNode {
      // 1
      let node = StarNode()
      let thePosition = CGPoint(x: position.x * scaleFactor, y: position.y)
      node.position = thePosition
      node.name = "NODE_STAR"
    &nbsp;
      // 2
      var sprite: SKSpriteNode!
      sprite = SKSpriteNode(imageNamed: "Star")
      node.addChild(sprite)
    &nbsp;
      // 3
      node.physicsBody = SKPhysicsBody(circleOfRadius: sprite.size.width / 2)
    &nbsp;
      // 4
      node.physicsBody?.dynamic = false
    &nbsp;
      return node
    }

 |

The code above should all look familiar by now:

1. You instantiate your `StarNode` and set its position.
2. You then assign the star's graphic using an `SKSpriteNode`.
3. You've given the node a circular physics body – you use it for collision detection with other objects in the game.
4. Finally, you make the physics body static, because you don't want gravity or any other physics simulation to influence the stars.

Now add the following code to `init(size:)`, just _before_ where you create the player node. You want the stars to be behind the player in the foreground node and so you need to add them before you add the player node.

| ----- |
|

    // Add a star
    let star = createStarAtPosition(CGPoint(x: 160, y: 220))
    foregroundNode.addChild(star)

 |

Build and run the game. Tap to start and watch the player sprite collide with the star.

![11-FirstStar][24]

The Uber Jumper bonks its head on the star. That wasn't the plan! Why do you think that happened? Take a guess.

| ----- |
| Solution Inside: Solution |  [Select][20][Show&gt;][20] |
|

The physics engine handles the collision between the player and star nodes. The player node's physics body hits the star node's physics body, which is static and thus immovable. The star node blocks the player node.

 |  |

### Collision Handling and Bit Masks

To handle collisions between the player and star nodes, you need to capture the collision event and call the `GameObjectNode` method `collisionWithPlayer`.

This is a good time to take a look at _collision bit masks_ in Sprite Kit.

When setting up collision information for your physics bodies, there are three bit mask properties you can use to define the way the physics body interacts with other physics bodies in your game:

* `categoryBitMask` defines the collision categories to which a physics body belongs.
* `collisionBitMask` identifies the collision categories with which this physics body should collide. Here, "collide" means to bump into each other as physical objects would. For example, in a third-person shooter you may want your player sprite to collide with enemy sprites, but pass through other player sprites.
* `contactTestBitMask` tells Sprite Kit that you would like to be informed when this physics body makes contact with a physics body belonging to one of the categories you specify. For example, in your game you want Sprite Kit to tell you when the player sprite touches a star or a platform. Using the correct combination of settings for the `contactTestBitMask` and the `collisionBitMask`, you can tell Sprite Kit to let objects pass through each other but still notify you when that happens so you can trigger events.

First, define your categories. Open _GameObjectNode.swift_ and add the following `struct` above the `class` definitions:

| ----- |
|

    struct CollisionCategoryBitmask {
      static let Player: UInt32 = 0x00
      static let Star: UInt32 = 0x01
      static let Platform: UInt32 = 0x02
    }

 |

Go back to _GameScene.swift_. To set up the player node's collision behavior, add the following code to the bottom of `createPlayer`, just before the `return` statement:

| ----- |
|

    // 1
    playerNode.physicsBody?.usesPreciseCollisionDetection = true
    // 2
    playerNode.physicsBody?.categoryBitMask = CollisionCategoryBitmask.Player
    // 3
    playerNode.physicsBody?.collisionBitMask = 0
    // 4
    playerNode.physicsBody?.contactTestBitMask = CollisionCategoryBitmask.Star | CollisionCategoryBitmask.Platform

 |

Let's take a closer look at this code block:

1. Since this is a fast-moving game, you ask Sprite Kit to use precise collision detection for the player node's physics body. After all, the gameplay for Uber Jump is all about the player node's collisions, so you'd like it to be as accurate as possible! (it costs few more cpu cycles, but the fun is considerably more!)
2. This defines the physics body's category bit mask. It belongs to the `CollisionCategoryPlayer` category.
3. By setting `collisionBitMask` to zero, you're telling Sprite Kit that you don't want its physics engine to simulate any collisions for the player node. That's because you're going to handle those collisions yourself!
4. Here you tell Sprite Kit that you want to be informed when the player node touches any stars or platforms.

Now set up the star node. Add the following code to the bottom of `createStarAtPosition:`, just before the `return` statement:

| ----- |
|

    node.physicsBody?.categoryBitMask = CollisionCategoryBitmask.Star
    node.physicsBody?.collisionBitMask = 0

 |

This is similar to your setup for the player node. You assign the star's category and clear its `collisionBitMask` so it won't collide with anything. In this case, though, you _don't_ set its `contactTestBitMask`, which means Sprite Kit won't notify you when something touches the star. You already instructed Sprite Kit to send notifications when the player node touches the star, which is the only contact with the star that you care about, so there's no need to send notifications from the star's side.

Sprite Kit sends notifications for the node contacts you've registered by calling `didBeginContact` on its `SKPhysicsContactDelegate`. Set the scene itself as the delegate for the physics world by adding the `SKPhysicsContactDelegate` protocol to the `GameScene` class definition. It should now look like this:

| ----- |
|

    class GameScene: SKScene, SKPhysicsContactDelegate {
      // Layered Nodes
      ...

 |

Now register the scene to receive contact notifications by adding the following line to `init(size:)`, just after the line that sets the gravity:

| ----- |
|

    // Set contact delegate
    physicsWorld.contactDelegate = self

 |

Finally, add the following method to _GameScene.swift_ to handle collision events:

| ----- |
|

    func didBeginContact(contact: SKPhysicsContact) {
      // 1
      var updateHUD = false
    &nbsp;
      // 2
      let whichNode = (contact.bodyA.node != player) ? contact.bodyA.node : contact.bodyB.node
      let other = whichNode as GameObjectNode
    &nbsp;
      // 3
      updateHUD = other.collisionWithPlayer(player)
    &nbsp;
      // Update the HUD if necessary
      if updateHUD {
        // 4 TODO: Update HUD in Part 2
      }
    }

 |

Let's take a closer look at this code:

1. You initialize the `updateHUD` flag, which you'll use at the end of the method to determine whether or not to update the HUD for collisions that result in points.
2. `SKPhysicsContact` does not guarantee which physics body will be in `bodyA` and `bodyB`. You know that all collisions in this game will be between the player node and a `GameObjectNode`, so this line figures out which one is _not_ the player node.
3. Once you've identified which object is _not_ the player, you call the `GameObjectNode`'s `collisionWithPlayer:` method.
4. This is where you will update the HUD, if required. You'll implement the HUD in Part Two, so there's nothing but a comment here for now.

Build and run the game. Tap to start. Upon collision, the star provides the player sprite with a sizable boost and then removes itself from the scene. Good work!

![12-FirstStarCollision][25]

That was a lot of heavy lifting there! Take a well-deserved break before moving on. In the next section, you're going to add a new star type—the _uber star_—as well as a cool sound effect.

_Note:_ If you want to know more about detecting contacts and collisions in Sprite Kit feel free to check out "[iOS Games by Tutorials][26]", which contains 3 solid chapters on Sprite Kit physics.

## Multiple Star Types

Uber Jump will have two different kinds of stars: those worth a single point and special stars worth five points. Each star type will have its own graphic. To identify the star type, you'll have an enumeration in the `StarNode` class.

At the top of _GameObjectNode.swift_, add the following enumeration:

| ----- |
|

    enum StarType: Int {
      case Normal = 0
      case Special
    }

 |

To store the star type, add the following property to the top of the `StarNode` class:

You now need to specify a star type when creating a star, so in _GameScene.swift_, add `starType` as a parameter to the `createStarAtPosition:` method signature so that it looks like this:

| ----- |
|

    func createStarAtPosition(position: CGPoint, ofType type: StarType) -&gt; StarNode {

 |

Inside `createStarAtPosition(position: ofType:)`, replace the three lines of code that create and add the `SKSpriteNode` with the following:

| ----- |
|

    node.starType = type
    var sprite: SKSpriteNode!
    if type == .Special {
      sprite = SKSpriteNode(imageNamed: "StarSpecial")
    } else {
      sprite = SKSpriteNode(imageNamed: "Star")
    }
    node.addChild(sprite)

 |

First, you set the star type. Then, when you create the sprite, you check to see which star type it is so you can get the correct graphic.

All that remains is to specify a star type when you create the star. In `init(size:)` in _GameScene.swift_, find the line where you call `createStarAtPosition:` and replace it with this:

| ----- |
|

    let star = createStarAtPosition(CGPoint(x: 160, y: 220), ofType: .Special)

 |

You specify that you would like this star to be a `StarType.Special` type.

Build and run. Your star is now pink! Later, you'll add a scoring system and the differences in star types will become more apparent.

![13-FirstStarSpecial][27]

## Ping! Adding a Sound Effect

It would be nice if, when the player collides with the star, they got an audible cue. Download the star sound effect [here][28] and drag it into your Xcode project. Make sure that _"Destination: Copy items if needed"_ is checked and that your _UberJump_ target is selected.

In Sprite Kit, you use `SKAction` to play sounds. Open _GameObjectNode.swift_. At the top of `StarNode`, add the following class property:

| ----- |
|

    let starSound = SKAction.playSoundFileNamed("StarPing.wav", waitForCompletion: false)

 |

Now all that remains is to run the sound action when the player node collides with the star.

Within `collisionWithPlayer()` in `StarNode`, replace `self.removeFromParent()` with:

| ----- |
|

    // Play sound
    runAction(starSound, completion: {
      // Remove this Star
      self.removeFromParent()
    })

 |

This runs the `SKAction`, plays the sound file, and removes the star when the action is complete.

Build and run. As the player sprite collides with the star, you'll hear a delightful, twinkly ping. :]

## Game Objects: Platforms

Your final task in this first part of the series is to add a platform to the scene. You'll represent platforms with a new subclass of `GameObjectNode` so that you get all the associated goodness from that class. As with the stars, you'll have two types of platforms: one that is indestructible and one that disappears as soon as the player sprite has jumped off of it.

In _GameObjectNode.swift_, add the following enumeration above the `GameObjectNode` class definition to define the two platform types:

| ----- |
|

    enum PlatformType: Int {
      case Normal = 0
      case Break
    }

 |

Next you'll create the `PlatformNode` class. Add the following code to _GameObjectNode.swift_:

| ----- |
|

    class PlatformNode: GameObjectNode {
      var platformType: PlatformType!
    &nbsp;
      override func collisionWithPlayer(player: SKNode) -&gt; Bool {
        // 1
        // Only bounce the player if he's falling
        if player.physicsBody?.velocity.dy &lt; 0 {
          // 2
          player.physicsBody?.velocity = CGVector(dx: player.physicsBody!.velocity.dx, dy: 250.0)
    &nbsp;
          // 3
          // Remove if it is a Break type platform
          if platformType == .Break {
            self.removeFromParent()
          }
        }
    &nbsp;
        // 4
        // No stars for platforms
        return false
      }
    }

 |

Let's take a closer look at this code:

1. As is common in this type of game, the player node should only bounce if it hits a platform while falling, which is indicated by a negative `dy` value in its `velocity`. This check also ensures the player node doesn't collide with platforms while moving up the screen.
2. You give the player node a vertical boost to make it bounce off the platform. You accomplish this the same way you did for the star, but with a less powerful boost. You want stars to be important, right?
3. If this is a `platformType.Break` platform, then you remove the platform from the scene.
4. Finally, the Uber Jumper doesn't get points from bouncing on or off platforms so there is no need to refresh the HUD.

To add a platform to your scene, open _GameScene.swift_ and add the following method:

| ----- |
|

    func createPlatformAtPosition(position: CGPoint, ofType type: PlatformType) -&gt; PlatformNode {
      // 1
      let node = PlatformNode()
      let thePosition = CGPoint(x: position.x * scaleFactor, y: position.y)
      node.position = thePosition
      node.name = "NODE_PLATFORM"
      node.platformType = type
    &nbsp;
      // 2
      var sprite: SKSpriteNode!
      if type == .Break {
        sprite = SKSpriteNode(imageNamed: "PlatformBreak")
      } else {
        sprite = SKSpriteNode(imageNamed: "Platform")
      }
      node.addChild(sprite)
    &nbsp;
      // 3
      node.physicsBody = SKPhysicsBody(rectangleOfSize: sprite.size)
      node.physicsBody?.dynamic = false
      node.physicsBody?.categoryBitMask = CollisionCategoryBitmask.Platform
      node.physicsBody?.collisionBitMask = 0
    &nbsp;
      return node
    }

 |

This method is very similar to `createStarAtPosition(_:ofType:)`, but note these main points:

1. You instantiate the `PlatformNode` and set its position, name and type.
2. You choose the correct graphic for the `SKSpriteNode` based on the platform type.
3. You set up the platform's physics, including its collision category.

Now to add the platform, insert the following code in `init(size:)`, just before the line that creates the star:

| ----- |
|

    // Add a platform
    let platform = createPlatformAtPosition(CGPoint(x: 160, y: 320), ofType: .Normal)
    foregroundNode.addChild(platform)

 |

Build and run the game. Tap to start and watch as the player sprite gets a boost from the star and then bounces on the platform!

![15-FirstPlatform][29]

## Where to Go From Here?

Well done! As you've seen, creating a game like _Mega Jump_ is not that hard at all. You've got all the basics of Uber Jump in place and are on your way to building a great game.

You can [download the Xcode project][30] with everything you've done so far in this tutorial series.

In [Part Two][31] of the series, you'll implement the accelerometer for movement on the x-axis. You'll also load an entire level from a property list and add a scoring system. That's a lot to look forward to, right? ;]

Please post a comment below if you have any questions, thoughts or suggestions for future tutorials!

[1]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/01/FeatureImage.png
[2]: http://www.raywenderlich.com/63229/make-game-like-mega-jump-spritekit-part-12
[3]: http://www.raywenderlich.com/u/tobystephens
[4]: /?p=84434 "Sprite Kit Tutorial for Beginners"
[5]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/26-HUDStars.png
[6]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/11/NewProject-700x413.png
[7]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/11/NewProjectName-700x413.png
[8]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/11/DeleteGameScene.png
[9]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/11/FirstRun-303x500.png
[10]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/11/NewProjectPortrait-700x426.png
[11]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/UberJumpGraphics.zip "UberJump Graphics Resources"
[12]: http://cdn5.raywenderlich.com/wp-content/uploads/2014/11/uj_asset_files.png
[13]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/03/uj_bg_thumb.png
[14]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/03/Player@2x.png
[15]: /?p=45152 "Sprite Kit Tutorial: Animations and Texture Atlases"
[16]: http://lostgarden.com "lostgarden.com"
[17]: http://www.lostgarden.com/2007/03/lost-garden-license.html
[18]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/05-BackgroundNode-287x500.png
[19]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/06-ForegroundNode-286x500.png
[20]:
[21]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/07-TapToStart-283x500.png
[22]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/11/NewClassGameObjectNode-700x413.png
[23]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/10-ForcesAndBounce-700x175.png
[24]: http://cdn4.raywenderlich.com/wp-content/uploads/2014/01/11-FirstStar-283x500.jpeg
[25]: http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/12-FirstStarCollision-283x500.jpeg
[26]: /?page_id=48022
[27]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/13-FirstStarSpecial-283x500.jpeg
[28]: http://cdn3.raywenderlich.com/wp-content/uploads/2014/01/StarPing.wav
[29]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/15-FirstPlatform-283x500.png
[30]: http://cdn1.raywenderlich.com/wp-content/uploads/2014/11/UberJump-Part-1.zip
[31]: /?p=87232 "How To Make A Game Like Mega Jump With Sprite Kit and Swift: Part 2/2"
 