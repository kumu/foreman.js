Foreman.js
==========

**Foreman.js** simplifies running background jobs within the browser.  It's built on James Padolsey's [operative](https://github.com/padolsey/operative) and Petka Antonov's [bluebird](https://github.com/petkaantonov/bluebird), and degrades gracefully in older browsers.

Jobs are executed asynchronously and you're welcome to use callbacks or promises to handle the result.

```
Foreman.source = "/javascripts/worker.js" # your worker source

# Callback-flavor
Foreman.execute("factorial", 5, function(result) {...})

# Promise-flavor (preferred)
Foreman.execute("factorial", 5)
  .then(function(result) {...})
  .catch(function(error) {...}) # caught() for older browsers 
  .finally(function() {...})    # lastly() for older browsers
```

# Defining Jobs

Foreman uses a single script to define the available jobs. If you want to
define your jobs in separate files you'll need to concatenate them
into a single script 
(see [gulp](http://gulpjs.com/) or [grunt](http://gruntjs.com/)).

Job definition follows a simple template:

```
Foreman.define("job-name", function(payload, job) { ... })

# TODO: Consider binding the job func to the job itself to
# avoid the additional parameter
this.progress(0.5)
this.cancel("array must only contain numbers")
this.trigger("event")
```

Here's a simple job definition that sums the values within an array:

```
# /javascripts/worker.js
#
# A simple sum example:
#
Foreman.define("sum", function(array, job) {
  var sum = 0;
  
  for (var i = 0; i < array.length; i++) {
    sum += array[i];
    job.progress(i / array.length);
  }
  
  return sum;
})
```

With the job defined, here's how you would execute it through `Foreman`:

```
Foreman.source = "/javascripts/worker.js"

Foreman.execute("sum", [1, 2, 3])
  .then(function(sum) {
    console.log("sum is " + sum);
  })
```

# Getting Started

## Initialization

All jobs are driven by a single script.

```
# worker.js must also include foreman.worker.js
Foreman.source = "/path/to/worker.js"
```

Additional defaults can be configured through `Foreman.defaults`:

```
Foreman.defaults = {...}
```

## Creating Jobs

(Not sure if we need worker/job/task separation or can use job alone.)

```
worker = Foreman.hire(id)
job = Foreman.build(id, payload)
task = Foreman.execute(id, payload)
```

## Monitoring Jobs

### Central Monitoring

All jobs can be centrally monitored through the main `Foreman` controller.

```
# Event-based handlers
Foreman.on("event", (event) -> ...)

# Hook-based handlers. Cannot be removed once assigned.
# (Go with a middleware approach instead?)
Foreman.onJob((job) -> ...)
Foreman.beforeJob((job) -> ...)
Foreman.onStart((job) -> ...)
Foreman.onProgress((job) -> ...)
Foreman.onComplete((job) -> ...)
Foreman.onCancel((job) -> ...)
Foreman.onError((job) -> ...)
Foreman.afterJob((job) -> ...)
```

### Direct Monitoring

If you're only interested in a specific job you can add event listeners directly.

```
job = Foreman.execute(id, payload)
  .on("start", (job) -> ...)
  .on("progress", (job) -> ...)
  .on("complete", (job) -> ...)
  .on("cancel", (job) -> ...)
  .on("error", (job) -> ...)
```

### Foreman.status()

The `status` helper returns the status of all running jobs.

```
Foreman.status()
# => [{job: job, progress: progress}, {...}]
```

## Job Stats / Instrumentation

All jobs maintain stats about their execution, exposed through the `attr` and `attrs` helpers.

```
job.attr("payload")
job.attr("status") # pending|running|complete|terminated
job.attr("start")
job.attr("end")
job.attr("duration")
job.attr("progress")
job.attrs() # {start: ..., end: ...}
```

## Job Termination

Jobs can be terminated gracefully through `cancel` or forcefully through `kill`.

```
job.cancel(message)
job.kill(message)
```

Similar methods are available on the `Foreman` instance:

```
Foreman.cancel(job_or_job_id, message) # graceful
Foreman.kill(job_or_job_id, message) # forceful
```

## Inline Job Definition

```
Foreman.define("sum", (array) -> ...)
```

## Map / Reduce Helpers

Because map / reduce operations are so common we've included built-in helpers.

```
Foreman.map(array, (value) -> ...)
Foreman.reduce(array, (value) -> ...)
```

## Running Multiple Foremen

```
foreman = Foreman.spawn("/path/to/worker.js")
```
