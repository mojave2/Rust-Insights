- [`Box`](#box)
- [`Rc` and `Arc`](#rc-and-arc)
- [`alloc` module](#alloc-module)
- [`vec` module](#vec-module)
  - [`Vec` struct: a contiguous growable array type.](#vec-struct-a-contiguous-growable-array-type)

The `alloc` library provides smart pointers and collections for managing heap-allocated values.

## `Box`

- 根据文档 `Box` Type 是一个 smart pointer type, 但将 `Box` 理解成一个 container 会更加形象。当使用 `Box::new` 来创建一个 pointer value 时，Rust 会在 heap 上分配内存存储 contained content，并在 stack 上创建一个 owning pointer，通常把这个 stack pointer 称为一个 `Box` value。和普通 value 一样 Box value 也只能有唯一的 owner。
- 由于 Rust 在编译阶段需要知道每个数据类型的内存大小，这给递归形式的数据结构的定义带来了很大困难。递归形式数据结构的典型例子就是链表的节点，节点类型的 fields 包含了节点类型自身。`Box` 可以很好地解决这个问题，通过为递归的 field 提供一层封装。
- 在概念理解上 Box 和普通的 value 差不多，只是存储机制不同；而在使用上 Box 与 Reference 基本一样，比如 `mut Box<String>` 和 `&mut String`。

## `Rc` and `Arc`

- `Rc/Arc` 在内部实现原理上和 `Box` 很像，也可以理解成一个 container。在使用 `Rc::new` 或 `Arc::new` 创建一个对应的 pointer value 时，Rust 都会在 Heap 上分配内存存储 contained content，在 stack 上创建一个 owning pointer。
  - 但和 `Box` 不同的是 `Rc/Arc` pointer 可以有多个 owner，共同 share ownership。当所有的 owner 对被 drop 之后，heap space 才会释放。
  - 在通过 clone 创建新的 shared owners 时，会在 stack 上创建一个新的 pointer，其指向同一个 heap container。
  - Rust 是内存和线程安全的，由于 `Rc/Arc` 有多个 owner，所以 `Rc/Arc` 是 immutable 的。
  - `Rc` 和 `Arc` 类型基本一致，唯一差别就是 `Arc` 可以在线程间共享，而 `Rc` 只能在一个线程内共享。
- 软件设计中 `Rc/Arc` 常用来定义公共数据，然后在多个主体（如线程）共同持有。`Rc/Arc` 类型也常常和具有 interior mutability 的类型（如 `Cell`, `RefCell`, `Mutex`, Atomics）一起使用，可以在多个主体之间进行同步数据。
  - Interior mutability 简单理解就是一个变量没有使用 `mut` 定义，但其内容却依然是 mutable 的。
- 在一般的使用上，Rc 与 reference 没有太大差别，比如一个 `Rc<String>` 和 `&String` 一样使用。

Inherited mutability

## `alloc` module

- 这个 module 包含了一些 memory allocation APIs, 比较 low-level, 不直接使用。
- `alloc` 在 heap 上分配一块指定 layout 的内存
- `dealloc` 在 heap 上指定位置上释放指定 layout 的内存
- `realloc` 在 heap 上已分配位置处，改变原有分配内存的大小 (shrink or grow)

    ```rust
    use std::alloc::{alloc, dealloc, Layout};

    unsafe {
        let layout = Layout::new::<u16>();
        let ptr = alloc(layout);

        *(ptr as *mut 16) = 42;
        assert_eq!(*(ptr as *mut 16), 42);

        dealloc(ptr, layout);
    }
    ```

## `vec` module

- 这个模块定义了一些与 Vector 相关的 structs。
- Vector 是特定类型 `T` 的数据集，这些 `T` values 连续地分布在一片 heap 内存上。Vector 的大小是动态的，可以不断插入或抛出 `T` values。
- Vector 分配的内存空间，会随着 value pushing 而动态增长，但不会随 value poping 而动态收缩。
- Vector 的索引效率是 `O(1)`, 尾部 pushing/poping value 是 `O(1)`。

### `Vec` struct: a contiguous growable array type.

- `pub struct Vec<T, A: Allocator = Global> { /* private fields */ }`
- `Vec::new()`, `Vec::with_capacity(usize)`
- `vec.push(T)`, `vec.extend(iter: T)`
- `vec.pop()`
- `vec.len()`
