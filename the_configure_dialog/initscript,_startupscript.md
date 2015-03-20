# InitScript, StartupScript

The *InitScript / StartupScript* tab lets you choose the *InitScript* and *StartupScript*. 

![The Init Script Tab](https://dl.dropboxusercontent.com/u/8112069/scripting-autodesk-revit-with-revitpythonshell/initScript-startupScript.png)

## InitScript

The *InitScript* is a script that is run whenever an interactive shell is opened. You can place imports, functions and variable definitions in the *InitScript* to make your life in interactive mode easier. Just keep in mind, that when you save code developed this way to a command (FIXME: external script?) you will need to add these to the external script.

The default contents of the *InitScript* are:

```python
# these commands get executed in the current scope
# of each new shell (but not for canned commands)
from Autodesk.Revit.DB import *
from Autodesk.Revit.DB.Architecture import *
from Autodesk.Revit.DB.Analysis import *

uidoc = __revit__.ActiveUIDocument
doc = __revit__.ActiveUIDocument.Document
selection = list(__revit__.ActiveUIDocument.Selection.Elements)

from Autodesk.Revit.UI import TaskDialog
from Autodesk.Revit.UI import UIApplication
def alert(msg):
    TaskDialog.Show('RevitPythonShell', msg)

def quit():
    __window__.Close()
exit = quit

# a fix for the __window__.Close() bug introduced with the non-modal console
class WindowWrapper(object):
    def __init__(self, win):
        self.win = win
    def Close(self):
        self.win.Dispatcher.Invoke(lambda *_: self.win.Close())
    def __getattr__(self, name):
        return getattr(self.win, name)
__window__ = WindowWrapper(__window__)
```
The contents of this file sometimes changes from version to version of RevitPythonShell. You can always find the newest version in the repository:
https://code.google.com/p/revitpythonshell/source/browse/trunk/RevitPythonShell/init.py

## StartupScript

The *StartupScript* is run when RevitPythonShell is first initialized as part of the `IExternalApplication.OnStartup` event, just after the RevitPythonShell ribbon has been populated. In addition to the standard RevitPythonShell variables available to external scripts, the *StartupScript* has access to `__uiControlledApplication__` the instance of `UIControlledApplication` passed to Revit `IExternalApplication` implementations. 

Possible, real-world uses for this include:

* creating custom ribbon panels (e.g. for deploying your addin)
* starting a web server (see chapter "Webserver example")
* unattended / "batch" processing of revit files

(FIXME: allow *multiple* StartupScripts and move to separate tab)

The initial *StartupScript* installed with RevitPythonShell looks like this:

```python
# script that is run when RevitPythonShell starts in the IExternalApplication.Startup event.
try:
    # add your code here
    # ...
    __window__.Close()  # closes the window 
except:
    import traceback       # note: add a python27 library to your search path first!
    traceback.print_exc()  # helps you debug when things go wrong
```

The contents of this file sometimes changes from version to version of RevitPythonShell. You can always find the newest version in the repository:
https://code.google.com/p/revitpythonshell/source/browse/trunk/RevitPythonShell/startup.py
