# Variables

You can use the *Variables* tab to manage a set of configuration variables that you can access from your scripts with the `__vars__` dictionary. The `__vars__` dictionary has string keys and string values.

```python
>>> __vars__['foo'] = 'bar'
>>> __vars__['foo']
'bar'
>>>
````

