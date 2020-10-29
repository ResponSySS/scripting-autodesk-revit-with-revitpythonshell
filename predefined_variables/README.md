# Predefined variables

In addition to the builtin variables you would expect in python, the RevitPythonShell adds some special variables to enable access to Autodesk Revit.

Some of these variables can only be understood in the context of implementing `IExternalCommand.Execute` in the Revit API, so check there if you are not quite sure what to make of a specific variable.

| Variable | Description |
| -- | -- |
| `__revit__` | A reference to the `Autodesk.Revit.Application` instance, obtained from the `ExternalCommandData` aurgument passed to plugins. |
| `__commandData__` | The actual `ExternalCommandData`argument passed to the RevitPythonShell plugin when you clicked "Open Python Shell" or launched a script from the ribbon. On closing the interactive shell window, the contents of `__message__` will be assigned back so Revit has access to it.|
| `__elements__` | The `ElementSet` passed to the RevitPythonShell when you clicked "Open Python Shell" or launched a script from the ribbon. |
| `__result__` | This is set to `IExternalCommand.Result.Succeeded`, but you can change it if you want. When the interactive shell is closed or a script ends, the RevitPythonShell returns the value of this variable as the result of the `IExternalCommand.Execute` method.|
|`__vars__` | This is a `IDictionary<string, string>` of user defined variables as defined in the configuration  file|
| `__uiControlledApplication__` | (only for the *StartupScript*) A reference to the `UIControlledApplication`instance.
| `__window__` | A reference to the current output window. Basically, only `__window__.Close()` is guaranteed to work, but poke around!|
| `__file__` | External scripts and those deployed with RpsAddin support the `__file__` builtin - the variable contains the full path to the source file being executed. For scripts that are read from a dll (ExternalCommandAssemblyBuilder / RpsAddin) the absolute path to the dll is used as the base directory of the script path (e.g. 'C:\myfolder\myaddin.dll\myscript.py') |

A script that is invoked as a *StartupScript* (when Revit is starting up) has access to the `UIControlledApplication` instance just like regular Revit plugins. You can use this to customize the ribbon for your plugin - this is especially useful when developing self-contained RpsAddins for deployment. See "Using the StartupScript to modify the RibbonPanel" for more information. (FIXME)

## Other predefined variables

Depending on the contents of your *InitScript*, when you open an interactive shell, you will find other variables predefined too. The default *InitScript* contains these values:

- the contents of the namespaces `Autodesk.Revit.DB`, `Autodesk.Revit.DB.Architecture`, `Autodesk.Revit.DB.Analysis`
- `uidoc`: an alias for `__revit__.ActiveUIDocument`
- `doc`: an alias for `__revit__.ActiveUIDocument.Document
- `selection`: an alias for `list(__revit__.ActiveUIDocument.Selection.Elements)
- `TaskDialog` (the class)
- `UIApplication` (the class)
- `alert(msg)`: shortcut for `TaskDialog.Show(...)`
- `quit()`: closes the window

## User defined variables (__vars__)
(FIXME)
- writeable dict
