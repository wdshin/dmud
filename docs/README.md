
# dmud - 'Proof of Concepts'

* What is the problem that you’re solving?
  * [Why I start building lpmud on chain](https://medium.com/@ElliotShin/why-i-start-building-lpmud-on-chain-46965b24b17e)
  * [Requirements on sending events in on-chain rpg](https://medium.com/@ElliotShin/requirements-on-sending-events-in-on-chain-rpg-ae504a9d1e13)
  * We want to help gamers to easily make interactive online games 100% on-chain. 
  * This is difficult because there is no easy way to 1) stream live information on-chain and 2) incentivize individual game creators . 
  * Streaming of information is extremely difficult using current blockchain networks and this limits the interactive nature of online games. 
  * This is one of the main reasons we have not seen MMOGs built 100% on-chain. The polling model can be adopted to a certain extent but this is not suitable or sustainable for a real time interactive game environment with mass players.
  * Also, there is no easy way to effectively incentivize micro studios and individual game creators for their creativity using a conventional web3 value share model. 
  * The conventional model is designed to incentivize players for their play but no real progress has been made in exploiting the creative inertia of gamers and amateur creators.

* Expand on the product that you're building
  * dMud V1 is a text based on-chain open world MMORPG. 
  * The users can use dMud to create their own text based MMORPG games or simply participate as players to play the games created by other users. 
  * The creators, also known as the wizards / immortals, are the architects of a game world. They have the power to design a whole game environment and rules using dMud Creator Studio, which is a text based interface for creators to create games using a natural language. 
  * The players are known as mortals, they can interact with other players to build guilds to complete quests and also fight other players to loot their game items (NFTs / FTs). 
  * At the guild, the players share their knowledge and game experience. The guild masters can make suggestions to wizards / immortals to improve the gameplay experience. 
  * The user generated games will be curated and rewarded according to active users, transactions and likes from players.
  * Technically, the dMud consists of two parts. 
    * The dMud VM which is a mainnet that will run the dMud games created by both dMud core team and users. The dMud Creator Studio is a place where users have tools and libraries to access all information required in building their own game. 
    * dMud V2 plans to leverage generative AI to add more visuals to games created on dMud.


# minimal implementation goals

1. interact with contracts
2. minting some general objects
3. tick 10k entities to hit and damage
4. sending events from on-chain to off-chain and listening events 
5. very simple game objects
6. very simple user object
7. very simple player object
8. very simple game world
9. very simple social world
10. entity component system in sui move

## system design

### references

* [unreal engine 5.1](https://docs.unrealengine.com/5.1/en-US/)
  * https://docs.unrealengine.com/5.1/en-US/unreal-engine-terminology/
    * The Blueprint Visual Scripting system is a complete gameplay scripting system that uses a node-based interface to create gameplay elements from within Unreal Editor. As with many common scripting languages, it is used to define object-oriented (OO) classes or objects in the engine. As you use Unreal Engine, you'll often find that objects defined using Blueprint are colloquially referred to as "Blueprints."
    * Object : Objects are the most basic class in Unreal Engine - in other words, they act like building blocks and contain a lot of the essential functionality for your Assets. Almost everything in Unreal Engine inherits (or gets some functionality) from an Object.
    * Class : A Class defines the behaviors and properties of a particular Actor or Object in Unreal Engine. Classes are hierarchical, meaning a Class inherits information from its parent Class (that is, the Class it was derived or "sub-classed" from) and passes that information to its children. Classes can be created in C++ code or in Blueprints.
    * Actor : An Actor is any object that can be placed into a level, such as a Camera, static mesh, or player start location. Actors support 3D transformations such as translation, rotation, and scaling. They can be created (spawned) and destroyed through gameplay code (C++ or Blueprints).
    * Casting : Casting is an action that takes an Actor of a specific class and tries to treat it as if it were of a different class. Casting can succeed or fail. If casting succeeds, you can then access class-specific functionality on the Actor you cast to.
    * A Component is a piece of functionality that can be added to an Actor.
      * When you add a Component to an Actor, the Actor can use the functionality that the Component provides. For  example:
        * A Spot Light Component will make your Actor emit light like a spot light.
        * A Rotating Movement Component will make your Actor spin around.
        * An Audio Component will give your Actor the ability to play sounds.
      * Components must be attached to an Actor and can't exist by themselves.
    * Pawn : Pawns are a subclass of Actor and serve as an in-game avatar or persona (for example, the characters in a game). Pawns can be controlled by a player or by the game's AI, as non-player characters (NPCs).
      * When a Pawn is controlled by a human or AI player, it is considered to be Possessed. Conversely, when a Pawn is not controlled by a human or AI player, it is considered to be Unpossessed.
    * Character : A Character is a subclass of a Pawn Actor that is intended to be used as a player character. 
      * The Character subclass includes a collision setup, input bindings for bipedal movement, and additional code for player-controlled movement.
    * Player Controller : A Player Controller takes player input and translates it into interactions in the game. Every game has at least one Player Controller in it. A Player Controller often possesses a Pawn or Character as a representation of the player in a game.
    * AI Controller : Just as the Player Controller possesses a Pawn as a representation of the player in a game, an AI Controller possesses a Pawn to represent a non-player character (NPC) in a game. By default, Pawns and Characters will end up with a base AI Controller unless they are specifically possessed by a Player Controller or told not to create an AI Controller for themselves.
    * Player State : A Player State is the state of a participant in the game, such as a human player or a bot that is simulating a player. Non-player AI that exists as part of the game world doesn't have a Player State.
      * Some examples of player information that the Player State can contain include:
        * Name
        * Current level
        * Health
        * Score
        * Whether they are currently carrying the flag in a Capture the Flag game.
      * For multiplayer games, Player States for all players exist on all machines and can replicate data from the server to the client to keep things in sync. This is different from a Player Controller, which will only exist on the machine of the player it represents.
    * Game Mode : The Game Mode sets the rules of the game that is being played. These rules can include:
        * How players join the game.
        * Whether or not a game can be paused.
        * Any game-specific behavior such as win conditions.
      * You can set the default Game Mode in the Project Settings and override it for different Levels. Regardless of how you choose to implement it, you can only have one Game Mode for each Level.
      * In a multiplayer game, the Game Mode only exists on the server and the rules are replicated (sent) to each of the connected clients.
    * Game State : A Game State is a container that holds information you want replicated to every client in a game. In simpler terms, it is 'The State of the Game' for everyone connected.
      * Some examples of what the Game State can contain include:
        * Information about the game score.
        * Whether a match has started or not.
        * How many AI characters to spawn based on the number of players in the world.
      * For multiplayer games, there is one local instance of the Game State on each player's machine. Local Game State instances get their updated information from the server's instance of the Game State.
    * Brush : A Brush is an Actor that describes a 3D shape, such as a cube or a sphere. You can place brushes in a level to define level geometry (these are known as Binary Space Partition or BSP brushes). This is useful if you want to quickly block out a level, for example.
    * Volume : Volumes are bounded 3D spaces that have different uses based on the effects attached to them. For example:
      * Blocking Volumes are invisible and used to prevent Actors from passing through them.
      * Pain Causing Volumes cause damage over time to any Actor that overlaps them.
      * Trigger Volumes are programmed to cause events when an Actor enters or exits them.
    * Level : A Level is a gameplay area that you define. Levels contain everything a player can see and interact with, such as geometry, Pawns, and Actors.
      * Unreal Engine saves each level as a separate .umap file, which is why you will sometimes see them referred to as Maps.
    * World : A World is a container for all the Levels that make up your game. It handles the streaming of Levels and the spawning (creation) of dynamic Actors.


* [unity engine](https://docs.unity.com/)


### goal

### terminology in dmud

* User
* Player
* Entity(Object) 
* Class 
* Actor 
* Casting
  * Cast 
* Component
* Pawn
* Character
* Player Controller
* AI Controller
* Player State
* Game Mode
* Game State
* Brush
* Volume
* Level
* World
  * Zone
  * Area
  * Cell
* Universe
  
### Responsiblities and Relationships among Entities

* User
  * ! is directly connected to a wallet.
  * ! should be created(minted) right after connecting to a Universe.
  * ! has informations about the outside.
    * has primary PFP
      * has other NFTs
    * has membership NFTs
    * has real name
    * has email
    * has blockchain wallet address
    * has device ids
  * ! can connect to a World.
  * ! can create a Player after connecting to a World.
  * ! has(owns) several Players(NFTs)
    * A Player belongs to one World
    * A Player can have several Commanders
  * ? can connect to Chatting Rooms(Channels)
* Player
  * is an mortal.
  * is connected to a User.
  * ! has Items(NFTs) of the World.
  * ! has Coins of the World.
  * ! has a PlayerState of the World.
  * ? has informations in the World.
  * ? has Achievements of the World.
  * ? can do Social Actions on other Players.
  * ? can have Social Relationships between other Players.

* Immortal (aka Wizard)
  * can create and edit the World.

* World
  * ? has Zones.
    * a Zone has Levels.
    * has an Entry(Starting) Level
  * ? has an Entry(Starting) Zone.
  * ! has a Game Mode.
  * ! has a Game State.
  * has at least one zone, one level , one game mode and one game state.

* ? Level 
  * has a parent Zone.
  * should be edited online by immortals.
  * ? should be imported from Level Editor
    * ! from dMUD Level Editor
    * from Unreal Engine , Unity Engine , 

  * Static Actors 
  * Dynamic Actor Spawner
  * Ticker
  * loads by loading static actors, dynamic actors 
  * starts by Ticker.


### input entities ( contracts )

* commander on chain
  * text commander on chain
    * The client sends an unverified command to the text commander.
    * This is interpreted according to the context and sent to the commander entity, and the commander entity controls the player.
  * packet commander
  

* player controller
  * entity that receives orders from the commander
  * An entity that issues commands to the player
  * A player can belong to any space.
    * If it does not belong to a space, it belongs to void.

### game entities

#### live actors

* player

#### static actors

#### game space entities

* world 
  * composed of some zones
* zone 
  * one processing unit that deals with live objects in ecs(Entity Component System) . 
  * composed of some areas
* area , room( in text mud )
  * player belongs to one area
  * synchronizing unit 
* cell
  * 2D = quad , 3D = voxel


### output entities

* events for player

