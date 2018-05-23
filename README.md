Overview
======

While the [ants of the highlands](https://codegolf.stackexchange.com/questions/135102/formic-functions-ant-queen-of-the-hill-contest) gather food and walk around each other, their distant cousins live in the forests below, growing fungus and aggressively raiding the ground and other nests for food. But like their highland counterparts, they want food, and they have armies of workers at their disposal to get at it. 

The Arena
------

The arena is a torodial 1000*2500 rectangular grid of cells. Each cell has one of eight colors, and at most one object at any one time. There are three types of objects of significance: 

  - Ants are either queens, which are spawned at the start of the game, or workers, which are spawned by queens. 
  - Food is a prized possession, and may be hoarded or consumed. 
  - Fungi are ant-placed objects which may be created at low cost and will grow slowly into food if left alone within the vicinity of food. 
  
In this game, adjacency refers to being in the [Moore Neighborhood](https://en.wikipedia.org/wiki/Moore_neighborhood) unless otherwise specified. Some rules instead specify the [von Neumann neighborhood](https://en.wikipedia.org/wiki/Von_Neumann_neighborhood). 

Game Overview
------

Each game, 16 submissions are put against each other in a game. If the competition has fewer than 16 submissions, all of them participate in a game, and if there are more, the 16 players are chosen randomly. 

At the beginning of each game, each cell is reset to the color 0 (white), each submission used in the game has its corresponding submission object constructed, and 2500 food, and 16 queens are randomly scattered onto the map. Queens are very unlikely to be adjacent to food or other queens, but this is not a guarantee. 

Then the game itself proceeds. The game lasts for 15000 turns, and each turn consists of the following phases, each of which is resolved concurrently in turn:

  - Decide: Each ant's views are determined based on the current state of the arena and its previous move history, and each ant's decision is determined and validated separately.
  - Move: Each ant's decision is executed simultaneously, and the presence of multiple objects in a single cell resolved once all ants that decided to move into a cell have done so.
  - Grow: Each object that can potentially change as a result of its surroundings has its neighborhood resolved, and the changes to the objects as a result happen simultaneously.

After the 15000 turns is up, the game ends. Submission objects are disposed of, caches trimmed down, food amounts logged, and the process started again if more scores need to be collected. 

Ant Senses
------

Each ant has a 5*5 area of sight centered on themselves. This area of sight is arranged to give the ants a relative sense of direction in the immediate term, but not innately in the long term. Ants can see the color of empty cells, and the properties of objects in filled cells, but not the color of filled cells. 

Each ant also has an internal state that can take one of eight values. This state is internal to that ant, and to transfer information between itself and other ants or to remember more than this, it must position itself, drop payloads, or mark the ground below it, all which are subject to enemy interference. 

Ant Actions
------

Using their senses, ants can decide to move or drop payloads, and also color their current cell and change their current state simultaneously, once per turn. The coloring happens on the cell that the ant is currently in, before it moves. 

Ants move orthogonally or diagonally one cell at a time. Worker ants are free to move into other cells at their own risk, but a queen may not move to a cell that her view shows is in the Moore neighborhood of a non-adjacent queen, nor to a cell that her view shows is in the von Neumann neighborhood of an adjacent queen. (However, it is legal for a queen to stay still on a cell adjacent to another queen.) Additionally, a queen without food may not move into a step into a cell already occupied by a laden worker, lest she step onto a laden worker that decides to stay still, which would without this restriction kill her. 
  
Instead of moving, ants may instead drop a payload into an adjacent cell. Food payloads cost one food, and can only be performed if the ant is carrying food. Fungus payloads cost nothing, and may be freely placed. Ant payloads may be performed only by the queen, cost one food, and consistently result in a new loyal worker. It is legal to drop payloads on oneself, but this will usually result in the dropped object being picked back up or destroyed during collision resultion, to no substantive effect. 

Ant Lifecycle
------

Both queen and worker ants start in state 0. Worker ants are oriented such that their "from" cell is the cell the queen occupied when spawning the worker, while queen ants have a random initial orientation. From there, it is up to submissions to perform ant-specific initialization and differentiation if desired. 

Once spawned, ants have an indefinite lifetime. Queen ants are guaranteed to live up to the end of the game, but worker ants may die by colliding with other ants. 

Ant Leeching
------

After movements and collisions are handled, a queen may neighbor one or more unladen enemy workers. If the number of such neighbors is equal to or less than the amount of food carried by that ant, then the queen loses food equal to the number of adjacent enemy workers. An unladen worker ant that neighbors one or more enemy queens that lose food this way gains one food. If a worker is next to multiple such enemy queens, it receives only one food, and one or more food is simply lost forever. 

This transfer does not occur between allied ants. To perform such food transfers, you must command the laden ant to drop its food in an adjacent cell, and have another ally be in that cell to automatically pick it up. 

Fungi
------

Fungi are a means of spawning more food. Fungus farms may potentially lead to bountiful harvests for submissions that invest the resources to spawn, watch over, guard and harvest these fungi. They may also be strategically planted to obscure the ground or probe for food from a distance. 

If food is within a 9 by 9 cell area centered around the fungus, then the fungus will grow. Internally, fungus takes 32768 steps to mature and spawn into food, and will gain 1 step per generation per piece of food placed in this vicinity. Therefore, placing more food in the vicinity of a fungus will speed up the fungus's growth, but will also make a nice feast for any ants that find it. If a fungus is not within the vicinity of food, it will not grow. 

Growth stages are visible to all ants, and work as a coarse indicator of the age of the fungus. 0 means that the fungus has taken no steps toward maturity, 1 means the fungus has taken 1-7 steps toward maturity, 2 means the fungus has taken 8-63 steps toward maturity, 3 means the fungus has taken 64-511 steps toward maturity, 4 means the fungus has taken 512-4095 steps toward maturity, and 5 means the fungus has taken 4096-32767 steps toward maturity. (If a fungus completes all 32768 steps of growth, it matures into food in time for ants to see it as food the next turn.)

Collision Resolution
======

As noted above, all decisions are executed at once, and collisions are resolved as they happen once surrounding ants have all their decisions executed. 

Ants
------

Collisions are an unavoidable consequence of multiple ants deciding to enter the same cell independently, or one or more ants entering a cell of an ant that decides not to move from it. Because ants may move only one cell at a time, up to 8 ants can move into the same cell at a time, and may run into an ant standing still in it. Queen-queen collisions are prevented by the rules restricting movement within the vicinity of other queens. 

Collisions of ants are resolved depending on the ants involved. For the purposes of collision resolution, an ant is considered moving if and only if it spent its turn moving itself to a different cell. A newly spawned worker is treated like a moving worker. 

  - A queen, if involved in a collision, is always the only survivor. She loses one food if she moved to a cell containing a laden worker that stayed still. As stated previously, a queen without food may not move to a cell already occupied by a laden worker, preventing the case where an unladen queen would step on a laden worker staying still. 
  - The collision of more than one moving laden ants in a worker-only collision results in all involved workers dying, leaving food behind in the cell. 
  - If a still laden worker is in a worker-only collision with no more than one moving laden worker, she survives, but loses her food if there was a moving laden worker in the collision. 
  - If a single ant in a worker-only collision held food and she moved, then she alone is the survivor. She loses the food she carried if she moved onto a worker ant that stood still. 
  - If none of ants in a worker-only collision held food, then they all die. 

Food and Fungi
------

Fungi do not stack. If multiple fungi are placed into a cell at once, collision resolution proceeds as if only one were placed in. Putting fungus in a cell already containing fungus will destroy the old fungus and create a new one. Fungus is destroyed if it shares its cell with any other object. 

A queen ant that stays or moves into a cell will pick up all contained and/or placed food in that cell. However, if a worker does the same, it can only pick up one of these pieces, and the rest are simply lost forever. If no ants are available to pick up the food, then only one food remains in the cell, with the rest lost forever. 

Submission
======

Each submission must be a Javascript class, with two methods holding special significance: a constructor, which is called once per game start to initialize each object before eachTurn is ever called, and an eachTurn method, which is called concurrently on all new ant views to determine what to do. 

The constructor is given no arguments, while eachTurn is given to arguments, an integer corresponding to the current ant's state, and a 25-length view array. 

A possible submission skeleton is as follows:

    class MyAnt {
    
        constructor() {
            //TODO: You fill this in
        }

        eachTurn(state, view) {
            //TODO: You also fill this in
        }

    }

Input
------

The view array contains objects of the following form: 

    {
        contents: Integer representing the contents of the cell, 0 for empty, 1 for food, 2 for fungus, 3 for ant
        details: Object giving details about the cell contents
    }

The details object yields further information about the object in the cell, and for each of the possible cell contents, it has the following forms: 

  - Empty Cell:

        {
            color: Integer from 0-7 inclusive representing color of the cell
        }

  - Cell with Food:

        {} // Yes, an empty object
    
  - Cell with Fungus:
  
        {
            stage: Integer from 0-5 representing fungus stage, a very coarse indicator of age and time to maturity
        }
    
  - Cell with Ant:
    
        {
            queen: Boolean representing whether the ant is a queen
            friend: Boolean representing whether the ant is ally or enemy
            food: Integer representing the current food stores of that ant
        }

Each view object provided represents reflects the true state of a cell in the arena, albeit with incomplete information. The array itself always corresponds to a flattened 5*5 cell area of the arena centered on the ant, rotated 0, 90, 180, or 270 degrees such that: 

  - For ants that have moved since spawning, either the cell to the top or to the top left correspond to the cell that the ant was in before the one currently inhabited
  - For worker ants that have not moved since spawning, either the cell to the top or to the top left correspond to the cell that the queen occupied the turn she spawned the worker
  - For queen ants that have not moved since spawning, the orientation is randomly determined at the start of the game and remains until the queen moves

Indices in the view array correspond to cells in english reading order: 

     0  1  2  3  4
     5  6  7  8  9
    10 11 12 13 14
    15 16 17 18 19
    20 21 22 23 24

There is no means of getting orientation information definitively either form inside or outside, and it must be inferred by current state and surroundings. 

Output
------

The output expected of the eachTurn method is an object with the following fields:

    {
        cell: (Mandatory) Integer in 0-8 inclusive, representing the cell to move to or to drop a payload to
        spawn: (Optional) If present, 0 for no payload, or 1, 2, or 3 for a payload of food, fungus, or worker, respectively
        state: (Optional) If present, an integer in 0-7 inclusive, representing the state to change to
        color: (Optional) If present, an integer in 0-7 inclusive, representing the color to color the old cell with
    }

The cell is interpreted with the same rotation as the view. Output cells 0-2 correspond to view indices 6-8, output cells 3-5 correspond to view indices 11-13, and output cells 6-8 correspond to view indices 16-18. If you compare this to the index chart above, it's in the same english reading order as the view array provided. 

No Side Effects
------

Submissions may not make state modifications except to themselves, and should take care so that internal state modifications from calls to eachTurn are not visible from outside. Debugging submissions using input and output is allowed, but once submitted as an answer, they are forbidden. Attempting to use input or output in a submission will result in an error and disqualification until edited out. 

Consistency
------

Submissions are expected to behave deterministically, and the eachTurn function is expected to be pure, returing the same decision object for a given combination of state and view, regardless of how eachTurn was previously called or is currently being called. 

Built-in functions may be called, if they are similarly externally pure. Math.abs() is fine, but Date.getTime() is not, for some examples. In particular, you are not allowed to call Math.random(). Supply your own pseudorandom numbers from constants, ant states, and ant views. 

Unless you have a good handle on javascript concurrency and are willing to stress-test the code for concurrency bugs, I recommend using well-tested concurrency patterns and performing only idempotent modifications of objects, or avoiding modification in the eachTurn object altogether. 

Resource Limits
------

Each game submission starts with 1 second of reserve time. Each call to the constructor and eachTurn add 1 millisecond to the reserve time, then decrement from the reserve time the time they took to execute. Calls are memoized, and don't count against a submission if they are found in cache. If a submission exhausts its reserve time, it is disqualified. 

Submissions are also limited to 64 Megabytes of memory at any one time. Unlike the time limit, this limit is not enforced automatically, but if a submission turns out to consistently hog memory during games it participates in, I will use a memory profiler to determine if this limit is exceeded. 

After Submission
======

Disqualification
------

To keep tournaments running smoothly, submissions are disqualified if their submission performs an invalid action. Disqualified submissions will be excluded from future games within a tournament after a disqualification, and will be kept out of future tournaments until the problem causing the disqualification is fixed. 

The following conditions are detected automatically, and therefore result in disqualification immediately: 

  - Exhausting reserve time, as described above
  - Returning an ill-formed or badly typed object from eachTurn
  - Throwing an exception from the constructor or eachTurn
  - Attempting to spawn food or workers with no food
  - Attempting to spawn a worker with another worker
  - Attempting to move a queen too close to another queen
  - Attempting to move an unladen queen to a cell containing a laden worker
  - Performing input/output from submitted answers

It might seem harsh to disqualify for a single wrong move or bug, rather than consider it "no move" or ignoring it, but by insisting on correctness from entries, I can focus my efforts on keeping tournaments running quickly and smoothly. This is not supposed to be an additional challenge, so a reason is given for any disqualification, and an explanation given, with specific input and output given to help solve the disqualifying problem. 

Multiple Answers and Editing
------

You may provide multiple answers, provided that each one stands as a competitor in its own right, does not team up with other submissions, and at least in part is the product of your own substantive effort. You may take advantage of other submissions' weaknesses in an effort to achieve a higher score in comparison. Keep in mind that submissions come in, the chances of running into another particular submission will decrease. 

You may also edit your submissions to tune them however you choose. There are no guidelines about whether to create a new post or edit a currently existing post; the choice is yours. 

If you make a variation of another submission, remember to differentiate it, and if is derived from someone else's work, remeber to credit your sources. 

Scoring
------

At the end of a game, submissions are ranked on how much food their queens held at the end of the game. Submissions which score exactly equal to each other simply share the average of the ranks they would have if they scored differently from each other. 

Submissions with the highest average rank over a multitude of games win. For this challenge, I will give out working first places from tournaments as scores accumulate enough significance that Dunn's test indicates that there is a single distinct first place with at least 98% confidence. 

Chat
======

For questions and extended discussion of this challenge, please use the [chat room](https://chat.stackexchange.com/rooms/77728/the-formic-forest). Comments on this post are likely to be cleared up from time to time, while chat room text will be kept around permanently. 

If you want to contribute to the specification itself, see the github repository hosting the latest changes to this specification, [right here](https://github.com/eaglgenes101/formic-forest-specification). 


