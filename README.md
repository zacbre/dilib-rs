# dilib-rs

A dependency injection library for Rust.

## Usage
```toml
[dependencies]
dilib = "0.1.3-alpha"
```

## Example

### Basic Usage

```rust
use std::sync::Mutex;
use dilib::*;

struct GetNumber(u32);

struct Counter {
    val: u32,
}

let mut container = Container::new();
container.add_scoped(|| String::from("Hello, world!"));
container.add_scoped_with_name("42", || GetNumber(42));
container.add_scoped_with_name("102", || GetNumber(102));
container.add_singleton(Mutex::new(Counter { val: 0 }));

let mut counter = container.get_singleton::<Mutex<Counter>>().unwrap();
counter.lock().unwrap().val = 5;

assert_eq!(container.get_scoped::<String>().unwrap(), "Hello, world!".to_string());
assert_eq!(container.get_scoped_with_name::<GetNumber>("42").unwrap().0, 42);
assert_eq!(container.get_scoped_with_name::<GetNumber>("102").unwrap().0, 102);
assert_eq!(container.get_singleton::<Mutex<Counter>>().unwrap().lock().unwrap().val, 5);
```

### With derive
Requires `derive` feature.

```rust
use std::sync::Mutex;
use dilib::*;

#[derive(Inject)]
struct Counter {
    #[inject(default="Counter")]
    label: String,
    val: Singleton<Mutex<usize>>
}

let mut container = Container::new();
container.add_singleton(Mutex::new(0_usize));
container.add_deps::<Counter>();

let c1 = container.get_scoped::<Counter>().unwrap();
*c1.val.lock().unwrap() = 12;

let c2 = container.get_scoped::<Counter>().unwrap();
assert_eq!(c1.label, c2.label);
assert_eq!(*c2.val.lock().unwrap(), 12);
```
