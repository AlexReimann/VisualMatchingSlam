# VisualMatchingSlam
Estimate frame-to-frame relative movement by matching pixels and using the iterative closest point algorithm

##Idea draft:

###Prerequisites
The change from frame to frame is relatively small when expecting slow / normal movement or high frame rate. Thus the positional movement of every point in the image is small.  
**The position of a point in the next frame will be in the neighbourhood of the old position**

Also the change of illumination should be small from frame to frame (does not always hold to true but is set as requirement).
Each sensor has noise and a point in space will most likely not have the exactly same color value from one frame to the next. *But* the value will be close to the same (with a normal distribution around the actual value).  
**The points will be registered with about the same color value in the following frame**

Edges are a special case of the aforementioned. Points at edges change much more than points on surfaces.  
**The noise of points at (physical? TODO: check) edges are much higher**

In the real world uniform same colored surfaces without any color change are rare. Illumination is almost always directed which leads to a color gradient.  
**Uniform surfaces without color gradient are almost nonexistent in the real world**

###The hard things
Looking at the prerequisites it should be possible to match a point from one frame to the next frame by looking for the same / closest color value in the next frame. The problem is that points in the neighbourhood, although there is a gradient, have a most likely a high resemblance with each other. Together with the noise this can lead to points easily being matched wrongly.  
**Neighbourhood points can be easliy mismatched because of close resemblance and noise**

The iterative closest point (ICP) algorithm is in general robust against wrong matches, *as long as the average is correct*. This is because it is an error minimization scheme.  
**ICP is robust against noise as long as the average matching is correct**

Thus the question: How will the matching distribution look like? Or can the matching error be avoided?

As stated before edges have higher noise and mess up things even more. But they also carry much information, because forms are defined by their edges after all (though this is kind of philosophical).  
**Edges have to be treated special because of the high noise but also high information**

###Thoughts on the matching error
Assuming a uniformly colored surface, further on called "wall", and a directed light source, further called "lamp", the surface has a color gradient. The gradient is because of the directed illumination and goes in the same direction. The wall has uniform color and points with the same distance to the lamp have their color changed in the same way. Thus points on the wall with the same distance to the lamp will have the same color value (ignoring the noise).  
In 2D this leads to circles on the wall with the same color. In 3D it is some kind of waves which probably also have some fancy mathematical name I don't know about. Anyways, the important thing is that points with the same value are connected as they are on the same circle / wave. So they are in the neighbourhood and not uniformly distributed.  
**Points with the same value color are in the neighbourhood with a non-uniform distribution dependent on the illumination**
**The matching error will thus be non-uniform, which screws up ICP**

*So what can be done?*

###Thoughts on fixing the matching error
The color gradient can be determined in each frame for each pixel. Matching some gradient lines of each frame (I remember vaguely that the guys from http://vision.in.tum.de/ did something like that somewhen for slam) might work, with some math magic. But screw math I want something stupid.  
Using the gradient to identify the matching also doesn't help as they will be the same along the gradient line.
Can we instead somehow make the matching error be uniformly distributed, somehow taking out the gradient? This sounds weird at first.

The gradient makes it possible to match stuff on a uniform surface. If we remove it somehow, the points will look the same and the matching will not work anymore.  
So basically the directed illumination only changes the matching problem from the whole surface to the gradient line. Which then leaves us with the classical aperture problem (of optical flow).

----------------------

###Tackling the arperture problem
The problem is that multiple points in near vicinity have the same value and thus it can not be determind which points to match. If there is just a line, you don't see if you go up, down or move at all, the image stays the same.  
But in the actual case the images most likely have a shit lot of information. There is not just one line but stuff all over the place. Looking at how *all* things move gives the solution. Each pixel has multiple possible matches (especially because of noise). Each of these matches can be seen as a vote for a possible movement (this is stolen from how Hough transforms are used).  
**Each possible match can be used as a cast for a possible movement / transformation**

But ICP does optimization based on given matches. We don't know which are the correct matches so what do we do? Of course, come up with some custom weird scheme where we have no idea if it works or not: Guess the transformation and check if it fits the votings of the points (which we have no idea how far they are away, but whatever). This kind of seems to lead to some bundle adjustment. So basically assume some "random" transformations and check if they would fit, if stuff gets better then...  
Maybe this is not the right thing? Should use ICP somehow.

Can we use the votings to get a coherent visual flow?
