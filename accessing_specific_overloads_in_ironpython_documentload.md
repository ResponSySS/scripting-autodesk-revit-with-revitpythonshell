# Accessing specific overloads in IronPython (Document.Load)

The RevitPythonShell makes working with Revit a bit easier, because, you know, Python. The specific flavour of Python used is IronPython - a port of the Python language to the .NET platform. IronPython lets you call into .NET objects seamlessly. In theory. Except when it doesen't. [All abstractions are leaky](http://www.joelonsoftware.com/articles/LeakyAbstractions.html).

This article is all about a specific leak, based on an impedance mismatch between the object model used in .NET and that used in Python: In C#, you can [overload a method](https://msdn.microsoft.com/en-us/library/ms229029%28v=vs.110%29.aspx). A simple way to think about this is to realize that the name of the method includes its *signature*, the list of parameter (types) it takes. Go read a book if you want the gory details. Any book. I'm just going to get down to earth here and talk about a specific example:

The [`Document.LoadFamily`](http://revitapisearch.com/html/2966229b-60b0-404d-5ffe-e4c4d85d2d7a.htm) method.

The standard method for selecting a specific overload is just calling the function and having IronPython figure it out.

I guess the place to read up on how to call overloaded methods is here: http://ironpython.net/documentation/dotnet/dotnet.html#method-overloads. To quote: 

> When IronPython code calls an overloaded method, IronPython tries to select one of the overloads at runtime based on the number and type of arguments passed to the method, and also names of any keyword arguments.

This works really well if the types passed in match the signatures of a specific method overload well. IronPython will try to automatically convert types, but will fail with a `TypeError` if more than one method overload matches.

The `Document.LoadFamily` method is special in that one of its parameters is marked as `out` in .NET - according to the standard IronPython documentation (REF) that should translate into a tuple of return values - and it does, if you know how. It is just non-intuitive - see [this question on Stack Overflow](http://stackoverflow.com/questions/31471089/how-to-pick-the-right-loadfamily-function-in-revitpythonshell):

> revitpythonshell provides two very similar methods to load a family.
>
>```python
>LoadFamily(self: Document, filename:str) -> (bool, Family)
>LoadFamily(self: Document, filename:str) -> bool
>```
>
>So it seems like only the return values are different. I have tried to calling it in several different ways:
>
>(success, newFamily) = doc.LoadFamily(path)
>success, newFamily = doc.LoadFamily(path)
>o = doc.LoadFamily(path)
>But I always just get a bool back. I want the Family too.

What is happening here is that the c# definitions of the method are:

```c#
public bool LoadFamily(
	string filename
)
```
and
```c#
public bool LoadFamily(
	string filename,
	out Family family
)
```

The IronPython syntax candy for `out` parameters, returning a tuple of results, can't automatically be selected here, because calling `LoadFamily` with just a string argument matches the first method overload. 

You can get at the overload you are looking for like this:

```python
import clr
family = clr.Reference[Family]()
# family is now an Object reference (not set to an instance of an object!)
success = doc.LoadFamily(path, family)  # explicitly choose the overload
# family is now a Revit Family object and can be used as you wish
```
This works by creating an object reference to pass into the function and the method overload resultion thingy now knows which one to look for.

Working under the assumption that the list of overloads shown in the RPS help is the same order as they appear, you can also do this:

```python
success, family = doc.LoadFamily.Overloads.Functions[0](path)
```

and that will, indeed, return a tuple `(bool, Autodesk.Revit.DB.Family)`. I just don't think you should be doing it that way, as it introduces a dependency on the order of the method overloads - I wouldn't want that smell in my code...

Note, that this has to happen inside a transaction, so a complete example might be:

```python
import clr
t = Transaction(doc, 'loadfamily')
t.Start()
try:
	family = clr.Reference[Family]()
    success = doc.LoadFamily(path, family)
    # do stuff with the family
    t.Commit()
except:
    t.Rollback()
```
