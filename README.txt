Documentation Author: Niko Procopi 2019

This tutorial was designed for Visual Studio 2017 / 2019
If the solution does not compile, retarget the solution
to a different version of the Windows SDK. If you do not
have any version of the Windows SDK, it can be installed
from the Visual Studio Installer Tool

Welcome to the Loop Reflection Tutorial!
Prerequesites: Basic Reflection

The main focus of this tutorial is to change the
addReflectionToPixColor function, to bounce many times.
This allows us to have reflections, of reflections, of
reflections... as many times as we want.

First, we add a parameter called "maxBounces". This
is the maximum number of times a ray can bounce, but
it is not guranteed that a ray will bounce this many 
times. The ray will stop bouncing when it stops hitting
geometry, or when it reaches the maximum number of bounces

We make a variable for the reflected vector that goes
to a point (either from eye, or from other geometry)
	vec3 reflectedRayToPoint;
	
We make a hitinfo for what is hit
	hitinfo reflectHit;
	
We create a final color that will eventually be
returned. This color holds all reflections
	vec3 color = vec3(0);
	
We loop through all the bounces
	for(int i = 0; i < maxBounces; i++)
	
If maxBounes is set to zero, and if this loop is never
entered, then the function will return 0, for black,
which means no reflection

First, we reflect the ray that hit the geometry.
The first time we enter this loop, this will be 
the ray from the eye to the geometry. With each 
iteration of the loop, it will be a ray from 
geometry to other geometry
	reflectedRayToPoint = reflect(dir, triangles[rayHitPoint.index].normal);
	
After reflecting, check if more geometry is hit
	if(intersectTriangles(rayHitPoint.point, reflectedRayToPoint, reflectHit))
	
If geometry was hit, then render the pixel that was hit,
and then continue bouncing to see if more pixels are hit, 
then render those, and continue the pattern until the loop ends
	color += addLightColorToPixColor(lightPos, reflectedRayToPoint, reflectHit, lightIntensity);

If no geomtry was hit when the ray bounced, then it means
our ray went into the black void, or in future tutorials,
maybe it doesn't hit other geometry due to the geometry
being too close or too far. When this happens, we exit
the loop, with the "break" command.

After the loop, we return the final color
	
Lastly, we go into our trace() function where we call
addReflectionToPixColor, and we add one parameter,
which determines the maximum number of bounces that
a ray can have. It is not guarranteed that a ray will
bounce this many times (before leaving the scene into
the black void), but this is the maximum number possible.
It can be set to any number: 0, 5, 20, 100, anything

How to improve:

Uniform buffers
	Keep the triangles array, make it empty
	Put the plane, and each cube in a seperate uniform buffer
	Give them each a model matrix
	Use a compute shader to move them, each with their own model matrix
	Have the compute shader write to the triangles array, just like
	how the compute shader wrote the the vertex buffer in Basic Compute Particles
	
More Optimization:
	Give each model a spherical radius, so that rays check for collision
	with the spheres, before checking for collision with each polygon that
	is inside the sphere. After this, keep doing more spatial partitioning

Textures
	Between "float lightIntensity = 6.0;" and "vec3 pixColor = ..."

	i.point is the 3D coordinate that a ray hits a triangle
	triangles[i.index].a is one point on the triangle
	triangles[i.index].b is one point on the triangle
	triangles[i.index].c is one point on the triangle
	UV coordinates dont exist yet
	Use barycentric coordinates to get (a+b+c=1) variables
	to compare points of triangle to point intersecting
	Then use (a+b+c=1) to compare points of triangle's UVs
		to get the UV coordinate of the point you hit
	For Debugging, do pixColor = uv * 0.1
		Then do vec3 pixColor = sample(texture, uv) * 0.1
	However
		This will not change the color of squares in reflection
		triangles[eyeHitPoint.index].color needs to be replaced with 
		reflectHit.point (like i.point) to calculate UVs all over again
		
Skybox
	On lines 274, in Reflection FragmentShader.glsl
	When you see if(intersectTriangles ...
	Should have "else ..." to get skybox color
	We already have direction, we just need color from cubemap
	We do NOT change the if(intersectTriangles ... at line 241,
	because that is for shadows
	On line 346, change "return vec4(0,0,0,1)" to 
	return the skybox color. This will be the actual background
	wallpaper skybox