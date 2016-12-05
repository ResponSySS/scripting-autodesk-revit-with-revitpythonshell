# Collecting Elements

Here is some examples on collecting elements using the  `FilteredElementCollector`.
More information about the class can be found [here](http://www.revitapidocs.com/2016/263cf06b-98be-6f91-c4da-fb47d01688f3.htm)

```python
# Collect all Walls
walls = FilteredElementCollector(doc).OfClass(Wall)

# Collecting all Instances from category Furniture
collector = FilteredElementcollector(doc)
furniture_instances = collector.OfCategory(BuiltInCategory.OST_Furniture).WhereElementIsNotElementType()
```
