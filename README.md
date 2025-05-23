# ColonialCartographer

I'd like to turn my design document into a working game. I want to make sure we use vanilla JS. Can you help? I want to start with the file structure, and making sure we get the math, rendering, memory, and MVC right.

Introduction
One thing you notice playing historical strategy games is the perfect information the player has. There are historical issues, especially considering who or what the player is playing as (a historical figure, a nation, a Civilization, the spirit of history, etc), but also gameplay limitations from information overload, quickly devolving into a spreadsheet simulator.
One solution to this issue is to have information be a key feature of the gameplay. Instead of exploring the map and earning perfect clarity about it, it is revealed in pieces, sometimes inaccurately. The game is no longer just decision-making about actions, but about what you know and what you believe to be true about the game in progress.
The first core feature, the map, came to me from several inspirations. The first is not actually a strategy game at all, but from first-person 3D horror games (The Voidness, LiDAR Exploration Program, someone’s here) where you use a LiDAR scanner to reveal the level geometry with dots. Although there are no monsters lurking in the shadows of a turn-based strategy game, the underlying idea is the same: you receive information through intentional actions and the feedback they give.
The obvious inspiration for the early game design is Sid Meier’s Civilization, especially Civilization 5. The look and feel of the game, a decade after its release, remains timeless. The UI, design, and mechanics created a simple, seamless, unending experience, aided by the game’s immense modding scene.
Other inspirations are more scattered. One of the core gameplay elements, placing the map fragments on the board, came from imaging an empty Settlers of Catan board, and placing that game’s tiles on top of another. Slightly later, I remembered hearing about a version of Catan that involved placing down the starting towns and roads before revealing any tiles, then rolling dice to determine the terrain.
The historical background for the game, 1492-1898, comes from my various readings of historical sources from Americanists. “The Atlantic World: A History, 1400 - 1888” (Egerton, Games, Landers, Lane, Wright) is an essential background read. “Changes in the Land” (Cronon), “Native People of Southern New England, 1500–1650” (Bragdon), “This Land Is Their Land” (Silverman), and “The Unredeemed Captive” (Demos) are similarly essential for the early history of New England. “A Voyage Long and Strange” (Horwitz) filled in much of my knowledge of the rest of America. I’ve also studied numerous articles about the population, depopulation, and colonization of indigenous American nations. “American Nations” (Woodward) has expanded my understanding of America’s cultural boundaries. Finally, Edmund Morris’ biographical series of Theodore Roosevelt rounds out my knowledge of the closing of the western frontier in 1898, hence the game’s ending.
The working title for the game is “Colonial Cartographer.”
Game Mechanics
Colonial Cartographer is a historical 4X, turn-based strategy game, with many mechanics very similar to Sid Meier’s Civilization 5.
Wealth and Turns
We start with a very simplified resource management system, with one abstract Wealth  amount the player has (yields)
The Player can earn Wealth each turn and expend Wealth on actions, like Explorations or combining fragments
Tiles
The gameworld is a grid of hexagons, again like Civilization
Very important to get hexagon geometry and tiling correct!
Flat-topped hexagons
Use hexagon math everywhere (q, r, s), only convert to square coordinates with user interaction / displaying grid on the screen
Tiles can have a terrain, features, and resources
Terrains are arbitrary and have associated Movement Costs, Visibility, Yields, and Display Colors
Terrain types utilize inheritance, so Land and Water terrains share similar qualities; most obviously, for unit movement
Right now, the terrain types and their colors (assume movement costs = 1, visibility = 0, yields = 0 for now) are:
Plains, Brown-Green
Grassland, Deep Green
Desert, Sand
Tundra, Gray
Snow, White
Coast, Light Blue
Ocean, Darker Blue
Tiles can have an arbitrary number of features, but only one of each upper-level feature class (Vegetation, Elevation)
The three vegetation features are:
No Vegetation (By default, this is the absence of a vegetation feature)
Forest
Jungle
The three elevation features are:
Flat (By default, this is the absence of an elevation feature)
Hills
Mountain
Tiles can have an arbitrary number of resources, and each resource will have a specific quantity per tile. Resources are not yet fully implemented
True Map System
The True Map is the world of the game, the same as the map in games like Civilization
It is an arbitrary rectangle grid of hexagon tiles
It can be randomly generated or customized
The True Map is not directly revealed to the player, but exists in the data model
One key feature is that the player must construct their own version of the map
Game Board & Anchor Points
The player starts with an empty game board, which appears as a hexagon grid with no tiles revealed
I think the “fog of war” should be a nice beige, like the parchment of an old map
The game starts on turn one with one tile at the origin (0,0) as their first anchor points
We will give the player a cartesian grid coordinate, but will not use it internally
Anchor Points are tiles on the map that the player:
Knows their absolute position, and their position in relation to each other
Knows the terrain of the tile (and all associated resources/features)
The player starts with one Anchor Point, and can earn more through major game milestones
For now, this is unimplemented, but eventually, players will be able to create more anchor points
The player’s Colony / First City will be on their first Anchor Point
These Cities will function similarly to those in Civilization, but for now, new city creation is not implemented
Fragments
The player builds the map from fragments they receive, which correspond to the True Map
Fragments are an arbitrary group of tiles that may or may not be 100% accurate to the True Map
This means we should efficiently handle fragment objects, and also drawing the border around an arbitrary group of connected hexagons, including those with interior gaps
Because the player needs to use their judgement on the accuracy of fragments, they may include less specific terrain detail than exists in the True Map, so terrain types might be wrong, or less specific (just Land or Water instead of the subclasses), and feature/resources might also be different
I think of fragments as just miniature versions of the full map, but obviously are not perfect subsections
The player can drag fragments from the sidebar onto the game board
Fragment moving, both between the sidebar and the game board and on the game board itself should feel very snappy
Fragments have borders that can be toggled on and off
Need to make sure mitered corners are correct and efficient
Fragment Ordering
There are no restrictions on if or where the players can place a fragment, it is entirely up to their judgement
To manage conflicts between overlapping fragments, the player choose the relative order of all fragments, as if they were choosing how to stack sheets of paper (really more like cutouts, since the arbitrary fragments aren’t perfect rectangles)
The order of fragments is managed at the fragment level, not tile-by-tile
This means we need to efficiently handle how to display what tiles, and what tiles to display when we move a fragment on the game board (the second-highest tiles)
Fragment order is managed universally in the sidebar
Fragments are displayed in a list, and can be moved up or down in the order with arrows
This order is also the display order
Fragments that are completely obscured (no tiles visible) should have an indication
In addition to toggling fragment borders, the player can also toggle individual fragments visible/not visible
The player can toggle what conflicts they want to see, which show as red dots on the game board
Any overlap between fragments
Overlap where tiles in fragments don’t have the same terrain type
Currently, you can’t really change the relative order without changing the universal order, because the stack of fragments is a directed path graph; even non-overlapping fragments must have a relative order
The player can choose between different modes of interacting with fragments:
Dragging fragments on, off, or around the game board (left click + CTRL)
Bringing a fragment down in order (left click + Z)
Bringing a fragment up in order (left click + X)
Otherwise, panning the map (left click) does not move any fragments
Clicking on a fragment in the side bar can open a mini-map showing only that fragment
The collection of fragments and their order, essentially what is displayed to the player and what they see, is the Working Map (which could be thought of as its own fragment, except it contains both Landmark Token Slots and the Landmark Tokens)
Landmarks
Landmarks help the player connect fragments, but have few restrictions on how they’re used, leaving room for player judgement
Landmark Tokens are implemented like pins that can be dragged onto the map and appear over a tile; a Landmark only has one Landmark Token
Because fragments might have conflicting sources landmarks, they have tiles that show a Landmark Token Slot, that look like the Landmark Token but slightly grayed out
Some Landmarks might turn out to be entirely fictitious, in which case, it would be advisable for the player not to place down that Landmark Token
Landmarks are not inherent parts of the True Map, but are dynamically generated
Anchor Points are considered Landmarks, but have immovable tokens. Fragments that are generated in relation to Anchor Points will contain at least the Anchor Point Landmark Token slot, but the player should be smart enough to place these correctly (but again, we won’t restrict them!)
Landmarks relate to the terrain they are on
These are mostly arbitrary based on the terrain; while they are based on a position in the world, they don’t actually have any descriptive information stored, just randomly generated names for now (“White Forest” “Lake Winfred”)
Explorations & Fragment Generation
Fragments are generated by Explorations, which is how the player gets a mostly accurate view of the world while introducing non-random errors
Explorations are simulations of a unit traversing the terrain of the True Map and recording the details
An Exploration has an explorer unit, an amount of stored Wealth, and a Logbook
The unit can see a limited radius around iit each turn with 100% accuracy
The unit expends Wealth moving each turn
A unit has a limited domain, either Land or Water; for now, we will start with a unit that can only move on land
The most important unit behavior is how it pathfinds, because it will have both a model of the world, but obviously is moving through the True Map
How to handle pathfinding through fog of war?
Need to make sure pathfinding on hex grid is efficient
 The Logbook is made up of Entries about the terrain
An Entry can be a single tile, a group of contiguous tiles, or a navigable path between two tiles
Each Entry takes up a certain amount of space in the Logbook, which has limited total space
Entries have priority, which is a cardinal value out of 10 (multiple entries can have the same priority level)
If the Logbook reaches capacity, the Exploration will need to manage the memory of the Entries
The main way the Exploration manages the Logbook is through simplification. Because single tiles take up a lot of memory and have low priority, they will often be combined into a new or existing group of tiles
Simplification introduces non-random “errors” to fragments that the player will only determine through fragment generation
The position and relative position of Anchor Points, Landmarks, and the path between the unit’s current position and the nearest Anchor Point, are considered highest priority, followed by regions of tiles, then single tiles
The Exploration starts with a copy of the Working Map in the Logbook, and will only create Entries in the Logbook that contradict with the Working Map (or, obviously, are not on it, in the fog of war unexplored tiles)
The unit’s sight is also considered a free Logbook entry
The Exploration has a temporary map, which is a combination of the Working Map, the unit’s sight, and Logbook entries, which it uses to navigate (pathfind)
The Exploration calculates how much Wealth it will take to return to an Anchor Point based on its temporary, and will try to always return to an Anchor Point before it runs out of Wealth
The Logbook (and the completed temporary map) becomes a fragment when the Exploration finishes, when it returns to an Anchor Point
The fragment does not contain much of the original Working Map, but should only have those tiles that either a) contradict the working map or b) contain the path between newly discovered tiles and the Anchor Point it finishes at
Landmarks are generated dynamically. Any tile is a candidate to have an Exploration designate it a Landmark, based on certain conditions, like:
The distance to another Landmark (probably not within a few tiles)
Its position relative to different terrain types (A lone mountain among plains)
Its position relative to the same terrain type (The westernmost portion of a lake)
If the Exploration has not designated any Landmark, it will designate the tile where it decides to return as a Landmark if it is more than x tiles away from another Landmark (Maybe 5 tiles?)
When an Exploration designates a new Landmark, it will mark it down in the Logbook (high priority), and if successfully completed, will reward the player with a new token
An Exploration doesn’t necessarily have to designate new Landmarks, especially if the player doesn’t send it into unexplored territory
Yes, it is entirely possible that an Exploration does not generate any new information, that is how much agency the player has!
The player will select a tile for a unit to try to navigate to. If the unit reaches this destination tile before it estimates that it needs to begin its return (based on the amount of stored Wealth), it will begin its return to an Anchor Point then
Other Game Notes
Map Fundamentals
* True map exists behind the scenes (land/water, terrain biomes, landmarks)
* Players only see what they've discovered through fragments
Fragment System
* Fragment Types:
   * Voyage/scouting fragments (paths across the map with varying accuracy)
   * Observation fragments (reports about an area from a specific vantage point)
   * Distant fragments (vague reports of far-off locations)
* Fragment Generation:
   * Based on player actions (exploration, trade routes, military campaigns)
   * Generated relative to landmarks or anchor points
   * Contains both accurate and inaccurate information
   * Early game: High frequency, lower accuracy, variable sizes
   * Late game: Less frequent, higher accuracy, variable sizes
Landmark System
* True Landmarks: Fixed features on the actual map (mountains, lakes, settlements, anchor points)
* Reported Landmarks: What appears in fragments (may be accurate, misplaced, or fictional), these appear as semi-transparent icons on a hex
* Landmark Implementation:
   * Landmarks exist as separate "token" elements players place on the map
   * Players determine landmark placement based on evidence from fragments
   * Landmark tokens can be moved as new evidence comes in
   * Some landmarks may eventually be determined fictional and removed
   * The landmark token is fully opaque, so it can be placed on fragments
Fragment Management
* Placement Mechanics:
   * Players place fragments on hex grid
   * Players decide stacking order (which takes precedence)
   * The stack order determines which information is considered "canon"
* Fragment Consolidation:
   * Players can combine fragments that share landmarks
   * Combined fragments replace individual fragments (one-way process)
   * Player determines stacking order when combining
   * Creates cleaner map and represents growing knowledge
Anchor Points System
* Function: Represents areas of absolute certainty on the map, unmovable hexes that are accurate to the true map
* Acquisition: Through major developments:
   * Establishing new cities
   * Building highways/infrastructure
   * Adopting new survey technology
   * Conducting official government surveys
* Progression:
   * Start with single anchor point (initial settlement, Colony)
   * Gradually acquire more through development
   * Initially costly, becomes easier with technological advancement
Knowledge Evolution
* Early Game (Exploration Phase):
   * High uncertainty, many conflicting fragments
   * Few anchor points, mostly exploration-focused
   * Maps primarily based on natural landmarks
* Mid Game (Consolidation Phase):
   * Growing certainty in core territories
   * More anchor points at major settlements
   * Fragment consolidation becomes important
* Late Game (Governance Phase):
   * Stable, reliable maps in developed territories
   * Network of anchor points, mostly at infrastructure
   * New fragments primarily at frontiers or disputed areas