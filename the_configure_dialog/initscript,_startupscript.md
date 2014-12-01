# InitScript, StartupScript

The *Init Script* tab lets you choose the *InitScript* and *StartupScript*. 

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

