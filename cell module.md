Rust安全机制是给定一个T类型的对象，它只能存在以下两种情形中的一种：
- 有多个指向它的不可变引用（&T)
- 至多有一个指向它的可变引用（&mut T)，此属性具有排它性

以上规则会被Rust Complier强制执行，但是这样有时候就会显得不太灵活，因为可能会有这样的情况：T类型的对象已经有（多个）指向它的不可变引用（&T），但是我们还是想改变它的值，此时就得需要一个指向它的可变引用（&mut T），明显Rust编译器是不允许我们这样做的，那该怎么办呢？
幸运的是，Rust为我们提供了一种叫做共享的可变的容器，使我们以一种可控的方式来改变引用的值。我们称这种为“**内部可变性**”，它有两种类型：Cell<T> 和 RefCell<T>， 有了它们，我们就可以改变的引用（&T）类型的值，而不需要可变的引用（&mut T）类型。
Cell<T>提供了下面一些方法：

- 如果类型T实现了Copy， **get**方法获取内部的值， **update**方法提供了使用函数来更新内部的值， 并且返回新的值。
```
  impl<T: Copy> Cell<T> {
   pub fn get(&self) -> T {
          // SAFETY: This can cause data races if called from a separate thread,
          // but `Cell` is `!Sync` so this won't happen.
          unsafe { *self.value.get() }
      }
   pub fn update<F>(&self, f: F) -> T
      where
          F: FnOnce(T) -> T,
      {
          let old = self.get();
          let new = f(old);
          self.set(new);
          new
      } 
  }    
```
- 如果类型T被 `?Sized` 界定， `as_ptr`返回指向内部值的原始指针， `get_mut`返回内部值的可变引用， `from_mut`方法会生成一个&Cell\<T\>，其参数的类型是&mut T.
```
  impl<T: ?Sized> Cell<T> {
     pub const fn as_ptr(&self) -> *mut T {
        self.value.get()
    }
    pub fn get_mut(&mut self) -> &mut T {
        self.value.get_mut()
    }
    pub fn from_mut(t: &mut T) -> &Cell<T> {
        // SAFETY: `&mut` ensures unique access.
        unsafe { &*(t as *mut T as *const Cell<T>) }
    }
 } 
```
- 如果类型实现了Default，`take`方法用类型的`default()`替换Cell的内部值，并返回原来的值。
```
  impl<T: Default> Cell<T> {
     pub fn take(&self) -> T {
        self.replace(Default::default())
    }
  }
```
- 对于其它所有的类型，`new`方法创建一个给定值的新的Cell， `set`方法用给定的值更新内部值，并`drop`原来的值， `swap`方法会交换两个Cell的内部的值，如果两个Cell引用的相同的对象，就什么也不做，`as_slice_of_cells` 返回一个 `&[Cell<T>]` 类型的对象，其参数是 `&Cell<[T]> `类型
```
  impl<T> Cell<T> {
    pub const fn new(value: T) -> Cell<T> {
        Cell { value: UnsafeCell::new(value) }
    }
    pub fn set(&self, val: T) {
        let old = self.replace(val);
        drop(old);
    }
    pub fn swap(&self, other: &Self) {
        if ptr::eq(self, other) {
            return;
        }
        // SAFETY: This can be risky if called from separate threads, but `Cell`
        // is `!Sync` so this won't happen. This also won't invalidate any
        // pointers since `Cell` makes sure nothing else will be pointing into
        // either of these `Cell`s.
        unsafe {
            ptr::swap(self.value.get(), other.value.get());
        }
    }
    pub fn as_slice_of_cells(&self) -> &[Cell<T>] {
        // SAFETY: `Cell<T>` has the same memory layout as `T`.
        unsafe { &*(self as *const Cell<[T]> as *const [Cell<T>]) }
    }
  }    
```
需要注意的是它们都是非多线程安全的，因为都没有实现Sync Trait： 
```
   impl<T: ?Sized> !Sync for Cell<T> {} 
``` 
```
  impl<T: ?Sized> !Sync for RefCell<T> {}
 ```
  
如果我们有一个
```
struct A {
  value: T
}
```
这里value是不可变的，但是有时候我们想改变其值，怎么办呢？

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

