rasterization (光栅化): 把三维空间的几何形体显示在屏幕上 = project geometry primitives onto the screen.
![[Pasted image 20241124224201.png]]
- **view coordinate system**: eye point (camera position) at its origin; line of sight along negative z-axis; up direction in positive y-axis.

Model transformation: Find a good place and arrange people
Matrix that performs model transformation = model matrix

model matrix (定义本来的样子和所在的位置), 
projection matrix (选择投影)选择透视投影或者正交投影
view matrix (定义 camera 所在的位置和看的方向)


Translate and Scale:
![[Pasted image 20241124234127.png]]

# WebGL
a WebGL program = a JS program executing in conjunction with shader programs.
The majority of the WebGL API is 
- about setting up state to supply data to our GLSL (GL Shader Language) programs.
- just a rasterizer
WebGL Shader Model:
![[Pasted image 20241206144938.png]]
VBOs: Vertex Buffer Objects = Store positions, normals, indices

4 ways a shader can receive data:
1. Attributes and Buffers
	1. Buffers = arrays of **binary data** you upload to the GPU; way of getting vertex and other per vertex data onto GPU. Things like positions, normals, texture coordinates, vertex colors; 
	2. Attributes=specify how to pull data out of your buffers and **provide them to your vertex shader**. tell a particular attribute which buffer to pull the positions out of, what type of data it should pull out (3 component 32 bit floating point numbers), what offset in the buffer the positions start, and how many bytes to get from one position to the next.
2. Uniforms=global variables you set before you execute your shader program.
3. Textures=arrays of data you can randomly access in your shader program;
4. Varyings=a way for a vertex shader to pass data to a fragment shader.

WebGL only cares about 2 things: clip space coordinates and colors.
![[Pasted image 20241125115342.png]]
Clip space in `x` goes from -1 to +1. That means 0 is in the center and positive values will be to the right of that.

bind points = internal global variables inside WebGL. bind a resource (e.g., position buffer) to a bind point; functions refer to the resource through the bind point.
`gl_Position` and `gl_FragColor` are defined in GLSL. 
## Vertex shader
- Per vertex normal, texture, lighting, color. 
- Responsible for all vertex coordinate transformations; Determine final vertex position. 


## Fragment shader
 - operates on each pixel
 - compute final color of each pixel
 - performs texture lookup
 - discards fragments; sets fragment color.


## Buffer
`gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);` binds positionBuffer to ARRAY_BUFFER bind point. 
## space transformations

LocalSpace -> WorldSpace -> ViewSpace -> ClipSpace -> Screen Space
![[Pasted image 20241128154329.png]]
The chain of matrix transformations (allows to) is used to provide these changes. 
The transformations that we can use in vector spaces are scale, translation and rotation.
### transformation matrix
Base case:
![[Pasted image 20241128163759.png]]
![[Pasted image 20241128163822.png]]
![[Pasted image 20241128165042.png]]
![[Pasted image 20241128165201.png]]
![[Pasted image 20241128165413.png]]


matrix multiplication is not commutative: different order to apply transformations => different results. 
chain several transformations together by multiplying matrices one after the other = single matrix that encodes the full transformation.
Since we are using column vectors we will have to read a chain of transformation right to left. 

_rotated around the Y axis for 90° clockwise_, then _rotated 180° around the X axis_, and then _translated into (1.5, 1, 1.5)_. the transformation matrix will be:
![[Pasted image 20241128170119.png]]
In World Space the X axis is now oriented as the Z axis of that space therefore it's now (0,0,1). The Y axis is now flipped upside down, hence (0,-1,0). The Z axis is now oriented as the X axis, (1,0,0).
explain why X axis change to (0,0,1): 首先按照原来的 Y axis 顺时针转 90 就会转到原来的 Z axis 上. 然后按照 X axis 的180度 rotate 就和 X axis 没关系了
why Y -> (0,-1,0): 1) Y axis rotation 不影响 2) X axis rotation: X axis 已经变到原来的 Z axis 上了, 所以 Y -> -Y
why Z -> (1,0,0): 1) Y axis rotation: 变到之前的 -X 2) X axis rotation: 因为此时 X axis 已经在原先的 Z, 所以 Z 又会绕着新的 X 转, 转到原来的 X 的位置

## local space
又叫做 model space
**Local coordinate system**: scenario=consider only single character/object. 我们自己 configure 选 origin, sphere-shaped objects 通常 modeled with their center at the origin. game character models usually with origin positioned at feet level, and y-axis running through center of the body.
## world space
Since every object will be in its own position and orientation in the world, every one has a different Model to World transformation matrix.

**World (global) coordinate system**: scenario=multiple characters (each with their own local coordination system) appear in single space. adjust position (world transformation) of each characters to avoid overlapping.
When all the objects have been transformed into a common space (the World Space) their vertices will be relative to the World Space itself.

Scene: a scene is made up of a collection of objects specified in world coordinates
![[Pasted image 20241128190126.png]]


## view space
With all the objects at the right place in world space we now need to project them to the screen.
view (eye) space = position in local camera space

## clip space
clip space 又叫做 Projection Space.
coordinates in clip space are called window coordinates.

Normalized Device Coordinates (NDC): This is a coordinate system used in the pipeline of rendering 3D graphics. NDCs are a standardized way to represent coordinates so that they are **device-independent**. Coordinates in this system are typically defined within a range (like 1 to 1 for both x and y).

clip space = what you get when you apply projection matrix.
Orthographic (正交) projection & Perspective (透视) projection. 
Clip space coordinates always go from -1 to +1 no matter what size your canvas is. clipping window is used to select the part of the scene that is to be displayed. 
![[Pasted image 20241125122250.png]]
Vertex shader gets called once for each vertex. store clip space value of current vertex into `gl_Position`. GPU takes those values and stores them internally. 

### Clip Lines in 2D Window Coordinates
Brute Force:
![[Pasted image 20241128190459.png]]
![[Pasted image 20241128190640.png]]
problems of Brute Force: calc line intersections is computationally expensive

### Polygon Clipping
Sutherland-Hodgman: 
![[Pasted image 20241205231654.png]]


## screen space
screen space = clip space after rasterization. 
coordinates in screen space are called device coordinates.
clip space to screen space
![[Pasted image 20241125120755.png]]
Window: A world-coordinate area selected for display. 
Viewport: An area on a display device to which a window is mapped.
![[Pasted image 20241128184657.png]]
![[Pasted image 20241128185418.png]]


## homogeneous coordinates
property of homogeneous coordinates:
![[Pasted image 20241205235835.png]]

## Projections

边缘不直接投影，边缘通过插值填充（Edges are not projected. Edges are filled in by interpolation）.
Perspective projection is in viewing transformation. In order to find this transformation. Find relationship between transformed points (Q) and original points (P). 
![[Pasted image 20241206000317.png]]
## Graphics Pipeline
![[Pasted image 20241206113051.png]]
To install and use a shader in OpenGL:
![[Pasted image 20241206144153.png]]

## webgl dataflow
Graphics primitives include: lines, circles, arcs, rectangles, etc.
### GLSL
subset of C++ language, case sensitive, no char/string types. 
`uniform`: constant across an entire primitive
`attribute`: VS per vertex information
`varying`: VS write, FS read qualifier
`gl_Position`: type=vec4, description=vertex position
`gl_FragColor`: type=vec4, description=final fragment color

## Texture Mapping
**Texture** – an image meant for the storage of some information that is later mapped to an object.
An image mapped to geometry is called a texture.

Textures 
- represent variations in reflected spectral light over a surface 
- color patterns that represent 
	- complex surface geometry 
	- complex surface composition 
- Textures add realism to simple geometry

Texturing Surface: 
![[Pasted image 20241206174648.png]]
### Backward Mapping
![[Pasted image 20241206211553.png]]

### 3D Texture Map



### Interpolation Across Triangle: Barycentric coordinates
**Interpolation** – process of finding a previously unknown value between a number of known values.

重⼼坐标
motivation of interpolation:
- specify values at vertices
- Obtain smoothly varying values across triangles
to interpolate, we need:
- texture coordinates
- colors
- normal vectors
interpolate by barycentric coordinates. 

![[Pasted image 20241206180936.png]]


![[Pasted image 20241206181202.png]]

![[Pasted image 20241206181953.png]]

![[Pasted image 20241206182013.png]]

![[Pasted image 20241206182730.png]]
### Sampling Texture
very useful references: https://cglearn.eu/pub/computer-graphics/textures-and-sampling
That is why we would rather have an image mapped to our object and pick the colors from that image. Images can be regarded as mathematical functions and in fact you do not have to use an image, but could just sample some interesting function. We will see those kinds of techniques in the Procedural Generation topic, because those functions are usually generated to produce some random-looking but specific pattern. Here however we will see how can we use just an image (that you can draw in an image editing program for example) as the storage of color information on some object's surface.
**Sampling** – process of getting a finite number of values from a function, map, image.

Bilinear Interpolation:
![[Pasted image 20241206224442.png]]Mipmap:
property utilized: texture images must be a power of 2. 
![[Pasted image 20241206224923.png]]
