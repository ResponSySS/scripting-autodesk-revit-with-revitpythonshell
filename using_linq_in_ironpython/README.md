# Using Linq in IronPython

```python
import clr
clr.AddReference("System.Core")
import System
clr.ImportExtensions(System.Linq)

# will print 3 and 4 :)
[2, 3, 4].Where(lambda x: x != 2).ToList().ForEach(System.Console.WriteLine)
```


