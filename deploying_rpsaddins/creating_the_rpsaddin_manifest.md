# Creating the RpsAddin manifest

A sample RpsAddin manifest looks like this:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RpsAddin>
  <RibbonPanel text="Hello World">
    <!-- the script is always searched relative to the location of the RpsAddin xml file -->
    <PushButton text="Hello World!" src="helloworld.py"/>
  </RibbonPanel>
</RpsAddin>
```

This example is taken from the *Examples* folder in the RevitPythonShell code. The file is called `HelloWorld.xml`. The name of the file is important, since the "Deploy RpsAddin" feature uses that file to name the resulting RpsAddin dll - in this case it will be called `HelloWorld.dll`.

Let's look at the various parts of the manifest:

* `<RpsAddin>`: this is the root tag describing an RpsAddin manifest file
* `<RibbonPanel>`: you can create as many panels on the ribbon as you like - they will be placed on the "Add-Ins" ribbon in Revit.
  * `text`: the `text` attribute contains the text of the resulting ribbon panel 
  * `<PushButton>`: each ribbon panel may have one or more push buttons. Each such push button has a `text` attribute that is displayed in the interface and a `src` attribute that is the path (either absolute or relative to the RpsAddin manifest file) to the external script being configured.

If you would like to add a `SplitButton`, you can do so like this:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RpsAddin>
  <RibbonPanel text="Showing off my SplitButtons">
    <SplitButton text="SplitButton #1">
      <!-- the script is always searched relative to the location of the RpsAddin xml file -->
      <PushButton text="Hello World!" src="helloworld.py"/>
      <PushButton text="Goodbye cruel World!" src="helloworld.py"/>
    </SplitButton>
    <SplitButton text="SplitButton #2">
      <PushButton text="Gotta go" src="helloworld.py"/>
      <PushButton text="real fast!" src="helloworld.py"/>
    </SplitButton>
    <PushButton text="Enough with the Kindergarten jokes already!" src="helloworld.py"/>
  </RibbonPanel>
</RpsAddin>
```

You can also add a *StartupScript* that will be run when the RpsAddin is loaded by Revit at startup:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RpsAddin>
  <StartupScript src="good_morning_world.py"/>
  <RibbonPanel text="Hello World">
    <!-- the script is always searched relative to the location of the RpsAddin xml file -->
    <PushButton text="Hello World!" src="helloworld.py"/>
  </RibbonPanel>
</RpsAddin>
```

You could even create an RpsAddin manifest file without any push buttons at all and just use the *StartupScript* to create the user interface on the Revit ribbon. See "Using the StartupScript to modify the RibbonPanel" on how to do this. Note that if you do this, the scripts won't be automatically added to the resulting RpsAddin dll. You can use the `File` tag to add files to the output directory. Use the installer to then bundle them up for installation:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<RpsAddin>
  <StartupScript src="good_morning_world.py"/>
  <File src="helloworld.py" />
</RpsAddin>
```
In the example above, the file `helloworld.py` is copied to the output directory. Your startup script (`good_morning_world.py`) then needs to reference this file.