# Graphics Programming

## THE SIERPINSKI GASKET

   <img title="a title" style="float: right;" alt="Alt text" src="assets\Sierpinski_Gasket.JPG">

1. Pick an initial point $p = (x, y, 0)$ at random inside the triangle.
2. Select one of the three vertices at random.
3. Find the point $q$ halfway between $p$ and the randomly selected vertex.
4. Display $q$ by putting some sort of marker, such as a small circle, at the corresponding location on the display.
5. Replace $p$ with $q$.
6. Return to step 2.

The psuedo-code for this algorithim will be,

```
function sierpinski()
{
    initialize_the_system();
    p = find_initial_point();

    for (some_number_of_points)
    {
        q = generate_a_point(p);
        display_the_point(q);
        p = q;
    }

    cleanup();
}
```

The real code will look like

The strategy used in this algorithm is known as <b>immediate mode graphics</b> and, until recently, was the standard method for displaying graphics, especially where interactive performance was needed. One consequence of immediate mode is that there is no memory of the geometric data. With our first example, if we want to display the points again, we would have to go through the entire creation and display process a second time.

```js
function sierpinski()
{
    initialize_the_system();
    p = find_initial_point();

    for (some_number_of_points)
    {
        q = generate_a_point(p);
        store_the_point(q);
        p = q;
    }

    display_all_points();
    cleanup();
}
```

In our second algorithm, because the data are stored in a data structure, we can redisplay the data. The method of operation is known as <b>retained mode graphics</b> and goes back to some of the earliest special-purpose graphics display hardware. This approach has one major flaw. Suppose that we wish to redisplay the same objects in a different manner, as we might in an animation. The geometry of the objects is unchanged but the objects may be moving. Displaying all the points as we just outlined involves sending the data from the CPU to the GPU each time that we wish to display the objects in a new position. For large amounts of data, this data transfer is the major bottleneck in the display process. Consider the following alternative scheme:

```js
function sierpinski()
{
    initialize_the_system();
    p = find_initial_point();

    for (some_number_of_points)
    {
        q = generate_a_point(p);
        store_the_point(q);
        p = q;
    }

    send_all_points_to_GPU();
    display_data_on_GPU();
    cleanup();
}
```

As before, we place data in an array, but now we have broken the display process into two parts: storing the data on the GPU and displaying the data that have been stored. If we only have to display our data once, there is no advantage over our previous method; but if we want to animate the display, our data are already on the GPU and redisplay does not require any additional data transfer, only a simple function call that alters the location of some spatial data describing the objects that have moved.

## PROGRAMMING TWO-DIMENSIONAL APPLICATIONS

The three-dimensional coordinate system has been used by WebGL. Although, by setting the $z=0$. A coordinate is usually represented by a point $\textbf{p}=(x,y,z)$ or column vector $\textbf{p}=\begin{bmatrix} x & y & z\end{bmatrix}^\intercal$. In WebGL, the terms vertex and point are used in somewhat different ways. A vertex is a location in space; in computer graphics, we employ two-, three-, and four-dimensional spaces.

We want to start with a simple programme, so we'll place all of the data we want to show within a cube centred at the origin, with a diagonal running from $(1, 1, 1)$ to $(1, 1, 1)$. This is referred as as <b>clip coordinates</b>. Objects outside this cube will be removed or <b>clipped</b> and will not appear on the screen.

The programme might be written using a simple array of two items to hold the `x` and `y` values of each point. In JavaScript, we might create an array like this:

```js
var p = new Float32Array([x, y]);
var n = p.length;
```

`p` is just a contiguous array of standard 32-bit floating-point numbers. We can initialize an array component wise like,

```js
p[0] = x;
p[1] = y;
```

We can produce much cleaner code if we first define a two-dimensional point object and its actions. We developed such objects and methods and included them in the MV.js package. Numeric data is stored within these objects using JavaScript arrays.

The types in the OpenGL ES Shading Language (GLSL) that we use to construct our shaders correspond to the functions and three- and four-dimensional objects defined in MV.js. As a result, using MV.js should make all of our code examples clearer than if we had used regular JavaScript arrays. Although these functions were designed to be GLSL-compliant, because JavaScript does not provide operator overloading like C++ and GLSL, we wrote functions for arithmetic operations involving points, vectors, and other kinds. Nonetheless, code pieces written in MV.js, such as

```js
var a = vec2(1.0, 2.0);
var b = vec2(3.0, 4.0);
var c = add(a, b); // returns a new vec2
```

may be utilised in the programme and simply transformed into shader code. Individual components can be accessed via indexing, much like an array ( a[0] and a[1] ). The code below creates 5000 points by starting with the vertices of a triangle in the plane $z=0$:

```js
const numPoints = 5000;

var vertices = [vec2(-1.0, -1.0), vec2(0.0, 1.0), vec2(1.0, -1.0)];

var u = scale(0.5, add(vertices[0], vertices[1]));
var v = scale(0.5, add(vertices[0], vertices[2]));
var p = scale(0.5, add(u, v));

points = [p];

for (var i = 1; i < numPoints; ++i) {
    var j = Math.floor(Math.random() * 3);

    p = scale(0.5, add(points[i - 1], vertices[j]));
    points.push(p);
}
```

The python equivalent to this code is,

```py
import math
import random
from matplotlib import pyplot as plt


class vec2:

    def __init__(self, x: int | float, y: int | float) -> None:
        self.x = x
        self.y = y


def add(u: vec2, v: vec2) -> vec2:
    return vec2(u.x + v.x, u.y + v.y)


def scale(factor: int | float, u: vec2) -> vec2:
    return vec2(u.x * factor, u.y * factor)


numPoints = 5000

vertices = [vec2(-1.0, -1.0), vec2(0.0, 1.0), vec2(1.0, -1.0)]

u = scale(0.5, add(vertices[0], vertices[1]))
v = scale(0.5, add(vertices[0], vertices[2]))
p = scale(0.5, add(u, v))

points = [p]

for i in range(1, numPoints):
    j = math.floor(random.random() * 3)
    p = scale(0.5, add(points[i - 1], vertices[j]))
    points.append(p)

x = [u.x for u in points]
y = [u.y for u in points]

plt.title = f'SIERPINSKI GASKET as generated with {numPoints} random points'
plt.scatter(x, y, marker='.', color='black')
plt.tight_layout()
plt.show()
```