## 什么是slice：
> slice 是一块内存的视图，表示为一个指针（pointer)和长度(length). 这块内存是连续的，`连续的`意味着其中的每个元素与其相邻的元素要保持相同的距离被存放。

## slice有几种形式：
- shared type: &[T]
- mutated type: &mut [T]
