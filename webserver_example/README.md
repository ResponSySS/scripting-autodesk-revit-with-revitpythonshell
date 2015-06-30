# Webserver example

This is a more elaborate example that shows how to embedd a webserver in Autodesk Revit and use it to automate tasks.

How do you access the BIM from outside Revit? With the Revit API it is easy to access the outside world from *within* Revit. Sometimes you want to write software that needs to read a schedule from a `.rvt` document - from *outside* of Revit.

As an example, say you have a shell script that reads in schedule data from a Revit document and saves it to a CSV file.

One way to solve this is to have Revit act as a web server, say, http://localhost:8080. You could then use curl:

```
curl http://localhost:8080/schedules/my_schedule_name > my_local_file_name.csv
```

Let us build a RevitPythonShell script that allows you to do just that: Export any schedule in the BIM as a CSV file through a web service. Depending on the URL requested, you could return a screenshot of the current view or ways to open / close documents:

```
curl http://localhost:8080/screenshot
curl http://localhost:8080/open/Desktop/Project1.rvt
```

This is a variation on the [non-modal dialog issue](http://thebuildingcoder.typepad.com/blog/2010/04/asynchronous-api-calls-and-idling.html) ([see here too!](http://thebuildingcoder.typepad.com/blog/2013/12/replacing-an-idling-event-handler-by-an-external-event.html)). We want to run a web server in a separate thread, but have handling requests run in the main Revit thread so that we have access to the API. We will be using an [external event](http://help.autodesk.com/view/RVT/2016/ENU/?guid=GUID-0A0D656E-5C44-49E8-A891-6C29F88E35C0) to solve this.

The web server itself uses the [`HttpListener`](https://msdn.microsoft.com/en-us/library/system.net.httplistener.aspx)class, which runs in a separate thread and just waits for new connections. These are then handled by pushing them into a queue and notifying the `ExternalEvent` that a new event has happened. 

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

- a communication channel `contexts` is created for sending web requests (stashed as `HttpListenerContext` instances) to the `ExternalEvent` thread.
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
    
    def append(self, c):
        self.contexts.Enqueue(c)
        
    def pop(self):
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
            'schedules': get_schedules
            # add other handlers here
        }
        
    def Execute(self, uiApplication):
        while self.contexts:
            context = self.contexts.pop()
            request = context.Request
            parts = request.RawUrl.split('/')[1:]
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

```

The `Execute` method here does the grunt work of working with the .NET libraries and delegating requests to the specific handlers. You can extend this class can by adding new handlers to it. In fact, you don't even need to extend the class to add handlers - just register them in the `handlers` dictionary.

Each handler takes a list of path elements and a `UIApplication` object. The handler runs in the Revit API context. It should return an HTTP error code, a content type and a string containing the response.

An example of such a handler is `get_schedules`:

```python
def get_schedules(args, uiApplication):
    '''add code to get a specific schedule by name here'''
    print 'inside get_schedules...'
    from Autodesk.Revit.DB import ViewSchedule
    from Autodesk.Revit.DB import FilteredElementCollector
    from Autodesk.Revit.DB import ViewScheduleExportOptions
    import tempfile, os, urllib

    doc = uiApplication.ActiveUIDocument.Document
    collector = FilteredElementCollector(doc).OfClass(ViewSchedule)
    schedules = {vs.Name: vs for vs in list(collector)}

    if len(args):
        # export a single schedule
        schedule_name = urllib.unquote(args[0])
        if not schedule_name.lower().endswith('.csv'):
            # attach a `.csv` to URL for browsers
            return 302, None, schedule_name + '.csv'
        schedule_name = schedule_name[:-4]
        if not schedule_name in schedules.keys():
            return 404, 'text/plain', 'Schedule not found: %s' % schedule_name
        schedule = schedules[schedule_name]
        fd, fpath = tempfile.mkstemp(suffix='.csv')
        os.close(fd)
        dname, fname = os.path.split(fpath)
        opt = ViewScheduleExportOptions()
        opt.FieldDelimiter = ', '
        schedule.Export(dname, fname, opt)
        with open(fpath, 'r') as csv:
            result = csv.read()
        os.unlink(fpath)
        return 200, 'text/csv', result
    else:
        # return a list of valid schedule names
        return 200, 'text/plain', '\n'.join(schedules.keys())
```
When you write your own handler functions, make sure to implement the function signature: `rc, ct, data my_handler_function(args, uiApplication`.

In `get_schedules`, a `FilteredElementCollector` is used to find all
`ViewSchedule` instances in the currently active document. Using a [dict
comprehension](https://www.python.org/dev/peps/pep-0274/) is a nifty way to quickly make a lookup table for checking the arguments.

The `args` parameter contains the components of the url after the first part,
which is used to select the handler function. So, if the requested URL were,
say, `http://localhost:8080/schedules`, then `args` would be an empty list. In
this case, we just return a list of valid schedule names, one per line - see
the `else` at the bottom of the function.

If the URL were, say `http://localhost:8080/schedules/My%20Schedule%20Name`,
then the `args` list would contain a single element, `"My%20Schedule%20Name"`.
The `%20` encoding is a standard for URLs and is used to encode a space
character. We use `urllib` to unquote the name.

In order to make the function work nicely with a browser, it is nice to have a
`.csv` ending to it - we redirect to the same URL with a `.csv` tacked on if it
is missing! The code for handling the redirect can be found in the [full sample
script on GitHub](https://github.com/daren-thomas/rps-sample-scripts/blob/master/StartupScripts/RpsHttpServer/rpshttpserver.py).
Notice how the HTTP return code 302 is used as the return value for `rc` - you
can [look up all the HTTP return codes
online](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html), we will only
be using 200 (OK), 302 (Found - used for redirects) and 404 (Not Found).

Next, the script checks to make sure the schedule name is a valid schedule in the document. A 404 return code is used to indicate an error here. 

The actual code for returning a schedule makes use of a technique described in
Jeremy Tammik's blog post [The Schedule API and Access to Schedule
Data](http://thebuildingcoder.typepad.com/blog/2012/05/the-schedule-api-and-access-to-schedule-data.html).
The `ViewSchedule.Export` method is used to write the schedule to a temporary
file in CSV format and then read back into memory before deleting the file on
disk. This is a bit of a hack and coming up with a better solution is left as
an exercise for the reader...

The last piece in our puzzle is the `RpsServer`:

```python
class RpsServer(object):
    def __init__(self, externalEvent, contexts, port=8080):
        self.port = port
        self.externalEvent = externalEvent
        self.contexts = contexts

    def serve_forever(self):
        try:
            self.running = True
            self.listener = HttpListener()
            prefix = 'http://localhost:%i/' % self.port
            self.listener.Prefixes.Add(prefix)
            try:
                print 'starting listener', prefix
                self.listener.Start()
                print 'started listener'
            except HttpListenerException as ex:
                print 'HttpListenerException:', ex
                return
            waiting = False
            while self.running:
                if not waiting:
                    context = self.listener.BeginGetContext(
                        AsyncCallback(self.handleRequest),
                        self.listener)
                waiting = not context.AsyncWaitHandle.WaitOne(100)
        except:
            traceback.print_exc()

    def stop(self):
        print 'stop()'
        self.running = False
        self.listener.Stop()
        self.listener.Close()

    def handleRequest(self, result):
        '''
        pass the request to the RevitEventHandler
        '''
        try:
            listener = result.AsyncState
            if not listener.IsListening:
                return
            try:
                context = listener.EndGetContext(result)
            except:
                # Catch the exception when the thread has been aborted
                self.stop()
                return
            self.contexts.append(context)
            self.externalEvent.Raise()
            print 'raised external event'
        except:
            traceback.print_exc()
```

This class implements the `serve_forever` function that starts an
`HttpListener` on a specified port and uses `handleRequest` to pass any
requests on to the external event for processing inside the Revit API context.
