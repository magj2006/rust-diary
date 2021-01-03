Rust安全机制是给定一个对象T，它只能存在以下两种情形中的一种：
- 多个指向它的不可变引用（&T)
- 至多一个指向它的可变引用（&mut T)，此属性具有排它性

以上规则会被Rust Complier强制执行，但是这样就会不太灵活，因为可能会遇到对象T已经有（多个）不可变引用（&T），但是我们还想改变它，想有一个它的可变引用（&mut T）的情形，这时该怎么办呢？
幸运的是，Rust为我们提供了共享的可变的容器。Cell<T> 和 RefCell<T> 就是为此场景而生，它们提供了一种“**内部可变**”的机制，你可以通过共享引用来改变它们类型的值。
需要注意的是它们都是非多线程安全的，因为都没有实现Sync Trait： ```impl<T: ?Sized> !Sync for Cell<T> {}``` ```impl<T: ?Sized> !Sync for RefCell<T> {}```

如果我们有一个```struct A {
  value: T
}```， 这里value是不可变的，但是有时候我们想改变其值，怎么办呢？

所以如果你想把&T变成为&mut T，被认为是一种未定义的行为。
幸运的是，标准库提供了一个UnsafeCell Trait， 通过用它封装我们的T，就可以实现&T到&mut T的转变。
如标准库中Cell<T> RefCell<T> 内部都是通过定义一个UnsafeCell<T>来实现内部可变性。
```
pub struct Cell<T: ?Sized> {
    value: UnsafeCell<T>,
}
```
```
pub struct RefCell<T: ?Sized> {
    borrow: Cell<BorrowFlag>,
    value: UnsafeCell<T>,
}
```

