# Plugins

## Protocol

Plugins use JSON-RPC over stdin/stdout (much in the same way VSCode plugins do). The protocol is split into two stages.

The first stage of the protocol deals with the initial discovery of the plugin. A plugin is started up and then asked to reply with its configuration. Much the same was as commands, plugins have a signature that they respond to Nu with.  Once Nu has this signature, it knows how to later invoke the plugin to do work.

The second stage is the actual doing of work.  Here the plugins are sent either a stream of data where they act over the stream element-wise as a filter, or they take all the elements at once in a final processing step as a sink.

## Discovery

Plugins are discovered by Nu by checking all directories available in the current PATH. In each directory, Nu is looking for executable files that match the pattern `nu_plugin_*` where `*` matches anything after the `nu_plugin_`.  On Windows, this has a similar pattern of `nu_plugin_*.exe` or `nu_plugin_*.bat`.

Once a matching file has been discovered, Nu will invoke the file and pass to it the first JSON-RPC command: config. Config replies with the signature of the plugin, which is identical to the signature commands use. (TODO: add link).

Nu continues in this way until it has traveled across all directories in the path.

After it has traversed the path, it will look in two more directories: the target/debug and the target/release directories. It will pick one or the other depending whether Nu was compiled in debug mode or release mode, respectively.  This allows for easier testing of plugins during development.

