Design Review
===
Author: Daniel Kingsbury

### Summary:

A Unity-inspired game authoring environment allowing game designers with no programming skills to build arcade-style 2D scrolling platformers using a variety of visual tools and minimal scripting.


### Overall Design:

The Game Center--the central launch interface--provides access both to the Authoring environment (where games can be created) and the Player environment (where created games can be played). The Authoring environment uses just the static object development API of the Engine to design levels, while the Player uses both this part of the API jointly with the engine's dynamic mechanisms for object interaction and scene progression.   

The Game Center's is a concise, organized, extensive visual extremity that allows players to concisely track and store information and provides access to avenues to create new information. More specifically, it first gives players access to the authoring environment through a simple button, then completely propels them into the Authoring environment. The user creates his or her data in the Authoring environment by generating instances of Engine classes and data types, which are then made accessible to the Game Center (which initialized the authoring environment originally). 

The Game Center passively displays the creations of the authoring environment with thumbnails. The engine API appears to the user of the authoring environment like a showcase of tool-like classes, which can be instanced in concert to produce a combination that allows the user even greater functionality than the classes would have independently.

The Engine provides a range of tools (Components) with enough generality and compatibility that they can be mixed and matched in different combinations like replaceable parts to form a wide and effective variety of Entities (essentially game objects). The authoring does not use the Engine instances they design, but packages them up into data packages (games) accessible from the Game Center.

Along with access to the Authoring environment (which creates unified packages of Engine instance data), the Game Center also provides access to the Player, which loads and executes this data in the context of a Game Loop. The Game Loop processes and displays the results of each iteration of the simulation, then proceeds to the next iteration. 

The Player does not actually handle the interactions of the engine data types instanced by the authoring internally. Rather, the player is given access to a small subset of the engine's API which allows the initial configurations of the engine to be set. The engine handles the interaction of objects and the updating of frames. It runs indefinitely from the initial configuration, constantly packaging small windows of displayable data which are tracked by the Player so that it can visualize them. The Player essentially uses the engine to perform all caluclations that update the scene, and only displays the small number of visualizable components accessible to them. The Player--much like the authoring--packages the data type instances to send to the Game Center in the form of saved games and high scores. 

In summary, the Game Center launches the user into an environment to create inputs (the authoring) or an environment to interpret the inputs (the Player). 

The design is not particularly tied to a specific genre. The low dependency between the authoring and engine (the use of general scripts
which can be parsed uniquely and run from only the initial configurations without having to be manually, dynamically controlled by the
authoring or player) gave the Authoring environment the freedom to append the engine of their choosing with its own expected syntax. These engines could each provide radically different functionality and give the user access to an unlimited number of genres. 

To accommodate a different genre, we assume the author must know the syntax of the particular engine when creating scripts. A unique instance of the Game class is all that is needed to represent a game. The game data type holds a list of Scene instances, representing the levels. It also hosts a list of GameObject objects, which are the user-defined building blocks of the level, and are instanced uniquely in each level and stored in each Scene. GameObjects contain Strings defining how their Engine counterparts should be initialized, and what components they should receive. 

Instances extend these components with their own individual extended logic and specific component values. Scenes also contain a string that must initialize all the other non-object non-instance in the game, essentially describing the rules of interaction between objects -- and by extension -- their instances.

### General Engine Analysis:

The API of the engine is two-pronged. It contains a static part defining what kinds of objects can be created and what components they can have, accessible only through the ECS package. It also contains a dynamic part-- a few concise methods for updating a scene-- only accessible through the Controller class. The Player feeds instructions into the Controller in the form of string which
the Engine parses into instances of a concise set of key data types in the Controller. 

Some of these data types host sub-instructions which are parsed and distributed throughout the engine dynamically every new iteration of the game loop, propagating the Engine instance continually from the initial configuration. The Controller distributes the instructions in a predefined order by calling various methods of a group of handler classes, which are the vessels in which the instructions are placed and sent out to the tools the outer modules wished to access originally. 

These tools are called Components and Events. Components describe the properties of the instances the user designed in the authoring environment. Events manipulate these components dynamically.

The Components and Events of the Engine appear to outer modules as a passive, directly inaccessible showcase of the full
range of behaviors and states possible for all instances, manipulable only through adder and getter methods, and for
event activation, merely the Manager.call method. Outer, non-Engine modules tell the Controller in the initial String of instructions
how the module wishes to use the event/component showcase (the toolkit).

One design issue that needs to be fixed is the high number of public getters in setters in Components. These methods are necessary, because components are intended to be passive variable containers; we should have alternatively made the getters and setters protected and placed them in a package with a handler that could arbitrate their use.

The Engine Controller could stand to be improved. Rather than giving other modules multiple access points to privileged internal tools, I condensed the access point into one class, the Controller class. The Controller is initialized by the Player, who gives it the Strings created by the authoring environment, after which the Engine parses these to create its starting data types that define the initial configuration of the system. 

While I feel that I localized the access sufficiently, I feel like there is some room for improvement. Ideally, every game loop, the Player would only have to call one engine.run method that would completely update the back end trackers and the contents of the Player interface. The Player does only call only one engine method in the loop, controller.updateScene() (outside the loop in the key listener, the Player also calls controller.parseKey()) but once the game loop calls this method, it calls methods to interpret the updated contents of the engine, which involves some dependencies. 

These methods have direct access to engine getters so that they can read the current state of the engine and ensure that the front end parallels it. Because they have access to the public getters, they also have access to the public setters, which is potentially much more power and responsibility than the Player should have. For instance, the Player gets the map of instances from the Engine after it updates the scene, then ensures its front end representations of the model instances in the engine are updated to reflect the changes exactly. 

Some of these methods were very particular, so at one point both the Player team and Engine team were alternating on developing the Player Stage (which has the game loop) so that we could integrate and ensure that the Player was getting the right components in the right way and setting them to parallel their front end counterparts. Alternatively, this should have only happened once. The engine should have only written one method in the Player that somehow bound the imageView variables (location x, location y, width, height, image filename) ONCE to their engine counterparts when the Controller is initialized, so that all updates to game object components in the engine every loop would automatically update the front end. This way, only an initialization method for synchronizing the player and engine, one key listener engine call, and one game loop engine call would be needed for the Player and Engine to communicate.


## Engine AI: 

AIEvent is an abstract superclass that contains a bank of protected methods (and private helper methods) that can be used in synchronization to encode behavior in GameObjects. GameObjects carry strings in their Logic Component, which can be parsed and executed every loop, potentially calling some AI logic methods. The string parser does not have direct access to the AIEvent classes, but can access them through subclasses of AIEvents, which can call specific AIEvent protected methods when activated.

This structure is coherent and cuts off privileged access from outside packages, minimizing dependency vry effectively. Furthermore, the methods are well-thought, many-layered, and take advantage of the extensibility of Components to create additional subclasses that provide the AI a wider variety of ways to interact with the states of GameObjects. For instance, I was easily able to create an AimComponent and LOSComponent (line of sight component) that allowed GameObjects the capability to shoot missiles. The methods in AI Event are concise and take advantage of multiple shared helper methods, for example, the general math methods for calculating and adjusting angles.


### Engine Manager and Event Hierarchy:

The Engine Manager and Event hierarchy mediate a lot of the syntactical rigidity of the engine's syntax. They require only that the caller execute the manager's call method and specify the Entity instance they would like to perform the Event on, the Event class they would like to activate, and its case-specific, potentially unlimited parameters. 

This standardized event-calling allows for maximum extensibility. If someone wants to extend the engine to have some more capabilities, they just find the location of the Event in the Event hierarchy, or they create their own package with a new abstract event superclass and a family of superclasses. The calling convention remains the same, because it takes unlimited parameters.

First, the Manager class is instanced by the Controller and given access to some of the central game data -- namely the set of objects and maps of instances -- so it can handle interactions between objects and perform events on different object pairs. The Manager has a
public .call method that is used to activate a general event. 

Events are extended by InstanceDependentEvents, which are extended by ComponentDependentEvents, which are in turn extended by MotionEvent, AIEvent, AimModifierEvent, and HealthModifierEvent. Each of these requires different packages or classes, and takes different kinds of paramaters, but their calling procedure is standardized through the manager. 

Dependency is minimized this way, because Events (and therefore the data types they influence) have privileged access only through the manager's call method, and events remain completely encapsulated from all outer management classes: even the Controller. The Events are only given the authority to impact Components, and contain methods which manipulate them in unique ways, sometimes in concert with other Components.


### Flexibilty

I am most responsible for designing the Engine API, specifically the Engine Controller's API (not the passive Component/Event API).
This API only requires a String from the Player (loaded from a file) which it parses to initialize the data types it uses to track
and update the objects in the scene. To the designer of a scripting-based authoring module, this is very flexible, because it allows the authoring to create based based on any kind of engine that is also based on scripting. 

But there are setbacks to this design. The game script is developed through a combination of GUI tools and a console. The engine's syntactically specific API requires the authoring to either create a compiler to correct and expedite the author's creation of scripts,
or a vastly more extensive GUI to increase efficiency at the expense of the freedom script writing allows. 

The designers of the authoring environment minimized auth-engine dependencies so much that any engine is attachable and compatible in the place of the current one. No references at all are made to specific engine classes, so it is extremely encapsulated-- in fact, possibly too encapsulated. The minimal points of access between the authoring and engine leaves the authoring environment too uninformed about the engine to take advantage of its full range of capabilities. 

However, scripting is easy to extend because it serves as a foundational process -- a foundational philosophy of how data should be created and packaged for transfer (through Strings). Strings do not have to be manually created by the user, and independent, fully encapsulated GUIs can be added to the authoring environment that can write the Strings in a different way, after which the Strings are simply added to the terminal with the existing user code. 


### Alternate Designs

One of the APIs that has evolved hugely from its original vision is the Event API. We originally relied heavily on a class called the Entity Manager, which behaved like a disorganized toolkit: it had the same access privileges as the Manager, but harbored a mess of unrelated public methods that handled all varieties of Components. As a consequence, it had direct, universal access to all the different Components, and could modify them freely. 

Events were simply a hierarchy of classes that redundantly wrapped Entity Manager methods so that these events could be stored in maps and triggered once the terms represented by the key were satisfied. We also had a hierarchy of Conditionals. Every event owned a List of Conditionals, and would only be activated if the conditionals were all satisfied.

One of the developers of the Authoring Environment suggested that our event/conditional structure was redundant, and was forcing the engine to hard-code all possible combinations of entity manager methods and wrap them despite the fact that they already existed and were fully functional on their own.

He instead recommended Groovy scripting: rather than storing events in maps, we stored Strings in maps, which could be parsed
to call any number of Entity Manager methods in any combination. This would obsolete Conditionals, which were essentially just ways
of packaging boolean expressions. 

Scripting would remove the redundancy of events and allow the user to type out as complicated boolean conditionals as they pleased, rather than requiring the Engine to rewrite and hard-code some basic possibilities in the form of Conditionals. Once he proposed this to us, we discussed these pros separately and agreed to refactor. Later on, the Entity Manager was deprecated and organized into a hierarchy of abstract Event subclasses with different families of protected methods available to them. The element of scripting was remained, and now activates all events the same way through the Manager.call method.

The changes definitely improved the API, namely by making it extremely flexible. The Strings carried around by the Engine can contain
any information the author wishes them to. 

For the first week and a half, Entity-Component design was not in our horizon. We had a GameObject superclass and a hierarchy of
subclasses that specified unique extensions to this. But because some superclasses contained variables and methods that could be
used by the same object, code was getting duplicated, and the superclass hierarchy was becoming to extensive. Entity-Component design made methods and features attachable and replaceable between objects. 

The Engine team shared anecdotes of times they had been pressured into code duplication by the existing model, so we discussed alternatives and one of our team members introduced the entity-component design pattern. Components classes were inevitably very passive, and simply packaged passive variables supplemented by getters and setters. However, these minor cons were easily outshined by the extensibility provided by ECS: GameObject defaults do not have to be hard-coded, and can be custom-defined by the author using the infinite combinations of components possible. 

Entity-Component design is clearly the superior design model. Data is saved and transferred concisely between modules solely through an instance of the Game object. The Game object contains a list of GameObjects, as well as a list of scenes, each of which contains a list of Instances, which are built from GameObjects.

Alternatively, we were planning on the authoring environment directly creating the Engine Controller's key data types and saving them
to file. Our concern was that there was no apparent way -- at least with JSON -- to save these data structures directly to file.
We could only get JSON to save strings, so we were forced to represent our data types with Strings. The Engine team designed a Parser
class with an abundance of adder methods, which can be called from a string to build the Maps and Sets the controller needs
entry by entry. 

### Conclusions

The best feature of our design is our minimal dependency, namely our ability to attach any kind of engine to the authoring
environment, and the fact that the engine has only one access point. Designing the Controller has given me a concept of how to create
a concise API by creating controller-type classes to take in one or two data types from external modules and use that data type
to perform internal method calls throughout the engine. It has also given me ideas about how to condense information into one
well structured data type (in our case, the Game object) to be exchanged between modules.

The worst feature of the project is the scripting reliance in the authoring environment. Having scripting capability is not an issue,
but there absolutely must be a better system of supplemental GUIs to guide the process of String creation, because game making is
as of now a painstaking ordeal. I have learned to avoid using Strings alone to carry information, and in the future if I want to
pass information with Strings, I will try to use them in conjunction with custom made classes which can easily interpret the contents
of the String, requiring less precise and complex syntax from the user.

My biggest strength as a coder is in my adaptability. I am able to produce out-of-the box ideas quickly and test them quickly to determine whether they are worth using, and I am good at determining and accepting well in advance whether or not I should disband an idea. I write code that is easy to extend and refactor, so these changes tend not to be too revolutionary. 

My favorite part of coding is by far planning collaboratively. I'm very social, and I also require knowing almost exactly what I will be implementing before I begin, otherwise my code will lack direction, conviction, and concision. Planning allows me to keep in touch with my big-picture goals with a project, and reminds me of my purpose.

One of the key strategies I've developed is locating portions of a design that can be isolated into a unified, independent
part (a module or class hierarchy), and then minimizing the access points to this module. Sometimes reducing access points prevents modules from accessing the full utility of a module, so this process must be very intentional and cautious. 
