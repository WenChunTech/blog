# Rust偏僻知识点


<!--more-->
# Rust偏僻知识点

## Cow(Clone on Write)

两个可选值：

- `Borrowed`，用于包裹对象的引用（通用引用）；
- `Owned`，用于包裹对象的所有者；

`Cow` 提供

1. 对此对象的不可变访问（比如可直接调用此对象原有的不可变方法）；
2. 如果遇到需要修改此对象，或者需要获得此对象的所有权的情况，`Cow` 提供方法做克隆处理，并避免多次重复克隆。

`Cow` 的设计目的是提高性能（减少复制）同时增加灵活性，因为大部分情况下，业务场景都是读多写少。利用 `Cow`，可以用统一，规范的形式实现，需要写的时候才做一次对象复制。这样就可能会大大减少复制的次数。

它有以下几个要点需要掌握：

1. `Cow<T>` 能直接调用 `T` 的不可变方法，因为 `Cow` 这个枚举，实现了 `Deref`；
2. 在需要写 `T` 的时候，可以使用 `.to_mut()` 方法得到一个具有所有权的值的可变借用；注意，调用 `.to_mut()` 不一定会产生克隆；
   -  在已经具有所有权的情况下，调用 `.to_mut()` 有效，但是不会产生新的克隆；
   - 多次调用 `.to_mut()` 只会产生一次克隆。

3. 在需要写 `T` 的时候，可以使用 `.into_owned()` 创建新的拥有所有权的对象，这个过程往往意味着内存拷贝并创建新对象；
   - 如果之前 `Cow` 中的值是借用状态，调用此操作将执行克隆；
   - 本方法，参数是`self`类型，它会“吃掉”原先的那个对象，调用之后原先的对象的生命周期就截止了，在 `Cow` 上不能调用多次；

举例:写一个函数，过滤掉输入的字符串中的所有空格字符，并返回过滤后的字符串。

```rust
use std::borrow::Cow;

fn remove_spaces<'a>(input: &'a str) -> Cow<'a, str> {
    if input.contains(' ') {
        let mut buf = String::with_capacity(input.len());

        for c in input.chars() {
            if c != ' ' {
                buf.push(c);
            }
        }

        return Cow::Owned(buf);
    }

    return Cow::Borrowed(input);
}
```



## HRTBS(Higher-Ranked Trait Bounds)

1. HRTBS主要用于解决函数参数含有闭包,并且闭包参数含有引用
2. 语法:for<'a> T: Trait<'a>

例如如下代码实际是不能编译通过的,因为编译器不能推断出返回哪个引用

```rust
fn call_on_ref_zero<F>(f: F) where F: Fn(&i32, &i32) -> &i32 {
    let zero = 0;
    f(&zero, &zero);
}
```

尝试添加生命周期参数,还是不能通过编译,因为zero变量的生命周期是短于'a的.

```rust
fn call_on_ref_zero<'a, F>(f: F) where F: Fn(&'a i32, &'a i32) -> &'a i32 {
    let zero = 0;
    f(&zero, &zero);
}
```

我们尝试使用HRTBS试试

```rust
fn call_on_ref_zero<F>(f: F) where for<'a> F: Fn(&'a i32, &'a i32) -> &'a i32 {
    let zero = 0;
    f(&zero, &zero);
}
```

另一个例子

```rust
fn foo<'b, F>(f: F) -> &'b str
where
    for<'a> F: Fn(&'a str) -> &'a str,
{
    let s = "hello";
    f(s)
}

fn bar(s: &str) -> &str {
    &s[..1]
}

fn main() {
    let s = foo(bar);
    println!("{s}");
}
```

最后一个例子:

```rust
fn foo<'a>(f: Box<dyn Fn(&'a i32)>) {
    let x = 1;
    f(&x);
	{
		let y = 2;
		f(&y);
	}
}
```

第一次调用 `f(&x)` 时生命周期 `'a` 等于变量 `x` 的生命周期；而在第二次调用 `f(&y)` 时，生命周期 `'a` 又等于了变量 `y` 的生命周期；而变量 `x` 和变量 `y` 的生命周期显然是不同的。因此无法用一个静态的生命周期来描述 `'a` ，我们希望的是，闭包 `f` 在具体调用时绑定具体的生命周期，比如调用 `f(&x)` 时绑定的是 `x` 的生命周期，而调用 `f(&y)` 时绑定的是 `y` 的生命周期。

使用HRTBS修改如下:

```rust
fn foo(f: Box<dyn for<'a> Fn(&'a i32)>) {
    let x = 1;
    f(&x);
	{
		let y = 2;
		f(&y);
	}
}
```

这样生命周期 `'a` 就不再是静态的了，他会随着闭包 `f` 的调用绑定到不同的生命周期：`f(&x)` 调用时绑定到 `x` 的生命周期，`f(&y)` 调用时绑定到 `y` 的生命周期。

## PhantomData

PhantomData主要用于无界生命周期(unbounded lifetime)或者需要drop struct非自身字段,例如:

```rust
use std::marker:

struct Vec<T> {
    data: *const T, // *const是可变的！
    len: usize,
    cap: usize,
    _marker: marker::PhantomData<T>,
}
```

`PhantomData`模式表

| Phantom 类型                | `'a` | `'T`                   |
| :-------------------------- | :--- | :---------------------|
| `PhantomData<T>`            | -    | 协变（可触发drop检查）    |
| `PhantomData<&'a T>`        | 协变 | 协变                    |
| `PhantomData<&'a mut T>`    | 协变 | 不变                    |
| `PhantomData<*const T>`     | -    | 协变                   |
| `PhantomData<*mut T>`       | -    | 不变                   |
| `PhantomData<fn(T)>`        | -    | 逆变(*)                |
| `PhantomData<fn() -> T>`    | -    | 协变                   |
| `PhantomData<fn(T) -> T>`   | -    | 不变                   |
| `PhantomData<Cell<&'a ()>>` | 不变 | -                      |

(*)如果发生变性的冲突，这个是不变的

我们可以使用泛型结构体来实现对同一种类对象不同子类对象的区分，例如，我们的系统中要设计这样一个功能，将用户分为免费用户和付费用户，而且免费用户在体验免费功能之后，如果想升级成付费用户也是可以的。按照我们常规的思维，可能是定义两个结构体 `FreeCustomer` 以及 `PaidCustomer`，但是我们可以通过泛型结构体来实现，例如：

```
struct Customer<T> {
    id: u64,
    name: String,
}
```

不过，我们这里的 `T` 又无处安放，所以又不得不使用 `PhantomData`，它就像一个占位符，但是又没有大小，可以为我们持有在声明时使用不到的数据：

```rust
use std::{
    marker::PhantomData,
    sync::atomic::{self, AtomicU64},
};

static NEXT_ID: AtomicU64 = AtomicU64::new(0);

struct Customer<T> {
    id: u64,
    name: String,
    phantom: PhantomData<T>,
}

struct FreeFeature;
struct PaidFeature;

trait Free {
    fn feature1(&self);
    fn feature2(&self);
}

trait Paid: Free {
    fn paid_feature(&self);
}

/// 为 Customer<T> 实现需要的方法
impl<T> Customer<T> {
    fn new(name: String) -> Self {
        Self {
            id: NEXT_ID.fetch_add(1, atomic::Ordering::Relaxed),
            name,
            phantom: PhantomData,
        }
    }
}

/// 免费用户可以升级到付费用户
impl Customer<FreeFeature> {
    fn advance(self, payment: f64) -> Customer<PaidFeature> {
        println!(
            "{}（{}） 将花费 {:.2} 元升级到付费用户",
            self.name, self.id, payment
        );
        self.into()
    }
}

/// 所有客户都有权使用免费功能
impl<T> Free for Customer<T> {
    fn feature1(&self) {
        println!("{} 正在使用免费功能一", self.name)
    }

    fn feature2(&self) {
        println!("{} 正在使用免费功能二", self.name)
    }
}

/// 付费用户才能使用的功能
impl Paid for Customer<PaidFeature> {
    fn paid_feature(&self) {
        println!("{} 正在使用付费功能", self.name)
    }
}

/// 允许使用免费用户转换成付费用户
impl From<Customer<FreeFeature>> for Customer<PaidFeature> {
    fn from(c: Customer<FreeFeature>) -> Self {
        Self::new(c.name)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_customer() {
        // 一开始是免费用户
        let customer = Customer::<FreeFeature>::new("MichaelFu".to_owned());
        customer.feature1();
        customer.feature2();

        // 升级成付费用户，可能使用付费功能和普通功能
        let customer = customer.advance(99.99);
        customer.feature1();
        customer.feature2();
        customer.paid_feature();
    }
}

```



## Thread_local

thread_local是一种将数据存储到全局变量中的方法，程序中的每个线程都有自己的副本。线程不共享这个数据，所以访问不需要同步,thread_local中声明的变量，在线程结束后会被释放，比如如果有10个线程中使用了该thread_local变量,则这10个线程结束时将有10个该类型的变量被释放。这适用于thread_per_core架构的异步运行时(类似于nginx),而不是tokio这种work-stealing scheduler(任务窃取).

举个例子:

```rust
use std::borrow::Cow;
use std::cell::RefCell;
use std::thread;

fn main() {
    thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

    FOO.with(|f| {
        assert_eq!(*f.borrow(), 1);
        *f.borrow_mut() = 2;
    });

    // 每个线程开始时都会拿到线程局部变量的FOO的初始值
    let t = thread::spawn(move || {
        FOO.with(|f| {
            assert_eq!(*f.borrow(), 1);
            *f.borrow_mut() = 3;
        });
    });

    // 等待线程完成
    t.join().unwrap();

    // 尽管子线程中修改为了3，我们在这里依然拥有main线程中的局部值：2
    FOO.with(|f| {
        assert_eq!(*f.borrow(), 2);
    });
}

```



## Barrier(线程屏障)

使用 `Barrier` 让多个线程都执行到某个点后，才继续一起往后执行：

```rust
use std::sync::{Arc, Barrier};
use std::thread;

fn main() {
    let mut handles = Vec::with_capacity(5);
    let barrier = Arc::new(Barrier::new(5));

    for _ in 0..5 {
        let b = barrier.clone();
        handles.push(thread::spawn(move|| {
            println!("before wait");
            b.wait();
            println!("after wait");
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }
}

```

## once_cell和lazy_static

once_cell 和 lazy_static 都是 Rust 中用于实现单例模式（Singleton）的库。

1. once_cell 适用于程序初始化加载配置文件这种场景
2. LazyCell和LazyLock适用于初始化某个变量,比如说正则的compile,数据库连接等场景.

```rust
#![feature(once_cell)]
use std::cell;
use std::collections::HashMap;
use std::sync;
use std::sync::Once;

static INIT: Once = sync::Once::new();
static mut SUM: u64 = 0;

fn init_sum() -> u64 {
    unsafe {
        INIT.call_once(|| {
            println!("this is first init");
            SUM = (1..100000).sum();
        });
        SUM
    }
}

fn main() {
    let split_line = "*".repeat(100);
    let first_result = init_sum();
    println!("first result: {first_result}");
    let second_result = init_sum();
    println!("second result: {second_result}");

    println!("{split_line}");

    let once = cell::OnceCell::<HashMap<&str, &str>>::new();
    let dict = once.get_or_init(|| {
        println!("this is hash map init once");
        let mut map = HashMap::new();
        map.insert("lang", "rust");
        map.insert("edition", "2021");
        map
    });

    println!("once dict: {dict:?}");

    let dict_twice = once.get_or_init(|| {
        println!("this is hash map init twice");
        let mut map = HashMap::new();
        map.insert("lang", "python");
        map.insert("version", "1.10");
        map
    });
    println!("twice dict: {dict_twice:?}");

    println!("{split_line}");

    let lazy_init = cell::LazyCell::new(|| env!("PATH"));
    println!("lazy_init: {}", *lazy_init);

    // thread safe lazy cell
    let lazy_init_safe = sync::LazyLock::new(|| env!("PATH"));

    print!("lazy_init_safe: {}", *lazy_init_safe);

}

```

## impl A and T: A

在 impl 中被声明的类型参数，至少要满足下面三种形式：

1. impl<T> Foo<T>， T 出现在实现的Self 类型Foo<T> 中 。
2. impl<T> SomeTrait<T> for Foo ， T出现在要实现的 trait 中 。
3. impl<T, U> SomeTrait for T where T: AnotherTrait<AssocType=U> ， 出现在 T 的 trait 限定的关联类型中。

**参考资料:**

- [【Rust】幽灵数据（PhantomData） | MichaelFu (fudenglong.site)](https://blog.fudenglong.site/2022/05/17/Rust/phantom-data/)
- [3.10 PhantomData（幽灵数据） | 第三章、所有权 |《Rust 高级编程 2018》| Rust 技术论坛 (learnku.com)](https://learnku.com/docs/nomicon/2018/310-phantom-data/4721)
- [Rust高阶生命周期绑定 (xiaopengli89.github.io)](https://xiaopengli89.github.io/posts/rust-hrtbs/)

---

> 作者:   
> URL: https://blog-12x.pages.dev/rust%E5%81%8F%E5%83%BB%E7%9F%A5%E8%AF%86%E7%82%B9/  

