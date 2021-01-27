# Minimal Node DCP Worker

This is a minimal DCP worker for NodeJS.

It doesn't do anything fancy at all.

It is intended to be used as an example for developers writing their own workers.

* Some documentation: https://docs.dcp.dev/module-dcp_worker.html
* Official worker: https://github.com/Distributed-Compute-Labs/dcp-worker
* DCP Client: https://www.npmjs.com/package/dcp-client

*Caution:* `Worker` constructor will be changing slightly in the future. See code to stay future-proof: https://github.com/wesgarland/dcp-minimal-node-worker/blob/develop/dcp-mnw#L39-L41.

If you have any problems, find me or my team on Slack; #dev-connection on https://dcp-devs.slack.com/. 

## Worker Architecture
A DCP Worker is a Supervisor and a collection of Evaluator instances. The Supervisor is a NodeJS program which talks to our Scheduler.

## DCP Client
The `dcp-client` package contains a giant bundle of code, which includes the Supervisor and supporting libraries.  Initializing this package injects some modules into the running NodeJS environment under the `dcp/` namespace; it also downloads the running config from the current scheduler, and optionally (disabled by default) an updated version of the package.

## DCP Evaluator 
The Evaluator is a C++ program which embeds a recent version of Google's v8 JavaScript engine. We have added enough logic to implement a trivial event loop and JS functions to read/write from a socket (or stdio).  There are no functions exposed to JS which can read or write to disk or open network connections; the only communication is through the Supervisor socket.

The Evaluator can run as either a daemon with one instance of v8 on a given thread, or an inetd-style service giving per-process isolation.  Per-process isolation is particularly helpful, as Google's engine crashes its process (purposefully - `SIGABRT`) under certain circumstances, such as JS code exhausting the stack. This is the same as a tab crashing in Chrome. 

### Launching
Remember if you are embedding the Worker that you need a running Evaluator to connect to.  There is no requirement that the Evaluator run on the same host as the Supervisor, however it is strongly recommended that all  Evaluators connected to the Supervisor are on the same host.  Heterogeneous Evaluator support is on the long-term road map.

The Evaluator binary itself is very stable, but the software it runs and the `dcp-client` package change regularly.  You need to have a reasonably up to date `dcp-client` to connect to the production scheduler (https://scheduler.distributed.computer/).

The software that the Evaluator runs is JavaScript passed on the command line; this software is provided by the `dcp-worker` dependency of the `dcp-client` you are using; the list of scripts that it needs to run to work properly is in `evaluator-set-up-files.json`.

If you are using one of our packaged workers (Windows `msi`, Debian `deb`), you probably already have an Evaluator running as an OS service....but if you are doing something custom: the `dcp-evaluator-start` script in this package knows how to parse the json, launch an evaluator, etc.

The correct path to `dcp-evaluator-start` can be determined programmatically via

```javascript
const thePath = require.resolve("dcp-worker/dcp-evaluator-start")
```

See the `start-evaluator` shell script in this repository for a reasonably future-proof way to launch the Evaluator from your own process manager.
