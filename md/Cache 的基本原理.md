> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/102293437)

对于没有接触过底层技术的朋友来说，或许从未听说过 cache。毕竟 cache 的存在对程序员来说是透明的。在接触 cache 之前，先为你准备段 code 分析。

```
int arr[10][128];

for (i = 0; i < 10; i++)
        for (j = 0; j < 128; j++)
                arr[i][j] = 1;
```

如果你曾经学习过 C/C++ 语言，这段 code 自然不会陌生。如此简单的将`arr`数组所有元素置 1。 你有没有想过这段 code 还有下面的一种写法。

```
int arr[10][128];

for (i = 0; i < 128; i++)
        for (j = 0; j < 10; j++)
                arr[j][i] = 1;
```

功能完全一样，但是我们一直在重复着第一种写法（或许很多的书中也是建议这么编码），你是否想过这其中的缘由？文章的主角是 cache，所以你一定猜到了答案。那么 cache 是如何影响这 2 段 code 的呢？

为什么需要 cache
-----------

在思考为什么需要 cache 之前，我们首先先来思考另一个问题：我们的程序是如何运行起来的？

我们应该知道程序是运行在 RAM 之中，RAM 就是我们常说的 DDR（例如： DDR3、DDR4 等）。我们称之为 main memory（主存）。当我们需要运行一个进程的时候，首先会从磁盘设备（例如，eMMC、UFS、SSD 等）中将可执行程序 load 到主存中，然后开始执行。在 CPU 内部存在一堆的通用寄存器（register）。如果 CPU 需要将一个变量（假设地址是 A）加 1，一般分为以下 3 个步骤：

1.  CPU 从主存中读取地址 A 的数据到内部通用寄存器 x0（ARM64 架构的通用寄存器之一）。
2.  通用寄存器 x0 加 1。
3.  CPU 将通用寄存器 x0 的值写入主存。

我们将这个过程可以表示如下：

![](https://pic1.zhimg.com/v2-1aa0caac22aec470dd15d0a7ca1f4c80_r.jpg)

其实现实中，CPU 通用寄存器的速度和主存之间存在着太大的差异。两者之间的速度大致如下关系：

![](https://pic2.zhimg.com/v2-cce58cab829ecc2755f6797b41bea821_r.jpg)

CPU register 的速度一般小于 1ns，主存的速度一般是 65ns 左右。速度差异近百倍。因此，上面举例的 3 个步骤中，步骤 1 和步骤 3 实际上速度很慢。当 CPU 试图从主存中 load/store 操作时，由于主存的速度限制，CPU 不得不等待这漫长的 65ns 时间。如果我们可以提升主存的速度，那么系统将会获得很大的性能提升。如今的 DDR 存储设备，动不动就是几个 GB，容量很大。如果我们采用更快材料制作更快速度的主存，并且拥有几乎差不多的容量。其成本将会大幅度上升。我们试图提升主存的速度和容量，又期望其成本很低，这就有点难为人了。因此，我们有一种折中的方法，那就是制作一块速度极快但是容量极小的存储设备。那么其成本也不会太高。这块存储设备我们称之为 cache memory。在硬件上，我们将 cache 放置在 CPU 和主存之间，作为主存数据的缓存。 当 CPU 试图从主存中 load/store 数据的时候， CPU 会首先从 cache 中查找对应地址的数据是否缓存在 cache 中。如果其数据缓存在 cache 中，直接从 cache 中拿到数据并返回给 CPU。当存在 cache 的时候，以上程序如何运行的例子的流程将会变成如下：

![](https://pic2.zhimg.com/v2-bc15d8c0612599fc3de51c4382e07aa5_r.jpg)

CPU 和主存之间直接数据传输的方式转变成 CPU 和 cache 之间直接数据传输。cache 负责和主存之间数据传输。

多级 cache 存储结构
-------------

cahe 的速度在一定程度上同样影响着系统的性能。一般情况 cache 的速度可以达到 1ns，几乎可以和 CPU 寄存器速度媲美。但是，这就满足人们对性能的追求了吗？并没有。当 cache 中没有缓存我们想要的数据的时候，依然需要漫长的等待从主存中 load 数据。为了进一步提升性能，引入多级 cache。前面提到的 cache，称之为 L1 cache（第一级 cache）。我们在 L1 cache 后面连接 L2 cache，在 L2 cache 和主存之间连接 L3 cache。等级越高，速度越慢，容量越大。但是速度相比较主存而言，依然很快。不同等级 cache 速度之间关系如下：

![](https://pic1.zhimg.com/v2-0910f3308b1d0e425c308307869a3f68_r.jpg)

经过 3 级 cache 的缓冲，各级 cache 和主存之间的速度最萌差也逐级减小。在一个真实的系统上，各级 cache 之间硬件上是如何关联的呢？我们看下 Cortex-A53 架构上各级 cache 之间的硬件抽象框图如下：

![](https://pic3.zhimg.com/v2-155a251f204f87982b21b742002ef136_r.jpg)

在 Cortex-A53 架构上，L1 cache 分为单独的 instruction cache（ICache）和 data cache（DCache）。L1 cache 是 CPU 私有的，每个 CPU 都有一个 L1 cache。一个 cluster 内的所有 CPU 共享一个 L2 cache，L2 cache 不区分指令和数据，都可以缓存。所有 cluster 之间共享 L3 cache。L3 cache 通过总线和主存相连。

多级 cache 之间的配合工作
----------------

首先引入两个名词概念，命中和缺失。 CPU 要访问的数据在 cache 中有缓存，称为 “命中” (hit)，反之则称为 “缺失” (miss)。多级 cache 之间是如何配合工作的呢？我们假设现在考虑的系统只有两级 cache。

![](https://pic2.zhimg.com/v2-4974c1f109f00f887fceda68b37bd3f5_r.jpg)

当 CPU 试图从某地址 load 数据时，首先从 L1 cache 中查询是否命中，如果命中则把数据返回给 CPU。如果 L1 cache 缺失，则继续从 L2 cache 中查找。当 L2 cache 命中时，数据会返回给 L1 cache 以及 CPU。如果 L2 cache 也缺失，很不幸，我们需要从主存中 load 数据，将数据返回给 L2 cache、L1 cache 及 CPU。这种多级 cache 的工作方式称之为 inclusive cache。某一地址的数据可能存在多级缓存中。与 inclusive cache 对应的是 exclusive cache，这种 cache 保证某一地址的数据缓存只会存在于多级 cache 其中一级。也就是说，任意地址的数据不可能同时在 L1 和 L2 cache 中缓存。

直接映射缓存 (Direct mapped cache)
----------------------------

我们继续引入一些 cache 相关的名词。cache 的大小称之为 cahe size，代表 cache 可以缓存最大数据的大小。我们将 cache 平均分成相等的很多块，每一个块大小称之为 cache line，其大小是 cache line size。例如一个 64 Bytes 大小的 cache。如果我们将 64 Bytes 平均分成 64 块，那么 cache line 就是 1 字节，总共 64 行 cache line。如果我们将 64 Bytes 平均分成 8 块，那么 cache line 就是 8 字节，总共 8 行 cache line。现在的硬件设计中，一般 cache line 的大小是 4-128 Byts。为什么没有 1 byte 呢？原因我们后面讨论。

这里有一点需要注意，cache line 是 cache 和主存之间数据传输的最小单位。什么意思呢？当 CPU 试图 load 一个字节数据的时候，如果 cache 缺失，那么 cache 控制器会从主存中一次性的 load cache line 大小的数据到 cache 中。例如，cache line 大小是 8 字节。CPU 即使读取一个 byte，在 cache 缺失后，cache 会从主存中 load 8 字节填充整个 cache line。又是因为什么呢？后面说完就懂了。

我们假设下面的讲解都是针对 64 Bytes 大小的 cache，并且 cache line 大小是 8 字节。我们可以类似把这块 cache 想想成一个数组，数组总共 8 个元素，每个元素大小是 8 字节。就像下图这样。

![](https://pic1.zhimg.com/v2-3e0de5f8b95e27dbd41328c9d089224c_r.jpg)

现在我们考虑一个问题，CPU 从 0x0654 地址读取一个字节，cache 控制器是如何判断数据是否在 cache 中命中呢？cache 大小相对于主存来说，可谓是小巫见大巫。所以 cache 肯定是只能缓存主存中极小一部分数据。我们如何根据地址在有限大小的 cache 中查找数据呢？现在硬件采取的做法是对地址进行散列（可以理解成地址取模操作）。我们接下来看看是如何做到的？

![](https://pic2.zhimg.com/v2-e8deb539258684ad9d4dffef08b02c09_r.jpg)

我们一共有 8 行 cache line，cache line 大小是 8 Bytes。所以我们可以利用地址低 3 bits（如上图地址蓝色部分）用来寻址 8 bytes 中某一字节，我们称这部分 bit 组合为 offset。同理，8 行 cache line，为了覆盖所有行。我们需要 3 bits（如上图地址黄色部分）查找某一行，这部分地址部分称之为 index。现在我们知道，如果两个不同的地址，其地址的 bit3-bit5 如果完全一样的话，那么这两个地址经过硬件散列之后都会找到同一个 cache line。所以，当我们找到 cache line 之后，只代表我们访问的地址对应的数据可能存在这个 cache line 中，但是也有可能是其他地址对应的数据。所以，我们又引入 tag array 区域，tag array 和 data array 一一对应。每一个 cache line 都对应唯一一个 tag，tag 中保存的是整个地址位宽去除 index 和 offset 使用的 bit 剩余部分（如上图地址绿色部分）。tag、index 和 offset 三者组合就可以唯一确定一个地址了。因此，当我们根据地址中 index 位找到 cache line 后，取出当前 cache line 对应的 tag，然后和地址中的 tag 进行比较，如果相等，这说明 cache 命中。如果不相等，说明当前 cache line 存储的是其他地址的数据，这就是 cache 缺失。在上述图中，我们看到 tag 的值是 0x19，和地址中的 tag 部分相等，因此在本次访问会命中。由于 tag 的引入，因此解答了我们之前的一个疑问 “为什么硬件 cache line 不做成一个字节？”。这样会导致硬件成本的上升，因为原本 8 个字节对应一个 tag，现在需要 8 个 tag，占用了很多内存。tag 也是 cache 的一部分，但是我们谈到 cache size 的时候并不考虑 tag 占用的内存部分。

我们可以从图中看到 tag 旁边还有一个 valid bit，这个 bit 用来表示 cache line 中数据是否有效（例如：1 代表有效；0 代表无效）。当系统刚启动时，cache 中的数据都应该是无效的，因为还没有缓存任何数据。cache 控制器可以根据 valid bit 确认当前 cache line 数据是否有效。所以，上述比较 tag 确认 cache line 是否命中之前还会检查 valid bit 是否有效。只有在有效的情况下，比较 tag 才有意义。如果无效，直接判定 cache 缺失。

上面的例子中，cache size 是 64 Bytes 并且 cache line size 是 8 bytes。offset、index 和 tag 分别使用 3 bits、3 bits 和 42 bits（假设地址宽度是 48 bits）。我们现在再看一个例子：512 Bytes cache size，64 Bytes cache line size。根据之前的地址划分方法，offset、index 和 tag 分别使用 6 bits、3 bits 和 39 bits。如下图所示。

![](https://pic2.zhimg.com/v2-f6fdf760d314f146941e2192957f1a81_r.jpg)

直接映射缓存的优缺点
----------

直接映射缓存在硬件设计上会更加简单，因此成本上也会较低。根据直接映射缓存的工作方式，我们可以画出主存地址 0x00-0x88 地址对应的 cache 分布图。

![](https://pic2.zhimg.com/v2-b3d111caabc93c638bb08bde5026d711_r.jpg)

我们可以看到，地址 0x00-0x3f 地址处对应的数据可以覆盖整个 cache。0x40-0x7f 地址的数据也同样是覆盖整个 cache。我们现在思考一个问题，如果一个程序试图依次访问地址 0x00、0x40、0x80，cache 中的数据会发生什么呢？首先我们应该明白 0x00、0x40、0x80 地址中 index 部分是一样的。因此，这 3 个地址对应的 cache line 是同一个。所以，当我们访问 0x00 地址时，cache 会缺失，然后数据会从主存中加载到 cache 中第 0 行 cache line。当我们访问 0x40 地址时，依然索引到 cache 中第 0 行 cache line，由于此时 cache line 中存储的是地址 0x00 地址对应的数据，所以此时依然会 cache 缺失。然后从主存中加载 0x40 地址数据到第一行 cache line 中。同理，继续访问 0x80 地址，依然会 cache 缺失。这就相当于每次访问数据都要从主存中读取，所以 cache 的存在并没有对性能有什么提升。访问 0x40 地址时，就会把 0x00 地址缓存的数据替换。这种现象叫做 cache 颠簸（cache thrashing）。针对这个问题，我们引入多路组相连缓存。我们首先研究下最简单的两路组相连缓存的工作原理。

两路组相连缓存 (Two-way set associative cache)
---------------------------------------

我们依然假设 64 Bytes cache size，cache line size 是 8 Bytes。什么是路（way）的概念。我们将 cache 平均分成多份，每一份就是一路。因此，两路组相连缓存就是将 cache 平均分成 2 份，每份 32 Bytes。如下图所示。

![](https://pic1.zhimg.com/v2-4653656ec3d4d5942bae805df6723690_r.jpg)

cache 被分成 2 路，每路包含 4 行 cache line。我们将所有索引一样的 cache line 组合在一起称之为组。例如，上图中一个组有两个 cache line，总共 4 个组。我们依然假设从地址 0x0654 地址读取一个字节数据。由于 cache line size 是 8 Bytes，因此 offset 需要 3 bits，这和之前直接映射缓存一样。不一样的地方是 index，在两路组相连缓存中，index 只需要 2 bits，因为一路只有 4 行 cache line。上面的例子根据 index 找到第 2 行 cache line（从 0 开始计算），第 2 行对应 2 个 cache line，分别对应 way 0 和 way 1。因此 index 也可以称作 set index（组索引）。先根据 index 找到 set，然后将组内的所有 cache line 对应的 tag 取出来和地址中的 tag 部分对比，如果其中一个相等就意味着命中。

因此，两路组相连缓存较直接映射缓存最大的差异就是：第一个地址对应的数据可以对应 2 个 cache line，而直接映射缓存一个地址只对应一个 cache line。那么这究竟有什么好处呢？

两路组相连缓存优缺点
----------

两路组相连缓存的硬件成本相对于直接映射缓存更高。因为其每次比较 tag 的时候需要比较多个 cache line 对应的 tag（某些硬件可能还会做并行比较，增加比较速度，这就增加了硬件设计复杂度）。为什么我们还需要两路组相连缓存呢？因为其可以有助于降低 cache 颠簸可能性。那么是如何降低的呢？根据两路组相连缓存的工作方式，我们可以画出主存地址 0x00-0x4f 地址对应的 cache 分布图。

![](https://pic4.zhimg.com/v2-9db10cd5b86e5a10f08980ab1d1cfc07_r.jpg)

我们依然考虑直接映射缓存一节的问题 “如果一个程序试图依次访问地址 0x00、0x40、0x80，cache 中的数据会发生什么呢？”。现在 0x00 地址的数据可以被加载到 way 1，0x40 可以被加载到 way 0。这样是不是就在一定程度上避免了直接映射缓存的尴尬境地呢？在两路组相连缓存的情况下，0x00 和 0x40 地址的数据都缓存在 cache 中。试想一下，如果我们是 4 路组相连缓存，后面继续访问 0x80，也可能被被缓存。

因此，当 cache size 一定的情况下，组相连缓存对性能的提升最差情况下也和直接映射缓存一样，在大部分情况下组相连缓存效果比直接映射缓存好。同时，其降低了 cache 颠簸的频率。从某种程度上来说，直接映射缓存是组相连缓存的一种特殊情况，每个组只有一个 cache line 而已。因此，直接映射缓存也可以称作单路组相连缓存。

全相连缓存 (Full associative cache)
------------------------------

既然组相连缓存那么好，如果所有的 cache line 都在一个组内。岂不是性能更好。是的，这种缓存就是全相连缓存。我们依然以 64 Byts 大小 cache 为例说明。

![](https://pic3.zhimg.com/v2-1e61e8d13030ed4f0b42c2d1a854ffce_r.jpg)

由于所有的 cache line 都在一个组内，因此地址中不需要 set index 部分。因为，只有一个组让你选择，间接来说就是你没得选。我们根据地址中的 tag 部分和所有的 cache line 对应的 tag 进行比较（硬件上可能并行比较也可能串行比较）。哪个 tag 比较相等，就意味着命中某个 cache line。因此，在全相连缓存中，任意地址的数据可以缓存在任意的 cache line 中。所以，这可以最大程度的降低 cache 颠簸的频率。但是硬件成本上也是更高。

一个四路组相连缓存实例问题
-------------

考虑这么一个问题，32 KB 大小 4 路组相连 cache，cache line 大小是 32 Bytes。请思考以下 2 个问题：

1.  多少个组？
2.  假设地址宽度是 48 bits，index、offset 以及 tag 分别占用几个 bit？

总共 4 路，因此每路大小是 8 KB。cache line size 是 32 Bytes，因此一共有 256 组（8 KB / 32 Bytes）。由于 cache line size 是 32 Bytes，所以 offset 需要 5 位。一共 256 组，所以 index 需要 8 位，剩下的就是 tag 部分，占用 35 位。这个 cache 可以绘制下图表示。

![](https://pic4.zhimg.com/v2-ad47fa00875dcca7ea3e58b828edaeef_r.jpg)

Cache 分配策略 (Cache allocation policy)
------------------------------------

cache 的分配策略是指我们什么情况下应该为数据分配 cache line。cache 分配策略分为读和写两种情况。

### 读分配 (read allocation)

当 CPU 读数据时，发生 cache 缺失，这种情况下都会分配一个 cache line 缓存从主存读取的数据。默认情况下，cache 都支持读分配。

### 写分配 (write allocation)

当 CPU 写数据发生 cache 缺失时，才会考虑写分配策略。当我们不支持写分配的情况下，写指令只会更新主存数据，然后就结束了。当支持写分配的时候，我们首先从主存中加载数据到 cache line 中（相当于先做个读分配动作），然后会更新 cache line 中的数据。

Cache 更新策略 (Cache update policy)
--------------------------------

cache 更新策略是指当发生 cache 命中时，写操作应该如何更新数据。cache 更新策略分成两种：写直通和回写。

### 写直通 (write through)

当 CPU 执行 store 指令并在 cache 命中时，我们更新 cache 中的数据并且更新主存中的数据。**cache 和主存的数据始终保持一致**。

### 写回 (write back)

当 CPU 执行 store 指令并在 cache 命中时，我们只更新 cache 中的数据。并且每个 cache line 中会有一个 bit 位记录数据是否被修改过，称之为 dirty bit（翻翻前面的图片，cache line 旁边有一个 D 就是 dirty bit）。我们会将 dirty bit 置位。主存中的数据只会在 cache line 被替换或者显示的 clean 操作时更新。因此，主存中的数据可能是未修改的数据，而修改的数据躺在 cache 中。**cache 和主存的数据可能不一致。**

同时思考个问题，为什么 cache line 大小是 cache 控制器和主存之间数据传输的最小单位呢？这也是因为每个 cache line 只有一个 dirty bit。这一个 dirty bit 代表着整个 cache line 是否被修改的状态。

实例
--

假设我们有一个 64 Bytes 大小直接映射缓存，cache line 大小是 8 Bytes，采用写分配和写回机制。当 CPU 从地址 0x2a 读取一个字节，cache 中的数据将会如何变化呢？假设当前 cache 状态如下图所示 (tag 旁边 valid 一栏的数字 1 代表合法。0 代表非法。后面 Dirty 的 1 代表 dirty，0 代表没有写过数据，即非 dirty)。

![](https://pic3.zhimg.com/v2-ff2a4d78af3ff8d411e092a96941fd6a_r.jpg)

根据 index 找到对应的 cache line，对应的 tag 部分 valid bit 是合法的，但是 tag 的值不相等，因此发生缺失。此时我们需要从地址 0x28 地址（请注意 cacheline 大小对齐）加载 8 字节数据到该 cache line 中。但是，我们发现当前 cache line 的 dirty bit 置位。因此，cache line 里面的数据不能被简单的丢弃，由于采用写回机制，所以我们需要将 cache 中的数据 0x11223344 写到地址 0x0128 地址（这个地址根据 tag 中的值及所处的 cache line 行计算得到）。这个过程如下图所示。

![](https://pic3.zhimg.com/v2-1630dc6c3c099fdc1b92c8f33f1eea32_r.jpg)

当写回操作完成，我们将主存中 0x28 地址开始的 8 个字节加载到该 cache line 中，并清除 dirty bit。然后根据 offset 找到 0x52 返回给 CPU。

问题解答
----

回到最初提到的问题。不知你是否已经明白其中的原因。我们下篇文章仔细展开该问题。

[undefined](https://zhuanlan.zhihu.com/p/102326184)

### 扩展阅读

我们这里一直避开了一个关键问题。我们都知道 cache 控制器根据地址查找缓存并判断是否命中 cache，这里的地址究竟是虚拟地址 (virtual address，VA) 还是物理地址(physical address，PA)？可以前往下面的文章一探究竟。

[undefined](https://zhuanlan.zhihu.com/p/107096130)

### Note

1.  tag array 存储在硬件 cache 里，占用真实 cache 内存。但是我们提到 cache size 的时候，并没有考虑 tag 的占用。所以计算时，请忽略 tag 占用。

### 还有问题？

![](https://pica.zhimg.com/v2-a07ab0e5d8eb76b7de2989209842e361_l.jpg?source=f2fdee93)smcdef4 次咨询5.09825 次赞同去咨询