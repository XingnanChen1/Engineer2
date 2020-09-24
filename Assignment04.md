
## Assignment 04 Write-up

### Downloads: 
[MyGame_x86](https://github.com/XingnanChen/Engineer2/blob/master/Assignment04/MyGame_OpenGL.zip?raw=true)  
[MyGame_x64](https://github.com/XingnanChen/Engineer2/blob/master/Assignment04/MyGame_D3D.zip?raw=true)


### Assignment Objectives：
- Make an interface in the Graphics that game programmers can submit data(background color, mesh, effect) from MyGame to Graphics.  
- Make the effects and meshes reference counted.  
- Change the mesh and effect status when pressed a key.  


### ScreenShots
Game Running  
![Image](Assignment04/gamerunning.gif)  
 
### Implementation:
1. Submitting Background Color:  
- Create a SubmitBackgroundColor() function in Graphics to save the data from MyGame to s_DataBeingSubmittedByApplicationThread.   
- Implement the SubmitDataToBeRendered() function(which is inherited from iApplication) in MyGame to submit the color data to Graphic by calling the submission functions exposed by Graphics. 
In myGame, the user can invoke SubmitBackgroundColor() like this:

```cpp
Graphics::SubmitBackgroundColor((sinf(i_elapsedSecondCount_systemTime) + 1) / 2,(1 + cosf(i_elapsedSecondCount_systemTime)) / 2,0.f, 1);   
```  
- When rendering the background color by using ClearUp, I use the cached data from s_dataBeingRenderedByRenderThread.   

2. Reference Counting is used to determine when a pointer should be deleted.  
Using cMesh as an example:   
In cMesh class:   
- Add the header and three macros in cMesh class.  

header:  

```cpp
<Engine/Assets/ReferenceCountedAssets.h>  
```

macros:  
```cpp
EAE6320_ASSETS_DECLAREREFERENCECOUNTINGFUNCTIONS()  
EAE6320_ASSETS_DECLAREDELETEDREFERENCECOUNTEDFUNCTIONS( cMesh )  
EAE6320_ASSETS_DECLAREREFERENCECOUNT()  
```

- Change the constructor, deconstructor, and initialize function to private.  
- Create a factory function called Load() to initialize and return a cMesh object.   

3. Submitting Meshes and Effects pairs  

In Graphics:  
Create SubmitMeshAndEffect() function and expose it to MyGame to receive the data. Then cache those meshes and effects into s_DataBeingSubmittedByApplicationThread. Since we are using two copies of the data to synchronize the application loop thread and the render thread. We can’t render the data directly and have to cache it first. After the previous frame is rendered, the cached data will be swap to the current rendering data and then the current frame will be rendered.  

In MyGame:  
Initialize all meshes and effects data in SubmitDataToBeRendered() function by creating array of meshdata and effectdata struct. Then invoke SubmitMeshAndEffectData() function to submit the data to render thread. The usage of the interface looks as following:  
```cpp
struct sVertex_mesh  
{  
	float x, y, z;  
}; 
struct meshData
{
	eae6320::Graphics::VertexFormats::sVertex_mesh* vertexData;
	size_t vertexCount;
	uint16_t* indexData;
	size_t indexCount;
}; 
struct effectData
{
	std::string vertextShaderPath;
	std::string fragmentShaderPath;
};
```
First define the data in myGame class:  
 ```cpp
eae6320::Graphics::VertexFormats::sVertex_mesh* vertexData;
eae6320::Graphics::VertexFormats::sVertex_mesh* vertexData_b;

uint16_t* indexData;
uint16_t* indexData_b;

meshData* meshdata;
effectData* effectdata;
```
Then init the data and submit it:  
 ```cpp
vertexData = new Graphics::VertexFormats::sVertex_mesh[vertexCount];
{
	vertexData[0].x = 0.0f;
	vertexData[0].y = 0.0f;
	vertexData[0].z = 0.0f;

	vertexData[1].x = 0.8f;
	vertexData[1].y = 0.0f;
	vertexData[1].z = 0.0f;

	vertexData[2].x = 0.8f;
	vertexData[2].y = 0.8f;
	vertexData[2].z = 0.0f;

	vertexData[3].x = 0.0f;
	vertexData[3].y = 0.8f;
	vertexData[3].z = 0.0f;
}
const size_t indexCount = 6;

indexData = new uint16_t[indexCount];
{
	indexData[0] = 0;
	indexData[1] = 1;
	indexData[2] = 2;
	indexData[3] = 0;
	indexData[4] = 2;
	indexData[5] = 3;
}

meshdata = { vertexData,vertexCount,indexData,indexCount };

effectdata = { "data/Shaders/Vertex/standard.shader", "data/Shaders/Fragment/myShader.shader" };

//parameters are meshdata effectdata and mesh_effect_pair_counts
Graphics::SubmitMeshAndEffect(meshdata, effectdata, 1);
```  
The meshdata took 16 bytes on OpenGL and 32 bytes on D3D;  
The effectdata took 56 bytes on OpenGL and 80 bytes on D3D;  
We can't make it any smaller since all these data are needed for graphics engine to render the frame.  
For meshdata, all memebers takes 4 byte, reordred won't affect anything. And the vertex data and the index data are already in the form of pointer.  
For effectdata, these memebers are just std::string, there isn't too much thing we can do to reduce it's size.  

The sDataRequiredToRenderAFrame struct is defined as:  
```cpp
struct sDataRequiredToRenderAFrame
{
	eae6320::Graphics::ConstantBufferFormats::sFrame constantData_frame;
	//background color
	float r, g, b, a;

	//mesh

	meshData* meshdatas[2];
	effectData* effectsdatas[2];

	size_t mesh_effect_count;

};
```

On OpenGL platform:  
constantData_frame takes 144 bytes;  
background color takes 4 * 4 = 16 bytes;  
meshdatas takes 16 * 2 = 32 bytes;  
effectData takes 56 * 2 = 112 bytes;  
mesh_effect_count takes 4 bytes;  

The struct takes 144 + 16 + 32 + 112 + 4 = 308 bytes;  
Since there are two copies of the struct, it takes 616 bytes in total.  

On D3D platform:  
constantData_frame takes 144 bytes;  
background color takes 4 * 4 = 16 bytes;  
meshdatas takes 32 * 2 = 64 bytes;  
effectData takes 80 * 2 = 160 bytes;  
mesh_effect_count takes 8 bytes;  

The struct takes 144 + 16 + 64 + 160 + 8 = 392 bytes;  
Since there are two copies of the struct, it takes 784 bytes in total.  

4. Responding to input  

To make rendering data is controlled by users' input, we have to implement the UpdateSimulationBasedOnInput() function. We can declare a bool state variable such as "isSecondMeshRemoved" in our game. And change the state of it based on the input in UpdateSimulationBasedOnInput() function. And finally in SubmitDataToBeRendered() function,
we changed the submit data by the state of "isSecondMeshRemoved" like:  
```cpp
if(isSecondMeshRemoved)
	Graphics::SubmitMeshAndEffect(meshdata, effectdata, 1);
else
	Graphics::SubmitMeshAndEffect(meshdata, effectdata, 2);
```
Then our game simulation status will respond to user's input correctly.  

To make the game simulate based on time, we can implement the UpdateSimulationBasedOnTime() function in our game. Use the similar idea from responding the input, we can change the state based on time instead of input. Using the same method, we can easily make our bottom mesh blink every second elapesed!  
