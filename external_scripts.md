# External Scripts

External scripts are the RPS equivalent of `ExternalCommand`s in the Revit API - except you can edit them and re-run them without restarting Revit!

As you work with the interactive shell (or better, the IronPython pad below the shell), you might create a set of statements that are reusable. If you save these to a file, you can add them to the Revit Ribbon panel in the Add-ins section. They will show up as buttons that you can press to have the script executed. By opening the script with your favorite editor, you can change what happens when the button get's pressed.

![The External Scripts tab](the_configure_dialog/2015.03.18_Configure_RevitPythonShell.png)

Use the "Configure" dialog to configure the external scripts. On the "External Scripts" tab, you can add new external scripts, remove them and edit them. Each entry in the list of external scripts has these properties:

- a "Name": this is the text that is shown next to the button in the interface
- a "Group": use a shared group name to add multiple external scripts to a "SplitButton" (see the "Button fixe" example below) or leave this blank to create stacked buttons (like "Button one" etc. in the example below)
- a "Path": the full path to the python script to be run when the button is clicked.

When you click "Save", you will be reminded to restart Revit in order to show changes to the Ribbon. The reason being, that Revit Add-ins can only manipulate the Ribbon when Revit starts. So. Close Revit and re-open it to see the following panel in your Add-ins section of the Revit Ribbon:

![The RPS Ribbon panel with external scripts](the_configure_dialog/2015.02.27_Ribbon_Panel.png)

If you want to create more fancy controls, you should read the section "More control over the Ribbon Panel".