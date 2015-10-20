# Using LINQ in IronPython

You can use LINQ in IronPython by importing the extension methods from a namespace. This same technique can be used to use other extension methods too! 

```python
import clr
clr.AddReference("System.Core")
import System
clr.ImportExtensions(System.Linq)

# will print 3 and 4 :)
[2, 3, 4].Where(lambda x: x != 2).ToList().ForEach(System.Console.WriteLine)
```

This is not quite the same as the syntactic sugar you might be used to from c#. It could still come in handy if you want to use some of the more advanced LINQ methods.

BTW: You can also use python list comprehensions for most use cases of LINQ! 


