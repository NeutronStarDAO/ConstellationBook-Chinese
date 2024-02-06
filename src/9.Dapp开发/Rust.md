## 1 小时入门 Rust：屏蔽复杂语法，直接在开发中学习

<br>

### 结构体

在 Rust 中，结构体（structure）是用来定义自定义数据类型的一种方法。结构体可以包含多个不同类型的数据字段，并且每个字段都可以被访问和操作。结构体在 Rust 中用于创建复杂的数据类型，例如，它们可以表示面向对象编程中的类，并且也可以实现多重继承。

结构体在 Rust 中的作用与 Java 的类类似，都可以用来描述对象，封装数据和方法。不过 Rust 的结构体与 Java 的类的语法和实现有所不同，比如 Rust 中的结构体可以选择实现关联函数，而 Java 的类则不能。

<br>

### Vec

`Vec` 是 Rust 中的动态数组类型，它允许在运行时增加或删除元素。它的大小可以随着需求的增加而自动增加，而且也可以随着元素的删除而自动减小。它提供了许多方便的方法，如 push，pop，iterate 等，方便处理集合数据类型。

<br>

### 关联函数

在 Rust 中，关联函数是使用 `impl` 关键字声明的静态函数，它不依赖于任何具体实例。它们通常用于创建一个新的实例或返回一个常量，例如：

```rust
struct Circle {
    radius: f64,
}

impl Circle {
    fn new(radius: f64) -> Self {
        Circle { radius }
    }

    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}
```

在上面的代码片段中，`new` 函数是一个关联函数，它创建一个新的圆形实例，而 `area` 函数是一个实例函数，它接受一个圆形实例作为参数。

<br>

### 实例

在 Rust 中，一个实例就是结构体或者枚举的一个具体的对象，可以通过它来调用方法，访问字段等。

例如：

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };
    println!("The area of the rectangle is {} square pixels.", rect.area());
}
```

这里我们定义了一个名为`Rectangle`的结构体，其中有两个字段 `width` 和 `height`。然后我们实现了一个名为 `area` 的方法，该方法可以返回矩形的面积。最后我们在 `main` 函数中创建了一个矩形实例，并使用它调用了 `area` 方法。

<br>

### 枚举

Rust 的枚举是一种可以将多个具有相关性的命名值组合在一起的数据类型。枚举中的每个命名值称为枚举项，可以是一个整数值或其他类型的值。枚举在 Rust 中是一种很强大的数据类型，可以让你在代码中更容易表达概念和意图。

举个例子，你可以定义一个枚举来表示星期：

```rust
enum Weekday {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}
```

这个枚举有7个枚举项，每个枚举项都代表了一个星期中的一天。你可以使用枚举项来表示一个星期中的某一天：

```rust
let today = Weekday::Monday;
```

<br>
