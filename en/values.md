# Values

A Value is the basic datatype in Nu. Currently, it covers the following possible value types:

```rust
pub enum Value {
    Primitive(Primitive),
    Row(crate::data::Dictionary),
    Binary(Vec<u8>),
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

Nu comes with two 'big' number types: BigInt for integers and BigDecimal for decimal numbers.
