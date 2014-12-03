# Variables

You can use the *Variables* tab to manage a set of configuration variables that you can access from your scripts with the `__vars__` dictionary. The `__vars__` dictionary has string keys and string values.

![the variables tab](https://dl.dropboxusercontent.com/u/8112069/scripting-autodesk-revit-with-revitpythonshell/variables-0.png)

Use the *Add* button to add a new value to the list. You should see a new entry in the variables list called `<name>`. The *Name* and *Value* text boxes below the list are populated with the corresponding values of the selected list item. You can change both the name and the value as needed. You can have as many such variables as you like.

![adding a value to the variables tab](https://dl.dropboxusercontent.com/u/8112069/scripting-autodesk-revit-with-revitpythonshell/variables-1.png)

The main advantage of using the variables system is that it provides a way to specify configuration for scripts. By using the `__vars__` dictionary to request paths, port numbers or anything else that might change from one machine to another and needs to be configured by the user.



```python
>>> __vars__['foo'] = 'bar'
>>> __vars__['foo']
'bar'
>>>
```

