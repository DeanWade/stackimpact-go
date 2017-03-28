# StackImpact Go Agent

## Overview

StackImpact is a performance profiling and monitoring service for production Go applications. It gives developers continuous visibility with line-of-code precision into application performance, such as CPU, memory and I/O hot spots as well execution bottlenecks, allowing developers to optimize applications and troubleshoot issues before they impact customers. Learn more at [stackimpact.com](https://stackimpact.com/).



#### Features

* Automatic hot spot profiling for CPU, memory allocations, network, system calls and lock contention.
* Automatic bottleneck tracing for HTTP handlers and HTTP clients.
* Error and panic monitoring.
* Health monitoring including CPU, memory, garbage collection and other runtime metrics.
* Alerts on hot spot anomalies.
* Multiple account users for team collaboration.

Learn more on the [features](https://stackimpact.com/features/) page (with screenshots).


#### Documentation

See full [documentation](https://stackimpact.com/docs/) for reference.



## Requirements

Linux, OS X or Windows. Go version 1.5+.


## Getting started


#### Create StackImpact account

Sign up for a free account at [stackimpact.com](https://stackimpact.com/).


#### Installing the agent

Install the Go agent by running

```
go get github.com/stackimpact/stackimpact-go
```

And import the package `github.com/stackimpact/stackimpact-go` in your application.


#### Configuring the agent

Start the agent by specifying the agent key and application name. The agent key can be found in your account's Configuration section.

```go
agent := stackimpact.NewAgent();
agent.Start(stackimpact.Options{
	AgentKey: "agent key here",
	AppName: "MyGoApp",
})
```

Other initialization options:
* `AppVersion` (Optional) Sets application version, which can be used to associate profiling information with the source code release.
* `AppEnvironment` (Optional) Used to differentiate applications in different environments.
* `HostName` (Optional) By default, host name will be the OS hostname.
* `Debug` (Optional) Enables debug logging.


Example:

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/stackimpact/stackimpact-go"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello world!")
}

func main() {
	agent := stackimpact.NewAgent()
	agent.Start(stackimpact.Options{
		AgentKey: "agent key here",
		AppName: "Basic Go Server",
		AppVersion: "1.0.0",
		AppEnvironment: "production",
	})

	// use MeasureHandlerFunc or MeasureHandler to additionally measure HTTP request execution time.
	http.HandleFunc(agent.MeasureHandlerFunc("/", handler)) 
	http.ListenAndServe(":8080", nil)
}
```


#### Measuring code segments

*The use of Segment API is optional.*

To measure the execution time of arbitrary parts of the application, the Segment API can be used. The agent continuously watches segment execution time and initiates profiling when anomalies are detected.

```go
// Starts measurement of execution time of a code segment.
// To stop measurement, call Stop on returned Segment object.
// After calling Stop, the segment is recorded, aggregated and
// reported with regular intervals.
segment := agent.MeasureSegment("Segment1")
defer segment.Stop()
```

```go
// A helper function to measure HTTP handler execution by wrapping http.Handle method parameters.
// Usage example:
//   http.Handle(agent.MeasureHandler("/some-path", someHandler))
pattern, wrappedHandler := agent.MeasureHandler(pattern, handler)
```


```go
// A helper function to measure HTTP handler function execution by wrapping http.HandleFunc method parameters.
// Usage example:
//   http.HandleFunc(agent.MeasureHandlerFunc("/some-path", someHandlerFunc))
pattern, wrappedHandlerFunc := agent.MeasureHandlerFunc(pattern, handlerFunc)
```


#### Monitoring errors

*The use of Error API is optional.*


To monitor exceptions and panics with stack traces, the error recording API can be used.

Recording handled errors:

```go
// Aggregates and reports errors with regular intervals.
agent.RecordError(someError)
```

Recording panics without recovering:

```go
// Aggregates and reports panics with regular intervals.
defer agent.RecordPanic()
```

Recording and recovering from panics:

```go
// Aggregates and reports panics with regular intervals. This function also
// recovers from panics.
defer agent.RecordAndRecoverPanic()
```


#### Analyzing performance data in the Dashboard

Once your application is restarted, you can start observing regular and anomaly-triggered CPU, memory, I/O, and other hot spot profiles, execution bottlenecks as well as process metrics in the [Dashboard](https://dashboard.stackimpact.com/).


#### Troubleshooting

To enable debug logging, add `Debug: true` to startup options. If the debug log doesn't give you any hints on how to fix a problem, please report it to our support team in your account's Support section.


## Overhead

The agent overhead is measured to be less than 1% for applications under high load.
