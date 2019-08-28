# Commands

Commands are the building blocks for pipelines in Nu. They do the action of the pipeline, whether creating data, changing data as it flows from inputs to outputs, or viewing data once it as exited the pipeline. There are two types of commands: internal commands, those commands built to run inside of Nu, and external commands, commands that are outside of Nu and communicate with standard Unix-style stdin/stdout.

## Internal commands

All commands inside of Nu, including plugins, are internal commands. Internal commands communicate with each other using streams of Tagged<Value> and ShellError (TODO: Link to data types)

### Signature

Commands use a light typechecking pass to ensure that arguments passed to them can be handled correctly. To enable this, each command provides a Signature which tells Nu:

* The name of the command
* The positional arguments (eg, in `start x y` the `x` and `y` are positional arguments)
* If the command takes an unbounded number of additional positional arguments (eg, `start a1 a2 a3 ... a99 a100)
* The named arguments (eg, `start --now`)
* If the command is a filter or a sink

With this information, a pipeline can be checked for potential problems before it's executed.

## External commands

## Communicating between internal and external commands


