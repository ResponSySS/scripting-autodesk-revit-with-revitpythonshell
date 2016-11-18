Here is some code showing how to write the URL parameter of a `WallType` with the [RevitPythonShell](https://github.com/architecture-building-systems/revitpythonshell). This was created as an answer to the [RevitPythonShell group forum](https://groups.google.com/forum/#!forum/revitpythonshell). This method should work for other families as well!

```python
# find a WallType to work with...
collector = FilteredElementCollector(doc).OfClass(WallType)
wt = [wt for wt in collector 
      if Element.Name.__get__(wt).startswith('_dpv AW1')][0]
print Element.Name.__get__(wt)
parameter = wt.get_Parameter(BuiltInParameter.ALL_MODEL_URL)
transaction = Transaction(doc, 'setting URL')
try: 
    transaction.Start() 
    parameter.Set('http://systems.arch.ethz.ch')
    transaction.Commit()
except: 
    transaction.Rollback()
```

The tricky part is the line with `wt.get_Parameter(BuiltInParameter.ALL_MODEL_URL)`. I found the `ALL_MODEL_URL` parameter name using the [RevitLookup tool](https://github.com/jeremytammik/RevitLookup) - which can be launched directly from RPS if you have it installed (just type `lookup(wt)`). The basic steps are:

1. retrieve the parameter (`Element.get_Parameter` - you can use the `BuiltInParameter` enumeration for standard parameters...)
2. set the value of the parameter (`Parameter.Set`)
3. since this changes the BIM model, you need to wrap it all in a `Transaction`.