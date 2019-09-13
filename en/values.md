# Values

A Value is the basic datatype in Nu. Currently, it covers the following possible value types:

```rust
pub enum Value {
    Primitive(Primitive),
    Row(crate::data::Dictionary),
    Table(Vec<Tagged<Value>>),

    Block(Block),
}
```

Where Primitive is:

```rust
pub enum Primitive {
    Nothing,
    Int(BigInt),
    Decimal(BigDecimal),
    Bytes(u64),
    Binary(Vec<u8>),
    String(String),
    Pattern(String),
    Boolean(bool),
    Date(DateTime<Utc>),
    Path(PathBuf),

    // Stream markers (used as bookend markers rather than actual values)
    BeginningOfStream,
    EndOfStream,
}
```

Let's look at these in reverse order to see how Primitives build towards our full Value types:

## Primitive

A Primitive data type is a fundamental type in Nu. While these have similarities to the fundamental datatypes of programming languages, there are a few differences because of the shell capabilities of Nu.

Nu comes with two 'big' number types: BigInt for integers and BigDecimal for decimal numbers. This allows Nu in the future to do mathematical operations and maintain precision for longer.

Other data types that are perhaps a bit different from the norm:

* Bytes(u64) = filesize in number of bytes
* Binary(Vec<u8>) = an array of bytes
* Pattern(string) = a glob pattern (like the `nu*` in `ls nu*`)
* Path(PathBuf) = a filepath
* BeginningOfStream = a marker to denote the beginning of a stream
* EndOfStream = a marker to denote the end of a stream

## Value

In addition to the primitive types, Nu supports aggregate data types. Collectively, these aggregate types are called Values.

Currently, Nu supports 4 Value types: Row, Table, Block, and Primitive.

### Tables and Rows

Nu uses a set of terms that match a bit more closely to spreadsheets. Rather than having lists of objects, Nu has a table, which contains rows. Each Row contains contains the column names and their corresponding Values.

### Blocks

Blocks represent code that is ready to be executed by the evaluator. One example of this is the condition in `where {$it > 10}`. 
