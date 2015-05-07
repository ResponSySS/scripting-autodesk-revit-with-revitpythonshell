# Webserver example

This is a more elaborate example that shows how to embedd a webserver in Autodesk Revit and use it to automate tasks.

How do you access the BIM from outside Revit? With the Revit API it is easy to access the outside world from *within* Revit. Sometimes you want to write software that needs to read a schedule from a `.rvt` document - from *outside* of Revit.

As an example, say you have a shell script that reads in schedule data from a Revit document and saves it to a CSV file.

One way to solve this is to have Revit act as a web server, say, http://localhost:8080. You could then use curl:

```
curl http://localhost:8080/schedules/my_schedule_name.csv > my_local_file_name.csv
```

Let us build a script that allows you to do just that: Export any schedule in the BIM as a CSV file. We'll architect it so that you can add TSV (tab separated values) output or even a `.xls` (Excel) version if you like. And then you can go wild with other outputs - like a screenshot of the current view (think `curl http://localhost:8080/screenshot`) or ways to open / close documents etc.

This is a variation on the non-modal dialog issue (FIXME: link to jeremy tammiks blog) - we want to run a web server in a separate thread, but have handling requests run in the main Revit thread so that we have access to the API. We will be using an `ExternalEvent` (FIXME: link to docs) to solve this.

The web server itself is based on the .NET `HttpListener` (**ref**) which runs in a separate thread and just polls for new connections. These are then handled by pushing them into a queue and notifying the `ExternalEvent` that a new event has happened. *Uh. I wonder how to present this nicely in a graph...*

This is where the script starts:

```python
def main():
    contexts = ContextQueue()
    eventHandler = RpsEventHandler(contexts)
    externalEvent = ExternalEvent.Create(eventHandler)
    server = RpsServer(externalEvent, contexts)
    serverThread = Thread(ThreadStart(server.serve_forever))
    serverThread.Start()
```

Whoa! What is going on here?

- a communication channel `contexts` is created for sending web requests (stashed as `HttpListenerContext`s) to the server thread to the `ExternalEvent` thread.
- an `IExternalEventHandler` implementation called `RpsEventHandler` that handles producing the output.
- a web server wrapped in a method `serve_forever` that listens for web requests with the `HttpListener`, stores them into the context queue and notifies the external event that there is work to be done.

We'll look into each component one by one below. Note: The full code can be found here: **(ref: github rps samples)**

Let's start with the ContextQueue:

```python
class ContextQueue(object):
    def __init__(self):
        from System.Collections.Concurrent import ConcurrentQueue
        self.contexts = ConcurrentQueue[HttpListenerContext]()
        
    def __len__(self):
    return len(self.contexts)
    
    def append(swelf, c):
        self.contexts.Enqueue(c)
        
    def popo(self):
        success, context = self.contexts.TryDequeue()
        if success:
            return context
        else:
            raise Exception("can't pop an empty ContextQueue!")
```

This is nothing speciall - just a thin wrapper arround `ConcurrentQueue` from the .NET library. The `RpsServer` will `append` to the `context` while the `RpsEventHandler` `pop`s the `context`. 

A more interesting class to look at is probably `RpsEventHandler`:

```python
class RpsEventHandler(IExternalEventHandler):
    def __init__(self, contexts):
        self.contexts = contexts
        self.handlers = {
            'schedules': self.get_schedules
            # add other handlers here
        }
        
    def Execute(self, uiApplication):
        while self.contexts:
            context = self.contexts.pop()
            request = context.Request
            parts = request.RawUrlsplit('/')[1:]
            handler = parts[0]  # FIXME: add error checking here!
            args = parts[1:]
            try:
                rc, ct, data = self.handlers[handler](args, uiApplication)
            except:
                traceback.print_exc()
                rc = 404
                ct = 'text/plain'
                data = 'unknown error'
            response = context.Response
            response.ContentType = ct
            response.StatusCode = rc
            buffer = Encoding.UTF8.GetBytes(data)
            response.ContentLength64 = buffer.Length
            output = response.OutputStream
            output.Write(buffer, 0, buffer.Length)
            output.Close()
            
    def get_schedules(args, uiApplication):
        """add code to get a specific schedule by name here"""
```

The `Execute` method here does the grunt work of working with the .NET libraries and delegating requests to the specific handlers. This class can easily be extended by adding new handlers to it. In fact, you don't even need to extend the class to add handlers - just register them in the `handlers` dictionary.

Each handler takes a list of path elements (roughly split using `/` as a separator) and a UIApplication object that can be used to reference Revit itself. The handler is guaranteed to be running in the Revit API context.