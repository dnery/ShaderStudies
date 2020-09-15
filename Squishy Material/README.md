# Squishy material breakdown

I wanted to replicate the effect you get when you press a squishy material against a glass window. Like pressing your
face against the glass of a fish tank, and seeing what it looks like from the inside. You'll see the material being
called "bouncy" throughout this note. Here's what it'll look like:

<img src="m_bouncy_demo1.gif" width="800"/>

The exercise here is basically displacing vertices, so I've broken down the task into 2 parts:
*   An **orthogonal** (in relation to the instigator plane) displacement factor
*   A **parallel** (in relation to the instigator plane) displacement factor

<img src="m_bouncy_guide1.jpg" width="800"/>

These factors are expressed as vectors and in the end we'll add them both.

## Orthogonal displacement


#### Quick maths

The 2 things we need answered to build this effect are:
*   Has our object intersected with the instigator plane yet?
*   How much has our object clipped through the plane? (For each "particle" of our object)

<img src="m_bouncy_guide2.png" width="800"/>

Well, first we begin with a couple of truths: **vector projection** and **dot product**, and make a simple deduction to
get the length of a projection without knowing the angle between both vectors involved:

<img src="m_bouncy_guide3.png" width="800"/>

Applying that to our case, here's how we obtain the **shortest distance** between a particle (or vertex) in our object's
surface and the instigator plane:

<img src="m_bouncy_guide4.png" width="800"/>

In fact, since we derive our distance from a dot product, we can also tell when our object's crossed the plane
boundaries just by looking at the sign of the value:

<img src="m_bouncy_guide5.png" width="800"/>

This gives us the numerical amount of displacement we'll need. To get a direction, we just multiply it by the plane
normal; since we only want to displace when clipping through the plane, this is the direction that takes our vertex
back towards the boundary. We're ready to write the shader.


#### Material graph

Highlighted are the nodes related to operations I didn't exactly talk about so far:

<img src="m_bouncy_guide6.jpg" width="800"/>

1.  Since it's up to us to provide a vector of length 1, we use `Normalize` just to make sure
2.  Since we'll only displace when clipping through (negative value), we do a `Min` with `0`, so the vertex won't be
affected when it's not touching the plane
3.  It's important to know the sign (`+` or `-`) to identify when clipping happens, but the displacement must be
positive (*towards* some direction, not *away* from it) so we do an `Abs`
4.  As mentioned before, the `Dot` operation gave us a scalar distance value, but we're displacing a vertex in 3D space
and we need a proper 3D vector or direction, so we `Multiply` with the plane normal, which is exactly the direction we
want to go back to when clipping through the plane

Create a material instance from the base material we have so far so that you can customize `V3_PlaneNormal` and
`V3_PlanePoint` more easily. This is what we have:

<img src="m_bouncy_demo2.gif" width="800"/>

You can see we're impeding the vertices from clipping through the plane, they kinda "squish" or "bounce off" the surface
which is exactly what we want. To complete the effect we also need some spread, so onto the next part.


## Parallel displacement addition


#### Quick maths

To "squish" the surface we also need to "spread" it, and we probably want to do it from the center of our object towards
the edges of the contact area. For our editor sphere, this center can be the actor position. We project both surface and
center vectors onto the plane and do a subtraction to get the direction `T`:

<img src="m_bouncy_guide7.png" width="800"/>

Just remember that both `D1` and `D2` are exactly the distances we learned how to calculate in the previous section.


#### Quick maths

So we know how to get the direction for this displacement, but how about the amount? Well, I used a value proportional
to the amount we displaced the vertex orthogonally to the plane in the previous section. I just didn't want it to be
*linearly* proportional, so I used a `Sqrt` and multiplied by a scalar parameter that I can use to control the effect
through the material instance (click to expand):

<img src="m_bouncy_guide8.jpg" width="800"/>

The graph looks considerably bigger, but not at all more complicated:

1.  This is most of the math for calculating the projected points; nothing new here, just maybe more redundant
2.  This is what I use to control the parallel displacement amount; **attenuate** the amount obtained from the
previous orthogonal displacement, and **multiply** by a scalar param that I can tweak in the editor

So here's what we got, finally:

<img src="m_bouncy_demo3.gif" width="800"/>

You can see that the "spreading" is not really physically accurate, and one way to improve it a little would be to
probably start deforming the vertices *slightly before* reaching the plane surface. Also, in a more realistic scenario,
any part of the sphere that sticks to the plane shouldn't "slide" across the surface after contact like ours does.
Still, this was some great practice and I'm calling it a wrap for now.


## Notes

*   The model used is a simple editor sphere, you can find it in the `Modes` plane
*   The wall or "instigator plane" is a simple editor box, stretched and squeezed, with a default **wireframe
material,** I just made it two-sided
*   The texture I sampled in the shader is a simple checker texture I found on the web, I'm including the PSD file here
and you can import it directly into unreal
*   A natural step forward for this is **making the plane dynamic,** that is, setting the point and normal parameters
during runtime; it's mostly actor behavior code so I didn't cover it here, but **you can set material parameters during
runtime** and if you feel willing, it's an excellent exercise
