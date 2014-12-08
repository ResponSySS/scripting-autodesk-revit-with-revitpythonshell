# What is a RpsAddin

An RpsAddin refers to a Revit Addin that was written using the RevitPythonShell and then bundled for deployment using the "Deploy RpsAddin" feature of the RevitPythonShell.

RpsAddins allow you to easily deploy your scripts without having to install and configure RevitPythonShell on each machine. It does this by creating a Revit DLL that can be used as an Addin. This DLL will use some runtime glue code located in `RpsRuntime.dll` to provide IronPython integration for the Addin. The DLL also contains your scripts and a manifest for the addin as string resources.