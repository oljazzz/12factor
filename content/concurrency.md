## VIII. Concurrency
### Scale up via the process model

Any computer program, once run, is represented by one or more processes.  Web apps have taken a variety of process-execution forms.  For example, PHP processes run as child processes of Apache, started on demand as needed by request volume.  Java processes take the opposite approach, with the JVM providing one massive uberprocess that grabs more system resources (CPU and memory) as needed, with concurrency managed internally via threads.  In both cases, the running process(es) are only minimally visible to the developers of the app.

In the twelve-factor app, processes are a first class citizen.  Processes in the twelve-factor app take strong cues from [the unix process model for running service daemons](http://adam.heroku.com/past/2011/5/9/applying_the_unix_process_model_to_web_apps/).  Using this model, the developer can architect their app to handle diverse workloads by assigning each type of work to a *process type*.  For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.

The process model truly shines when it comes time to scale up.  The share-nothing, horizontally partitionable nature of twelve-factor app processes means that adding more concurrency is a simple and reliable operation.  The array of process types and number of processes of each type is known as the *process formation*:

<img src="http://s3.amazonaws.com/adamheroku_blog/process_diagram.png" style="border: 1px solid #777" />

Twelve-factor app proceses [should never daemonize](http://dustin.github.com/2010/02/28/running-processes.html) or write PID files.  They count on the operating system's process manager (Upstart, launchd, or a distributed process manager on a cloud platform) to manage STDIN/STDOUT, restarts after crashes, and restarts requested by the user (such as when deploying new code).

Concurrency of process types should never be referenced or managed directly by the app, since it varies between deploys.  The process model offers a clean way for the underlying platform to scale up and distributed the app across many physical machines, without needing to modify the code of the app.  Attempting to manage process concurrency at the operating system level from the app (tools such as `mongrel_cluster` for Ruby, or command line switches such as `--concurrency` passed to Python's `celeryd`) are violations of the twelve-factor system.
