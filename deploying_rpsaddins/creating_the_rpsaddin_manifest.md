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

You can also add a `StartupScript` that will be run when the RpsAddin is loaded by Revit at startup:

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
