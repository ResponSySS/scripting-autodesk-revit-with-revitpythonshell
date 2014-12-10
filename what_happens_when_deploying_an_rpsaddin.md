# What happens when deploying an RpsAddin

FIXME: add a figure here showing the files and how they relate.

This is what happens behind the scenes when deploying an RpsAddin:

* RevitPythonShell reads in the RpsAddin manifest file (the file you select in the Open File Dialog)
* RevitPythonShell creates a DLL file in an output directory (FIXME: check output directory name) with a class that implements IExternalApplication and also subclasses RevitPythonShell.Rps(FIXME: look this up too)
* RevitPythonShell adds all scripts mentioned in the RpsAddin manifest file as resources to this DLL
* RevitPythonShell copies the relevant runtime files (RpsRuntime.dll, but also the IronPython runtime dlls) to the output directory

If you want to deploy the resulting addin, you still need to create a Revit addin manifest file (not to be confused with the RpsAddin manifest file) and place that in the appropriate location (e.g. `%APPDATA%\Autodesk\Revit\Addins\2015`). You will also need to add any other files / python modules your scripts load. See "Creating the Installer" for more information.