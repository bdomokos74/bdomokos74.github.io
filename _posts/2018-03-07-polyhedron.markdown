
# Regular polyhedrons from paper

I wanted to create the five regular polihedrons from paper with my kids. They like to fold and glue paper, also they will study these at school at some point... So let's see how to make them from paper.



```python
import math
import numpy as np
import matplotlib.pyplot as plt
from pylab import rcParams
rcParams['figure.figsize'] = 15, 8
```

### Draw a tetrahedron net from triangles


```python
def draw_triangle(ax, x0, y0, r, alpha):
    delta = np.pi*2/3
    x = [x0]
    y = [y0]
    for i in range(1, 4):
        x.append(x[i-1]+r*np.cos(alpha))
        y.append(y[i-1]+r*np.sin(alpha))
        alpha += delta
    ax.plot(x, y, color='C0')
    return list(zip(x, y))

def draw_tetrahedron_net1(ax, x0, y0, r):
    r0 = draw_triangle(ax, x0, y0, r, np.pi/3)
    draw_triangle(ax, x0, y0, r, 0)
    draw_triangle(ax, x0, y0, r, -np.pi/3)
    draw_triangle(ax, r0[1][0], r0[1][1], r, -np.pi/3)

def draw_tetrahedron_net2(ax, x0, y0, r):
    draw_triangle(ax, x0, y0, r, np.pi/3)
    draw_triangle(ax, x0, y0, r, 0)
    r0 = draw_triangle(ax, x0, y0, r, -np.pi/3)
    draw_triangle(ax, r0[1][0], r0[1][1], r, 0)
    
_, ax_tetra = plt.subplots(1, 2)
for ax in ax_tetra:
    ax.set_aspect('equal', 'datalim')
draw_tetrahedron_net1(ax_tetra[0], 10, 10, 3)
draw_tetrahedron_net2(ax_tetra[1], 10, 10, 3)
plt.show()
```


![png](/assets/output_3_0.png)


###  Cube
The cube has 11 different nets, the here I only show two


```python
def draw_square(ax, x0, y0, a, alpha):
    delta = np.pi/2
    x = [x0]
    y = [y0]
    for i in range(1, 5):
        x.append(x[i-1]+a*np.cos(alpha))
        y.append(y[i-1]+a*np.sin(alpha))
        alpha = alpha-delta
    ax.plot(x, y, color='C0')
    return list(zip(x, y))

def draw_cube_net1(ax, x0, y0, a):
    x = x0
    y = y0
    for i in range(4):
        r = draw_square(ax, x, y, a, 0)
        x = r[3][0]
        y = r[3][1]
    draw_square(ax, x0-a, y0, a, 0)
    draw_square(ax, x0+a, y0, a, 0)
    
def draw_cube_net2(ax, x0, y0, a):
    x = x0
    y = y0
    for i in range(4):
        r = draw_square(ax, x, y, a, 0)
        x = r[3][0]
        y = r[3][1]
    draw_square(ax, x0-a, y0-a, a, 0)
    draw_square(ax, x0+a, y0, a, 0)
    
_, ax = plt.subplots(1, 1 )
ax.set_aspect('equal', 'datalim')
draw_cube_net1(ax, 10, 10, 5)
draw_cube_net2(ax, 30, 10, 5)
```


![png](/assets/output_5_0.png)


### Octahedron


```python
def draw_octahedron_net(ax, x0, y0, r):
    draw_triangle(ax, x0, y0, r, 2*np.pi/3)
    draw_triangle(ax, x0, y0, r, np.pi/3)
    draw_triangle(ax, x0, y0, r, 0)
    r0 = draw_triangle(ax, x0, y0, r, -np.pi/3)
    r1 = draw_triangle(ax, r0[1][0], r0[1][1], r, 0)
    draw_triangle(ax, r1[1][0], r1[1][1], r, np.pi/3)
    draw_triangle(ax, r1[1][0], r1[1][1], r, 0)
    draw_triangle(ax, r1[1][0], r1[1][1], r, -np.pi/3)
_, ax_octa = plt.subplots(1, 1)
ax_octa.set_aspect('equal', 'datalim')
draw_octahedron_net(ax_octa, 10, 10, 3)
plt.show()
```


![png](/assets/output_7_0.png)


### The dodecahedron can be put together from pentagons.
Plot the pentagon and half of the net.


```python
alpha = 360/5/360*2*math.pi

def calc_pentagon(*args, **kwargs):
    alpha = 360/5/360*2*np.pi
    if 'a' in kwargs:
        a = kwargs['a']
        m = a/(2*np.tan(alpha/2))
        r = m/np.cos(alpha/2)
    elif 'r' in kwargs:
        r = kwargs['r']
        a = 2*r*np.sin(alpha/2)
        m = a/(2*np.tan(alpha/2))
    else:
        raise Exception("Missing input a or r")
    u = np.sin(alpha)*r
    v = r-np.cos(alpha)*r
    w = r-v+m
    z = u-a/2
    b = 2*a*np.sin(alpha/4)
    c = 2*a+b
    return (a, b, c, r, u, v, w, z)

def print_pentagon(r):
    a, b, c, r, u, v, w, z = r
    print("a={:.02f}\nb={:.02f}\nc={:.02f}\nr={:.02f}\nu={:.02f}\nv={:.02f}\nw={:.02f}\nz={:.02f}".format(a, b, c, r, u, v, w, z))
    
def draw_pentagon(ax, x0, y0, a, **kwargs):
    ax.set_aspect('equal', 'datalim')
    alpha = 0
    backwards = 1;
    if 'backwards' in kwargs:
        backwards = -1
    delta = backwards*(360/5)/360*np.pi*2
    if 'alpha' in kwargs:
        alpha = float(kwargs['alpha'])
    x = [x0]
    y = [y0]
    arr_alpha = [alpha]
    for i in range(1, 6):
        x.append(x[i%5-1]+a*np.cos(alpha))
        y.append(y[i%5-1]+a*np.sin(alpha))
        alpha = alpha + delta
        arr_alpha.append(alpha)
    ax.plot(x, y)
    return list(zip([x[i] for i in range(5)], [y[i] for i in range(5)], [arr_alpha[i] for i in range(5)]))

def draw_pentagon_notation(ax):
    for i in range(5):
        ax.plot([10+3/2, ret[i][0]], [10+a/(2*np.tan(alpha/2)), ret[i][1]], "y--", linewidth=1)
    ax.plot([10-z, 10], [10, 10], 'y--', linewidth=1)
    ax.text(9.5, 9.8, 'z', fontdict={'size':18})
    ax.plot([10-z, 10-z], [10, 10+w], 'y--', linewidth=1)
    ax.text(9.2, 11, 'w', fontdict={'size':18})
    ax.plot([10-z, 10-z], [10+w, 10+w+v], 'y--', linewidth=1)
    ax.text(9.2, 13.8, 'v', fontdict={'size':18})
    ax.plot([10-z, 10+a/2], [10+w+v, 10+w+v], 'y--', linewidth=1)
    ax.text(10, 14.8, 'u', fontdict={'size':18})
    ax.text(10.6, 11.2, 'r', fontdict={'size':18})
    ax.text(12.9, 13.8, 'a', fontdict={'color': 'C0', 'size':18})

def draw_half_dodecaeder_net(ax):
    dots = plot_pentagon(ax_array[1], 50, 50, 10 , alpha=(2*18/360*2*np.pi))
    for i in range(1,6):
        d = dots[i%5]
        plot_pentagon(ax_array[1], d[0], d[1], 10, alpha=d[2], backwards=True)

_, ax_array = plt.subplots(1, 2, gridspec_kw = {'width_ratios':[1, 1]})

ax = ax_array[0]
ret = plot_pentagon(ax, 10, 10, 3)
draw_pentagon_notation(ax)
draw_half_dodecaeder_net(ax_array[1])
plt.show()

```


![png](/assets/output_9_0.png)


Calculate sizes of the net:


```python
ret = calc_pentagon(a=3)
a, b, c, r, u, v, w, z = ret

ret2 = calc_oct(a=c)
print_oct(ret2)
```

    a=7.85
    b=4.85
    c=20.56
    r=6.68
    u=6.35
    v=4.62
    w=7.47
    z=2.43



```python

def draw_icosahedron_net(ax, x0, y0, r):
    r0 = draw_triangle(ax, x0, y0, r, 0)
    draw_triangle(ax, x0, y0, r, -np.pi/3)
    r1 = draw_triangle(ax, x0, y0, r, -2*np.pi/3)
    #ax.plot([r1[1][0]], [r1[1][1]], 'ro') # draw a dot to find it
    draw_triangle(ax, r1[1][0], r1[1][1], r, -np.pi/3)
    return (r0[1][0], r0[1][1])

_, ax_icosa = plt.subplots(1, 1)
ax_icosa.set_aspect('equal', 'datalim')
start = (10, 10)
for i in range(5):
    start = draw_sub(ax_icosa, start[0], start[1], 3)
    
plt.show()
```


![png](/assets/output_12_0.png)


### Final result

![Final](/assets/poly.jpg)
