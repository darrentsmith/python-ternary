# python-ternary

This is a plotting library for use with [matplotlib](http://matplotlib.org/index.html) to make [ternary plots](http://en.wikipedia.org/wiki/Ternary_plot)
plots in the two dimensional simplex projected onto a two dimensional plane.

The library provides functions for plotting projected lines, curves (trajectories), scatter plots, and heatmaps. There are [several examples](https://github.com/marcharper/python-ternary/blob/master/examples.py) and a short tutorial below.

Most ternary functions expect the simplex to be partititioned into some number of steps, determined by the scale parameter. A few functions will do this partitioning for you, but when working with real data or simulation output, you may have partitioned already. if you are working with probability distributions, just use `scale=1`. Otherwise the scale parameter effectively controls the resolution of many plot types (e.g. heatmaps).

# Basic Plotting Functions

The easiest way to use python-ternary is with the wrapper class `TernaryAxesSubplot`,
which mimics Matplotlib's AxesSubplot. Start with

```
scale = 20
figure, ternary_ax = ternary.figure(scale=scale)
```

You can also wrap a matplotlib AxesSubplot object:

```
figure, ax = pyplot.subplots()
ternary_ax = TernaryAxesSubplot(ax=ax)
```

With `ternary_ax` you can use many of the usual matplotlib functions:

```
figure, ternary_ax = ternary.figure(scale=scale)
ternary_ax.set_title("Scatter Plot", fontsize=20)
ternary_ax.scatter(points, marker='s', color='red', label="Red Squares")
ternary_ax.legend()
```

If you need to act directly on the underyling matplotlib axes, you can access them:

```
ax = ternary_ax.get_axes()
```

Most drawing functions can take standard matplotlib keyword arguments such as [linestyle](http://matplotlib.org/api/lines_api.html#matplotlib.lines.Line2D.set_linestyle) and linewidth.

## Simplex Boundary and Gridlines

The following code draws a boundary for the simplex and gridlines.

![Ternary Plot -- Boundary and Gridlines](https://raw.githubusercontent.com/marcharper/python-ternary/images/readme_images/boundary_and_gridlines.png)

```
from matplotlib import pyplot
import ternary

## Boundary and Gridlines
scale = 40
figure, ternary_ax = ternary.figure(scale=scale)

# Draw Boundary and Gridlines
ternary_ax.boundary(color="black", linewidth=2.0)
ternary_ax.gridlines(color="blue", multiple=5)

# Set Axis labels and Title
fontsize = 20
ternary_ax.set_title("Simplex Boundary and Gridlines", fontsize=fontsize)
ternary_ax.left_axis_label("Left label $\\alpha^2$", fontsize=fontsize)
ternary_ax.right_axis_label("Right label $\\beta^2$", fontsize=fontsize)
ternary_ax.bottom_axis_label("Bottom label $\\Gamma - \\Omega$", fontsize=fontsize)

# Remove default Matplotlib Axes
ternary_ax.clear_matplotlib_ticks()

pyplot.show()
```

## Drawing lines

You can draw individual lines between any two points with draw_line and lines parallel to the axes with draw_horizonal_line, draw_left_parallel_line, and draw_right_parallel_line:

```
import ternary

scale = 40
figure, ternary_ax = ternary.figure(scale=scale)

# Draw Boundary and Gridlines
ternary_ax.boundary(color="black", linewidth=2.0)
ternary_ax.gridlines(color="blue", multiple=5)

# Set Axis labels and Title
fontsize = 20
ternary_ax.set_title("Various Lines", fontsize=20)
ternary_ax.left_axis_label("Left label $\\alpha^2$", fontsize=fontsize)
ternary_ax.right_axis_label("Right label $\\beta^2$", fontsize=fontsize)
ternary_ax.bottom_axis_label("Bottom label $\\Gamma - \\Omega$", fontsize=fontsize)

# Draw lines parallel to the axes
ternary_ax.horizontal_line(16)
ternary_ax.left_parallel_line(10, linewidth=2., color='red', linestyle="--")
ternary_ax.right_parallel_line(20, linewidth=3., color='blue')
# Draw an arbitrary line
p1 = ternary.project_point((12,8,10))
p2 = ternary.project_point((2, 26, 2))
ternary_ax.line(p1, p2, linewidth=3., marker='s', color='green', linestyle=":")

ternary.show()
```

The line drawing functions accept the matplotlib keyword arguments of [Line2D](http://matplotlib.org/api/lines_api.html)

![Ternary Plot -- Various Lines](https://raw.githubusercontent.com/marcharper/python-ternary/images/readme_images/various_lines.png)

## Curves

Curves can be plotted by specifying the points of the curve, just like matplotlib's plot. Simply use:

```
ternary.plot(points)
```

Points is a list of tuples or numpy arrays, e.g. [(0.5, 0.25, 0.25), (1./3, 1./3, 1./3)]. Ternary assumes that the points are probability distributions (e.g. x+y+z=1) unless you specify otherwise. Again you can specify axes and line options:

```
import ternary

## Sample trajectory plot
figure, tax = ternary.figure(scale=1.0)
tax.boundary(color='black')
tax.set_title("Plotting of sample trajectory data", fontsize=20)
points = []
with open("curve.txt") as handle:
    for line in handle:
        points.append(map(float, line.split(' ')))
tax.gridlines(multiple=0.2, color="black")
tax.plot(points, linewidth=2.0, label="Curve")
tax.legend()
tax.show()
```

![Ternary Curve Plot](https://raw.githubusercontent.com/marcharper/python-ternary/images/readme_images/trajectory.png)

There are many more examples in [this paper](http://arxiv.org/abs/1210.5539).

## Scatter Plots

Similarly, ternary can make scatter plots:

```
import ternary

### Scatter Plot
scale = 40
figure, ternary_ax = ternary.figure(scale=scale)
ternary_ax.set_title("Scatter Plot", fontsize=20)
ternary_ax.boundary(color="black", linewidth=2.0)
ternary_ax.gridlines(multiple=5, color="blue")
# Plot a few different styles with a legend
points = random_points(30, scale=scale)
ternary_ax.scatter(points, marker='s', color='red', label="Red Squares")
points = random_points(30, scale=scale)
ternary_ax.scatter(points, marker='D', color='green', label="Green Diamonds")
ternary_ax.legend()

ternary_ax.show()

```

![Ternary Scatter Plot Example](https://raw.githubusercontent.com/marcharper/python-ternary/images/readme_images/scatter.png)

## Heatmaps

Ternary can plot heatmaps in two ways and two styles. Given a function, ternary will evaluate the function at the specified number of steps (scale). The simplex can be split up into triangles or hexagons (thanks to contributor btweinstein for the hexagonal heatmap functionality). There is a large set of examples [here](http://people.mbi.ucla.edu/marcharper/stationary_stable/3x3/incentive.html).

For example:

```
def shannon_entropy(p):
    """Computes the Shannon Entropy at a distribution in the simplex."""
    s = 0.
    for i in range(len(p)):
        try:
            s += p[i] * math.log(p[i])
        except ValueError:
            continue
    return -1.*s
```

We can get a heatmap of this function as follows:

```
import ternary
scale = 60

figure, ternary_ax = ternary.figure(scale=scale)
ternary_ax.set_title("Scatter Plot", fontsize=20)

ternary_ax.heatmap_of_function(shannon_entropy, boundary=True, style="triangular")
ternary_ax.boundary(linewidth=2.0)
ternary_ax.set_title("Shannon Entropy Heatmap")

ternary_ax.show()
```

In this case the keyword argument *boundary* indicates whether you wish to evaluate points on the boundary of the partition (which is sometimes undesirable). Specify `style="hexagonal"` for hexagons. Large scalings can use a lot of RAM.

You may specify a [matplotlib colormap](http://matplotlib.org/examples/color/colormaps_reference.html) in the cmap_name argument.

![Ternary Heatmap Examples](https://raw.githubusercontent.com/marcharper/python-ternary/images/readme_images/heatmap_shannon.png

Ternary can also take a dictionary mapping `(i,j) for i + j + k = scale` to a float as input for a heatmap, using the function

```
ternary.heatmap(d, scale, cmap_name=None, axes_subplot=None, scientific=True)
```

![](https://camo.githubusercontent.com/30fb63ec53deb0fda2c892c0732a97620699500b/687474703a2f2f692e696d6775722e636f6d2f64555a6b3355302e6a7067)

[](https://camo.githubusercontent.com/b66c280914cb4a38130b83a3eb4311f94274aefb/687474703a2f2f692e696d6775722e636f6d2f6935516a5147542e6a7067)



# Test

You can run the test suite as follows:

```
python -m unittest discover ternary
```

# Citation

Please cite as follows:

```
Marc Harper, Python-ternary: A python library for ternary plots, 2011-2015, available at: https://github.com/marcharper/python-ternary
```

# Contributors

- Marc Harper [marcharper](https://github.com/marcharper)
- Bryan Weinstein [btweinstein](https://github.com/btweinstein): Hexagonal heatmaps

