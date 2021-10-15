# lecture-7-notes

### Enum example

To implement this data structure, we can first define the following enum.

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

Then we can create our list as shown below.

```rust
use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

When compiling this, we get an error. Why? Rust can’t figure out how much space to allocate for recursively defined types.

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

### Deref example

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

Here we have a basic struct MyBox, for which the new method creates a new box with an element of type T. We can allow this type to be dereferenced by implementing the `Deref` trait.

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

This would allow us to compile the following example.

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Behind the scenes in this example, what is actually run is this.

```rust
*(y.deref())
```

### Drop example

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("End of main");
}
```

When running this example, Rust automatically called drop for us when our instances went out of scope, calling the code we specified. Variables are dropped in the reverse order of their creation, so d was dropped before c. This example gives you a visual guide to how the drop method works; usually you would specify the cleanup code that your type needs to run rather than a print message.

You can also call drop manually.

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    c.drop();
    println!("End of main");
}
```

### Rc example 1

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

