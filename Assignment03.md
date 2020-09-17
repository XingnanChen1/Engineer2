## Assignment 03 Write-up

### Downloads: 
[MyGame_x86](https://github.com/XingnanChen/Engineer2/blob/master/Assignment03/MyGame_x86.zip?raw=true)  
[MyGame_x64](https://github.com/XingnanChen/Engineer2/blob/master/Assignment03/MyGame_x64.zip?raw=true)


### Assignment Objectives：
1. Make graphics system platform-independent.  
2. Add index buffer to mesh.  
3. Add parameters to mesh and effect initialization function to enable user defined input.  
4. Add another effect and another mesh.  

Optional Challenges:

5. change background color dynamically

### ScreenShots
Game Running  
![Image](Assignment03/gameRunning.gif)  
 
### Implementation:
When combining platform-specific codes into Graphics.cpp, we need to find out the platform-specific parts. Most of the platform-specific parts are managing buffers such as clearing the image buffer and depth buffer. Thus, I created a class called cBufferManager to encapsulate these functions, (maybe creating a namespace should be more suitable if we regard these features as Graphics' utility/helper function rather than buffer management). I declared the interfaces in the cBufferManager header file and implemented those interfaces in the corresponding platform-specific cpp file, and exclude build from the platform which it doesn't belongs to.  
In cBufferManager class, I declared these interfaces: ClearColor(), ClearDepth(), SwapBuffer(), InitalizeViews() and CleanUp(). Then we just need to declare a cBufferManager object in Graphics.cpp and invoke these interface accordingly.

For example, clearing the back image buffer:  
```cpp
s_buffer_manager.ClearColor(
        (sinf(elapsedSecondCount_simulationTime) + 1) / 2, 
        (1 + cosf(elapsedSecondCount_simulationTime)) / 2,
        0.f);
```  

From the game running gif at the begining, we can see the background color is changing dynamically. 
 
The effect initialization codes are like:  

```cpp
 if (!(result = s_effect.InitializeShadingData("data/Shaders/Vertex/standard.shader", "data/Shaders/Fragment/myShader.shader")))
 {
     EAE6320_ASSERTF(false, "Can't initialize Graphics without the shading data");
     return result;
 }
```
The user only need to specify the vertex and fragment shader file path.

At first, my effect object takes 16 bytes in OpenGL and 48 bytes in D3D. Let's discuss the member of effect class in each platform specifically.
There are four members:
```cpp
#if defined( EAE6320_PLATFORM_GL )
    GLuint m_programId = 0;
#endif
    cShader* m_vertexShader = nullptr;
    cShader* m_fragmentShader = nullptr;
    cRenderState m_renderState;
```
In OpenGL platform:
After reading code carefully, we can find that m_vertexShader and m_fragmentShader are no longer used after they're bound to the OpenGL program, thus we don't need store these two as class member. After removing these two members, the effect object reduced to 8 bytes(GLuint takes 4 bytes and cRenderState takes 1 byte, after aligment the object takes 8 bytes). 

In D3D platform:
The m_renderState itself takes 24 bytes because it has three pointer members(each pointer takes 8 bytes).
m_vertexShader and m_fragmentShader each takes 8 bytes(size of pointer in 64bit platform). After Alignment the object takes 48 bytes.
We can't change anything of m_vertexShader and m_fragmentShader, but we can change cRenderState to pointer to reduce the size.
Then the cRenderState reduced from 24 bytes to 8 bytes and the effect object is finally reduced to 24 bytes. We can't make it smaller because at least we have to store 3 pointers in this class under D3D platform.

The refactored code looks as follow:

```cpp
#if defined( EAE6320_PLATFORM_GL )
    GLuint m_programId = 0;
#endif
#if defined( EAE6320_PLATFORM_D3D )
    cShader* m_vertexShader = nullptr;
    cShader* m_fragmentShader = nullptr;
#endif
    cRenderState* m_renderState = nullptr;
```
After code refactoring. The effect object takes 8 bytes in OpenGL, while the effect object takes 24 bytes in D3D.

The mesh initialization codes are like:  

```cpp
const size_t vertexCount = 4;
VertexFormats::sVertex_mesh vertexData[vertexCount];
{
    // OpenGL is right-handed

    vertexData[0].x = 0.0f;
    vertexData[0].y = 0.0f;
    vertexData[0].z = 0.0f;

    vertexData[1].x = 1.0f;
    vertexData[1].y = 0.0f;
    vertexData[1].z = 0.0f;

    vertexData[2].x = 1.0f;
    vertexData[2].y = 1.0f;
    vertexData[2].z = 0.0f;

    vertexData[3].x = 0.0f;
    vertexData[3].y = 1.0f;
    vertexData[3].z = 0.0f;
}
const size_t indexCount = 6;
uint16_t indexData[indexCount]
{
    0,1,2,0,2,3
};

//use OpenGl right handed rule by default (counter clockwise)
if (!(result = s_mesh.InitializeGeometry(vertexData, vertexCount, indexData, indexCount)))
{
    EAE6320_ASSERTF(false, "Can't initialize Graphics without the geometry data");
    return result;
}
```
The user only need to specify the vertex data and index data and their size to use the interface.

Though the assignment asks us not to implement detecting winding order function, but I found a solution that's not hard to implement and took little processing time (if you are curios, please see [this]https://en.wikipedia.org/wiki/Curve_orientation and [this]https://stackoverflow.com/a/1180256). Since I think it's really pity that we already have plat form independent engine but we still have to depend on one specific winding order.
In implementing InitializeGeometry() function, we don't care about the winding order of the vertex data. It will detect the winding order automatically and change direction if needed. Takeing D3D platform as example:

```cpp
//if right hand then change direction
if(!isClockWise(vertexData,indexData))
{
    changeDirection(indexData, indexCount);
}
```

If it's right hand winding order under D3D platform it will change direction to left hand.

my mesh object takes 16 bytes in OpenGL and 24 bytes in D3D. 
These are members from mesh class:
```cpp
#if defined( EAE6320_PLATFORM_D3D )
    cVertexFormat* m_vertexFormat = nullptr;
    ID3D11Buffer* m_vertexBuffer = nullptr;
    ID3D11Buffer* m_indexBuffer = nullptr;

#elif defined( EAE6320_PLATFORM_GL )
    GLuint m_programId = 0;
    GLuint m_vertexBufferId = 0;
    GLuint m_vertexArrayId = 0;
    GLuint m_indexBufferId = 0;

#endif
```
 
We can't make the object smaller. These are the necessary data to store in the object. In OpenGL, 4 unsigned int takes 16 bytes, and in D3D, 3 pointer takes 24 bytes. 
 
Finally, declare another mesh and effect is easy, bind the new effect and draw the new mesh just below the previous effect and mesh will render the meshes with different effects sucessfully! 

# Optional Chanllange

When invoking clearcolor() in graphics.cpp, follow the same animation logic as the assignment 1. Make the color variate by elapsed simulation time, now the background will have the similar effect!(You can refer to the SubmitElapsedTime() function to get the elapsed simulation time.)
 
