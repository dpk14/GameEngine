# voogasalad

Design Review
Author: Daniel Kingsbury

Overall Design:

The simulation runs by continually updating a Grid object composed of Cell objects according to the relative placement
and orientation of the cell objects within the Grid. The Grid is essentially just a 2D array with general methods for
placing and filling cells and determining what is in the neighborhood of a cell, according to the Grid type. The Grid type is
extended into subclasses with different kinds of shapes, and thereby different neighbor relations. Cells are a class defined
by a location in the Grid, and are extended into subclasses with state and type variables based off the simulation rules.

The Simulation superclass contains a method for iterating through and updating the grid the main class gives it according to the rules of the particular simulation. It contains all methods specific to determining how cells behave and interact with each other according to their type or state. It handles creating a set of new cells for the next frame and creating an updated grid containing them. 

The Grid superclass contains two methods, getImmediateNeighbors (which grabs all) the cells that share a side with the given cell and getAllNeighbors(which gets all cells that share a side or corner). If I wanted a new kind of grid shape, I would simply create a new Grid subclass for that shape type and override or extend these two methods to accomodate the polygon shape. If I wanted to make a torroidal grid, I would just add a torroidal boolean and implement a method that extends the existing two to accomodate neighbors on opposing sides of the grid. 

All possible neighbor rules can be constructed from these two methods, because they are perfectly foundational. For instance, the Sugar Scape simuation involves a vision variable that allows you to acquire neighbors a certain way, so I simply used these existing neighbors methods in a new getVisibleNeighbors method which uses a queue to get neighbors of neighbors until a certain distance out.

The simulations can be put in three categories: probabilistic, quota, and manually assigned. For probabilistic configurations, sliders in the visual interface can be adjusted to send in a probability of each cell type, and the configuration end uses these values to
determine a random cell value at each location according to the set frequency. Quota type configuration files ensure that
a fixed quantity is assigned to each cell type. The visualization initializes the interface so that viewers can assign simulation specific values that will be fed into configuring the initial states. 

The visual interface allows the user to select the simulation type in order to initialize it, after which a grid is initialized from the configuration end, sent into the appropriate simulation class to be processed and updated according to the simulation rules, then sent back to be interpreted and presented by the visual interface in the updated state. To add a new simulation, the simulation maker (me) would create a new cell subclass containing state tracking variables specific to the simulation type, along with a simulation subclass, which contains all necessary user input variables and all the rules by which the cell is updated every iteration. 

The visual interface then creates slider bars, shapes, and graphs to represent all the different cell types, with their respective quantities and states. The configuration person then uses the values chosen by the user to read through an XML file and assign values accordingly, creating a list of all the cells to be displayed in the initial configuration. The dependencies are extremely clear. An advanceSimulation method exists in the simulation class, which is overridden to call methods specific to the particular cell type and initialize appropriate instances of cells. GetNeighbors methods remain the same, and can be easily extended or incorporated to determine relations between neighboring cells.

Readability:

The code is consistent in its layout and naming conventions, especially in the Grid and Simulation classes. For instance, we recycle and modify the same getNeighbor methods universally in every simulation, extending them differently based on the grid type. A lot of the methods are written like short phrases that sequentially lay out the narrative of the code. There is no universal style, however, as different people were assigned to different classes and aspects of the code. Also, some simulations were so unique that they required a different approach. 

For instance, Langton's Loop was so structurally specific that it does not align with the method layout of other simulations, e.g, once the cell finished replicating, splitter cells had to be added to the simulation after a certain count to complete the separation process. Conversely, other simulations did not have a structured end goal like this, and new cells with very particular goals were not introduced under specific conditions.


Flexibility/Extensibility:

Creating new simulations is a seamless process, because it is easy to create new types of Grids, because Simulation subclasses can be made with different getNeighbor methods; and judging on whether a neighbor is detected out of bounds, the Grid can be made to expand. New cell subclasses can be extended off existing ones, using existing state and tracking variables and adding more. The structure of other advanceSimulation methods can be copied over to the new Simulation subclass, only with new conditionals and other methods that handle the rules of movement according to neighboring states.

Only the Grid class is affected by a change in the grid data type, because all of the simulations but Langton's loop
do not directly call 2D array coordinates, only the getNeighbor methods contained within the Grid class. These methods, along with
other Grid classes referring to the internal 2D array data type, would have to be reconstructed. Only Langton's Loop calls
2D coordinates directly, but that is because it can only be implemented on a rectangular grid (because of the strict starting configuration) so using a map would be inefficient. 

Restructuring the code to accommodate changes in grid structure was not difficult. We merely had to create a Grid type and rewrite the neighbor methods within that class for different Grid structures, then replace all references to a 2D array in all of the existing simulation subclasses to references to Grids and Grid methods. The cell classes remained the same, and the actual structure of the simulation subclasses were not altered. The visualization was merely extended to also include the different Grid types
by merely copying and restructuring existing cell layout methods.


Alternate Designs:

One of our key decisions was to give very limited information and methods to the cell, instead increasing the control of the Simulation subclasses. Near the beginning of the design process, we alternatively suggested giving cells information about their neighbors; even ownership of the getNeighbors methods. This would allow cells to internally adjust their status and prevent reliance on one grid data type (because all relations would be defined locally between cells, rather than universally within the data type). 

However, we reasoned that this would be arbitrary, because we had structured the Simulation class to contain the 2D grid and iterate through it every time. By nature of using such a rigid data type (a 2D array), neighbors would already be implicitly defined by their location in the Grid, so it would be unnecessary to additionally define their relative locations locally. I prefer the choice we made, because it was easy to extend. If we were to try to do the extensions with neighbor relations defined within the cell, we would have to make a subclass of every single cell subclass to determine relations for different cell shapes (for instance, TriangleAgentCell and RectangleAgentCell).

Another significant simulation decision was to create an ArrayList of cells, which would be filled with cells that needed to be updated
as the code iterated through the 2D array, after which the 2D array was filled by this ArrayList. Our alternative suggestion was
to skip this step and simply recreate a new 2D array directly. The ArrayList intermediate allows us to randomize our selection through
a simple shuffle method, but we could still accomplish this by visiting random indices from the 2D array and sending updated copies
directly into the new ArrayList. I think this would maximize efficiency and significantly reduce the runtime, and as a result
I would prefer it.

Conclusions:

I think the best feature of the current design is the structure of the Grid class. None of the code for the simulation classes
is affected by the kind of Grid, because the neighbor relations are built internally to the Grid. All extentions for different grid
types will merely extend the existing getNeighbors methods, and will not impact the way the Simulation calls it. Implementing
it taught me to create superclass methods with universally important functionality while not being too specific. It taught me
how to section off what variables should be mutated, accessed, and manipulated by what classes, and limit the access or use of
those variables by other classes. 

I think the worst feature in our project is definitely the 2D grid. It is painfully inefficient to iterate over the entire cell array every frame, especially when a great deal of those cells aren't performing any actions. I believe we should have alternatively used a map, because random visitation is an intrinsic characteristic of that data type, it can also store coordinates, and a smaller hashMap can be built from it easily, containing only the cells that need to be moved. I learned that for purposes of seamless extension, I should try to use such rigidly structured data types as 2D arrays.

I should also not use ArrayLists if order doesn't matter (because it doesn't). In general, I should really deeply consider
the data types before I use them. I should ask whether order matters, whether adding and removing is more important than accessing.
I should also ensure that the data type I am using would not require otherwise redundant and inconvenient steps if an important, reasonable extension were introduced. I should definitely maintain my structure of classes, because they were very well planned this time around.
