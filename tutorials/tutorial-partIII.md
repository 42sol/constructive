# introduction to Openscad with the <u>Constructive</u> library for a new openscad user

### PART III

NOTE: THIS IS STILL A BCONSTRUCTION SITE.
LARGELY UNFINISHED WORK IN PROGRESS.

[Part III tutorial](./tutorial-partIII.md) shows advanced Features like grouping commands into a g() group, working with Parts, and combinig them into Assembly
--------------------

if you are unsure about particular basic commands used in the codes snipplets below, please refer to the [basic tutorial](./basic-tutorial.md).

see also:

[Part II tutorial](./tutorial-partII.md) shows somee basic object modification like reflectX(), cScale() ,or colors and then goes on to explain, how to work with sets of similar objects without for(), with: pieces(), span(), vals(), selectPieces(), etc..

For a more advanced use also look at the explanations inside the example below

https://github.com/solidboredom/constructive/blob/main/examples/mount-demo.scad

there is also another Example at:

https://github.com/solidboredom/constructive/blob/main/examples/pulley-demo.scad

-------------------

The easiest way to try out the Library is to download the [kickstart.zip](https://github.com/solidboredom/constructive/blob/main/kickstart.zip)

> NOTE: To run all code examples from this tutorial you will need only Openscad and
> a single file: constructive-compiled.scad put in the same Fodler as your own .scad files.
> the easiest way to start is to download the [kickstart.zip](https://github.com/solidboredom/constructive/blob/main/kickstart.zip)
> and then extract both files contained in it into same folder. Then you can open the tryExamples.scad from this folder with OpenScad, and then use this file to try the code Examples from the Tutorial, or anything else you like. Just Pessing F5 in Openscad to see the Results.

### g() groups several commands and parameters into single command,

most of Constructive commands can be used inside and outside of g() providing exactly the same result, So in most cases usage of g() is not essential but is preferred. It generally reduces model compilation time by Openscad and allows you to specify additional default values like height(x) and solid() for a code block,
for example:

```.scad
include <constructive-compiled.scad>

two()
  g( reflectZ(sides()), Z(10) ,TOUP()
    , height(7), solid(true) )
  {
    tube(d=20);
    g(turnXY(45), X(60)) box(20);
  }  
```

![screen](./partII-images/g.png)  

#### height(h) and solid(true/false)

specify the default value for Constructive primitive tube() and box(). (solid() only affects tube(), tubFast(), and  tubeSoftHole() not the box()) ) So inside of the g() block above, the h and solid arguments can be ommited when using box() or tube()  

------------------------------------------------

## assembling mechanical Parts from several Modules

#### assemble()

allows application of universal operations like adding and removing solids (i.e. boring holes or addings screws), only to a Part of the Model representing a specific mechanical (sub-)Part)

To start with,surrounding your code by an  assemble() block without any parameters simply allows you to use add() and remove() instead of Openscads native difference()
One of the advantages over difference() is that you can put positive parts(to be subtracted from) and negative parts(to be subtracted) in any order at any place of the block in any number, even inside movements and rotations. whereas with difference() you are forced to start with a single positive part (or you have to use a union() of several) and continue by negative parts which will be subtracted,forcing this order on you.

For Example:

```.scad
include <constructive-compiled.scad>

assemble()
{
  g(X(10),turnXY(45),solid())
  {
    remove()
      box(15,h=10);
    add()
      tube(d=10,h=20);
  }

  add()Z(-3)
    box(10,h=10);
}
```

![screen](./partII-images/assemble1.png)  

similar to g() command it is preferred to group
the preceding and following movement and rotation commands inside the according add statements() brackets

so that:

```.scad
include <constructive-compiled.scad>

assemble()
{
  Z(10) turnXY(45) TOUP() add() g(height(10)) X(20) box(30);
}
```

produces the same result as:

```.scad
include <constructive-compiled.scad>

assemble()
{
  add( Z(10), turnXY(45), TOUP(), height(10), X(20) ) box(30);
}
```

![screen](./partII-images/assemble2.png)  

the same applies to remove(),applyTo() and confineTo() described later

--------

#### confineTo()

> NOTE: Due to Openscads own issues in current versions of Openscad. confineTo can sometimes produce unpredictable results, so you might be better off uing the old goo intersection() instead, untill it is fixed

## splitting your model into building blocks called "Parts"

it is essential at cirtain complexity to split the model into parts,so you can later remove one part from another
the Parts areorthogonal to the usage of modules:
asingle part can be cnstructed by several modules, and several modules can add or remove to/from the same partintheir code.

the parts which you want to display need to be span_8_rotated_boxesin the assemble("part1,greatpart,screw,orAlikePart") argument;
if you decide to hide a Part just remove its name from the assemble(".....") string argment
So here an example:
TODO:.....

--------

#### applyTo()

specifies name of the part which will be affected by the following add() and remove()
to create this part you also need to pass the part's name to assemble()
example
assemble()
{}

#### assemble Part name prefixes

+name - use the list of part name already provided by apply(), but add name to this list

:name exclude ancestors,only proceed if a part with exactly this name is beeing assembled, do not proceed for ancestors(which is a standard behaviour without the :)

!name - exclude exact part with name, but not its ancestors or children (if part was included)

Example of prefix use:

```.scad
include <constructive-compiled.scad>

assemble("aaa,ccc,ddd",$derivedParts=["bbb,ccc"])
applyTo("ddd,bbb")
{
  add("ccc,+aaa,!bbb")
  echo($currentBody);
}
```

this outputs:

```
Compiling design (CSG Tree generation)...
ECHO: "Assembling: ", ["bbb", "aaa", "ccc", "ddd"]
ECHO: "aaa"
ECHO: "ccc"
ECHO: "ddd"
```

-----

##parametrizing bodies for specific operations or parts
it is possible to parametrize your code depending on whether it is beiing added or removed  or depending
on the name of the body which is constructed.
Adding and removing

#### margin()

allows create gaps between bodies

```.scad
include <constructive-compiled.scad>
assemble("rod,plate")
{
  g(X(10),turnXY(45),solid())
  {
    add("rod",remove="plate")
      tube(d=margin(16,1),h=20);

    add("plate")
      box(margin(20),h=3);
   }
}
```

the tube(d=margin(16,1)... part of the code will adust thediameter ofthe tube,so that d=16 when the tube is added (when constructing the part "rod") and d=17+1
when the body is removed (when creating hole in the part "plate")
as a result there will be a visible gap between two bodies:
 ![screen](./partII-images/margin.png)

the default margin is set to .8 by default so ommiting the second parameter in margin call, like  tube(d=margin(16),h=10);
will produce a 16mm tube when added and 16.8 mm when removed, resulting in a gap of the half of 0.8 on each side of the tube.

#### $removing variable

allows you code to change parameters depending on weather it is being removed from another object:

```.scad
include <constructive-compiled.scad>
assemble("rod,plate")
{
  g(X(10),turnXY(45),solid())
  {
    add("rod",remove="plate",
      Z($removing?5:0),turnXZ($removing?-35:15))
      tube(d=$removing?30:10,h=20);

    clear(grey)
       add("plate")
      box(margin(40),h=10);
   }
}
```

results in:
![screen](./partII-images/removing_var.png)

####Topics to cover
----

*simple constrains(touching/distance)
------

*bodyIs(body)
bodyIs(body)?(what+($removing? extra:0)):0;

-----

*removeFor(body,extra=$removeExtra,what=0)
----

*adjustFor
-----

*autoColor()
looks up color of a part in the global color table
and assigns it to a block

-----

*misc. 2D
arc(r,angle=90,deltaA=1,noCenter=false,wall=0)
addOffset(rOuter=1,rInner=0)
function arcPoints(r,angle=90,deltaA=1,noCenter=false)

------

For a **basic introduction** (specially if you are new to Openscad )
see the [beginners tutorial](./basic-tutorial.md) it explains Constructive Syntax for main Building blocks, like tube(), box() or bentStrip() and their placement and alignment in space like stack() , align(), X(),Y(),Z() or turnXZ()

[Part II tutorial](./tutorial-partII.md) shows somee basic object modification like reflectX(), cScale() ,or colors and then goes on to explain, how to work with sets of similar objects without for(), with: pieces(), span(), vals(), selectPieces(), etc..
