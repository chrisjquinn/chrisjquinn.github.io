---
layout: post
Title: Experiments
---

Experiments in Constraint-based Graphic Design
12 Dec 2019 · 16 min read — shared on Hacker News, Lobsters, Reddit and Twitter
Standard GUI-based graphic design tools only support a limited “snap to guides” style of positioning, have a basic object grouping system, and implement primitive functionality for aligning or distributing objects. They don’t have a way of remembering constraints and relationships between objects, and they don’t have ways of defining and reusing abstractions. I’ve been dissatisfied with existing tools for design, in particular for creating figures and diagrams, so I’ve been working on a new system called Basalt that matches the way I think: in terms of relationships and abstractions.

Basalt is implemented as a domain-specific language (DSL), and it’s quite different from GUI-based design tools like Illustrator and Keynote. It’s also pretty different from libraries/languages like D3.js, TikZ, and diagrams. At its core, Basalt is based on constraints: the designer specifies figures in terms of relationships, which compile down to constraints that are solved automatically using an SMT solver to produce the final output. This allows the designer to specify drawings in terms of relationships like “these objects are distributed horizontally, with a 1:2:3 ratio of space between them.” Constraints are also a key aspect of how Basalt supports abstraction, because constraints compose nicely.


I’ve been experimenting with this concept, off and on, for the last couple years. Basalt is far from complete, but the exploration has yielded some interesting results already. The prototype is usable enough that I made all the figures in my latest research paper and presentation with it.

If you want to read about the core ideas behind Basalt, take a look at the philosophy section. If you want to hear about my experience using Basalt to design real figures, skip ahead to the case studies section. If you want to see how gradient descent can be used to solve figures, go to the gradient descent section.

Philosophy
Basalt’s programming model is as follows. Designers write programs that produce figures described in terms of relationships. These relationships are compiled down to constraints, which are then solved automatically.

Basalt is a DSL embedded in a general-purpose programming language, so it inherits support for functional abstraction, classes, and so on. Its constraint-based approach is key to supporting abstractions that compose nicely.

Specifying drawings in terms of relationships
Basalt’s programming model allows drawings to be specified in terms of relationships between objects. The standard GUI-based tools for graphic design don’t have support for this. They have a limited “snap to guides” style of positioning, plus a basic object grouping system and basic functionality for aligning or distributing objects. They don’t encode figures in terms of primitive objects and constraints; instead, actions like aligning objects imperatively update positions, which is what the underlying representation stores. CAD tools have support for constraints, but they aren’t meant for graphic design.

Consider a figure with the following description. “There is a light grey canvas. In the center is a blue circle with a diameter that is half the canvas width. Circumscribed in the circle is a red square.” The picture looks like this:

Circumscribed square

Without using constraints, code to generate such a figure in Basalt looks like this:


width = 300
height = 300

bg = Rectangle(0, 0, width, height,
    style=Style({Property.fill: RGB(0xe0e0e0)}))
circ = Circle(x=150, y=150, radius=75,
    style=Style({Property.stroke: RGB(0x0000ff), Property.fill_opacity: 0}))
rect = Rectangle(x=96.97, y=96.97, width=106.06, height=106.06,
    style=Style({Property.stroke: RGB(0xff0000), Property.fill_opacity: 0}))

g = Group([bg, circ, rect])

c = Canvas(g, width=width, height=height)
The designer has to figure out where everything goes, manually computing the anchor points and size. Instead, with constraints, the designer can write down the relationship between the shapes, and the tool will solve the figure:


width = 300
height = 300

bg = Rectangle(0, 0, width, height, style=Style({Property.fill: RGB(0xe0e0e0)}))
circ = Circle(style=Style({Property.stroke: RGB(0x0000ff), Property.fill_opacity: 0}))
rect = Rectangle(style=Style({Property.stroke: RGB(0xff0000), Property.fill_opacity: 0}))

# The Group constructor takes two arguments: the first is a list of objects,
# and the second lists additional constraints, equations that relate the
# objects to one another

g = Group([bg, circ, rect], [
    # circle is centered
    circ.bounds.center == bg.bounds.center,

    # circle diameter is 1/2 of canvas width
    2*circ.radius == width/2,

    # rectangle is centered on circle
    rect.bounds.center == circ.bounds.center,

    # rectangle is a square
    rect.width == rect.height,

    # rectangle is circumscribed
    rect.width == circ.radius*2**0.5
])

c = Canvas(g, width=width, height=height)
Every primitive shape knows how to calculate its own bounds based on its internal attributes: for example, a circle’s bounds are (x−r,y−r)
 to (x+r,y+r)
, and a group’s bounds are defined in terms of min/max of the individual elements’ bounds. Attributes, as well as bounds, are allowed to be symbolic expressions: e.g. in the code above, the circle is not given a concrete center or radius, so these attributes are each automatically initialized to a fresh Variable(), and then the bounds depend on these variables. They are only assigned a concrete value when the constraint solver runs.

Basalt’s constraint-based approach allows the designer to think in terms of relationships and express those in the specification of the drawing itself, rather than having them be implicit and be manually solved by the designer.

The figure above could feasibly be drawn in Illustrator, though it might require taking out a pencil and paper to calculate positions of objects. Manually solving implicit constraints that are in the designer’s mind but not encoded in the tool, which is what designers currently do with existing programs, is painful and not scalable.

What if the figure design were changed slightly, for example the circle was to be inscribed in the rectangle? With Illustrator, it requires recomputing all the positions by hand; with Basalt, the change is one line of code:

-  # rectangle is circumscribed
-  rect.width == circ.radius*2**0.5
+  # circle is inscribed
+  rect.width == circ.radius*2
Side-by-side comparison of circumscribed and inscribed square

For a simple figure, making such a change by hand might be feasible, but what if the figure had hundreds of objects? With Illustrator, it would get out of hand to manually position all the objects precisely where they need to go based on the constraints in the designer’s mind. In Basalt, the constraint solver does the difficult job of determining exact positions for objects, and it scales well (for example, this figure has hundreds of objects).

Supporting abstraction
Constraints lead to a natural way of supporting abstraction, another key aspect necessary for designing sophisticated figures. Sub-components can be specified in terms of their parts and internal constraints. When these components are instantiated for use in a top-level figure (or a higher-level component), the constraints can simply be merged together.

Suppose we wanted to have an abstraction for a circumscribed square and then have four of them, arranged in a 2x2 grid, that fill the canvas. First, we can define the abstraction, a group consisting of a circle and rectangle, with some internal constraints:

def circumscribed_square():
    circ = Circle(style=Style({
        Property.stroke: RGB(0x0000ff),
        Property.fill_opacity: 0
    }))
    rect = Rectangle(style=Style({
        Property.stroke: RGB(0xff0000),
        Property.fill_opacity: 0
    }))
    return Group([circ, rect], [
        # rectangle is centered on circle
        rect.bounds.center == circ.bounds.center,
        # rectangle is a square
        rect.width == rect.height,
        # rectangle is circumscribed
        rect.width == circ.radius*2**0.5
    ])
Then, we can instantiate it multiple times and add the constraint that they are arranged in a 2x2 grid and are all the same size:

c1 = circumscribed_square()
c2 = circumscribed_square()
c3 = circumscribed_square()
c4 = circumscribed_square()

g = Group([c1, c2, c3, c4], [
    # arranged like
    # c1 c2
    # c3 c4
    c1.bounds.right_edge == c2.bounds.left_edge,
    c3.bounds.right_edge == c4.bounds.left_edge,
    c1.bounds.bottom_edge == c3.bounds.top_edge,
    # and all the same size
    c1.bounds.width == c2.bounds.width,
    c2.bounds.width == c3.bounds.width,
    c3.bounds.width == c4.bounds.width,
])
Finally, we can express the constraint that the figure fills the canvas:

width = 300
height = 300

bg = Rectangle(0, 0, width, height, style=Style({Property.fill: RGB(0xe0e0e0)}))

top = Group([bg, g], [
    # figure is centered
    g.bounds.center == bg.bounds.center,
    # and fills the canvas
    g.bounds.height == bg.bounds.height
])

c = Canvas(top, width=width, height=height)
When rendered, this code produces the following figure:

Four circumscribed squares

Constraints as an assembly language
Constraints aren’t necessarily what end-users should write directly, but constraints make for a nice assembly language: it’s easy to express higher-level geometric concepts in terms of Basalt’s constraints. For example, expressing that a set of objects are top-aligned is as simple as this:

def aligned_top(elements):
    top = Variable() # some unknown value
    return Set([i.bounds.top == top for i in elements])
Other concepts, like objects being horizontally or vertically distributed, being the same width or height, or being inset inside another object, can also be compiled down to constraints. Basalt provides some higher-level geometric primitives, and users can define their own.

Case studies
Adversarial turtle figure
Here is a figure I made for a paper (Figure 5), recreated in Basalt:

Adversarial turtle figure

The figure shows a number of images, along with a classification of those images given by the border color, laid out in a grid. The Basalt code for this figure defines two abstractions that are composed to make the figure:

def bordered_image(source: str, border_color, border_radius):
    image = Image(source)
    border = Rectangle(style=Style({Property.fill: border_color}))
    return Group([border, image], [
        inset(image, border, border_radius, border_radius)
    ])


def grid(elements: List[List[Element]], spacing):
    rows = [Group(i) for i in elements]
    return Group(rows, [
        # all elements are the same size
        same_size([i for row in rows for i in row.elements]),
        # elements in rows are distributed horizontally
        Set([distributed_horizontally(row, spacing) for row in rows]),
        # and aligned vertically
        Set([aligned_middle(row) for row in rows]),
        # rows are distributed vertically
        Set([distributed_vertically(rows, spacing)]),
        # and aligned horizontally
        aligned_center(rows)
    ])
This figure is simple enough that it would be possible to draw it in a traditional tool like Illustrator, but there are benefits to drawing it programmatically beyond avoiding the annoyance of manually laying out the figure in a GUI-based tool. The paper has 5 figures in this style, with varying images and sizes: using code allowed us to parameterize over these and do the work of designing the figure just once. Furthermore, the figures were meant to go in a paper, a fairly restricted format: for example, the width of our figures was fixed. We needed to decide on figure parameters, such as grid dimensions, border size, and image size, that would make the figures readable, and having the code generate the figures made it easy to explore this parameter space.

The original figures in the paper were actually made without Basalt, because Basalt didn’t exist at the time. Instead, I wrote Python code that directly painted pixels of an output image. The Basalt code is much nicer.

Robust ML logo
My brother Ashay and I designed the logo for Robust ML. Starting with the general ideas of shields and neural networks, Ashay drew a couple sketches on paper:

Robust ML logo sketches

Next, I sketched the logo in Illustrator. Even after we had the basic idea for the logo, it took a ton of iteration to figure out the details, including choosing the number of layers in the neural network, the number of nodes in each layer, and the spacing between the nodes in a given layer. Certain changes were easy, like tweaking colors using Illustrator’s recolor artwork feature. Other changes were extremely painful; for example, adding a node required a lot of manual labor, because it required moving nodes and lines as well as creating and positioning a new node and many new lines. Cumulatively, over dozens of iterations of the logo, I spent a couple hours just moving shapes around in Illustrator.

Since then, I have re-made the logo with Basalt, where exploring a parameter space is much easier with the help of the live preview tool.


Here is what the final output looks like:

Robust ML logo

Notary architecture figure
This is a figure made in TikZ, from a draft of a recent paper:

Notary architecture figure in TikZ

Here’s the code for the TikZ figure above. It uses lots of hard-coded positions, with commands like \draw [boxed] (-1,-2.5) rectangle (16.5,4). It was difficult to get the figure to look particularly pretty using TikZ. For the published version of the paper (Figure 1), I designed the figure using Basalt, also changing the styling a bit in the process:

Notary architecture figure in Basalt

The figure defines and makes use of a number of new abstractions built on top of Basalt’s primitives, including:

Component — a box, with text centered inside
Peripheral — a box with rounded corners, with text centered inside
Wiring — a bunch of line segments, with optional arrowheads, connecting two points, with optional label text
Describing these abstractions once and then using them many times made for a pleasant figure design experience. For example, Wiring was defined once and instantiated 11 times in the figure above. And of course, the entire figure was specified relationally. Concepts like the “Kernel domain” text being centered within its area were easy to express. The code makes use of many geometric primitives, expressing concepts such as a component’s inputs/outputs being evenly spaced along the edge.

Notary SoC illustration
Here’s another figure from the same paper (Figure 2):

Notary SoC illustration

The figures in the paper share abstractions for a consistent look; this figure makes use of the Component and Wiring abstractions from the previous figure. Constraint-based design was especially helpful for designing certain aspects of this figure, such as the spacing for the arrows at the bottom. For example, having the arrows spaced evenly along the entire width of the boxes looks pretty terrible; this was easy to test, only requiring a small change to the I/O Arrows abstraction:

-  distributed_horizontally(arrows.elements, periph_io_spacing),
-  arrows.bounds.center.x == periph.bounds.center.x
+  distributed_horizontally(arrows.elements),
+  arrows.bounds.left == periph.bounds.left,
+  arrows.bounds.right == periph.bounds.right
The constraint-based design, along with Basalt’s live preview tool, made it easy to select other parameters such that they looked good for the final figure.


Implementing this figure in code also made it easy to do things like matching the font size between the figure and the text in the paper. When designing figures in external tools, the graphics are often scaled afterwards to fit in the paper, but this messes with the effective font size. With Basalt, it was easy to design the overall figure, then size it appropriately so it could be included in the paper without rescaling, and finally tweak figure parameters so it could use the same font size as the paper.

Notary architecture animation
I needed to make a simplified version of the Notary architecture figure for a presentation. Furthermore, I needed to animate the figure over multiple slides to highlight and explain different aspects.

I started out implementing the presentation and the figure in Keynote, but designing the figure in Keynote was a bit of a pain due to having overlapping objects, as well as having details varying between phases of the build. I tried doing everything on one slide, but I ran into limitations with Keynote’s animation system (for example, it’s not possible to make an object appear, then disappear, then appear again). I tried duplicating some objects to work around this issue, but that quickly got out of hand, even with Keynote’s object list. I tried splitting the build over multiple slides, but then making global edits became annoying.

Finally, I switched to using Basalt for the figure (and switched the presentation to Beamer). It was straightforward to design a figure with details that varied depending on the build phase:


Aspects that changed based on the build phase took advantage of a BUILD variable, such as reset_color = red if BUILD == 3 else black. This BUILD variable was set via an environment variable, BUILD = os.environ['BUILD'], so different versions of the figure could be rendered by setting e.g. BUILD=2 and rendering the figure.

Notary noninterference figure build
The same presentation has another figure that is built up over a series of slides:


It uses the same BUILD variable approach as the previous figure to encode aspects that changed based on the build phase, such as the delimiters along the bottom of the figure:

labels = []
if BUILD >= 1: labels.append('Agent A runs')
if BUILD >= 2: labels.append('Deterministic start')
if BUILD >= 3: labels.append('Agent B runs')
if BUILD == 4:
    labels = ['Deterministic start']
delim = LabeledDelimiters(labels)
To render different phases of the build, the figure is rendered with different settings of the variable, e.g. BUILD=3.

This figure was designed to match the same visual language set up in the previous figure. This was easy to achieve by sharing the abstraction with the previous figure (instantiated here with slightly different parameters such as a lower wire count).

Again, Basalt’s live preview tool was helpful in choosing figure parameters, such that the figure looked good and the font size matched other figures in the presentation.


DFSCQ log figure
I wanted to try to recreate a complex figure from a paper (Figure 9) that I did not write. The original was made in Inkscape by one of the authors of the paper. The goal of the Basalt version was not to create a pixel-perfect recreation but to use Basalt’s approach of building up abstractions to see how it scales to complex figures. Here is a side-by-side comparison showing that Basalt is capable of replicating the figure:



The Inkscape figure is a large collection of manually positioned objects, presumably made with a bunch of copy-pasting. The Basalt figure, on the other hand, defines and uses a number of abstractions, all implemented as classes that build on top of Basalt’s primitives:

Layer — one of the stacked horizontal boxes with various contents, e.g. “LogAPI”
Blocks — a sequence of colored rectangles, e.g. the blocks to the right of the “activeTxn” label
Bracket — a “[” or “]” shape
BlockList — a list of blocks, consisting of an open bracket, a sequence of blocks separated by commas, and a close bracket, e.g. the contents to the right of the “committedTxns” label (uses Blocks and Bracket)
LabeledDelimiters — labels corresponding to parts of something, e.g. below the “disk log” (this abstraction is also used in the Notary noninterference figure above)
DiskLog — contents of a disk log, e.g. to the right of the “disk log” label (uses Blocks)
Explode — the pair of dotted lines showing one part of the figure exploded out, connecting subparts of certain stacked layers, e.g. in between the “DiskLog” and “Applier” layers
ArrowFan — arrows fanning out, e.g. in the “Applier” layer
This figure took some effort to implement. Basalt’s live preview tool was immensely helpful while working on this figure: seeing updates to the figure immediately after changing code was essential for building up this complex figure. The sliders were also somewhat helpful in getting the layout to match the original. Playing around with the parameters and seeing the figure redraw itself while maintaining its general structure is pretty neat:


Discussion
Solving constraints
Basalt allows designers to specify diagrams in terms of relationships and constraints, which boil down to equations over real variables. Basalt doesn’t place many restrictions over these equations, which gives great flexibility to the user, but it makes the underlying implementation more challenging because it has to solve these equations automatically. Basalt equations can have things like conjunctions and disjunctions, min/max terms, and products/quotients of variables, so in the general case it’s not possible to transform equations into some nice form like a linear program that’s computationally efficient to solve. Basalt equations fit into the logic of quantifier-free non-linear real arithmetic (QFNRA), so at least satisfiability is decidable, but solving isn’t necessarily going to be fast.

Currently, Basalt supports a couple different strategies for solving equations.

Z3

The current default uses the Z3 theorem prover, a powerful SMT solver that has support for QFNRA and much more. Encoding a constraints from Basalt into a Z3 query is straightforward, and in practice, Z3 works quite well.

Mixed integer linear programming

Basalt supports compiling a subset of constraints, ones that use linear real arithmetic along with a handful of operations like min/max, into a mixed integer linear program (MILP), which are then solved using a MILP solver, currently CBC through Google OR-Tools. Basalt’s MILP backend is strictly less general than the Z3 backend, and it seems to be slower than Z3 even on the queries it supports, but it was fun to implement.

Gradient descent

By far the least useful but most fun equation solver is one based on gradient descent. There’s a straightforward translation from Basalt constraints to a differentiable loss function, and this translation is sound. This means that the loss function has a global minima of 0
 if and only if the original constraints are satisfiable, and when the loss function does have a global minima of 0
, every global minima corresponds in a straightforward way to a satisfying assignment for the original variables.

Now of course, this says nothing about the characteristics of this loss function and whether it’s amenable to optimization via gradient descent. But I went ahead and implemented it anyways, using TensorFlow to handle the task of computing derivatives.

Unlike the other approaches, gradient descent computes the solution iteratively, so it’s possible to animate the solving of a figure, and the results are quite amusing:


Uniqueness of solutions
Sometimes, users produce figures where the set of constraints has multiple solutions; in other words, the figure is not fully constrained. In my experience, this is almost always indicative of a bug in the figure’s code, because it’s unlikely that the user has two different-looking figures in mind where either figure is acceptable.

Basalt can determine when this situation occurs, because it’s straightforward to encode as an SMT query. If the original set of constraints was C
, and it has a solution M=(x1=v1∧x2=v2∧⋯xn=vn)
, then we can ask the SMT solver if C∧¬M
 has a solution. If the new formula is unsatisfiable, then the original solution is unique, and the figure is fully constrained. In the case that the new formula is satisfiable, the SMT solver returns a concrete example of another valid solution. Comparing the two solutions can help a user figure out why a figure is under-constrained.

Next steps
There is still much work to be done on Basalt.

Right now, it’s an embedded DSL in Python, and the language is a bit clunky. Racket, a programming language for building programming languages, may be a better platform. Tej Chajed and I are currently working on building a more sophisticated version of Basalt as a Racket DSL.

Another thing I’m interested in exploring is having a visual editor for Basalt figures, a kind of hybrid between Basalt’s approach and existing GUI-based tools, because it’s often convenient to be able to drag objects around. This might involve using an approach similar to what’s used in CAD tools or perhaps something more exotic like the technique used in g9.js to have a two-way binding between the code and its visual representation.

Related work
There has been a long line of research in the area of constraint-based design, starting with Ivan Sutherland’s Sketchpad in 1963. As far as I know, most of the tools that use these ideas are GUI-based CAD programs, not practical graphic design tools.

Conclusion
Basalt is an approach to designing figures based on constraints. Basalt’s model allows a designer to think in terms of relationships between objects, and it makes it easy to build and reuse abstractions.

Basalt is still a work in progress. If you’re interested in hearing about updates, click here.

Thanks to Tej Chajed for many insightful discussions about Basalt, and thanks to Ashay Athalye, Kevin Kwok, and Curtis Northcutt for feedback on this post.