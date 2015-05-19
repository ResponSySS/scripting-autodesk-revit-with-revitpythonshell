# RevitLookup and RevitPythonShell

[RevitLookup](https://github.com/jeremytammik/RevitLookup) is a handy tool for interactively exploring Revit BIM databases. If you want to get anywhere with Revit API programming, you should look into RevitLookup. You should also be reading the RevitLookup's author Jeremy Tammik's blog [The Building Coder](http://thebuildingcoder.typepad.com/)!

The standard RevitPythonShell *InitScript* contains a section dedicated to RevitLookup:

```python
#------------------------------------------------------------------------------
import clr
from Autodesk.Revit.DB import ElementSet, ElementId

class RevitLookup(object):
    def __init__(self, uiApplication):
        '''
        for RevitSnoop to function properly, it needs to be instantiated
        with a reverence to the Revit Application object.
        '''
        # find the RevitLookup plugin
        try:
			rlapp = [app for app in uiApplication.LoadedApplications
					 if app.GetType().Namespace == 'RevitLookup'
					 and app.GetType().Name == 'App'][0]
        except IndexError:
            self.RevitLookup = None
            return
        # tell IronPython about the assembly of the RevitLookup plugin
        clr.AddReference(rlapp.GetType().Assembly)
        import RevitLookup
        self.RevitLookup = RevitLookup
        # See note in CollectorExt.cs in the RevitLookup source:
        self.RevitLookup.Snoop.CollectorExts.CollectorExt.m_app = uiApplication
        self.revit = uiApplication

    def lookup(self, element):
        if not self.RevitLookup:
			print 'RevitLookup not installed. Visit https://github.com/jeremytammik/RevitLookup to install.'
			return
        if isinstance(element, int):
            element = self.revit.ActiveUIDocument.Document.GetElement(ElementId(element))
        if isinstance(element, ElementId):
            element = self.revit.ActiveUIDocument.Document.GetElement(element)
        if isinstance(element, list):
            elementSet = ElementSet()
            for e in element:
                elementSet.Insert(e)
            element = elementSet
        form = self.RevitLookup.Snoop.Forms.Objects(element)
        form.ShowDialog()
_revitlookup = RevitLookup(__revit__)
def lookup(element):
    _revitlookup.lookup(element)

#------------------------------------------------------------------------------
```
What this does is define a global function `lookup` that will open up the "Snoop Objects" dialog from RevitLookup with the element passed. In fact, you can pass `ElementId`s, lists of `Element` objects or just plain integers, that are converted to `ElementId`s - obtain these from the *Manage->IDs of Selection* command in the Ribbon. Since the `Document` also derives from `Element`, you can inspect it too!

Here are some fun things to try, assuming you haven't removed anything from the standard *InitScript*:

Select an element in the BIM, start an interactive Python shell and type:

```python
>>> lookup(selection[0])
```

You should see something like this:

