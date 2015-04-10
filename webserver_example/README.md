# Webserver example

This is a more elaborate example that shows how to embedd a webserver in Autodesk Revit and use it to automate tasks.

How do you access the BIM from outside Revit? With the Revit API it is rather easy to access the outside world from *within* Revit, but say you wanted to write a shell script that needed to read a schedule from a .rvt document - how would you go about that?

Well… one way would be to have Revit act as a web server, say, http://localhost:8080. You could then use curl:

```
curl http://localhost:8080/schedules/my_schedule_name.csv > my_local_file_name.csv
```

Let us build a script that allows you to do just that: Export any schedule in the BIM as a .csv file. We'll architect it so that you can add a .tsv (tab separated values) output or even a .xls version if you like. And then you can go wild with other outputs - like a screenshot of the current view (think `curl http://localhost:8080/screenshot`) or ways to open / close documents etc.

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
- an `IExternalEventHandler` implementation called `RpsEventHandler` that is responsible for actually producing the output.
- a web server wrapped in a method `serve_forever` that listens for web requests with the `HttpListener`, stores them into the context queue and notifies the external event that there is work to be done.

We'll look into each component one by one below. Note: The full code can be found here: **(ref: github rps samples)**