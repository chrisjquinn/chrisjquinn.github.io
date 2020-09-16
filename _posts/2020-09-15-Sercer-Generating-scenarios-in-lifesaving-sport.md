---
layout: post
title: Sercer - Generating scenarios in lifesaving sport
---

With the usual question of *"What's lifesaving?"* the sport doesn't have much research affiliated with it compared to other sports. There are two parts to lifesaving sport; one very similar to swimming (Shown in the video below) and the other a 2 minute dash to save these fake casualties in all sorts of situations, called a <abbr title="Simulated Emergency Response Competition">SERC</abbr>. These scenarios come less abundantly in training, as they have to be thought of on the spot by a coach.

## The Goal
Make a tool that can generate these scenarios. <a href="https://github.com/chrisjquinn/Sercer" target="_blank">Sercer</a> goes about this via a desktop application in Java.

<iframe width="500" height="250" src="https://www.youtube.com/embed/OrOD3yczSIA" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<!-- More -->

## The Implementation

Making an application of such to generate these scenarios played on previous knowledge of Java, <a href="https://openjfx.io" target="_blank">JavaFx</a> was used to render buttons, canvases etc. and yields a nice result. Along with this, <a href="https://gluonhq.com/products/scene-builder/" target="_blank">Scene Builder</a> also does all the *XML* generation for you.


<!-- ![Sercer Screenshot](/assets/media/2020/09/15/sercer_screenshot.png) -->
<img src="/assets/media/2020/09/15/sercer_screenshot.png" alt="Screenshot of the Sercer application main window">

Even though the display looks very old-school with large pixel-like blocks on the birds eye view, this is made up for the speed in which the program can output something for a coach to use. For the various types of land & water scenarios possible, many processes are repeated. Each scenario is made through the following steps:

1. Choosing a land-based or water-based environment. Unless specified, a simple coin toss suffices.

2. Defining *in-bounds* and *out-of-bounds* areas. Either by using many bounding boxes, bodies of water, road-like shapes and finishing with a double buffer (decide in bounds first and fill the rest with out-of-bounds).

3. Populating these areas with casualties and aids.

Following this recipe, the scenarios vary greatly. This is great! Now coaches can use this as a template or just for their own inspiration of new ideas. The use of multiple windows allows communication for generating an environment or creating one; much like a version of MS Paint.

### The Object of Objects
Most developers now must already know from the subheading, the environment is defined via Java Objects. I took this approach as it is one that has been tought since first learning to program, along with the following explanation of defining the world as an object of objects which helps paint a picture.

The generator outputs an object that will contain our world. Our world is consisted of objects varying in nature; people, aids, walls, empty spaces, the list goes on. Abstraction of these world objects allowed the generator to not worry about whether this be a person or not, constraints such as being *in-bounds* has already been checked before being put into the world. Along with this, more "meta-data" such as the location of the environment or the weather can also be stored here. Operations on the world can be done as a whole, or by it's individual objects at their locations.


### Pros
Speed is always a key motivation for the development of applications in order to try and replace current systems. Here it comes double; the scenarios are generated and then rendered for a user. Beyond this, they can also be exported to `.pdf` or even `.serc` files (Thank you <a href="https://docs.oracle.com/javase/tutorial/jndi/objects/serial.html" target="_blank">Java Serialization</a>). The program also runs in one `.jar` file instead of a whole folder of classes and assets.


### Cons
In the low-level, the 2D birds eye view is represented with a 2D array. This is because the environments can change depending on it's size; some sparse some dense. Regardless of the environment at hand, we are now bound by **O(n<sup>2</sup>)** for a SERC **nxn** in length. Next, the application is already dated due to the choice of rolling out as one file, as JavaFx was removed from the JDK <a href="https://www.infoworld.com/article/3305073/removed-from-jdk-11-javafx-11-arrives-as-a-standalone-module.html" target="_blank">in 2018</a>.



## Running 
Firstly, making sure that having the <a href="https://www.oracle.com/java/technologies/javase-jre8-downloads.html" target="_blank">JRE 8 image</a> is the awkward part. Once this is installed on system, running the program is easy. This can be done either through clicking the `.jar` file or running:

`java -jar Sercer.jar`

With the above command assuming you have <a href="https://www.oracle.com/uk/java/technologies/javase-downloads.html" target="_blank">JDK</a> installed and are in the same directory of the file. Double clicking the file runs without the need of JDK.

You can also view the source code for <a href="https://github.com/chrisjquinn/Sercer" target="_blank">Sercer</a> in it's repository on GitHub.


## Pushing this further
Of course, no personal project can ever be said to be finished. The biggest (and next) step of this is to get this running on a subdomain, so that coaches can log-in to this web application and generate these scenarios. Not only does this make the application easier to access, better design can be used for the display.

-------

This work was part of my 3<sup>rd</sup> year dissertation & research project. If you wish to see the full report that dives into alot more areas; such as design, implementation and research, feel free to [contact me](mailto:quinn.christopherj@gmail.com).

