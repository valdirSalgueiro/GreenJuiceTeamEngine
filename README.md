# Green juice team engine
Homemade cross platform Open GLES 2.0 engine codenamed "greenjuice team engine". Its C based syntax for the core provides simple of use but still powerful API.

# Current supported platforms
![alt](https://raw.githubusercontent.com/valdirSalgueiro/sgsCrossPlatform/master/promotion/dl_android.png)
![alt](https://raw.githubusercontent.com/valdirSalgueiro/sgsCrossPlatform/master/promotion/dl_apple.png)
![alt](https://raw.githubusercontent.com/valdirSalgueiro/sgsCrossPlatform/master/promotion/dl_html5.gif)
![alt](https://raw.githubusercontent.com/valdirSalgueiro/sgsCrossPlatform/master/promotion/dl_uwp.png)
![alt](https://raw.githubusercontent.com/valdirSalgueiro/sgsCrossPlatform/master/promotion/xbox-icon-21.png)

# Quick start

## Requirements
Visual studio 2017 with c++ and mobile native modules installed

## Project structure 

The Visual studio 2017 solution file comes with projects for android, UWP(Windows and xbox) and ios platforms. These projects contain specifics for each platform. There is also a project that contains shared content for all these projects **"GreenJuiceTeam.Shared"** where you should put your game logic. It already contains some libraries to ease some common activities like font rendering, sound and low level rendering. 

The shared project contains a *Engine.cpp* class that we will use to build our first example. These class is instantiated in each of the platform example.

## Drawing a sprite

Engine cpp contains 3 main methods:
1. init() 
  Contains all your initialization code here.
2. update()
  Contains frame update logic.
3. render()
  Contains frame render logic.
  
Lets draw the green juice team logo just for fun. First lets load the logo image file (for now only PNG support is given) inside our *init* method.

```cpp
void Engine::init(int width_, int height_)
{
	startBatch(width_, height_); //important! need for rendering
	logo = new glImage(); //initialize our image object
	glLoadSprite("logo.png", logo); //load png file, important: files must be power of two images!
}
```

now we can render it at (0,0) in our render method:

```cpp
void Engine::render(float time) {
	spriteBatchBegin();
	glSprite(0, 0, GL2D_NONE, logo); // glsprite(x,y,MODE,image)
	spriteBatchEnd();
}
```

and thats it! :)

To see it running, set one of the application platforms(UWP, IOS or android) as your initiliazing project. Make sure  the target is set to x64!

# Low level 

This section is not necessary to develop games with the library, but still some good to know information. In most games you wont need to tinker with this kind of low level.

For (hopefully) more efficient sprite rendering the engine uses batch rendering.

Here are the main shaders:

fragment shader:
```glsl
uniform sampler2D sampler2d;
varying lowp vec4 vertexColor;
varying lowp vec2 texCoord;

void main(void)
{
	gl_FragColor = texture2D(sampler2d, texCoord)*vertexColor;
}
```

Here, nothing very special, just your common "texture" fragment shader. Things get a little dicey with the vertex shader though

```glsl
uniform mediump mat4 projMatrix;
uniform mediump mat4 im[16];         // input
varying mediump vec2 texCoord;
varying mediump vec4 vertexColor;

void main(void)
{
	mediump mat4 thisim = im[int(vertex.z)]; //input
	highp vec2 transVertex = vec2(thisim[2][0], thisim[2][1]) + mat2(thisim[0][0], thisim[0][1], thisim[1][0], thisim[1][1]) * (vertex.xy - vec2(thisim[2][2] - 0.5, thisim[2][3] - 0.5));
	gl_Position = vec4(transVertex, 1, 1) * projMatrix;
	vertexColor = vec4(thisim[3][0], thisim[3][1], thisim[3][2], thisim[3][3]);
	texCoord = (vertex.xy + vec2(0.5, 0.5)) * vec2(thisim[1][2], thisim[1][3]) + vec2(thisim[0][2], thisim[0][3]);
}
```

here the shader expects to be sent a batch with 16 "sprite information" condensed in the mat4 im[16] structure. If you change the batch size remeber to change this as well... anyway, the batch index is encoded in the "z" component of the z vertice. that are filled in buffer creation time. More specifically in this part:

```cpp
	GLfloat vertices[] = { -0.5f,-0.5f,0.0f, 0.5f,-0.5f,0.0f, 0.5f,0.5f,0.0f,    // triangle 1
						  -0.5f,-0.5f,0.0f, 0.5f,0.5f,0.0f, -0.5f,0.5f,0.0f };   // triangle 2

	GLfloat *tempVertices = new GLfloat[3 * 6 * BATCH_SIZE];
	for (int f = 0; f < BATCH_SIZE; f++) {
		memcpy(tempVertices + f * 3 * 6, vertices, 3 * 6 * sizeof(GLfloat));
		for (int g = 0; g < 6; g++) tempVertices[f * 3 * 6 + 2 + g * 3] = f;            // mark the index for each triangle
	}

	glBindBuffer(GL_ARRAY_BUFFER, vbo);
	glBufferData(GL_ARRAY_BUFFER, BATCH_SIZE * 6 * sizeof(GLfloat) * 3, tempVertices, GL_STATIC_DRAW);
```

so its basically 16 quads created before hand, and all with fixed position (from -0.5 to 0.5).

# Libraries
- Freetype
- Libpng
- Opengl Es 2
- EGL
- GLFW
- Zlib
- Tinyxml
- Emscripten

# Credits
- Valdir Salgueiro - Developer

# License
- Source code : MIT
