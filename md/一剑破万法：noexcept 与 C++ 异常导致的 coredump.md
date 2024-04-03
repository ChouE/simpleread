> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/609434714)

作为 C/C++ 程序员，最不想见到的就是 coredump。coredump 的原因有很多，今天我来谈谈其中一种十分常见的原因，那就是由于异常没有被 catch 导致的 coredump。

从一篇知乎文章讲起
---------

先看一位知友的文章：

[津的技术专栏：C++11 std::thread 异常 coredump 导致调用堆栈丢失问题的跟踪和解决（std::teminate）](https://zhuanlan.zhihu.com/p/456536345)

这篇文章讲的是这位知友遇到 std::thread 执行时程序出现 coredump，但经过 gdb 调试后却无法一眼看到问题代码的位置。有时候 coredump 不可怕，但是 core 栈不清晰最可怕。这是因为 coredump 的原因是在回调函数中抛出了异常，如果不是被 std::thread 回调，本身 C++ 异常导致的 coredump 在 gdb 调试时是能直观看到出问题的代码行的。

然后作者使用了极其高深而琐细的方法，最终定位到了引发 coredump 的代码。不得不说作者其实很厉害，我也从中学到不少。但其实这有更简便的方法，请听我细细道来！

### 演示代码

借用一下这位知友后来写的 demo 验证代码：

```
#include <iostream>
#include <thread>
#include <vector>
void thread_func() {
    std::cout << "thread_func start ..." << std::endl;
    std::vector<int> vec;
    //vec.push_back(1);
    //vec.push_back(2);
    std::cout << vec.at(1) << std::endl;
}
int main (void) {
    std::thread th1(thread_func);
    th1.join();
    return 0;
}
```

该程序运行后会触发一个 coredump。由于是 demo 代码，所以其实你一眼就能找到 bug 所在，但这不是重点，让我们假装不知，然后去排查。

### 典型的 coredump 堆栈

gdb 打开 coredump 文件后，`bt`命令展示的堆栈信息如下：

```
Program terminated with signal 6, Aborted.
#0  0x00007fa9f0015387 in raise () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install glibc-2.17-326.el7_9.x86_64 libgcc-4.8.5-44.el7.x86_64 libstdc++-4.8.5-44.el7.x86_64
(gdb) bt
#0  0x00007fa9f0015387 in raise () from /lib64/libc.so.6
#1  0x00007fa9f0016a78 in abort () from /lib64/libc.so.6
#2  0x00007fa9f0b41a95 in __gnu_cxx::__verbose_terminate_handler() () from /lib64/libstdc++.so.6
#3  0x00007fa9f0b3fa06 in ?? () from /lib64/libstdc++.so.6
#4  0x00007fa9f0b3fa33 in std::terminate() () from /lib64/libstdc++.so.6
#5  0x00007fa9f0b963c5 in ?? () from /lib64/libstdc++.so.6
#6  0x00007fa9f03b4ea5 in start_thread () from /lib64/libpthread.so.0
#7  0x00007fa9f00ddb0d in clone () from /lib64/libc.so.6
```

这是一个非常典型的 coredump 文件。请记住不管你在实际生产过程中是多么复杂的 C++ 程序，只要 coredump 文件中有`signal 6`、`int raise()`、`int abort()`这三个关键字，基本就可以大概率确认这是一起由于异常没有被 catch 而导致的 coredump。

在实际生产过程中采用原作者的排查方法无疑比较繁琐的，而且未必有这样的条件（因为涉及到修改 libstdc++ 的源码，重新编译，重新连接）。其实我说的简便的方法就是 C++11 开始引入的`noexcept`关键字！

### 修改演示代码

来给回调函数加上 noexcept 声明：

```
#include <iostream>
#include <thread>
#include <vector>
void thread_func() noexcept {
    std::cout << "thread_func start ..." << std::endl;
    std::vector<int> vec;
    //vec.push_back(1);
    //vec.push_back(2);
    std::cout << vec.at(1) << std::endl;
}
int main (void) {
    std::thread th1(thread_func);
    th1.join();
    return 0;
}
```

重新编译执行，然后 gdb 调试 coredump 文件。这次的 core 堆栈如下：

```
Program terminated with signal 6, Aborted.
#0  0x00007f35b2889387 in raise () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install glibc-2.17-326.el7_9.x86_64 libgcc-4.8.5-44.el7.x86_64 libstdc++-4.8.5-44.el7.x86_64
(gdb) bt
#0  0x00007f35b2889387 in raise () from /lib64/libc.so.6
#1  0x00007f35b288aa78 in abort () from /lib64/libc.so.6
#2  0x00007f35b33b5a95 in __gnu_cxx::__verbose_terminate_handler() () from /lib64/libstdc++.so.6
#3  0x00007f35b33b3a06 in ?? () from /lib64/libstdc++.so.6
#4  0x00007f35b33b29b9 in ?? () from /lib64/libstdc++.so.6
#5  0x00007f35b33b3624 in __gxx_personality_v0 () from /lib64/libstdc++.so.6
#6  0x00007f35b2e4c8e3 in ?? () from /lib64/libgcc_s.so.1
#7  0x00007f35b2e4cc7b in _Unwind_RaiseException () from /lib64/libgcc_s.so.1
#8  0x00007f35b33b3c46 in __cxa_throw () from /lib64/libstdc++.so.6
#9  0x00007f35b3408b17 in std::__throw_out_of_range(char const*) () from /lib64/libstdc++.so.6
#10 0x0000000000401595 in std::vector<int, std::allocator<int> >::_M_range_check (this=0x7f35b2851e60, __n=1) at /usr/include/c++/4.8.2/bits/stl_vector.h:794
#11 0x0000000000401313 in std::vector<int, std::allocator<int> >::at (this=0x7f35b2851e60, __n=1) at /usr/include/c++/4.8.2/bits/stl_vector.h:812
#12 0x0000000000400fde in thread_func () at demo.cpp:9
#13 0x000000000040262f in std::_Bind_simple<void (*())()>::_M_invoke<>(std::_Index_tuple<>) (this=0xd32040) at /usr/include/c++/4.8.2/functional:1732
#14 0x0000000000402589 in std::_Bind_simple<void (*())()>::operator()() (this=0xd32040) at /usr/include/c++/4.8.2/functional:1720
#15 0x0000000000402522 in std::thread::_Impl<std::_Bind_simple<void (*())()> >::_M_run() (this=0xd32028) at /usr/include/c++/4.8.2/thread:115
#16 0x00007f35b340a330 in ?? () from /lib64/libstdc++.so.6
#17 0x00007f35b2c28ea5 in start_thread () from /lib64/libpthread.so.0
#18 0x00007f35b2951b0d in clone () from /lib64/libc.so.6
```

看 #12 的位置已经指出了 demo.cpp 的第 9 行，即：

```
std::cout << vec.at(1) << std::endl;
```

简述一下原理就是，在函数没有声明 noexcept 的时候，异常会继续向调用函数抛出，直到遇到 noexcept 的函数，或者一直抛给 main 函数，然后触发 coredump。声明了 noexcept 的函数预期就是里面不会抛出异常的，当抛出了异常的时候，由于和函数预期不符，程序会调用 std::teminate() 来立即终止进程，从而保留更接近抛出异常的问题代码的第一现场！

bRPC 社区的案例
----------

通过前面的解读，我们可以发现发生在回调函数中未被 catch 的异常所引发的 coredump，在不加 noexcept 声明的情况下，其堆栈信息颇为隐晦。在 C++ 在线服务中，回调函数自然必不可少，不管是多线程或者是协程的代码，都会用到回调函数。比如实现接口的代码都是被 RPC 框架所调用的回调函数。

前面的 demo 中，回调函数是极为简单的，但在实际生产环境中，业务逻辑十分复杂，因此存在大量的函数嵌套调用，稍不注意异常就会被连续抛到 RPC 框架的调度逻辑中，此时更难以觉察，甚至会误导业务程序员以为是 RPC 框架自身的 bug。比如在 bRPC 社区中就多次出现这样的 issue：

*   [#2081：http_rpc_protocol.cpp 中 core 掉了, 看起来不像是业务代码导致的](https://github.com/apache/brpc/issues/2081)
*   [#1437：运行一段时间，就会 core 在 brpc 内部](https://github.com/apache/brpc/issues/1437)
*   [#165：在 brpc 接口内部 core，但是使用 gdb 分析时遇到问题](https://github.com/apache/brpc/issues/165)

**这些对于 bRPC 的误解，其实才是本文写作的初衷**。

以`#1437`为例，看一下其中的 core 堆栈信息：

```
#0 0x00007f635e6fb597 in raise () from /lib64/libc.so.6
#1 0x00007f635e6fcdc8 in abort () from /lib64/libc.so.6
#2 0x00007f635f0029d5 in __gnu_cxx::__verbose_terminate_handler() () from /lib64/libstdc++.so.6
#3 0x00007f635f000946 in ?? () from /lib64/libstdc++.so.6
#4 0x00007f635efff909 in ?? () from /lib64/libstdc++.so.6
#5 0x00007f635f000574 in __gxx_personality_v0 () from /lib64/libstdc++.so.6
#6 0x00007f635ea99903 in ?? () from /lib64/libgcc_s.so.1
#7 0x00007f635ea99e37 in _Unwind_Resume () from /lib64/libgcc_s.so.1
#8 0x000056490472bc40 in operator() (this=, obj=) at incubator-brpc-0.9.7/src/brpc/destroyable.h:35
#9 ~unique_ptr (this=, __in_chrg=) at /usr/include/c++/4.8.2/bits/unique_ptr.h:184
#10 ~DestroyingPtr (this=, __in_chrg=) at incubator-brpc-0.9.7/src/brpc/destroyable.h:41
#11 brpc::policy::ProcessNsheadRequest (msg_base=) at incubator-brpc-0.9.7/src/brpc/policy/nshead_protocol.cpp:325
#12 0x00005649046e7eda in brpc::ProcessInputMessage (void_arg=void_arg@entry=0x5649087f8840) at incubator-brpc-0.9.7/src/brpc/input_messenger.cpp:135
#13 0x00005649046e8bf3 in operator() (this=, last_msg=0x5649087f8840) at incubator-brpc-0.9.7/src/brpc/input_messenger.cpp:141
#14 brpc::InputMessenger::OnNewMessages (m=0x7f5fa8f07040) at /usr/include/c++/4.8.2/bits/unique_ptr.h:184
#15 0x000056490479339d in brpc::Socket::ProcessEvent (arg=0x7f5fa8f07040) at incubator-brpc-0.9.7/src/brpc/socket.cpp:1017
#16 0x00005649046bcaca in bthread::TaskGroup::task_runner (skip_remained=skip_remained@entry=1) at incubator-brpc-0.9.7/src/bthread/task_group.cpp:296
#17 0x00005649046bcdcb in bthread::TaskGroup::run_main_task (this=this@entry=0x5649085aa4e0) at incubator-brpc-0.9.7/src/bthread/task_group.cpp:157
#18 0x00005649046b720e in bthread::TaskControl::worker_thread (arg=0x564907f066e0) at incubator-brpc-0.9.7/src/bthread/task_control.cpp:76
#19 0x00007f635f2b2dc5 in start_thread () from /lib64/libpthread.so.0
#20 0x00007f635e7c04dd in clone () from /lib64/libc.so.6
```

通过`ProcessNsheadRequest()`这个函数，可知这是一个 nshead 协议的 bRPC 服务，nshead 是百度内部一个古老的 RPC 协议，bRPC 也支持该协议。如果是更通用的 baidu_std 协议的 bRPC 服务，那么堆栈信息中应该是`ProcessRpcRequest()`，比如 issue`#165`。

直观来看，确实是 core 在了 bthread 调度的地方，`run_main_task() -> task_runner() -> ProcessEvent() -> OnNewMessages()` 因此导致 bRPC 的使用者，将矛头直指 bRPC，但真相并非如此。其本质是接口 service 的业务实现中出现了未被 catch 的异常。

但具体是什么代码抛出的异常，其实仍然难以排查，如果是增量上线引入的代码，通过 review 本次上线的代码应该可以发现端倪。我们应该勤于在自己的业务函数中加上 noexcept 声明，这样更为及时地获取准确的 coredump 堆栈信息。但若要避免线上经常出现此类问题，则需要我们养成一个好编码习惯，请继续阅读。

C++ 在线服务与异常的最佳实践
----------------

以下经验不止适用于 bRPC 服务，其他 C++ RPC 框架的使用者也应该能从中受益。

### 不在服务运行时抛异常

由于 C++ 的异常规格与 Java 差异较大，对于是否该使用 C++ 的异常，C++ 圈子内向来争论不休。

我个人的经验是：在在线服务中，不应当在服务运行时主动 throw 异常。这里的服务运行中主要指的是请求处理的业务代码中。虽然异常意味着本次请求已经完全不可能继续正常处理。但若主动抛出异常，而本函数内或函数的整个调用链上都遗漏了对这种异常的 catch，那么服务就会 core 掉。从而导致同期能够正常处理的请求也得不到处理。

当然这里说的是单进程多线程 / 协程的服务，对于多进程单线程处理请求的服务而言，单进程 coredump 该服务仍然可以继续工作。不过这种多进程的模式在在线服务中不太流行。

另外服务运行时不 throw 异常还包括一些其他的背景线程。比如服务内有一个词典组件，该组件会定期热加载词典文件。加载过程在运行在一个单独的线程中的，这种线程内的函数也要避免 throw 异常。

当然凡事并无绝对，受限于业务场景，有些时候也存在一些 **workaround**。下文会有讨论。

### 服务启动时可以抛异常

对于一个在线服务而言，除了运行时的请求处理代码，还有一部分代码是在服务启动时、开始处理外部请求之前执行的。比如各类资源的初始化操作，所以该阶段也可以称作初始化阶段。此时由于并不处理请求，因此在初始化异常的时候可以抛出异常，直截了当地终止服务。彼时通过查看 coredump 堆栈，可以快速发现是哪一处初始化失败了。

### 勤于给函数加上 noexcept 声明

即使遵守了前面的准则，我们不主动 throw 异常，但未必能完全规避异常。比如在使用标准库或者某些第三方库的时候，仍然有可能抛出异常。这时就需要我们在可能抛异常的第一现场加上异常对应的 catch 逻辑，从而避免其继续跑到上层调用的函数中。

因此我们要勤于给函数加上 noexcept 声明，当然无差别的 noexcept 声明，确实会让代码略显冗余。确认哪些函数是必要添加的位置，从勤于变成善于，这需要一些经验。当然无差别的 noexcept 也并非不可取。

### lambda 表达式添加 noexcept 声明

除了普通函数、成员函数外，lambda 函数也可以添加 noexcept 声明。比如：

```
#include <iostream>
#include <thread>
#include <vector>
int main (void) {
    std::vector<int> vec;
    std::thread th1([&]() noexcept {
        std::cout << "thread_func start ..." << std::endl;
        //vec.push_back(1);
        //vec.push_back(2);
        std::cout << vec.at(1) << std::endl;
    });
    th1.join();
    return 0;
}
```

### 兜底：service 函数加上 noexcept

以 bRPC 的 echo_server 为例，下面是一个提供 Echo 接口的服务。实现该接口的执行逻辑就是自定义类型继承 proto 生成的 Service 父类，然后覆写虚函数 Echo。这时我们可以该这个 Echo 函数加一下 noexcept 声明。

虽然抛出异常的代码未必就在 Echo 中，而可能是 Echo 层层调用的千里之外的某个函数中。但加上 noexcept 之后，当业务代码抛出异常时，也不会让人误以为的 core 在 RPC 框架中，避免干扰排查方向。故而是一种兜底的做法。

```
class EchoServiceImpl : public EchoService {
public:
    EchoServiceImpl() {}
    virtual ~EchoServiceImpl() {}
    virtual void Echo(google::protobuf::RpcController* cntl_base,
                      const EchoRequest* request,
                      EchoResponse* response,
                      google::protobuf::Closure* done) noexcept {
             ...
        }
    }
};
```

### 是否应该使用标准库 / 第三方库中会抛出异常的函数？

我们需要熟悉哪些标准库的函数或者第三方库的函数会抛异常。比如 STL 容器中 at() 函数都是会做越界检查的，会抛异常。我个人强烈建议程序员自己做边界检查，避免使用 at()。比如：

```
vector<int> v;
...
if (i < v.size()) {
    ...  // 使用v[i]
}

map<string, float> m;
...
auto it = m.find(key);
if (it != m.end()) {
   auto& value = it->second;
   ...  // 使用value
}
```

当然这样严格的使用限制虽然避免了线上 coredump 的风险，但是可能会导致自己业务逻辑的 bug 无法被发现。比如在你预期的逻辑中，使用`v[i]`或`m[key]`的时候永远不会越界。但是你在实现时考虑不周，在某些极少数的边界情况出现了越界。彼时由于做了边界检查规避了 coredump，导致功能上线了很长时间，而未发现有 bug。这后果有时候可能更严重。

我们也可以给上面的 if 都补一个 else 去做日志打印或者报警之类的功能，但如果想更快发现 bug，避免 bug 产生实际影响，那么我建议你在这种情况下，使用 at()，并且给整个函数加上 noexcept 声明，从而让 coredump 快速定位。这也就是我前面提到的『不在服务时抛异常』的一些 workaround 情况。

noexcept specifier
------------------

### 基本介绍

前面我所提到的在函数声明中加入 noexcept 声明的用法，被称为`noexcept specifier`。其实这只是 noexcept 这个关键字的其中一个用法，还有另外一个用法，我们稍后会讲。先关注`noexcept specifier`。可以参考：

[noexcept specifier (since C++11)](https://en.cppreference.com/w/cpp/language/noexcept_spec)

所谓 noexcept，其实是 noexcept(true) 的简化，同样可以声明成 noexcept(false) 表示可能会抛异常。

```
void foo() noexcept(true);  // 等价于 void foo() noexcept;
void bar() noexcept(false); // 基本等价于 void bar();
```

自定义函数在没有加 noexcept 或 noexcept(true) 声明的时候，其默认是 noexcept(false)。但对于一些特殊函数即使在没有显式添加 noexcept 声明时，也可能是 noexcept(true) 的。比如所有析构函数在 C++11 以后默认是 noexcept 的。这是语法规范的一部分。当然你也可以显式地修改它：

```
class A {
public:
    ~A() noexcept(false) {}
}
```

**注意，本文以下内容中在未特殊指明的时候，所说的 noexcept 声明，都指的是 noexcept(true) 含义的 noexcept 声明。**

### noexcept 与多态

如果在类中把某个虚函数声明成 noexcept，那么在继承这个类的子类中，其同名函数必须也要声明成 noexcept，否则编译直接失败。

这对于框架与组件库的设计者来说是一个极好的功能，方便限制住使用方在实现子类的时候不会漏掉 noexcept，从而减少后续排查 coredump 的麻烦。但是请注意，对于框架和组件库中一些已有的函数如果之前没有加 noexcept，后续就不要再加了。因为会破坏前向兼容性，导致使用者存量的代码无法编译通过！

因此作为框架与组件库的设计者应该在第一次释出新函数接口的时候，就考虑加上 noexcept 声明。

再说个题外话，当在子类中需要覆写的虚函数同时使用 override 和 noexcept 的时候，要保证 noexcept 在前，override 在后。

```
class A {
public:
    A() {}
    ~A() {}
    virtual void echo() noexcept {}
};
class B : public A {
public:
    ~B() override {}
    void echo() noexcept override {}
};
```

### noexcept 与直接 throw

通常当你给一个函数加上`noexcept`声明的时候，就不应该在这个函数中再显式地`throw`异常了。

```
void foo() noexcept {
    ...
    throw runtime_error("... error");
}
```

对于高版本 g++ 编译的时候会出现编译警告：

```
In function ‘void foo()’:
warning: throw will always call terminate() [-Wterminate]
     throw runtime_error("... error");
```

如果开启 g++ 的`-Werror`、`-Werror=terminate`编译选项，则会直接编译失败。 值得一提的是 g++4.8 是没有这个编译检查的，不会有编译警告或者导致编译失败。具体从哪个版本引入的这项检查，我没有去深究，至少我用过的 g++7 是有的。

当然如果在 noexcept(true) 函数中调用了一个内部会 throw 异常的函数，这种情况是不会编译警告或编译失败的。

### noexcept 与函数的声明与定义

当函数声明与定义分离时，如果在声明函数的头文件中的加入了 noexcept 声明。那么在定义函数的源文件中也要加上 noexcept。而前面我们所提到的 override 关键字在函数声明与定义分离的时候，只能在函数声明的时候添加！

noexcept operator
-----------------

前面提到的 noexcept 用法都是`noexcept specifier`，其实它还有另外一个用法是`noexcept operator`，用于判定一个表达式是否是 noexcept 的。

```
#include <iostream>
using namespace std;
void foo() noexcept {
}

void bar() noexcept(false) {  // 或者 void bar() {
} 

int main() {
    cout<<"foo() check noexcept:" << noexcept(foo()) << endl;
    cout<<"bar() check noexcept:" << noexcept(bar()) << endl;
    return 0;
}
```

运行后输出：

```
foo() check noexcept:1
bar() check noexcept:0
```

注意，`noexcept operator`判断是一个表达式，不是函数。所以要使用`noexcept(foo())`而不是`noexcept(foo)`。其实这不难理解，因为 foo 本身可能存在重载：

```
#include <iostream>
using namespace std;
void foo() noexcept {
}

void foo(int i) {
} 

int main() {
    cout<<"foo() check noexcept:" << noexcept(foo()) << endl;
    cout<<"foo(1) check noexcept:" << noexcept(foo(1)) << endl;
    return 0;
}
```

运行后输出：

```
foo() check noexcept:1
foo(1) check noexcept:0
```

另外尽管这个例子中我是用 cout 来输出的，但是其实`noexcept operator`是在编译期间求值的，也就是说程序运行时`noexcept operator`是无开销的。

不信你可以这样来做一下测试：

```
#include <iostream>
using namespace std;
void foo() noexcept {
}

void bar() noexcept(false) {  // 或者 void bar() {
} 

int main() {
    static_assert(noexcept(foo()), " foo is not noexcept");
    static_assert(noexcept(bar()), " bar is not noexcept");
    return 0;
}
```

这个代码在编译阶段就会失败，编译输出：

```
error: static assertion failed: bar is not noexcept
     static_assert(noexcept(bar()), " bar is not noexcept");
```

`noexcept operator`也可以用来检测，类的成员函数，前面我们提到过类的析构函数默认都是 noexcept(true) 的，对于类中其他默认的函数，在不加声明的时候具体是 noexcept(true) 还是 noexcept(false)，是比较复杂的。在本文中不过度展开了，有兴趣可以阅读： [https://en.cppreference.com/w/cpp/language/noexcept](https://en.cppreference.com/w/cpp/language/noexcept)

noexcept 不是 coredump 万金油!
-------------------------

请注意虽然本文标题十分**标题党**地使用了『一剑破万法』的说法，但是这个『万法』仅仅指的是各类 C++ 异常（Exception)，对于其他原因导致的 coredump，比如访问非法内存地址触发 coredump，noexcept 并不会有太大帮助！

所以 noexcept 并不是排查 coredump 的万金油，它只对异常导致的 coredump 有效。

高版本 g++/gdb 能解决问题吗？
-------------------

高版本的 g++ 和 gdb 对于回调函数中异常没被 catch 导致的 coredump 堆栈不清晰的问题有优化吗？来做个测试。我这里有一个 g++9.4，gdb9.2 的环境。

先试一下前面提到的知友的那个未加 noexcept 声明的 demo 代码：

```
#include <iostream>
#include <thread>
#include <vector>
void thread_func() {
    std::cout << "thread_func start ..." << std::endl;
    std::vector<int> vec;
    //vec.push_back(1);
    //vec.push_back(2);
    std::cout << vec.at(1) << std::endl;
}
int main (void) {
    std::thread th1(thread_func);
    th1.join();
    return 0;
}
```

加上 - g 参数编译，重新触发 coredump，进行 gdb 调试：

```
#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:50
#1  0x00007fa0eec91859 in __GI_abort () at abort.c:79
#2  0x00007fa0eef3d911 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#3  0x00007fa0eef4938c in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#4  0x00007fa0eef493f7 in std::terminate() () from /lib/x86_64-linux-gnu/libstdc++.so.6
#5  0x00007fa0eef496a9 in __cxa_throw () from /lib/x86_64-linux-gnu/libstdc++.so.6
#6  0x00007fa0eef403ab in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#7  0x00005621d5f5b8fc in std::vector<int, std::allocator<int> >::_M_range_check (this=0x7fa0eeb19de0, __n=1)
    at /usr/include/c++/9/bits/stl_vector.h:1070
#8  0x00005621d5f5b6ed in std::vector<int, std::allocator<int> >::at (this=0x7fa0eeb19de0, __n=1) at /usr/include/c++/9/bits/stl_vector.h:1091
#9  0x00005621d5f5b36a in thread_func () at zhiyou.cpp:9
#10 0x00005621d5f5c15e in std::__invoke_impl<void, void (*)()> (__f=@0x5621d6810eb8: 0x5621d5f5b309 <thread_func()>)
    at /usr/include/c++/9/bits/invoke.h:60
#11 0x00005621d5f5c0f6 in std::__invoke<void (*)()> (__fn=@0x5621d6810eb8: 0x5621d5f5b309 <thread_func()>) at /usr/include/c++/9/bits/invoke.h:95
#12 0x00005621d5f5c088 in std::thread::_Invoker<std::tuple<void (*)()> >::_M_invoke<0ul> (this=0x5621d6810eb8) at /usr/include/c++/9/thread:244
#13 0x00005621d5f5c045 in std::thread::_Invoker<std::tuple<void (*)()> >::operator() (this=0x5621d6810eb8) at /usr/include/c++/9/thread:251
#14 0x00005621d5f5c016 in std::thread::_State_impl<std::thread::_Invoker<std::tuple<void (*)()> > >::_M_run (this=0x5621d6810eb0)
    at /usr/include/c++/9/thread:195
#15 0x00007fa0eef75de4 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#16 0x00007fa0eee69609 in start_thread (arg=<optimized out>) at pthread_create.c:477
#17 0x00007fa0eed8e133 in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

可以看出 #7、#8 能看出是 at 导致的越界异常，#9 指明了行号。看起来似乎不错，但在复杂的回调逻辑中表现如何呢？我们来试一下 bRPC 的 echo_server 的代码，原版代码：[https://github.com/apache/brpc/blob/master/example/echo_c%2B%2B/server.cpp](https://github.com/apache/brpc/blob/master/example/echo_c%2B%2B/server.cpp)

在 Echo 接口函数的底部中加上一个会抛出异常的代码：

```
virtual void Echo(google::protobuf::RpcController* cntl_base,
                      const EchoRequest* request,
                      EchoResponse* response,
                      google::protobuf::Closure* done) noexcept {
        // This object helps you to call done->Run() in RAII style. If you need
        // to process the request asynchronously, pass done_guard.release().
        brpc::ClosureGuard done_guard(done);

        brpc::Controller* cntl =
            static_cast<brpc::Controller*>(cntl_base);

        // The purpose of following logs is to help you to understand
        // how clients interact with servers more intuitively. You should 
        // remove these logs in performance-sensitive servers.
        LOG(INFO) << "Received request[log_id=" << cntl->log_id() 
                  << "] from " << cntl->remote_side() 
                  << " to " << cntl->local_side()
                  << ": " << request->message()
                  << " (attached=" << cntl->request_attachment() << ")";

        // Fill response.
        response->set_message(request->message());

        // You can compress the response by setting Controller, but be aware
        // that compression may be costly, evaluate before turning on.
        // cntl->set_response_compress_type(brpc::COMPRESS_TYPE_GZIP);

        if (FLAGS_echo_attachment) {
            // Set attachment which is wired to network directly instead of
            // being serialized into protobuf messages.
            cntl->response_attachment().append(cntl->request_attachment());
        }
        std::vector<int> v;
        std::cout<< v.at(0) << std::endl;  // 抛出越界异常！
    }
```

来进行编译，启动 echo_server。接着启动 echo_client 会向它发请求，echo_server 出现了 coredump，gdb 进行调试。coredump 堆栈如下：

```
#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:50
#1  0x00007f8be87d5859 in __GI_abort () at abort.c:79
#2  0x00007f8be8baf911 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#3  0x00007f8be8bbb38c in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#4  0x00007f8be8bba369 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#5  0x00007f8be8bbad21 in __gxx_personality_v0 () from /lib/x86_64-linux-gnu/libstdc++.so.6
#6  0x00007f8be89b5bef in ?? () from /lib/x86_64-linux-gnu/libgcc_s.so.1
#7  0x00007f8be89b65aa in _Unwind_Resume () from /lib/x86_64-linux-gnu/libgcc_s.so.1
#8  0x00005609e57d96f3 in brpc::detail::Destroyer<brpc::policy::MostCommonMessage>::operator() (this=<optimized out>, obj=<optimized out>)
    at ./src/brpc/destroyable.h:35
#9  std::unique_ptr<brpc::policy::MostCommonMessage, brpc::detail::Destroyer<brpc::policy::MostCommonMessage> >::~unique_ptr (this=<optimized out>, 
    __in_chrg=<optimized out>) at /usr/include/c++/9/bits/unique_ptr.h:292
#10 brpc::DestroyingPtr<brpc::policy::MostCommonMessage>::~DestroyingPtr (this=<optimized out>, __in_chrg=<optimized out>)
    at ./src/brpc/destroyable.h:41
#11 brpc::policy::ProcessRpcRequest (msg_base=<optimized out>) at src/brpc/policy/baidu_rpc_protocol.cpp:314
#12 0x00005609e59663db in brpc::ProcessInputMessage (void_arg=<optimized out>) at src/brpc/input_messenger.cpp:159
#13 0x00005609e5966f35 in brpc::InputMessenger::InputMessageClosure::~InputMessageClosure (this=<optimized out>, __in_chrg=<optimized out>)
    at src/brpc/input_messenger.cpp:194
#14 0x00005609e5967a6b in brpc::InputMessenger::OnNewMessages (m=0x7f8bc801aea0) at /usr/include/c++/9/bits/atomic_base.h:493
#15 0x00005609e5b39002 in brpc::Socket::ProcessEvent (arg=0x7f8bc801aea0) at src/brpc/socket.cpp:1093
#16 0x00005609e5aa682f in bthread::TaskGroup::task_runner (skip_remained=<optimized out>) at src/bthread/task_group.cpp:298
#17 0x00005609e5a9ddb1 in bthread_make_fcontext () at /usr/include/c++/9/bits/stl_iterator.h:803
```

发现并不清晰，对于大多数业务程序员来说颇具迷惑性。我们在 Echo 接口函数加上 noexcept 声明，再重走一遍上述流程，看一下新的 coredump 堆栈：

```
#0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:50
#1  0x00007f987922d859 in __GI_abort () at abort.c:79
#2  0x00007f9879607911 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#3  0x00007f987961338c in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#4  0x00007f9879612369 in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#5  0x00007f9879612d21 in __gxx_personality_v0 () from /lib/x86_64-linux-gnu/libstdc++.so.6
#6  0x00007f987940dbef in ?? () from /lib/x86_64-linux-gnu/libgcc_s.so.1
#7  0x00007f987940e281 in _Unwind_RaiseException () from /lib/x86_64-linux-gnu/libgcc_s.so.1
#8  0x00007f987961369c in __cxa_throw () from /lib/x86_64-linux-gnu/libstdc++.so.6
#9  0x00007f987960a3ab in ?? () from /lib/x86_64-linux-gnu/libstdc++.so.6
#10 0x000055e51d9cf0d6 in std::vector<int, std::allocator<int> >::_M_range_check (__n=0, this=<synthetic pointer>)
    at /usr/include/c++/9/bits/stl_vector.h:1089
#11 std::vector<int, std::allocator<int> >::at (__n=0, this=<synthetic pointer>) at /usr/include/c++/9/bits/stl_vector.h:1091
#12 example::EchoServiceImpl::Echo (this=<optimized out>, cntl_base=0x7f9854027df0, request=0x7f9854028120, response=0x7f9854029950, 
    done=<optimized out>) at server.cpp:73
#13 0x000055e51d9cd39a in example::EchoService::CallMethod (method=<optimized out>, done=<optimized out>, response=<optimized out>, 
    request=<optimized out>, controller=<optimized out>, this=<optimized out>) at echo.pb.cc:671
#14 example::EchoService::CallMethod (this=<optimized out>, method=<optimized out>, controller=<optimized out>, request=<optimized out>, 
    response=<optimized out>, done=<optimized out>) at echo.pb.cc:663
#15 0x000055e51dbf37a0 in brpc::policy::ProcessRpcRequest (msg_base=0x7f985401f460) at /usr/include/c++/9/bits/unique_ptr.h:381
#16 0x000055e51db3c37b in brpc::ProcessInputMessage (void_arg=<optimized out>) at src/brpc/input_messenger.cpp:159
#17 0x000055e51db3ced5 in brpc::InputMessenger::InputMessageClosure::~InputMessageClosure (this=<optimized out>, __in_chrg=<optimized out>)
    at src/brpc/input_messenger.cpp:194
#18 0x000055e51db3da0b in brpc::InputMessenger::OnNewMessages (m=0x7f986001aea0) at /usr/include/c++/9/bits/atomic_base.h:493
#19 0x000055e51dd0efa2 in brpc::Socket::ProcessEvent (arg=0x7f986001aea0) at src/brpc/socket.cpp:1093
#20 0x000055e51dc7c7cf in bthread::TaskGroup::task_runner (skip_remained=<optimized out>) at src/bthread/task_group.cpp:298
#21 0x000055e51dc73d51 in bthread_make_fcontext () at /usr/include/c++/9/bits/stl_iterator.h:803
```

可以看出 #10，#11，#12 清晰的展示出了越界异常，以及问题代码的行号 server.cpp:73，这一行就是 std::cout<<v.at(0)<< std::endl; 所在的行。

所以说高版本的 g++、gdb 对于本文所探讨的问题确实有优化，但对于复杂的回调链路依旧捉襟见肘，因此 noexcept 依旧是我们的首选。当然更高版本的 g++、gdb 我就没有环境测试了，其他同学有其他版本的测试结论可以评论回复。不过对于生产环境而言，升级编译器版本往往并不是自己或者自己团队所能决定的了，这是一项浩大的变更工作。