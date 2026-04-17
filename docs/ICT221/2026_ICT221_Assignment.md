# ICT221 Java Programming Assignment

This is Task 2 of ICT221, which is worth 40% of the course. Your solution must be clearly separated into two parts, in separate Java packages:

- **Part 1:** (in package **starsalvage.engine**): your Game Engine classes with Unit Tests.
- **Part 2:** (in package **starsalvage.gui**): a JavaFX GUI that allows users to play your game.

After you create the project in GitHub Classroom, two packages will be created automatically. Please keep that code structure unchanged.

## StarSalvage Game Description

Welcome to **StarSalvage**, a deep-space salvage adventure! You pilot a mining vessel through two perilous sectors of deep space, each represented as a 10x10 grid filled with drifting scrap, rogue drones, automated turrets, space mines and gravity wells. Your goal: achieve the highest possible score by collecting scrap, taking out hostile machinery and escaping each sector through its wormhole. All within a limited number of steps and before your hull is breached!

Unlike simple dungeon crawlers, the hazards in StarSalvage are **active**: drones patrol and pursue you, turrets cycle through charging and firing states, and mines drift across the map each turn. You'll need to plan ahead and, when you make a mistake, use the ship's onboard flight recorder to rewind time by up to five moves.

### Sector Layout and Items

Each 10x10 sector map contains:

| Symbol | Item          | Count | Location                                                | Behaviour                                                                                                  |
|--------|---------------|-------|---------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| E      | Entry         | 1     | Sector 1: bottom-left cell. Sector 2: same as the Wormhole from Sector 1. | Entry point into the sector.                                                                              |
| W      | Wormhole      | 1     | Random                                                  | Advances to the next sector or exits the game.                                                             |
| S      | Ship (Player) | 1     | Starts at Entry of Sector 1.                            | Moves in 4 directions: left, right, up, down.                                                              |
| #      | Asteroid      | \~8   | Random (also around sector boundaries)                  | Blocks movement. Can be destroyed by using a Drill Charge from the inventory.                              |
| X      | Gravity Well  | 5     | Random                                                  | Reduces ship's Hull when entered; remains active.                                                          |
| C      | Scrap Cargo   | 5     | Random                                                  | Increases player's score when collected; the cell becomes empty.                                           |
| D      | Scout Drone   | 2     | Random                                                  | Moving enemy with a three-stage behaviour (Patrol / Alert / Chase).                                        |
| T      | Sentry Turret | ***d*** | Random                                                | Stationary; cycles through Idle → Charging → Firing; fires along its row and column.                       |
| M      | Space Mine    | 2     | Random                                                  | Drifts one cell per turn in a fixed direction; reverses on collision.                                      |
| H      | Hull Patch    | 2     | Random                                                  | Restores the player's Hull; the cell becomes empty.                                                        |
| B      | Drill Charge  | 2     | Random                                                  | Picked up into the ship's inventory; can be deployed later to destroy one adjacent Asteroid.               |

### Game Settings and Gameplay

Starting the Game:

- Default maximum steps: 100
- Initial Ship Hull (HP equivalent): 10
- Initial Score: 0
- Users select/input game difficulty ***d*** (0--10, default 3) to adjust the challenge level.
- Users click "Run" to start and generate the Sector 1 map.
- All map items, except the Ship, should not overlap.

Advancing Sectors:

- Reach the Wormhole to move from Sector 1 to Sector 2, increasing difficulty ***d*** by 2 (max 10).
- Exit the game successfully by reaching the Wormhole on Sector 2.

Top Scores Feature:

- Record the top 5 plays (rank, score, date).
- Display the top 5 scores in the GUI, e.g., "#1 25 20/04/2026".

Game End Conditions:

- **Win:** Successfully exit the Sector 2 map (final score retained).
  - Celebrate a new top-5 score with a congratulatory message.
- **Lose:** Hull drops to 0 or steps reach the maximum (final score: -1).

Gameplay Mechanics:

- Move the ship in ***4 directions***: left, right, up, and down, inside the sector. Each action (move, wait, or drill) counts as one step.
- After each player action, **all active hazards take their turn** in a fixed order (see Active Hazards below).
- Interact with sector items:
  - **Scrap Cargo (C):** Collect (+2 score). The scrap is picked up, and the cell becomes empty.
  - **Hull Patch (H):** Restore 4 Hull (max Hull = 10). Regardless of current Hull, the Hull Patch is consumed immediately when entered, and the cell becomes empty.
  - **Gravity Well (X):** -2 Hull per encounter; the well remains active.
  - **Scout Drone (D):** -2 Hull, +2 score if you step onto its cell. The drone is destroyed and removed. If a drone moves onto your cell, the same effect applies.
  - **Sentry Turret (T):** Cycles through Idle --> Charging --> Firing each turn. On its Firing turn, it fires along 2 tiles of its row and column (-2 Hull if the Ship is in the line of fire). Stepping onto a Turret destroys it (+2 score); Hull cost depends on the Turret's current state (see Active Hazards below).
  - **Space Mine (M):** Stepping onto a Mine costs -4 Hull, +1 score, and the Mine is detonated (removed). If a Mine drifts onto the Ship's cell, the same effect applies.
  - **Drill Charge (B):** Picked up into the inventory (the cell becomes empty). See the Inventory section below for deployment rules.

### Active Hazards

After the player acts each turn (move, wait, or drill), all active hazards take their turns in a fixed order: first all Scout Drones, then all Sentry Turrets, then all Space Mines. Within each hazard type, the order is not specified and any consistent ordering is acceptable. Using a fixed turn order keeps the game deterministic, which is important for undo and for unit testing.

**Scout Drone (D)** maintains an **internal state** that determines its behaviour on each turn. It has three states: Patrol, Alert, and Chase. The drone evaluates its state **at the start of its turn**, then acts accordingly.

1. *Patrol* (default): The drone moves to a random walkable adjacent cell. After moving, if the Ship is now within scan range, the drone transitions to Alert (taking effect next turn).
2. *Alert*: The drone stays in place (does not move). If the Ship is within scan range, it then transitions to Chase (taking effect next turn). If the Ship has already moved out of scan range before the drone's turn, the drone transitions directly back to Patrol instead and takes a random move.
3. *Chase*: The drone moves one cell toward the Ship, choosing the direction (up, down, left, or right) that reduces the row-plus-column distance the most. After moving, if the Ship is now outside scan range, the drone transitions to Patrol (taking effect next turn). If a Chase drone tries to move toward the Ship but the closest cell is blocked (asteroid, another entity), the drone stays in place but remains in Chase. 

The scan range counts the row difference plus the column difference between the drone and the Ship: the Ship is in range if that sum is 3 or fewer. For example, a Ship that is 2 rows and 1 column away from the drone is in range (2+1=3); a Ship that is 2 rows and 2 columns away is out of range (2+2=4). 

**Sentry Turret (T)** maintains an **internal state** that determines its behaviour on each turn. The Turret is stationary and cycles through three states unconditionally. The cycle is deterministic and independent of the Ship's position. It evaluates its state **at the start of its turn**, then acts.

1. *Idle* (default): The Turret does nothing. Transitions to Charging (taking effect next turn). Stepping onto an Idle turret destroys it: +2 score, no Hull lost.
2. *Charging*: The Turret displays a visible warning (distinct icon in GUI, distinct symbol or annotation in CLI). It does not fire. Transitions to Firing (taking effect next turn). Stepping onto a Charging turret destroys it: +2 score, -2 Hull.
3. *Firing*: The Turret fires along its row and column, up to 2 tiles in each of the four cardinal directions. If the Ship occupies any of those tiles, the Ship loses 2 Hull. The Turret then transitions to Idle (taking effect next turn). Stepping onto a Firing turret destroys it: +2 score, -2 Hull.

The difficulty parameter ***d*** controls the number of turrets on the map.

When a Turret fires and the Ship is in the line of fire, the hit is resolved immediately during the Turret's turn. If there is an Asteroid between the Turret and the Ship along the row or column, the shot is blocked: it does not pass through Asteroids.

**Space Mine (M)** has no internal state machine but just a position and a drift direction (up, down, left, or right, assigned randomly at map generation). On every turn, the Mine attempts to move one cell in its drift direction. If the destination cell is walkable and empty, the Mine moves there. If the destination cell is blocked (asteroid, sector boundary, another hazard, or the Wormhole), the Mine reverses its drift direction and moves one cell in the new direction instead. If that cell is also blocked, the Mine stays in place (it is trapped between two obstacles). If the cell the Mine moves into contains the Ship (whether forward or after reversing), the Mine detonates: -4 Hull, +1 score, Mine removed.

### Inventory

The Ship has a simple inventory that tracks pickups not applied immediately:

- **Drill Charges (B):** stackable; no limit on carrying capacity for this assignment.
- Other pickups (Hull Patches, Scrap) apply immediately when entered, as described in Gameplay Mechanics.

In the GUI, a dedicated "Use Drill" button is active whenever the inventory holds at least one Drill Charge. After clicking "Use Drill", the player selects a direction (up/down/left/right) to target an adjacent Asteroid. In the CLI, the player enters `b` followed by a direction keyword.

If the selected direction does not contain an Asteroid, for example, the targeted cell is empty space, off the map edge, or occupied by any other item, the Drill Charge is still **consumed and wasted**, and a status message is displayed. Students should treat the drill as a physical device that fires when triggered regardless of what it hits.

Using a Drill Charge counts as one player action: it consumes one step from the step budget, all hazards take their turn afterwards, and the action is recorded by the flight recorder.

### Wait Action

The player may choose to *wait* instead of moving. A wait action advances the game by one turn: it consumes one step from the step budget and lets every hazard take its turn (drones advance their state, turrets advance their charge cycle, mines drift), but the Ship does not move. Waiting is a legitimate tactical option, for example, to let a Sentry Turret discharge its Firing turn while the Ship is safely off its row and column. Wait is exposed as `w` in the CLI and as a dedicated "Wait" button in the GUI.

### Undo

The ship's **flight recorder** lets the player reverse up to the last **5 player moves**. Undo must faithfully restore the Ship's position, Hull, score, inventory, and the state of every other entity on the map (drones' states and positions, turrets' charge phase, mines' positions and directions, picked-up items). Only the five most recent player moves are retained; older moves are discarded from the recorder. Redo is **not** required.

This feature is available in both the CLI (`z` for undo) and the GUI (a dedicated "Undo" button). A move counts as a "player move" for the purposes of the flight recorder whether it is a movement, a Drill Charge deployment, or a wait action.

### Interface and Controls

Both the Text Interface (CLI) and the Graphical Interface (GUI) are views onto the **same underlying game engine**. The classes in `starsalvage.engine` must not depend on a running JavaFX application.The engine should expose its state and accept commands through a clean Java API so that the CLI can drive a full game without launching JavaFX. The classes in `starsalvage.engine` should not directly read from `System.in` or print to `System.out` from within the core game-logic classes. The one exception is the `main()` method of GameEngine, which may use `System.in` and `System.out` to provide a text-based play loop, thinking of it as a lightweight test harness, not part of the engine's API. Adding the GUI must not break the CLI; both interfaces must continue to work at submission time and must drive exactly the same engine.

Keeping a CLI alongside the GUI is not a fallback but one of the most useful parts of this assignment. The CLI lets you step through a complete game in seconds without waiting for JavaFX to start, reproduce bugs deterministically from a short scripted sequence of commands, and write JUnit tests that drive the engine through a full play session. It is also the clearest evidence that your game logic is properly separated from the GUI layer.

- **Text Interface (CLI):** The game can be played in the text UI.
  - Control the Ship by typing keywords, e.g., `u` for up and `d` for down (and `l`, `r` for left and right).
  - Use `w` to wait in place for one turn.
  - Use `b <direction>` to deploy a Drill Charge (for example, `b u` drills the asteroid above the Ship).
  - Use `z` for undo (reverses the most recent player move, up to 5 moves deep).
  - Use the symbols in the item table to display items in the map.

- **Graphical Interface (GUI):**
  - Control Ship movement via **on-screen buttons**. No keyboard support is required.
  - Distinct icons for each item type, including a visibly different sprite for each Turret state (Idle / Charging / Firing).
  - Display and update steps, Hull, score, inventory, and current sector dynamically.
  - "Help" button for game instructions.
  - "Save" and "Load" buttons for game progress. Only a single save needs to be maintained.
  - "Wait" button to skip one turn in place.
  - "Undo" button wired to the flight-recorder feature.
  - A text area showing the game-play status for every action, such as:
    - You moved up one step.
    - You tried to move up one step but it is an asteroid.
    - You waited in place.
    - You collected a scrap cargo.
    - You drifted into a gravity well.
    - You destroyed a scout drone.
    - A sentry turret fires, but your ship is out of the line of fire.
    - A sentry turret fires and your ship loses 2 Hull.
    - You deployed a drill charge and destroyed an asteroid.
    - You deployed a drill charge but there was no asteroid to destroy.
    - You rewound time by one move.

### Programming Requirements

- Implement cell items and moving entities using **inheritance or interfaces** for clear class structures.
- Represent enemy state using a dedicated type (for example, an enum or a small class hierarchy) so that each state's behaviour is explicit in code.
- Maintain clean, modular, and reusable code for future expansion and ease of testing.

It is recommended that you implement the engine of the game first. You should write unit tests to check that your game logic, including enemy state transitions, the wait action, undo, and inventory use, works correctly.

Then you can use JavaFX to add a graphical user interface (GUI) to your game, which displays the 2D sector and allows the user to play the game. **You MUST use JavaFX, not any other Java GUI libraries.** Otherwise, the GUI part will be marked as 0.

## Learning Objectives

1. *Learn to use object-oriented design (OOD) to design Java programs;*
2. *Be able to use inheritance/interface and subtyping relationships between classes;*
3. *Be able to use association or composition relationships between classes;*
4. *Be able to model object state explicitly and reason about how state changes in response to events;*
5. *Be able to develop a comprehensive suite of unit tests for the core logic classes of an application (e.g., for the game engine);*
6. *Build simple graphical user interfaces using JavaFX.*

## Part 1: Game Engine (Weeks 8-10)

1. **Set up your Git Repository.** Follow the assessment link in GitHub Classroom provided in the task description in Cadmus to automatically create your own private GitHub repository containing one module with two packages: **starsalvage.engine** and **starsalvage.gui**.
   
   - Clone this repository onto your computer using IntelliJ.
   - Make sure you also have Java JDK 17 or later installed on your computer.
   
2. **Check you can run JavaFX.** This project uses the Gradle build tool to automatically download and install the necessary JavaFX and JUnit libraries when you build and run the project.

   After the build is complete, right-click on `src/main/java/starsalvage/gui/RunGame` and select **Run RunGame.main()**. You should see a blank JavaFX GUI pop up. Use this RunGame class every time you want to run your GUI.

3. **Data Design Decisions.** Think carefully about how to represent different cells in your sector, and how to represent the state of each moving enemy. You can use either inheritance or interfaces, or both, for cell contents. For enemy state, consider using an enum or a dedicated state class so that the transitions are explicit and testable.

4. **Implement and Test Your Game Engine.** `GameEngine.java` is provided as the base for engine development. You need to add more details to it, and you may also want to add other class files in the `starsalvage.engine` package to support engine development. For the classes in `starsalvage.engine`, write unit tests for them. Generally, you need to test every method in these classes. However, to keep this simpler, we do not require unit tests for: (1) constructors, (2) getters/setters, and (3) methods that involve the testing of input/output.

   Your unit tests **must** cover, at a minimum:
   - the state transitions of a Scout Drone under different Ship positions,
   - the three-phase cycle of a Sentry Turret and its line-of-fire calculation,
   - Space Mine drift and reflection at obstacles,
   - the inventory's pickup and Drill Charge deployment (including the case where the targeted direction has no Asteroid),
   - undo correctness over 5 sequential moves, including enemy turns, movements, wait actions, and Drill Charge uses.

5. **Text-based play.** The `main()` method of the GameEngine class should support text-based gameplay in the console. By the time you have finished implementing your game engine, one (or several) of your tests should be stepping through a complete game via the text-based UI.

6. **Class Relationships.** To get high marks, your engine needs to include several Java classes, with some association/composition relationships between them, and if possible, some inheritance/interface usage. Think about where you can best use these Java features. For example, enemy hierarchies, inventory and items, and the flight-recorder history are all natural candidates.

## Part 2: Game GUI (Weeks 10-11)

The goal of this stage is to use JavaFX to add an elegant and fully functional GUI to your game so that it can more easily be played on desktop computers. Your GUI should have the following features:

- event-handling of mouse events, including buttons for starting the game, moving up/down/left/right, waiting, using a Drill Charge, undo, save, load, and help;
- display of bitmap images (we recommend you use a large image for the background of the whole game, and small images for different items/cells in the map, so that the game looks professional and entertaining);
- **visually distinct** sprites or overlays for each Turret state (Idle / Charging / Firing), so the player can anticipate turret fire;
- multiple panels, with a main gaming panel to display the sector, plus one or more panels around the edges to display game options, Hull/score/steps, the inventory, the top-5 scores, and control/help buttons;
- a clean separation between the *back-end* (game engine) and *front-end* (GUI) classes using different Java package names, as described above.

Start by drawing one or two paper sketches of the GUI you plan to build. Take a photo of each sketch, as you will need to include these in your final report.

# Submission and Marking Criteria

Your report should be developed in Cadmus, based on the report template given in the Assessment / Task 2 area on Canvas. Your source code is hosted in your GitHub repository. You are reminded that your submission must be completely your own individual work, in your own words, and you must correctly reference any external sources that you use in your report, or in your code.

Your submission will be marked using the following criteria:

1. **[40%] Game Engine functionality and code:** this will be marked using the following criteria. ***Functionality:*** A playable and robust game with console input, including moving enemies with correct state transitions, a working inventory with Drill Charges, a working wait action, and a working undo of up to the last 5 moves. ***Code:*** Good OOD of the game engine classes with strong encapsulation of the data fields within classes; correct implementation of association or composition relationships between classes; good use of inheritance/interface and subtype polymorphism; explicit and well-organised modelling of enemy state; good choice of data structures; adherence to recommended Java coding style (naming conventions, code formatting, etc.); and concise, elegant code with no duplicated code.

2. **[15%] Game Engine JUnit Tests:** these tests will be marked according to how thoroughly they test all the Game Engine classes, including enemy state transitions, inventory behaviour, the wait action, and undo, and how well they follow correct JUnit naming conventions (e.g., tests of `starsalvage.engine.Foo.java` should be in the Java source file `starsalvage.engine.FooTest.java`). Unit tests are optional for the following methods: constructors, setters, getters, and methods that involve input/output.

3. **[25%] GUI functionality and code:** this will be marked using the following criteria. ***Functionality:*** A playable and robust game with GUI interface; good use of JavaFX widgets, layout panes, and controls to build an elegant and functional user interface; appropriate use of bitmap images in the GUI, including visually distinct sprites or overlays for the three Turret states; working Save/Load of game state; a working Wait button; and a working Undo button. ***Code:*** Good use of event-driven programming; good OOD of the GUI classes with strong encapsulation of the data fields within classes; good use of inheritance/interface and subtype polymorphism; adherence to recommended Java coding style (naming conventions, code formatting, etc.); concise, elegant code with no duplicated code; and good use of exception handling to catch and report any I/O or game errors. This will also demonstrate that your textual user interface still works after you have added the GUI.

4. **[20%] Documentation** (Report): this should contain four sections with 500--1000 words:

   A. *Introduction:* one paragraph to explain what your game does, how a player can win the game, what features you implemented, and how you represented enemy state and the flight-recorder (undo) history.

   B. *Your UML diagram:* that shows full details of all your game engine classes, plus the GUI classes that you have written. It should also show the main JavaFX classes you have used (just the class name is enough — you do not need to include all their data fields and methods), and the relationships between them and your classes. You do not need to include your unit testing classes in your UML diagram.

   C. *GUI sketches:* that show your original ideas for the GUI you plan to build for your game. This should be drawn by hand and then scanned or photographed and inserted into the report. It just needs to be a rough sketch, possibly with some labels to show the contents of each panel or the function of certain buttons, if that is not obvious.

   D. *Reflection:* a brief discussion of what went well in your game development, any difficulties that you encountered (including how you reasoned about enemy state transitions and the undo history), and what you would do differently if you were doing it again.
