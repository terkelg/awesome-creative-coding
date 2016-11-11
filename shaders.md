# Shaders
> The dark magic – master this, and you rule the world

## Definitions
### GLSL
GLSL is the shader language used for WebGL.

### GLSL Fragment shaders
Fragment shaders are responsible for giving each pixel their color.
Fragment shaders are run on each pixel: they're not at all aware of their surrounding pixels. This is a challenging constraint at times, but also why they're so speedy.

Fragment shaders also don't have much to use to find out where they are on the screen either. gl_FragCoord.xy will give you the exact position in pixels, but if you resize the screen the size of the object won't change to fit it.

So we pass in a uniform value that contains the size, or resolution, of the screen. A uniform is a value passed in from JavaScript that is the same for every pixel in the shader. It's useful for giving the fragment shader some context to work with: for example, you might also pass in the time in seconds to animate the output.

By dividing gl_FragCoord.xy by iResolution, we can get a value between 0 and 1 for the pixel's position on the screen:
```
vec2 p = gl_FragCoord.xy / iResolution;
```

### GLSL Vertex Shaders 

### Swizzling
Beyond being a great name — is a nice feature in GLSL for accessing the properties of a vector.
You can get a single float from a vector using .r, .g, .b or .a. For example:

```
vec4(1, 2, 3, 4).r == 1.0
vec4(1, 2, 3, 4).g == 2.0
vec4(1, 2, 3, 4).b == 3.0
vec4(1, 2, 3, 4).a == 4.0
```

But you can also create new vectors from combinations of their components like so:
```
vec4(1, 2, 3, 4).rb == vec3(1, 3)
vec4(1, 2, 3, 4).rgg == vec3(1, 2, 2)
vec4(1, 2, 3, 4).ggab == vec3(2, 2, 4, 3)
```
In addition to .rgba, you can also use .xyzw. These are equivalent, but if you're using the vector for a position instead of a color it's easy to reason about when using the latter.

### Finding the midpoint
The midpoint is just the average of two values, e.g. ```(x1 + x2) / 2```
To calculate the midpoint of a vector, you just have to calculate the midpoint of each of its values:
```
vec2((p1.x + p2.x) / 2.0, (p1.y + p2.y) / 2.0);
```
*2.0 because there's two values, remember average calculations*

That's a little verbose though. Can we make it shorter?

### Piecewise Operations
You can treat vectors a little like normal numbers: they can be added, multiplied, divided and subtracted just the same in GLSL!
```
p1 + p2 == vec2(p1.x + p2.x, p1.y + p2.y)
p1 - p2 == vec2(p1.x - p2.x, p1.y - p2.y)
p1 * p2 == vec2(p1.x * p2.x, p1.y * p2.y)
p1 / p2 == vec2(p1.x / p2.x, p1.y / p2.y)
```
This is called a piecewise operation, because it is applied to each piece of the vector individually.

You can even apply a piecewise operation to a vector using float, e.g.:
```
p1 + 1.0 == vec2(p1.x + 1.0, p2.x + 1.0)
p1 - 1.0 == vec2(p1.x - 1.0, p2.x - 1.0)
p1 * 5.0 == vec2(p1.x * 5.0, p2.x * 5.0)
p1 / 5.0 == vec2(p1.x / 5.0, p2.x / 5.0)
```

So to get the midpoint without using vec2 or any swizzling:
```
vec2 midpoint(vec2 p1, vec2 p2) {
  return p1 / 2.0 + p2 / 2.0;
}
```

## Boolean Comparisons
You can compare float values (but not vectors) using run-of-the-mill boolean operators, e.g.:
```
bool value = uv.x > 1.05;
bool value = uv.x <= 0.0;
bool value = uv.x != 0.5;
bool value = uv.y != 0.0 && uv.x != 0.0;
bool value = uv.y != 0.0 || uv.x != 0.0;
```

## Distance / length
You can check if a point is in a circle by seeing if its distance from the circle's center is smaller than the circle's radius. In GLSL we measure distance using the length() function:
```
float d = length(p2 - p1)
```
Much more concise than it would be in JavaScript!
All that's left to do is compare that distance to the radius and you should be able to draw that circle out to the screen.

## Signed Distance Functions (SDF)
http://jamie-wong.com/2016/07/15/ray-marching-signed-distance-functions/

Yep, those words probably make a lot less sense when they're put together like that.

To start: a **distance function** takes a point as input and returns the distance from a surface as output. In GLSL, you'd mark up a 2D Distance Function like so: ```float distanceFn(vec2 position);```

A signed distance function (SDF) is very similar, but it returns a negative value when it's inside the surface. You can now very quickly draw out that shape in 2D by checking if the SDF's value is less than zero.

## Smooth minimum
A smooth minimum function takes two values as input, and returns a smoothed out value between the two.
There's different types of smooth minimum with different tradeoffs, but here's a few taken from the ever-useful (iquilezles.org)[http://iquilezles.org/www/articles/smin/smin.htm]:

EXPONENTIAL
```
float smin( float a, float b, float k )
{
    float res = exp( -k*a ) + exp( -k*b );
    return -log( res )/k;
}
```

POLYNOMIAL
```
float smin( float a, float b, float k )
{
    float h = clamp( 0.5+0.5*(b-a)/k, 0.0, 1.0 );
    return mix( b, a, h ) - k*h*(1.0-h);
}
```

POWER
```
float smin( float a, float b, float k )
{
    a = pow( a, k ); b = pow( b, k );
    return pow( (a*b)/(a+b), 1.0/k );
}
```

# Links
- [jam3-lesson-webgl-shader-intro](https://github.com/Jam3/jam3-lesson-webgl-shader-intro)
- [jam3-lesson-webgl-shader-threejs](https://github.com/Jam3/jam3-lesson-webgl-shader-threejs)

# Tutorials
- https://github.com/stackgl/shader-school



"While a CPU has 2–8 big cores, a GPU has hundreds or even thousands of small ones. This makes it great at running code in parallel: provided a thread doesn't need to know anything about its neighbours, you can run a whole bunch of them really quickly at the same time without waiting for the others to finish."

Anki: Difference between CPU and GPU: 
