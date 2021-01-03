**[Sized](https://doc.rust-lang.org/std/marker/trait.Sized.html)** 表示类型在编译时就能确定其大小。

所有的类型参数都会有（实现）隐式的**Sized**界定，你可以使用语法<strong>?Sized</strong>来移除此界定。

```
struct Foo<T>(T);
struct Bar<T: ?Sized>(T);

// struct FooUse(Foo<[i32]>); // error: Sized is not implemented for [i32]
struct BarUse(Bar<[i32]>); // OK
```

有一个类外是trait的Self类型，它不会有隐式的Sized界定，因为这样会与trait object不兼容。
不过Rust允许你给trait添加Sized界定，但是之后你不能有它的trait object了。

```
trait Foo { }
trait Bar: Sized { }

struct Impl;
impl Foo for Impl { }
impl Bar for Impl { }

let x: &dyn Foo = &Impl;    // OK
// let y: &dyn Bar = &Impl; // error: the trait `Bar` cannot
                            // be made into an object
```
