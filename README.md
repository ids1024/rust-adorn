# rust-adorn

[![Build Status](https://travis-ci.org/Manishearth/rust-adorn.svg)](https://travis-ci.org/Manishearth/rust-adorn)

Python-style function decorators for Rust


Example usage:


```rust
#![feature(plugin, custom_attribute)]
#![plugin(adorn)]

#[adorn(bar)]
fn foo(a: &mut u8, b: &mut u8, (c, _): (u8, u8)) {
    assert!(c == 4);
    *a = c;
    *b = c;
}


fn bar<F>(f: F, a: &mut u8, b: &mut u8, (c, d): (u8, u8)) where F: Fn(&mut u8, &mut u8, (u8, u8)) {
    assert!(c == 0 && d == 0);
    f(a, b, (4, 0));
    *b = 100;
}

fn main() {
    let mut x = 0;
    let mut y = 1;
    foo(&mut x, &mut y, (0, 0));
    assert!(x == 4 && y == 100);
}
```

In this case, `foo` will become:

```rust
fn foo(a: &mut u8, b: &mut u8, (c, d): (u8, u8)) {
    fn foo_inner(a: &mut u8, b: &mut u8, (c, _): (u8, u8)) {
        assert!(c == 4);
        *a = c;
        *b = c;
    }
    bar(foo_inner, a, b, (c, d))
}
```

In other words, calling `foo()` will actually call `bar()` wrapped around `foo()`.


There is a `#[make_decorator]` attribute to act as sugar for creating decorators. For example,

```rust
#[make_decorator(f)]
fn bar(a: &mut u8, b: &mut u8, (c, d): (u8, u8)) {
    assert!(c == 0 && d == 0);
    f(a, b, (4, 0)); // `f` was declared in the `make_decorator` annotation
    *b = 100;
}

```

desugars to 

```rust
fn bar<F>(f: F, a: &mut u8, b: &mut u8, (c, d): (u8, u8)) where F: Fn(&mut u8, &mut u8, (u8, u8)) {
    assert!(c == 0 && d == 0);
    f(a, b, (4, 0));
    *b = 100;
}
```



I intend to add support for decorating impl items and default trait methods too. Also passing arguments to the decorator; though rustc's current metaitem support limits us to strings (varargs are possible though).