The RevitPythonShell exposes the Revit API to the Python programming language. And to keep with the spirit of Python, it includes a REPL (Read Evaluate Print Loop), the interactive shell that let's you try out code snippets *inside* a running Autodesk Revit instance! This offers a totally new way to create addins for Revit and Vasari with a more hands-on experience.

Features:

* Python language scripting (IronPython)
* Syntax highlighting
* full access to the Revit API
* export RevitPythonShell scripts as standalone-addins
* access to the *batteries included* python standard library as well as the .NET framework - mix the best of both worlds
* edit + run scripts without restarting Revit


Brainstorming chapters / sections:
* Ideas for chapters could include interface creation (I use winforms, but this opens a whole other realm - I find I spend more time coding the interface than I do manipulating the database!), and a whole lot of simple examples (geometry, revisions, sheet manipulation, family placement, family purging/renaming) etc.
* mention places to learn about Revit API programming (The Building Coder)
  * [The Revit API Developers Guide](http://help.autodesk.com/cloudhelp/2015/ENU/Revit-API/files/GUID-F0A122E0-E556-4D0D-9D0F-7E72A9315A42.htm)
* mention guides to convert c# code to IronPython
* go through the Revit API examples, translating them
* Contribute (tell people how to get in contact with bug fixes and new features from the community)
* including python libraries
* InitScript with src attribute
* can we have multiple StartupScripts?
* interactive non-modal REPL
* sample scripts from the SDK? go through them one at a time
* in fact: new goal: every friday, try out some new piece of API and blog it?
* Close Revit from the API (in StartupScript e.g.)
