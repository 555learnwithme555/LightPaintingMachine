# LightPaintingMachine

This is the Blender, Processing, and Arduino code that enable use of a CNC light painting machine to make pretty pictures and animations. A basic outline of how this works is as follows:

* Light paths are defined and animated in Blender. _VertexPathCreate.py_ is a Blender addon that makes adorning path objects to animated meshes easy.
* _PathExportTool.py_ is another Blender addon that iterates through each frame of animation and converts the light paths into a command sequence that can be followed by the Arduino
* The commands are sent through OSC and received by _LightPaintingRelay.pde_, a Processing sketch which relays the commands through serial to the Arduino.
* The serial commands are received by the Arduino, running _LightPaintingArduino.ino_. The Arduino executes the command sequence.
* When finished, the Arduino indicates it is finished by sending a serial command to Processing, which relays it back to Blender through OSC.
* When _PathExportTool.py_ in Blender receives the finished command, it sends the command sequence for the next exposure. This loop continues until all exposures are complete.

![alt text](Screenshots/SignalFlow.png)

**Using _VertexPathCreate.py_ to adorn light path objects to animated meshes**

Run _VertexPathCreate.py_ and enter edit mode to view the light path creation tool panel.

![alt text](Screenshots/VertexPathTool1.png)

Click the button to enter the path creation state.

![alt text](Screenshots/VertexPathTool2.png)

Select a series of verticies on the mesh and a path object will be generated, following the sequence of verticies you selected. Press finish or the enter key when you're done.

On the backend, this is automating the usually tedious process of creating empty objects attached to each vertex, creating a path that connects the verticies, and connecting the path to the empty objects using hook modifiers so that the path translates and deforms with the mesh.

_VertexPathCreate.py_ will automatically create a circle called _LightCircle_, which will be used as the bevel object for the light paths. You can change the size of this circle to change the diameter of the light paths. For accurate visual results, this should be set to the diameter of the light emitter on your machine.

_VertexPathCreate.py_ will also automatically create a material called _LightPathMaterial_ and assign the material of all created light paths to this material. Set up this material with no surface shader and an emission volume shader with a color of your choice. You can use different materials for each path, but it is important that each uses an emission shader because the color of this shader is used by _PathExportTool.py_ to send color commands to the machine.

**Running your light painting machine with _PathExportTool.py_, _LightPaintingRelay.pde_, and _LightPaintingArduino.ino_**

When you run _PathExportTool.py_, the light painting execution panel will appear in the tools pane when in object mode.

![alt text](Screenshots/ProcessingConsole.png)    

###### Paremeter setup

Before setting any paremeters, ensure that the unit of length in your Blender scene is set to your preference (use metric).

![alt text](Screenshots/PathExportTool.png)      

__Path Traversal Increment__ is the step increment used when converting light paths into a sequence of position commands. Each path will be traversed from 0 to 1 in steps of this paremeter. Generally the default value should be fine, for especially long paths a smaller value might be better.

__Path Traversal Threshold__ is the minimum distance from the current point on the path to the last recorded point for a new point to be recorded. As the path is traversed, a new point will be recorded when the distance to the last point is greater than this threshold. This is effectively a value of fidelity.
 
__Follow Black Paths__ will force the machine to follow paths that are black (invisible). These paths are normally ignored, but there are cases when you may want to force the machine to follow a black path to do manual obstacle avoidance where automatic obstacle avoidance does not suffice (more on this later).

An origin marker and bounding box that indicate the workspace of the machine will be created. Use the __Painter Bounds__ paremeter to match the size of the bounding box to the size of the physical machine, and use the __Painter Position__ paremeter to specify from where in the blender workspace the machine should operate.

__Light Painting Speed__ is how fast the machine will move. Generally this is a tradeoff between shoot time and machine vibration. Fast movement will often cause oscillation in the light, causing uneven, beady paths. Slower movement produces smoother paths.

__Invert Axes__ allows inversion of the X Y and Z axes. For my machine, the Z stage is operating upside down, so the Z axis is inverted.

__Prop Height Limit__ represents the Y position of the top of the tallest props in your scene. If you are not operating with the Z axis inverted, this would be the height of the bottom of your lowest props. This value is used in obstacle avoidance. If there is a prop in the way between the light position and the start of the next path, the light will retract to this height, move in the XY plane, and then drop down to the final position.

__Exposures Per Frame__ specifies how many exposures you want to divide each frame of animation into. This should be matched with the same number of exposures in Dragonframe.

__Exposure Time__ specifies the time of each exposure. This should be matched with the shutter speed in Dragonframe.

__Next Exposure Yield Threshold__ is a time value threshold. If the Arduino is not on the last exposure, it is about to move on to the next light path, and the time difference between the frame exposure time and the amount of time elapsed so far is less than this threshold, the Arduino will hold and wait to execute this path on the next exposure. In a scene with only short paths that can be drawn quickly, this value can be low. If there are long paths that take longer to draw, it is safer to keep this value higher. If the value is too low, a light path can get cut off by the end of the exposure and not be fully captured.

__Start Frame__ specifies the frame to start on.

__End Frame__ specifies the frame to end on.

**How this all interfaces with Dragonframe**

sdfg

**Incorporating motion control**

Motion control (or moco) will let you add camera movements and prop movements into your animations. For more info on how to create moco movements in Blender and export those movement to Dragonframe, check out this Blender addon: https://github.com/Defaultio/BlenderMoco/
