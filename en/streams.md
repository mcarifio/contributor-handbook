# Streams

Async streams form the foundation for how information flows from one end of the pipeline to the other. This allows nu to work with internal commands, external commands, and plugins in a way that's relatively seamless. 

There are two fundamental types for streams in Nu: InputStream and OutputStream

## InputStream

Let's look at the InputStream type a little closer:

```
BoxStream<'static, Tagged<Value>>
```

That is, it's an async stream who will send Tagged<Value>
