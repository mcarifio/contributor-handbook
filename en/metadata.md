# Metadata

All values that flow into and out of commands in Nu is tagged with metadata.  You'll see this commonly around the codebase as `Tagged<Value>`. 

Though the metadata that is tracked is still in its early days, we expect to expand this as Nu matures.

Currently, there are two pieces of metadata tracked on each value:

## Origin

Origin represents the location that the value came from originally. If the value was loaded from a file, it will be the filename. If it was loaded from a Url, it will be the url, and so on.

To save on space, and to keep metadata Copy-able, the origin is stored as a UUID. This identifer points to a lookup table where the UUID can be translated into the full origin.

## Span

A span are the start and ending location of value that was created or referenced on the commandline. These are most commonly seen represented as the underline underneath an error message.

While spans from programming languages traditionally also carry the file the span came from, here we assume the span always spans a value referenced on the commandline rather than in a source file. As Nu gets the ability to run its own source files, this will likely need to be revisited.
