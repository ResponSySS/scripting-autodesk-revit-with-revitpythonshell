# Variables

You can use the *Variables* tab to manage a set of configuration variables that you can access from your scripts with the `__vars__` dictionary. The `__vars__` dictionary has string keys and string values.

![the variables tab](https://dl.dropboxusercontent.com/u/8112069/scripting-autodesk-revit-with-revitpythonshell/variables-0.png)

Use the *Add* button to add a new value to the list. 

```python
>>> __vars__['foo'] = 'bar'
>>> __vars__['foo']
'bar'
>>>
```

