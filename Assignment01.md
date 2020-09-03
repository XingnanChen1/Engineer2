## Assignment 01 Write-up

### All requirements are implemented including optional ones. Press F1 will reduce the game simulation rate.

### Downloads:
[MyGame_x86_debug](https://github.com/XingnanChen/Engineer2/raw/master/Assignment01/MyGame_x86_debug.zip)  

[MyGame_x86_release](https://github.com/XingnanChen/Engineer2/raw/master/Assignment01/MyGame_x86_release.zip)  

[MyGame_x64_debug](https://github.com/XingnanChen/Engineer2/raw/master/Assignment01/MyGame_x64_debug.zip)  

[MyGame_x64_releasse](https://github.com/XingnanChen/Engineer2/raw/master/Assignment01/MyGame_x64_release.zip)  


### Assignment Objectives：
1. Create a static library and set up the configurations of it.
2. Create and configure my own game project; Log messages at the beginning and end of the game life cycle.
3. Make an animation that changing the triangle color by creating a new fragment shader.

Optional:
1. Decrease the simulation rate by pressing some key down.
2. Reduce shader content buffer declaration times.

### ScreenShots 
Animation  
![Image](Assignment01/AnimateColor.gif)  

Log Message  
![Image](Assignment01/LogPic.png)  

### Implementation Summary:
If you are familiar with VC++, all the steps are easy to understand.

1. Create Static library:  
  - Create a static library from existing codes.
  - Add property sheets in order.
  - Add the references(openglextension) it needs and add it as the reference to the project(Application) that needs it. 
  - Excluding the corresponding useless files for each target platform.
  - And Don't forget to force including platform specific code.
2. Game project: 
  - Create MyGame project from ExampleGame project(Note: change GUID and GameName in project settings); 
  - Modify "ExampleGame" to "MyGame" in codes.
  - Create BuildMyGameAssets the same way as creating MyGame; 
  - Set default startup project.
3. Animate color: 
  - Change the output color in *.Shader. 
  - Make the output value varying by elapsed time. 
  - The range of RGB should be within [0,1], sin(elapsedtime)/2 + 0.5 is a choice.   
4. Reduce simulation rate: Implemented in UpdateBasedOnInput() function in cMyGame.cpp, follow the same logic as exit game pressing exit key.
5. Content Buffer is now declared in content/shaders/contentbuffer.inc file, the shader using content buffer can include this file after including shaders/shader.inc. It's implmented by Macros to declare the content buffer using the correct data type on different platforms.

Briefly speaking, if you follow every step of the instruction, you shouldn't meet any problem. When I see anything is not as expected, I just read the description again carefully and correct the mistakes, every problem can be solved without googling anything.

### Questions:
#### Tell us which projects needed to add a reference to the new Graphics project? Did you find any projects whose code mentioned the Graphics namespace, but didn't need a reference added? If so, give an example, and explain why the code in question doesn't require a reference. 
Application needs to add the Graphics project reference.
ShaderBuilder has Graphics namespace but doesn’t need the reference because ShaderBuider doesn’t use any function from Graphics namespace.

#### Tell us about any thoughts you have about the engine code base that has been provided to you. How is it organized? Is there anything that you like or don't like about the organization? 
The engine code base is well organized. I like it, it seperate the features reasonably.

#### What do you think of the code style? Is there anything that you like or don't like about it?
Good. I don't care about the format argumants such as "whether there should be break line before every bracket of the if/else statement". As long as the code style is consistent, I'm fine with it.

#### Is there anything that you were initially confused about? Tell us why you were confused and what you figured out to not be confused. 
At first, I added the Graphics reference to the MyGame project. The solution can be built. I found Application project were using functions from Graphics project without having reference and the MyGame project didn’t use Graphics functions. Then I deleted the Graphics reference to MyGame and added it to Application project.

#### Discuss briefly your expectations of the class based on the first lecture and what you know so far, and tell us what you hope to learn from it. Take some time and think about what you personally might be able to get out of the class that aligns with your interests.
I am interested in how to write an organized engine and learn more about shader and rendering techniques. 

