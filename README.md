# Debugging ROS C++ nodes with Visual Studio Code
 
VSC = Visual Studio Code

FOR ALL KINDS OF DEBUGGING, THE WORKSPACE HAS TO BE COMPILED WITH DEBUGGING FLAG

    catkin_make -DCMAKE_BUILD_TYPE=Debug

and dont forget to compile back into normal or release build type afterwards :D

## Debugging nodes by running them (launching a separate GDB process)
Debug nodes by running them directly from VSC, with roscore running in the background

Go to debug window (Ctrl+Shift+D)

Click on Start Debugging button (top left corner green play button)

When "Select Envirement" option select pops up, choose C++(GDB/CDLL). This will create `launch.json` file where you can setup your preferences.

Inside `launch.json` the variables should be set like this:

    "request": "launch",

and the `"program"` tag should be set to show the path to the executable of the node. That is why before setarting to debug, the ROS workspace should already be built.

By default executables are set in `devel/lib/[ros_package_name]/[node_name]`

    "program": [path_to_workspace]/devel/lib/[ros_package_name]/[node_name]

where `[path_to_workspace]` should eather be absolute path, or if the VSC workspace is set in the ROS workspace folder, it can just be defined with `${workspaceFolder}`

Resulting in following setup file: 

    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Launch debugging process",
                "type": "cppdbg",
                "request": "launch",
                "program": "[path_to_workspace]/devel/lib/[ros_package_name]/[node_name]",
                "args": [],
                "stopAtEntry": false,
                "cwd": "${workspaceFolder}",
                "environment": [],
                "externalConsole": false,
                "MIMode": "gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            }
        ]
    }

When this is set up, the Start Debugging button can be clicked again and the node will be run with the debugging process attached.

## Debugging nodes from launch files (attaching to their GDB process)
### Overview
To debug nodes that are started within the launch files, there are two options:

**1.) To attach the VSC to a local process.**\
After running the roslaunch normally, VSC is attached to a pid of gdb process.

**PROS** - easy to setup, no VSC extensions needed, no modifcation of ros launch files.

**CONS** - debugging not enabled from the very beginning of program (although that could also be a PRO), but after you enable it by clicking `Start Debugging` button. Lots of clicking needed each time. And each time you need to insert sudo password to debug.

**2.) To attach the VS to a gdbserver.**\
In the launch file a fixed gdbserver port is set for the node that we want to debug. After running the launch file, VSC is attached to the fixed port (no searching amd redundant clicking needed). But to attach the VSC to the gdbserver, extensions need to be installed.

**PROS** - node waits for VSC to be attached, to begin running. Not a lot of clicking needed each time to debug

**CONS** - a smidge harder to set up, changes in ros launch files

### 1.) Attaching to the local process
Roslaunch file can be launched normally.

In the `launch.json` of VSC workspace (inside .vscode/ of the workspace) the following setup should be added (eather by clicking the "Add Configuration->(gdb) Attach" or by copy/pasting it from here )

    {
        "version": "0.2.0",
        "configurations": [
            { 
                "name": "Attach to process",
                "type": "cppdbg",
                "request": "attach",
                "program": "[path_to_workspace]/devel/lib/[ros_package_name]/[node_name]",
                "processId": "${command:pickProcess}",
                "MIMode": "gdb",
                
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            }
        ]
    }

After inserting the correct `"program"` variable, it can be run with Start Debugging button. When prompted, you can search for the node process to attach to by writing its name into the search window. You will also need to insert a pasword to begin debugging.

At the end of debugging session, the process should be **first deatached on VSC** and afterwads, the roslaunch can be canceled.


### 2.) Attaching to the gdbserver
In the launch, for the node that would be debbuged, the parameter `launch-prefix` should be set to a fix port for gdbserver. Example:

    <node pkg="debugging_cpp_pkg" 
        type="debugging_cpp_pkg_node" 
        name="debugging_cpp_pkg_noder" 
        output="screen"
        launch-prefix="gdbserver :1234"/>

And afterwards the roslaunch can be launched from terminal.

IN VSC the extention `Native Debug` should be installed and VSC restarted.

In the `launch.json` (inside .vscode/) of VSC workspace the following setup should be added (eather by clicking the "Add Configuration-> Connect to gdbserver" or by copy/pasting it from here )

    {
        "version": "0.2.0",
        "configurations": [
            {
                "type": "gdb",
                "request": "attach",
                "name": "Attach to gdbserver",
                "executable": "[path_to_workspace]/devel/lib/[ros_package_name]/[node_name]",
                "target": ":1234",
                "remote": true,
                "cwd": "${workspaceRoot}",
                "valuesFormatting": "parseText"
            }
        ]
    }

After inserting the correct `"program"` variable, it can be run with Start Debugging button.
At the end of debugging session, the process should be first deatached on VSC and afterwads, the roslaunch can be canceled.

The `launch-prefix` parameter should be removed whenewer we don't want to debug the node.

## Helpful reading

BLUE SAT ERP team guide to ros debugging
https://bluesat.com.au/a-dummys-guide-to-debugging-ros-systems/

NATIVE DEBUG EXTENSION
https://stackoverflow.com/questions/38089178/is-it-possible-to-attach-to-a-remote-gdb-target-with-vscode
