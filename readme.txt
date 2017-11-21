Provided testbed v1.0
for P&O: CS 2017-2018
by Wouter Baert

This document should contain all info you may need to demonstrate your autopilot on the provided testbed. Report issues/bugs or suggest improvements at https://github.com/p-en-o-cw-2017/provided-testbed/issues



Running the testbed and handling communication

In order to run the testbed, you need to set it up correctly so that an autopilot can connect to it. There are two ways to handle this communication: through the Java API or through sockets, which will be discussed separately.


Java API

In this approach the autopilot is included in the classpath of the testbed upon execution as an external library (with certain requirements which will be specified later). When the testbed initializes, it will use an AutopilotFactory class (specified by you) to instantiate a single autopilot. All further calls will be made using the interfaces.

This method has better performance (~700 UPS using SimpleAutopilot), however since you are in the same process as the testbed it also puts restrictions on how your autopilot runs. Namely, to avoid synchronization and other low-level issues, the testbed is purposefully written in such a way that, after the GUI, World, ... constructors are finished, all our code runs in the Swing thread. To be precise, the AutopilotFactory.createAutopilot() method is called from the constructor thread, whereas all Autopilot and AutopilotOutput methods are called from the Swing thread. This means you can still create your own threads and processes, yet you should watch out that you don't directly or indirectly cause multiple threads to access the same (testbed) data at the same time.

How to use:
1) Create a valid autopilot jar. This jar should include a package called "interfaces" which contains the following classes:
- Autopilot, AutopilotConfig, AutopilotInputs and AutopilotOutputs: these are defined at https://github.com/p-en-o-cw-2017/p-en-o-cw-2017/
- AutopilotFactory: This class can be specified by you, as long as it contains a method "public static Autopilot createAutopilot()" which creates and returns an autopilot for the testbed to communicate to. This method will be called once when the testbed starts.
If you're using eclipse, you can create the jar by selecting the classes/packages you wish to export, pressing right-click, selecting "Export...", selecting "JAR file", specifying an export location and exporting with the default settings.
2) You can now run the testbed from a terminal (in the same directory as both jars) with the following command:

On Linux and macOS:

    java -cp ProvidedTestbed.jar:<autopilot-library-name.jar> testbed.TestbedGUI

On Windows:

    java -cp ProvidedTestbed.jar;<autopilot-library-name.jar> testbed.TestbedGUI

As an example, you can run the testbed using a simple provided autopilot with the following command:

On Linux and macOS:

    java -cp ProvidedTestbed.jar:SimpleAutopilot.jar testbed.TestbedGUI

On Windows:

    java -cp ProvidedTestbed.jar;SimpleAutopilot.jar testbed.TestbedGUI

Sockets

In this approach the testbed and autopilot run in separate processes and communicate through sockets. In this method the testbed is the server, running at localhost with a specified port. Each time you start the testbed at most one autopilot can connect. In this set-up, the only "methods" which are called are those from the Autopilot interface; all other interfaces will have their contents serialized and deserialized as specified by the reader/writer classes at https://github.com/p-en-o-cw-2017/p-en-o-cw-2017/tree/master/autopilot-src-generated. This means that all method calls come from the testbed and go to the autopilot. The testbed will first send a byte indicating what method it is calling (0: simulationStarted, 1: timePassed, 2: simulationEnded), followed by the arguments (serialized interface contents). The testbed will then perform a busy wait until it receives the return value in serialized form. Since currently the only method without return type is simulationEnded, the testbed doesn't need to wait for the autopilot to finish so it continues after sending the 2-byte without the autopilot having to send anything back.

Although this method has worse performance (~180 UPS using SimpleAutopilot), you have more freedom with the way your autopilot runs.

How to use:
1) Start the testbed with the command:

On Linux and macOS:

    java -cp ProvidedTestbed.jar:SimpleAutopilot.jar testbed.TestbedGUI <port>

On Windows:

    java -cp ProvidedTestbed.jar;SimpleAutopilot.jar testbed.TestbedGUI <port>

Make sure to still include the interfaces in the classpath as explained in the Java API communication method (for example by including SimpleAutopilot.jar like in the command), since the linker will need these to execute the program (just to handle the imports, the relevant lines aren't executed if a port command is given). The testbed will now wait for an autopilot to connect to the specified port at localhost and its GUI won't respond.
2) Start your own autopilot and make sure it connects to the specified port. Communication details are specified above.
As an example, you can run a simple provided autopilot with the following command: "java -jar SimpleAutopilot.jar <port>"



Views


Custom view

This view allows you to easily position the camera in an arbitrary way as well as do some other stuff. Here are the controls:
WASD/ZQSD: Move in horizontal plane
Delete: Remove selected cube.
Left-click: Drag to move cubes.
Right-click: Start/end rotation, during rotation you can change the camera direction by moving the mouse.
Scrolling: While dragging a cube, scroll to adjust the cube-camera distance.


Chase cam

In this mode the camera will always be located on the horizontal projection of the drone's positive Z-axis (so not always "intuitively" behind the drone).


Orthographic view

These show the part of the world with Z-coordinates in between -20 and 220 along the horizontal direction. The amount of space shown along the vertical direction depends on the canvas size, as this allows us to preserve the in-world aspect ratio. Since the drone model itself would be very small and a bit hard to see in this view, it is represented by a "symbol", namely an enlarged model where its wings span 4m, unless they already span 4m anyway, in which case the model isn't scaled.


Autopilot view

The main thing to note about this view is that it simply shows the last image the autopilot (could have) accessed, so it only updates every time a world update is issued, so not based on GUI-induced changes like a change in the camera specification, a change in the cube configuration, ... If the drone is removed it the autopilot view won't be updated anymore either.



Control panel

Since there's a lot of fields, not all will be discussed separately in this document. Here are the most important things to know:
- You can start the simulation by setting "Time rate" to 1.
- Values aren't set until you press enter.
- Physical values are expressed in SI-units. Angles are expressed in degrees to make things user-friendly. On-screen-distances are expressed in pixels (e.g.: Angular speed is expressed in degrees/pixel).

Some further info on specific control panel components:
- Time rate: The ratio inWorldTime/realTime.
- Initial velocity: Will set the velocity of the drone, but only if no time has passed yet.
- Use new Euler angles: Whether or not to use the Euler angles as defined here: https://github.com/p-en-o-cw-2017/p-en-o-cw-2017/blob/master/Autopilot_v2.datatypes This is recommended since this is the fixed version, this option is mainly provided for backward compatibility.
- Camera height: Although this value is part of AutopilotConfig, you can't set it since it's already fully determined by the other three camera variables (this one was picked arbitrarily).
- Show unit vectors: Show the drone's "active" vectors as unit vectors. This is useful since they have different dimensions and their "absolute" size (relative to their respective unit) is fairly meaningless.
- Add cube: adds a single cube at the specified coordinate ("Center X/Y/Z").
- Generate cube: generates a specified amount of cubes at random positions (uniform distribution) within a specified radius around a specified coordinate ("Center X/Y/Z"). If "Space out cubes" is on, cubes will only be generated at positions where they don't overlap and aren't within 4m of the drone (this is done by trail-and-error, if the generation fails too often it will be aborted and reported to the standard output stream).
- Generate cubes in cylinder: Generates cubes as specified in M2.3 from the second assignment of the first semester.



Attribution

This testbed was made using the following libraries:
- JOGL by the JogAmp community: http://jogamp.org/jogl/www/
- JAMA by MathWorks and NIST: http://math.nist.gov/javanumerics/jama/
