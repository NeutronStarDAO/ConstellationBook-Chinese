## Motoko语言

Motoko 编程语言是一种新型、现代且类型安全的语言，适用于想在 IC 上构建下一代 DApp 的开发人员。Motoko 专门设计用于支持 IC 的独特功能并提供熟悉而强大的编程环境。作为一种新语言，Motoko 不断发展，支持新功能和其他改进。

Motoko 编译器、文档和其他工具都是[开源的](https://github.com/dfinity/motoko)。

Motoko 语言其主要有以下特点：

Motoko 使用的类型系统是静态类型系统，这意味着在编译时（而非运行时）就能够检查代码中的类型错误。以下是 Motoko 类型系统的一些特点和概念：

1. **[静态类型系统](https://internetcomputer.org/docs/current/motoko/main/motoko-introduction#types-are-static)：** Motoko 的类型系统是静态的，变量和表达式的类型在编译时已经确定，而不是在运行时。编译器可以在开发阶段捕获潜在的类型错误。Motoko 允许每个变量携带函数、对象或原始数据（例如，字符串、单词或整数）的值。Motoko 享有类型安全性，也称为类型完整性。也就是：[类型正确的 Motoko 程序不会出错](https://internetcomputer.org/docs/current/motoko/main/basic-concepts#type-soundness)。
2. **强类型：** Motoko 是强类型语言，即在编译时会强制执行类型规则，不允许隐式的类型转换。这样可以减少在运行时由于类型错误引起的问题。类型是 Motoko 表达式的一种承诺，从语言向开发者承诺程序的未来行为。在 Motoko 中的每个变量都有一个关联的类型，这个类型在程序执行之前就已经知道了。编译器会检查每个变量的使用，以防止运行时类型错误，包括空引用错误、无效字段访问等。
3. **类型推导（Type Inference）：** Motoko 支持类型推导，这意味着在很多情况下，开发者无需显式地注明变量的类型，编译器可以自动推断出类型。这有助于减少冗余的类型注释，同时保持类型安全。
4. **函数类型：** Motoko 是一种函数式编程语言，函数是一等公民（first-class citizens）。函数可以作为参数传递给其他函数，也可以作为返回值。函数的类型包括参数类型和返回类型。
5. **代数数据类型（Algebraic Data Types）：** Motoko 提供了代数数据类型，包括记录（records）和变体（variants）。记录用于表示有命名字段的结构化数据，而变体则用于表示具有不同构造的数据类型。这使得在代码中能够更清晰地表达数据结构和模式匹配。
6. **模式匹配：** 模式匹配是 Motoko 中处理复杂数据结构的一种强大方式。它用于根据数据的结构选择不同的执行路径。模式匹配在处理变体类型时尤为有用。增加了代码的可读性和表达能力。
7. **类型别名：** Motoko 允许开发者使用 `type` 关键字创建类型别名，开发者可以为复杂的类型定义一个简洁的名称，让代码更易于理解。
8. **接口（Interfaces）：** Motoko 提供了接口机制，允许开发者定义一组函数和属性，然后通过实现接口来确保对象符合特定的行为规范。这有助于实现代码的抽象和复用。

Motoko 的类型系统结合了函数式编程和现代静态类型语言的一些特性，旨在提供高度可读性、类型安全和灵活性。

<br>

### 原生地支持Canister智能合约

Motoko 原生地支持 Canister 智能合约

一个 Canister 智能合约（或简称 Canister ）被表示为一个 Motoko actor 。Actor 是一个自治对象，完全封装其状态并仅通过异步消息与其他 Actor 进行通信。

例如，此代码定义了一个有状态的`Counter` actor 

```js
actor Counter { // 定义一个命名为Counter的actor

  var value = 0;

  public func inc() : async Nat { // 定义一个公开的函数
    value += 1; // 全局变量value的值加1
    return value; 
  };
}
```

它的单个公共函数 `inc()` 可以由该actor和其他actor调用，以更新和读取其私有字段 `value` 的当前状态

<br>

### 以直接方式顺序编码

在 IC 上，Canister 可以通过发送异步消息与其他 Canister 进行通信。

异步编程很困难，但 Motoko 使你能够以更简单、顺序的方式编写异步代码。异步消息是返回 `future` 的函数调用，该 `await` 构造允许程序暂停执行，直到 `future` 完成。这个简单的功能避免了其他语言中显式异步编程的“回调地狱”问题

```js
actor Factorial {

  var last = 1;

  public func next() : async Nat {
    last *= await Counter.inc(); // await会暂停程序执行，等待Counter合约的inc函数完成且返回值
    return last;
  }
};

ignore await Factorial.next(); // 结果为1 * 1 = 1
ignore await Factorial.next(); // 结果为1 * 2 = 2
await Factorial.next(); // 结果为 2 * 3 = 6
```

<br>

### 现代类型系统

Motoko 的语法设计对于熟悉 JavaScript 和其他流行语言的人来说是直观的，而且 Motoko 提供了相应现代功能，例如健全的结构类型、泛型、变体类型和静态检查模式匹配。

```js
type Tree<T> = {
  #leaf : T;
  #branch : {left : Tree<T>; right : Tree<T>};
};

func iterTree<T>(tree : Tree<T>, f : T -> ()) {
  switch (tree) {
    case (#leaf(x)) { f(x) };
    case (#branch{left; right}) {
      iterTree(left, f);
      iterTree(right, f);
    };
  }
};

// 累加树的叶子节点的值
let tree = #branch { left = #leaf 1; right = #leaf 2 };
var sum = 0;
iterTree<Nat>(tree, func (leaf) { sum += leaf });
sum
```

<br>

### 自动生成IDL文件

Motoko actor 始终向其客户端呈现一个类型化接口，作为一组具有参数和（未来）结果类型的命名函数

Motoko 编译器（和 SDK）可以以一种称为 Candid 的中性语言格式声明接口，因此支持 Candid 的其他 Canister 、浏览器驻留代码和智能手机应用程序都可以使用该actor的服务。Motoko 编译器可以使用和生成 Candid 文件，从而允许 Motoko 与其他编程语言实现的Canister无缝交互（前提是它们支持 Candid 。

例如，之前的 Motoko `Counter` actor 有以下 Candid 接口：

```candid
service Counter : {
  inc : () -> (nat);
}
```

<br>

### 正交持久性

IC 在执行时会保留 Canister 的内存和其他状态。因此，Motoko actor 的状态，包括其内存中的数据结构，可以无限期地保存。Actor 状态不需要针对每条消息显式“重新存储”和“保存”到外部存储。

例如，在以下为 `Registry` 给名称分配顺序 ID 的 actor（容器）中，即使 actor 的状态在许多 IC 的节点机器上复制，并且通常不驻留，哈希表的状态也会在调用之间保留。

```js
import Text "mo:base/Text";
import Map "mo:base/HashMap";

actor Registry {

  let map = Map.HashMap<Text, Nat>(10, Text.equal, Text.hash); // 声明一个HashMap，其状态会保留在Canister中

  public func register(name : Text) : async () {
    switch (map.get(name)) {
      case null {
        map.put(name, map.size());
      };
      case (?id) { };
    }
  };

  public func lookup(name : Text) : async ?Nat {
    map.get(name);
  };
};

await Registry.register("hello");
(await Registry.lookup("hello"), await Registry.lookup("world"))
```

<br>

### 可升级

Motoko 提供了许多功能来帮助您利用正交持久性，包括允许您在升级容器代码时保留容器数据。

例如，Motoko 允许您将某些变量声明为 `stable` 。 `stable` 的变量值会在容器升级过程中自动保留。

写一个 stable 的计数器：

```js
actor Counter {

  stable var value = 0;

  public func inc() : async Nat {
    value += 1;
    return value;
  };
}

```

它可以安装到 Canister 中、递增 n 次，然后不间断地升级到更丰富的实现，例如：

```js
actor Counter {

  stable var value = 0;

  public func inc() : async Nat {
    value += 1;
    return value;
  };

  public func reset() : async () {
    value := 0;
  }
}
```

由于`value`已声明`stable`，因此服务的当前状态 n 会在升级后保留。计数将从 n 开始继续，而不是从 0 重新开始。

由于新接口与前一个接口兼容，引用该容器的现有客户端将继续工作，但新客户端将能够利用其升级的功能（附加 reset 功能）。

对于无法单独使用稳定变量解决的场景，Motoko 提供了用户可定义的升级 hooks ，这些 hooks 在升级之前和之后立即运行，并允许您将任意状态迁移到稳定变量。

<br>

### 还有更多

Motoko 还提供了很多开发人员生产力功能，包括子类型、任意精度算术和垃圾回收。

Motoko 不是、也无意成为实现 Canister 智能合约的唯一语言。如果它不能满足您的需求，还有一个适用于 Rust 编程语言的容器开发套件（CDK）。IC 的目标是使任何语言（具有对应的 WebAssembly 编译器）都能够生成在 IC 上运行的 Canister 智能合约，并通过语言中立的 Candid 接口与其他（可能是其他语言实现的）Canister 智能合约进行互操作。

其量身定制的设计意味着 Motoko 是互联网计算机上最简单、最安全的编码语言。

<br>
