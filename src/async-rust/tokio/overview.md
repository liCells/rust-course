# tokio 概览

对于 Async Rust，最最重要的莫过于底层的异步运行时，它提供了执行器、任务调度、异步 API 等核心服务。简单来说，使用 Rust 提供的 `async/.await` 特性编写的异步代码要运行起来，就必须依赖于异步运行时，否则这些代码将毫无用处。

## 异步运行时

Rust 语言本身只提供了异步编程所需的基本特性，例如 `async/.await` 关键字，标准库中的 `Future` 特征，官方提供的 `futures` 实用库，这些特性单独使用没有任何用处，因此我们需要一个运行时来将这些特性实现的代码运行起来。

异步运行时是由 Rust 社区提供的，它们的核心是一个 `reactor` 和一个或多个 `executor`(执行器):

- `reactor` 用于提供外部事件的订阅机制，例如 `I/O` 、进程间通信、定时器等
- `executor` 在上一章我们有过深入介绍，它用于调度和执行相应的任务( `Future` )

目前最受欢迎的几个运行时有:

- [`tokio`](https://github.com/tokio-rs/tokio)，目前最受欢迎的异步运行时，功能强大，还提供了异步所需的各种工具(例如 tracing )、网络协议框架(例如 HTTP，gRPC )等等
- [`async-std`](https://github.com/async-rs/async-std)，最大的优点就是跟标准库兼容性较强
- [`smol`](https://github.com/smol-rs/smol), 一个小巧的异步运行时

但是，大浪淘沙，留下的才是金子，随着时间的流逝，`tokio`越来越亮眼，无论是性能、功能还是社区、文档，它在各个方面都异常优秀，时至今日，可以说已成为事实上的标准。

#### 异步运行时的兼容性

为何选择异步运行时这么重要？不仅仅是它们在功能、性能上存在区别，更重要的是当你选择了一个，往往就无法切换到另外一个，除非异步代码很少。

使用异步运行时，往往伴随着对它相关的生态系统的深入使用，因此耦合性会越来越强，直至最后你很难切换到另一个运行时，例如 `tokio` 和 `async-std` ，就存在这种问题。

如果你实在有这种需求，可以考虑使用 [`async-compat`](https://github.com/smol-rs/async-compat)，该包提供了一个中间层，用于兼容 `tokio` 和其它运行时。

#### 结论

相信大家看到现在，心中应该有一个结论了。首先，运行时之间的不兼容性，让我们必须提前选择一个运行时，并且在未来坚持用下去，那这个运行时就应该是最优秀、最成熟的那个，`tokio` 几乎成了不二选择，当然 `tokio` 也有自己的问题：更难上手和运行时之间的兼容性。

如果你只用 `tokio` ，那兼容性自然不是问题，至于难以上手，Rust 这么难，我们都学到现在了，何况区区一个异步运行时，在本书的帮忙下，这些都不再是个问题：）

## tokio 简介

tokio 是一个纸醉金迷之地，只要有钱就可以为所欲为，哦，抱歉，走错片场了。`tokio` 是 Rust 最优秀的异步运行时框架，它提供了写异步网络服务所需的几乎所有功能，不仅仅适用于大型服务器，还适用于小型嵌入式设备，它主要由以下组件构成：

- 多线程版本的异步运行时，可以运行使用 `async/.await` 编写的代码
- 标准库中阻塞 API 的异步版本，例如`thread::sleep`会阻塞当前线程，`tokio`中就提供了相应的异步实现版本
- 构建异步编程所需的生态，甚至还提供了 [`tracing`](https://github.com/tokio-rs/tracing) 用于日志和分布式追踪， 提供 [`console`](https://github.com/tokio-rs/console) 用于 Debug 异步编程

### 优势

下面一起来看看使用 `tokio` 能给你提供哪些优势。

**高性能**

因为快所以快，前者是 Rust 快，后者是 `tokio` 快。 `tokio` 在编写时充分利用了 Rust 提供的各种零成本抽象和高性能特性，而且贯彻了 Rust 的牛逼思想：如果你选择手写代码，那么最好的结果就是跟 `tokio` 一样快！

以下是一张官方提供的性能参考图，大致能体现出 `tokio` 的性能之恐怖:
<img alt="tokio performance" src="https://pica.zhimg.com/80/v2-5f5ca10550ec936427c2919191331ae8_1440w.png" class="center"  />

**高可靠**

Rust 语言的安全可靠性顺理成章的影响了 `tokio` 的可靠性，曾经有一个调查给出了令人乍舌的[结论](https://www.zdnet.com/article/microsoft-70-percent-of-all-security-bugs-are-memory-safety-issues/)：软件系统 70%的高危漏洞都是由内存不安全性导致的。

在 Rust 提供的安全性之外，`tokio` 还致力于提供一致性的行为表现：无论你何时运行系统，它的预期表现和性能都是一致的，例如不会出现莫名其妙的请求延迟或响应时间大幅增加。

**简单易用**

通过 Rust 提供的 `async/await` 特性，编写异步程序的复杂性相比当初已经大幅降低，同时 `tokio` 还为我们提供了丰富的生态，进一步大幅降低了其复杂性。

同时 `tokio` 遵循了标准库的命名规则，让熟悉标准库的用户可以很快习惯于 `tokio` 的语法，再借助于 Rust 强大的类型系统，用户可以轻松地编写和交付正确的代码。

**使用灵活性**

`tokio` 支持你灵活的定制自己想要的运行时，例如你可以选择多线程 + 任务盗取模式的复杂运行时，也可以选择单线程的轻量级运行时。总之，几乎你的每一种需求在 `tokio` 中都能寻找到支持(画外音：强大的灵活性需要一定的复杂性来换取，并不是免费的午餐)。

### 劣势

虽然 `tokio` 对于大多数需要并发的项目都是非常适合的，但是确实有一些场景它并不适合使用:

- 并行运行 CPU 密集型的任务，`tokio` 非常适合于 IO 密集型任务，这些 IO 任务的绝大多数时间都用于阻塞等待 IO 的结果，而不是刷刷刷的单烤 CPU。如果你的应用是 CPU 密集型(例如并行计算)，建议使用 [`rayon`](https://github.com/rayon-rs/rayon)，当然，对于其中的 IO 任务部分，你依然可以混用 `tokio`
- 读取大量的文件, 读取文件的瓶颈主要在于操作系统，因为 OS 没有提供异步文件读取接口，大量的并发并不会提升文件读取的并行性能，反而可能会造成不可忽视的性能损耗，因此建议使用线程(或线程池)的方式
- 发送 HTTP 请求，`tokio` 的优势是给予你并发处理大量任务的能力，对于这种轻量级 HTTP 请求场景，`tokio` 除了增加你的代码复杂性，并无法带来什么额外的优势。因此，对于这种场景，你可以使用 [`reqwest`](https://github.com/seanmonstar/reqwest) 库，它会更加简单易用。


> 若大家使用 tokio，那 CPU 密集的任务尤其需要用线程的方式去处理，例如使用 `spawn_blocking` 创建一个阻塞的线程取完成相应 CPU 密集任务。
>
> 原因是：tokio 是协作式地调度器，如果某个 CPU 密集的异步任务是通过 tokio 创建的，那理论上来说，该异步任务需要跟其它的异步任务交错执行，最终大家都得到了执行，皆大欢喜。但实际情况是，CPU 密集的任务很可能会一直霸着着 CPU，此时 tokio 的调度方式决定了该任务会一直被执行，这意味着，其它的异步任务无法得到执行的机会，最终这些任务都会因为得不到资源而饿死。
>
> 而使用 `spawn_blocking` 后，会创建一个单独的 OS 线程，该线程并不会被 tokio 所调度( 被 OS 所调度 )，因此它所执行的 CPU 密集任务也不会导致 tokio 调度的那些异步任务被饿死


## 总结

离开三方开源社区提供的异步运行时， `async/await` 什么都不是，甚至还不如一堆破铜烂铁，除非你选择根据自己的需求手撸一个。

而 `tokio` 就是那颗皇冠上的夜明珠，也是值得我们投入时间去深入学习的开源库，它的设计原理和代码实现都异常优秀，在之后的章节中，我们将对其进行深入学习和剖析，敬请期待。

