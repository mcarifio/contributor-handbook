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

## Creating a plugin (in Rust)

In this section, we'll walk through creating a Nu plugin using Rust.

Let's create our project. For this example, we'll create a simple `len` command which will return the length of strings it's passed.

First off, we'll create our plugin:

```
> cargo new nu_plugin_dec
> cd new_plugin_dec
```

Next, we'll add `nu` to the list of dependencies to the Cargo.toml directory.  At the bottom of the new Cargo.toml file, add this new dependency on the `nu` crate:

```
[dependencies]
nu = "0.2.0"
```

With this, we can open up src/main.rs and create our plugin.

```rust
use nu::{
    serve_plugin, CallInfo, Plugin, Primitive, ReturnSuccess, ReturnValue, ShellError, Signature,
    Tagged, Value,
};

struct Len;

impl Len {
    fn new() -> Len {
        Len
    }

    fn len(&mut self, value: Tagged<Value>) -> Result<Tagged<Value>, ShellError> {
        match value.item {
            Value::Primitive(Primitive::String(s)) => Ok(Tagged {
                item: Value::int(s.len() as i64),
                tag: value.tag,
            }),
            _ => Err(ShellError::labeled_error(
                "Unrecognized type in stream",
                "'len' given non-string by this",
                value.tag.span,
            )),
        }
    }
}

impl Plugin for Len {
    fn config(&mut self) -> Result<Signature, ShellError> {
        Ok(Signature::build("len").filter())
    }

    fn begin_filter(&mut self, _: CallInfo) -> Result<Vec<ReturnValue>, ShellError> {
        Ok(vec![])
    }

    fn filter(&mut self, input: Tagged<Value>) -> Result<Vec<ReturnValue>, ShellError> {
        Ok(vec![ReturnSuccess::value(self.len(input)?)])
    }
}

fn main() {
    serve_plugin(&mut Len::new());
}
```

There are a few moving parts here, so let's break them down one by one.

First off, let's look at main:

```rust
fn main() {
    serve_plugin(&mut Len::new());
}
```

In main, we just call a single function `serve_plugin`. This will do the work of calling into out plugin, handling the JSON serialization/deserialization, and sending values and errors back to Nu for us.  To start it up, we pass it something that implements the `Plugin` trait.

Next, above main, is this implementation of the `Plugin` trait for our particular plugin.  Here, we'll implement the Plugin trait for our type, Len, which we'll see more of soon.  Let's take a look at how we implement this trait:

```rust
impl Plugin for Len {
    fn config(&mut self) -> Result<Signature, ShellError> {
        Ok(Signature::build("len").filter())
    }

    fn begin_filter(&mut self, _: CallInfo) -> Result<Vec<ReturnValue>, ShellError> {
        Ok(vec![])
    }

    fn filter(&mut self, input: Tagged<Value>) -> Result<Vec<ReturnValue>, ShellError> {
        Ok(vec![ReturnSuccess::value(self.len(input)?)])
    }
}
```

The two most important parts of this implementation are the `config` part, which is run by Nu when it first starts up. This tells Nu the basic information about the plugin: its name, the parameters it takes, and what kind of plugin it is. Here, we tell Nu that the name is "len" and we are a filter plugin (rather than a sink plugin).

Next, in the `filter` implementation, we describe how to do work as values flow into this plugin.  Here, we receive one value (a Tagged<Value>) at a time. We also return either a Vec of values or an error.  Return a vec instead of a single value allows us to remove values, or add new ones, in addition to working with the single value coming in.
  
Because the `begin_filter` doesn't do anything, we can remove it.  This would make the above:

```rust
impl Plugin for Len {
    fn config(&mut self) -> Result<Signature, ShellError> {
        Ok(Signature::build("len").filter())
    }

    fn filter(&mut self, input: Tagged<Value>) -> Result<Vec<ReturnValue>, ShellError> {
        Ok(vec![ReturnSuccess::value(self.len(input)?)])
    }
}
```

If that's the case, why have a `begin_filter`?  Let's look at the signature of `begin_filter` a little closer:

```rust
fn begin_filter(&mut self, _: CallInfo) -> Result<Vec<ReturnValue>, ShellError> {
    Ok(vec![])
}
```

Our `Len` command doesn't require any parameters, but if it did this is where we'd get them. From here, we could configure our filter, and then use that with each step in of the `filter` command over the input.

Next, let's look at `Len` itself to see what it's doing:

```rust
struct Len;

impl Len {
    fn new() -> Len {
        Len
    }

    fn len(&mut self, value: Tagged<Value>) -> Result<Tagged<Value>, ShellError> {
        match value.item {
            Value::Primitive(Primitive::String(s)) => Ok(Tagged {
                item: Value::int(s.len() as i64),
                tag: value.tag,
            }),
            _ => Err(ShellError::labeled_error(
                "Unrecognized type in stream",
                "'len' given non-string by this",
                value.tag.span,
            )),
        }
    }
}
```

We create a very simple `Len`, in fact, it has no structure at all. Instead, it's just a placeholder that will let us implement the plugin.

From here, we create two methods:

```rust
impl Len {
    fn new() -> Len {
        Len
    }
    // ...
}
```

The first method is optional, it's just a convenient way to create a new value of the `Len` type.  The real work is done in the second method:

```rust
impl Len {
    // ...
    
    fn len(&mut self, value: Tagged<Value>) -> Result<Tagged<Value>, ShellError> {
        match value.item {
            Value::Primitive(Primitive::String(s)) => Ok(Tagged {
                item: Value::int(s.len() as i64),
                tag: value.tag,
            }),
            _ => Err(ShellError::labeled_error(
                "Unrecognized type in stream",
                "'len' given non-string by this",
                value.tag.span,
            )),
        }
    }
}
```

This method will act over each element in the pipeline as it flows into our plugin.  For our plugin, we really only care about strings so that we can return the length of them.

We use Rust's pattern matching to check the type of the Value coming in, and then operate with it if it's a string.  The value is a Tagged<Value> so it carries with it where the value came from.  If the value isn't a string, we give an error and let the user know where the value came from that is causing the problem.  (Note, if we had wanted to also put an error underline under the command name, we could get the `name_span` from the CallInfo given to `begin_filter`)
    
Lastly, let's look at the top of the file:

```rust
use nu::{
    serve_plugin, CallInfo, Plugin, Primitive, ReturnSuccess, ReturnValue, ShellError, Signature,
    Tagged, Value,
};
```

Here we import everything we need -- types and functions -- to be able to create our plugin.

Once we have finished our plugin, to use it all we need to do is install it.  Once `nu` starts up, it will discover it and register it as a command we can use:

```
> cargo install --path .
> nu
```

