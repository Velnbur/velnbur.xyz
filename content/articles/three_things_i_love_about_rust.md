+++
title = "Three things I love about Rust's traits"
date = 2024-08-16
[extra]
toc = true
+++

Traits in Rust are superset of what is usually considered as "interfaces" in other languages.
Except, traits provide unique features which create new ways of solving common problems.
which I'll try to describe in this short article.

We'll go through the content by extending this definition of REST API client trait:

```rust
trait Client {
    /// Send data to `endpoint` with specified `method` and `body`.
    fn send(&self, method: &str, endpoint: &str, body: Vec<u8>);
}
```

# Traits vs Interfaces

The key feature and difference between typical "interfaces" is **default implementation**:

```rust
trait Client {
    fn send(&self, method: &str, endpoint: &str, _body: Vec<u8>) {
        println!("{method} {endpoint}");
    }
}
```

> This example is kind of obscure, so don't bother yourself thinking a lot about it.

This let's the end implementor use blank implementation for this trait:

```rust
impl Client for SomeClientImpl {}
```

Of course, the real example won't have default one, so let's assume we have
imlementation of `Client` using popular crate [`minreq`]:

```rust
struct MinreqClient;

impl Client for MinreqClient {
    fn send(&self, method: &str, endpoint: &str, body: Vec<u8>) {
        minreq::Request::new(method, endpoint)
            .with_body(body)
            .send()
            .unwrap()
    }
}
```

# Extension

One of the real examples of using **default implementation** is extension of trait's
methods:

```rust
trait Client {
    fn send(&self, method: &str, endpoint: &str, body: Vec<u8>);

    /// [`Self::send`] with empty body.
    fn send_with_empty_body(&self, method: &str, endpoint: &str) {
        self.send(method, endpoint, Vec::new())
    }
    // ^
    // |
    // - introduced method
}
```

As we added this method to all implementations (like for `MinreqClient` one),
this won't require others to change anything to all of them, and all of them
received new functionality for "free". Great examples of such pattern is
[`std::iter::Iterator`] which adds 75+ methods by implementing only `next`.

Also, this extension could be moved to another _"extension"_ trait:

```rust
trait ClientExt: Client {
    fn send_with_empty_body(&self, method: &str, endpoint: &str) {
        self.send(method, endpoint, Vec::new())
    }
}
```

And using generic implementation:

```rust
impl<T: Client> ClientExt for T {}
```

which litteraly means: _"implement `ClientExt` for anything that implements `Client`"_.
This pattern could be found in family of [`Stream`] traits ([`StreamExt`], [`TryStream`])
from [`futures`] crate.

# "Code generation"

This is just another practical usage of extension pattern, but I want you to
think about it in a way of automatic implementation of similiar methods for
different implementors.

Let's consider we have a REST API of pets:



[`futures`]: https://docs.rs/futures/latest/futures/index.html
[`minreq`]: https://docs.rs/minreq/latest/minreq/
[`std::iter::iterator`]: https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html
[`streamext`]: https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html
[`stream`]: https://docs.rs/futures/latest/futures/prelude/trait.Stream.html
[`trystream`]: https://docs.rs/futures/latest/futures/prelude/trait.TryStream.html
