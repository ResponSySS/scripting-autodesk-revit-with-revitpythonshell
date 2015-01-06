# Files and Locations

FIXME: move this to an appendix as most people won't care about this!

Create a list of all the files installed with RevitPythonShell. Also, a list of files (generally) created by RPS. Add a glossary for terms used here. Find a canonical way to refer to "canned commands" (external scripts, analog to `ExternalCommand`)

The RevitPythonShell installer creates the following folders:

* the *application folder* 
  * defaults to `C:\Program Files (x86)\RevitPythonShell2015` on most systems
  * note: the `2015` portion means that this is the version for Autodesk Revit 2015 - other versions will differ accordingly
* the *data folder*
  * `%APPDATA%\RevitPythonShell2015`, with `%APPDATA%` being a system variable that normally points to the `AppData\Roaming` subfolder of your user directory.
  * note: you can enter `%APPDATA%\RevitPythonShell2015` in the Windows Explorer and you will be taken directly to the correct folder. You can even `cd` there on the command line!

Also, it adds an addin manifest file to `%APPDATA%\Autodesk\Revit\2015` called `RevitPythonShell2015` that tells Revit to load the plugin on startup.

## Contents of the Application Folder

The application folder contains the code required to run RevitPythonShell as a 

## Contents of the Data Folder

