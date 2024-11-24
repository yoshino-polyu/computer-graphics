rasterization (光栅化): 把三维空间的几何形体显示在屏幕上 = project geometry primitives onto the screen.

- **Local coordinate system**: scenario=consider only single character/object. 我们自己 configure 选 origin, sphere-shaped objects 通常 modeled with their center at the origin. game character models usually with origin positioned at feet level, and y-axis running through center of the body.
- **World (global) coordinate system**: scenario=multiple characters (each with their own local coordination system) appear in single space. adjust position (world transformation) of each characters to avoid overlapping. 
![[Pasted image 20241124224201.png]]
- **view coordinate system**: eye point (camera position) at its origin; line of sight along negative z-axis; up direction in positive y-axis.

Model transformation: Find a good place and arrange people
matrix that performs model transformation = model matrix


View transformation: Find a good “angle” to put the camera
- View (视图) / Camera transformation
- Projection (投影) transformation
	- Orthographic (正交) projection
	- Perspective (透视) projection




Translate and Scale:
![[Pasted image 20241124234127.png]]

# WebGL
a WebGL program = a JS program executing in conjunction with shader programs.

4 ways a shader can receive data:
1. Attributes and Buffers
	1. Buffers = arrays of binary data you upload to the GPU; things like positions, normals, texture coordinates, vertex colors
	2. Attributes=specify how to pull data out of your buffers and **provide them to your vertex shader**. tell a particular attribute which buffer to pull the positions out of, what type of data it should pull out (3 component 32 bit floating point numbers), what offset in the buffer the positions start, and how many bytes to get from one position to the next.
2. Uniforms=global variables you set before you execute your shader program.
3. Textures=arrays of data you can randomly access in your shader program;
4. Varyings=a way for a vertex shader to pass data to a fragment shader.

WebGL only cares about 2 things: clip space coordinates and colors.
The majority of the WebGL API is about setting up state to supply data to our GLSL (GL Shader Language) programs.

bind points = internal global variables inside WebGL. bind a resource (e.g., position buffer) to a bind point; functions refer to the resource through the bind point. 
## clip space coordinates
Clip space coordinates always go from -1 to +1 no matter what size your canvas is.

## Vertex shader
A vertex shader's job is to compute vertex positions. Vertex shader provides the clip space coordinates.

## Fragment shader
 A fragment shader's job is to compute a color for each pixel of the primitive currently being drawn. Fragment shader provides the color.
 