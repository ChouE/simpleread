> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/460092901)

*   [引言](https://zhuanlan.zhihu.com/p/460092901/edit#%E5%BC%95%E8%A8%80)
*   [一、理解委托](https://zhuanlan.zhihu.com/p/460092901/edit#%E4%B8%80%E7%90%86%E8%A7%A3%E5%A7%94%E6%89%98)

*   [1.1 初识委托](https://zhuanlan.zhihu.com/p/460092901/edit#11-%E5%88%9D%E8%AF%86%E5%A7%94%E6%89%98)
*   [1.2 简单对委托的思考](https://zhuanlan.zhihu.com/p/460092901/edit#12-%E7%AE%80%E5%8D%95%E5%AF%B9%E5%A7%94%E6%89%98%E7%9A%84%E6%80%9D%E8%80%83)

*   [二、UE 委托的概要](https://zhuanlan.zhihu.com/p/460092901/edit#%E4%BA%8Cue%E5%A7%94%E6%89%98%E7%9A%84%E6%A6%82%E8%A6%81)

*   [2.1 三大类型](https://zhuanlan.zhihu.com/p/460092901/edit#21-%E4%B8%89%E5%A4%A7%E7%B1%BB%E5%9E%8B)
*   [2.2 支持绑定类型](https://zhuanlan.zhihu.com/p/460092901/edit#22-%E6%94%AF%E6%8C%81%E7%BB%91%E5%AE%9A%E7%B1%BB%E5%9E%8B)
*   [2.3 UE 委托特性](https://zhuanlan.zhihu.com/p/460092901/edit#23-ue%E5%A7%94%E6%89%98%E7%89%B9%E6%80%A7)

*   [三、TDelegate 单播 C++ 实现](https://zhuanlan.zhihu.com/p/460092901/edit#%E4%B8%89tdelegate%E5%8D%95%E6%92%ADc%E5%AE%9E%E7%8E%B0)

*   [3.1 定义与使用](https://zhuanlan.zhihu.com/p/460092901/edit#31%E5%AE%9A%E4%B9%89%E4%B8%8E%E4%BD%BF%E7%94%A8)
*   [3.2 TDelegate 的 C++ 实现分析](https://zhuanlan.zhihu.com/p/460092901/edit#32-tdelegate%E7%9A%84c%E5%AE%9E%E7%8E%B0%E5%88%86%E6%9E%90)
*   [3.3 FDefaultDelegateUserPolicy 策略模式](https://zhuanlan.zhihu.com/p/460092901/edit#33-fdefaultdelegateuserpolicy%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F)
*   [3.4 绑定](https://zhuanlan.zhihu.com/p/460092901/edit#34-%E7%BB%91%E5%AE%9A)
*   [3.5 创建](https://zhuanlan.zhihu.com/p/460092901/edit#35-%E5%88%9B%E5%BB%BA)
*   [3.6 执行](https://zhuanlan.zhihu.com/p/460092901/edit#36-%E6%89%A7%E8%A1%8C)
*   [3.7 内嵌眼花缭乱的模板类](https://zhuanlan.zhihu.com/p/460092901/edit#37-%E5%86%85%E5%B5%8C%E7%9C%BC%E8%8A%B1%E7%BC%AD%E4%B9%B1%E7%9A%84%E6%A8%A1%E6%9D%BF%E7%B1%BB)

*   [四、DelegateInstance 的 C++ 实现](https://zhuanlan.zhihu.com/p/460092901/edit#%E5%9B%9Bdelegateinstance%E7%9A%84c%E5%AE%9E%E7%8E%B0)

*   [4.1 TBaseUObjectMethodDelegateInstance 的细节](https://zhuanlan.zhihu.com/p/460092901/edit#41-tbaseuobjectmethoddelegateinstance%E7%9A%84%E7%BB%86%E8%8A%82)
*   [4.2 TCommonDelegateInstanceState 的细节](https://zhuanlan.zhihu.com/p/460092901/edit#42-tcommondelegateinstancestate%E7%9A%84%E7%BB%86%E8%8A%82)
*   [4.3 IBaseDelegateInstance 与 IDelegateInstance](https://zhuanlan.zhihu.com/p/460092901/edit#43-ibasedelegateinstance%E4%B8%8Eidelegateinstance)

*   [五、TMulticastDelegate 多播](https://zhuanlan.zhihu.com/p/460092901/edit#%E4%BA%94tmulticastdelegate%E5%A4%9A%E6%92%AD)
*   [六、TBaseDynamicDelegate 动态委托](https://zhuanlan.zhihu.com/p/460092901/edit#%E5%85%ADtbasedynamicdelegate%E5%8A%A8%E6%80%81%E5%A7%94%E6%89%98)
*   [结语](https://zhuanlan.zhihu.com/p/460092901/edit#%E7%BB%93%E8%AF%AD)

引言
--

写完[《UE4 智能指针及与 STL 的对比》](https://zhuanlan.zhihu.com/p/446500670)后，好久没有写 C++ 专有文章了，想写一篇了，但话题只能在 UE 内。由于 UE 的委托还是比较偏纯 C++ 设计，加上内部有不少宏和模板，想要完全理解，可能对 C++ 新手不是特别友好。我个人作为一名 C++ 元编程爱好者，想在这里谈谈个人浅见，如果还能帮助到大家那再好不过了。上次我写了一篇《由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思》介绍委托的部分知识点，重点分析了函数签名类型转换与委拖部分结构和借 new 重载转换，还入选了 UE 的社区周报，本文就想在此基础上全面介绍委托，如果想深入的了解强列建议两篇一起读。

[由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思](https://zhuanlan.zhihu.com/p/433019649)[UE4 智能指针及与 STL 的对比](https://zhuanlan.zhihu.com/p/446500670)[rayhunter：C++ 中 VS2019 下 STL 的 std::tuple 元组深入剖析](https://zhuanlan.zhihu.com/p/397720700)[C++ 代码可以魔法到什么程度？](https://www.zhihu.com/question/472749525/answer/2084936755)

一、理解委托
------

第一章节非常简单，适合完全不了解委托的新同学，已经了解过委托的建议直接跳过。

### 1.1 初识委托

C++ 原生并没有委托，不过像 C# 语言原生就有，但 C++ 几乎是无所不能，像智能指针，内存池及 GC 非常重要的设计，就由我们自己实现，委托也不例外。不过委托这词还是有点抽象，用个例子类比一下。

行军打仗，侦察兵负责侦察，炮兵负责攻击，侦察兵将发现的敌军据点报告给炮兵，炮兵实施火力打击。从程序语言设计层面考虑：模块 A 代表侦察兵；模块 B 代表炮兵。A，B 模块本身无直接关联，相互独立解耦，侦查兵发现敌方据点后并不直接或者说根本无法对其进行进攻，委拖就像侦查兵身上配备的无线电，非常小巧，把据点位置告之炮兵，炮兵进行真正火力打击。委托的注册就像炮兵本身有个固定频率的电台，提前告之侦察兵，就一流程就是委托的注册，发现敌方据点后可以向这个特定的频率发送通知。更进一步，侦察兵只把发现的据点信息告之炮兵叫单播委托；如果信息化，狙击以及冲锋三个小队都关心敌方哨兵情况，告之侦察兵，如果发现哨兵立马通知我们（就是多播注册），当侦察兵实际勘察发现敌方哨兵时，立马通知这三个小队，三个小队可以按自己的方法处理收到的消息，这叫多播委托。

上面是简单的实例来理解委托，我们从非常简单的 C++ 代码来理解一下委托。

```
using DelegateType = void (*)(int x, int y);

void ArtilleryAction(int x, int y)
{
    printf("Drop a bomb on the (%d,%d) position\r\n", x, y);
}

class Scout
{
public:
    Scout(DelegateType Delegate) :ArtilleryDelegate(Delegate) {}
    void FindInformation()
    {
        (*ArtilleryDelegate)(rand(), rand());
    }
private:
    DelegateType ArtilleryDelegate;
};

int main()
{
    Scout OneScout(ArtilleryAction);
    OneScout.FindInformation();
    return 0;
}
```

我们将`void (*)(int x, int y)`无返回值且接受两个 int 型参数的函数指针类型重定义为：**DelegateType**，就是一种超级简单的委托类型。然后写了 Scout 侦察兵类，只有一个成员 ArtilleryDelegate，就是刚定义的 DelegateType 委托类型，在 Scout 构造函数中提供了委托的初始化赋值，即对委托进行绑定注册，在我们的 main 函数中，将委托绑定到了`void ArtilleryAction(int x, int y)`函数中了。最后当然类中还给了一个简单测试函数：`FindInformation`, 表示侦察兵发现敌方据点坐标信息时，将信息传给已经绑定委托回调函数，调用后会打印：Drop a bomb on the （xx,xx） Position，总的来说这就是委托的简单模型，Scout 类只是按它的功能发现信息，传递发现的信息，并不关心信息到底谁处理，怎么处理。这样设计很容易将模块解耦和组合起来，比如注册时也可以换成其它的符合注册条件的函数，给我们比较大的自由。

### 1.2 简单对委托的思考

有了这些基础知识，我先提一些想法，也是 UE 设计要面临的，自己提前考虑一下：

*   _委托只能是简单的函数指针吗？_
*   委托能提前保存函数参数吗？类似 std::bind 可以将参数提前缓存起来
*   _委托能对不同的函数类型进行转换吗_
*   委托如果都支持上面功能，怎么设计，成员数据怎么保存
*   _委托如果支持成员函数，怎么处理成员对象时效安全问题_
*   委托怎么设计支持多播
*   委托对 UE 专有 UObject 怎么进行反射处理

二、UE 委托的概要
----------

### 2.1 三大类型

UE 的委托分为三大类，具体的枚举分类可以参见：DelegateCombinations.h 文件

*   _**1. 单播委托:** TDelegate 模板类，通常用`DECLARE_DELEGATE`及`DECLARE_DELEGATE_XXXXParams`宏进行声明，后面的宏表示带 XXXX 个参数_
*   **2. 多播委托:** TMulticastDelegate 模板类，通常用`DECLARE_MULTICAST_DELEGATE`及`DECLARE_MULTICAST_DELEGATE_<Num>Params`宏进行声明，后面的宏表示带 XXXX 个参数
*   _**3. 动态委托:** 继承于 TBaseDynamicDelegate，TBaseDynamicMulticastDelegate 模板类，通常用`DECLARE_DYNAMIC_DELEGATE`及`DECLARE_DYNAMIC_MULTICAST_DELEGATE`宏进行声明，分别表示单播与多播，另外还有包含参数个数的宏如`DECLARE_DYNAMIC_DELEGATE_OneParam`，其本质集成了 UObject 的反射系统，让其可以注册蓝图实现的函数及支持序列化存储到本机，UFunction 只需给函数名称及蓝图支持是其最大的特性，概念上并未逃出前两类，性能和功能弱于前两类。_
*   **2.1 Event:** 其继承多播，不单独讨论了。

以单播为例，我们可以看一下表格的映射关系

|

### 2.2 支持绑定类型

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>函数签名类型</th><th>宏声明</th></tr><tr><td>void Function()</td><td>DECLARE_DELEGATE(DelegateName)</td></tr><tr><td>void Function(&lt;Param1&gt;)</td><td>DECLARE_DELEGATE_OneParam(DelegateName, Param1Type)</td></tr><tr><td>void Function(&lt;Param1&gt;,&lt;Parm2&gt; )</td><td>DECLARE_DELEGATE_TwoParams(DelegateName, Param1Type, Param2Type)</td></tr><tr><td>void Function(&lt;Param1&gt;, &lt;Parm2&gt; ,...)</td><td>DECLARE_DELEGATE_&lt;Num&gt;Params( DelegateName, Param1Type, Param2Type, ... )</td></tr><tr><td>&lt;RetVal&gt; Function()</td><td>DECLARE_DELEGATE_RetVal(RetValType, DelegateName)</td></tr><tr><td>&lt;RetVal&gt; Function(&lt;Parm1&gt;)</td><td>DECLARE_DELEGATE_RetVal_OneParam(RetValType, DelegateName, Param1Type)</td></tr><tr><td>&lt;RetVal&gt; Function(&lt;Parm1&gt;,&lt;Parm2&gt;)</td><td>DECLARE_DELEGATE_RetVal_TwoParams(RetValType, DelegateName, Param1Type, Param2Type)</td></tr><tr><td>&lt;RetVal&gt; Function(&lt;Param1&gt;,&lt;Parm2&gt;,...)</td><td>DECLARE_DELEGATE_RetVal_&lt;Num&gt;Params( RetValType, DelegateName, Param1Type, Param2Type, ... )</td></tr></tbody></table>

UE 的委托很强大，一但 2.1 中类型声明确定后，只要能将要绑定的类型转换为声明的类型都能进行注册绑定，这个在[《由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思》](https://zhuanlan.zhihu.com/p/433019649)有分析，后面也会有一定的讨论。下面的表列举了单播可以绑定的类型，单播是 **Bind 开头**，如果是多播注册，只需将 **Bind 换成 Add**

> 这里先简单说一下，UE 的委托类似 std::bind 可带有 tuple 保存信息，官方称为 **payload**，比如声明委托函数原类型是`void()(int)`型，比如想绑定注册`void MyClass::Test(int，int，string)`函数，明显类型是不匹配的。因为待注册的函数首先是一个成员函数，会隐藏第一个参数 this 指针，后面还多了两个参数 int，string; 但只需将 this 指针保存并使得后面两个变量值，比如 18 与 “helloworld” 也能提前保存到 **payload** 中，调用时再还原，也是可以转换成功的，当然还要考虑指针的有效性问题，这背后肯定有复杂的设计。  

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>方式</th><th>用途</th></tr><tr><td>BindStatic(&amp;GlobalFunctionName)</td><td>绑定静态函数，要么是普通全局函数要么是静态成员函数</td></tr><tr><td>BindUObject(UObject, &amp;UClass::Function)</td><td>TWeakObjectPtr 为安全性设计要点，绑定 UObject 类成员函数，object 必有效</td></tr><tr><td>BindSP(SharedPtr, &amp;FClass::Function)</td><td>TWeakPtr 为安全性设计要点，绑定普通类成员函数，注意是 F 开头，指针对象必有效</td></tr><tr><td>BindThreadSafeSP(SharedPtr, &amp;FClass::Function)</td><td>TWeakPtr 为安全性设计要点，绑定普通类成员函数，注意是 F 开头，指针对象必有效，只是线程安全版，安全版问题可以参考开头链接：《UE4 智能指针及与 STL 的对比》</td></tr><tr><td>BindRaw(RawPtr, &amp;FClass::Function)</td><td>绑定普通类成员函数，注意是 F 开头，这次没有智能指针帮助，没有绑定对象内存的安全检查处理，一但对象内存回收，我们必手动处理 Unbind 或者移除，不然可能 crash</td></tr><tr><td>BindLambda(Lambda)</td><td>绑定 lambda 表达式，没有安全检查，自己处理好指针或引用的延迟调用问题</td></tr><tr><td>BindWeakLambda(UObject, Lambda)</td><td>绑定 lambda 表达式，UObject 有效性可以确保，但 Lambda 的其它引用或指针就不确保了</td></tr><tr><td>BindUFunction(UObject, FName("FunctionName"))</td><td>同时支持 UObject 的普通委托和 Dynamic 委托，因为它是用 UFunction 名称查询的，借了 UE 的序列化</td></tr><tr><td>BindDynamic(UObject, &amp;UClass::FunctionName)</td><td>仅仅可用于 Dynamic 委托，成员函数 FunctionName 必须有 UFUNCTION 宏声明过</td></tr></tbody></table>

### 2.3 UE 委托特性

1.  1. 委托内部特性

*   支持回调函数带返回值
*   内部最高支持 4 个 **payload**，如果 4 个参数还不够你用的话，请考虑将部分参数合为结构体或者类

> 这个我细看了一下，实际上 C++ 模板及 tuple 设计都是可变参，支持无限多个。只是 UE 在 TDelegate 模板类中对 DelegateInstance 实例模板类反查只提供了 4 个参数，所以建议为 4 个，如果不需要反查，可以传无限多。

*   委托类型支持多参数，可以见 2.1 的声明定义，暂时最多 9 个，请考虑将部分参数合为结构体或者
*   支持 const 函数，比用 const 成员函数与非 const 成员函数
*   强大的函数签名类型转换
*   委托本体与委托实例解耦及交互设计

> 个人理解最后两点为精华，无论从 C++ 模板语法层面还是从设计使用上，都是非常赞的

1.  多播委托通过`Broadcast()`一次触发所有注册的回调函数执行，不支持函数带有返回值
2.  Dynamic 动态委托主要就是集成了 UObject 的反射系统，让其可以注册蓝图实现的函数及支持序列化存储到本机。当然也可以绑定原生 C++ 函数代码，前提是这个函数被 **UFUNCTION** 标记（因为 UFUNCTION 标记过，这个函数就可以进行反射与序列化了）
3.  UE 的委拖内部通过 **payload** 保存数据，实质是 UE 自己实现的 TTuple，tuple 本身可以支持任意类型变量，所以这里可以让委托变得异常强大，有了这个可以轻松处理不同类型的函数转换。

三、TDelegate 单播 C++ 实现
---------------------

先看结构图, 图可能直接看不清，双击查看原图，画这个图费了我不少时间：

![](https://pic3.zhimg.com/v2-1e159439edd6b5bc632b9defa04cf236_r.jpg)

总括一下：TDelegate 是 UE 提供的单播委托，声明委托时目前限定了最多可携带 9 个参数。支持多种可调用对象（普通函数，成员函数，仿函数，lambda 等）到声明函数类型进行转换，同时转换的时还支持额外预保存 payload 参数，类似 std:bind 一样。TDelegate 委托自己没有成员，使用了策略模式继承于基类 FDelegateBase。基类 FDelegateBase 通过 new 两次重载提供对委托实例的内存分配，并且提供释放及访问方法，这样将委托本体与真正实例解耦开，委托本身不直接保存实例对象，实例由于有多种类型，通过中间层也可以很好的解决了类型擦除与转换的问题，委托本体的 size 也比较小，只有 12 字节，由于对齐原因，占 16 字节。

### 3.1 定义与使用

1.  定义单播委托 FOnMontageEnded 类型的实现

```
DECLARE_DELEGATE_TwoParams( FOnMontageEnded, class UAnimMontage*, bool );
```

1.  在`UAbilityTask_PlayMontageAndWait`类中定义委托对象 MontageEndedDelegate，作为类成员对象

```
class UAbilityTask_PlayMontageAndWait
{
    ...
private:
    FOnMontageEnded MontageEndedDelegate;
}
```

1.  在 cpp 代码中注册绑定委托的回调函数，并拷贝

```
// 委托的回调函数
void UAbilityTask_PlayMontageAndWait::OnMontageEnded(UAnimMontage* Montage, bool bInterrupted)
{
    if (!bInterrupted)
    {
        if (ShouldBroadcastAbilityTaskDelegates())
        {
            OnCompleted.Broadcast();
        }
    }

    EndTask();
}

void UAbilityTask_PlayMontageAndWait::Activate()
{
    // 注册
    MontageEndedDelegate.BindUObject(this, &UAbilityTask_PlayMontageAndWait::OnMontageEnded);
    // 委托拷贝给动画实列
    AnimInstance->Montage_SetEndDelegate(MontageEndedDelegate, MontageToPlay);
}
```

1.  执行：

```
void UAnimInstance::TriggerMontageEndedEvent(const FQueuedMontageEndedEvent& MontageEndedEvent)
{
    // 动画册实例执行委托
    MontageEndedEvent.Delegate.ExecuteIfBound(MontageEndedEvent.Montage, MontageEndedEvent.bInterrupted);
}
```

### 3.2 TDelegate 的 C++ 实现分析

1.  先手动展开一下委托定义的宏

```
DECLARE_DELEGATE_TwoParams( FOnMontageEnded, class UAnimMontage*, bool );
typedef TDelegate<void(class UAnimMontage*,bool)> FOnMontageEnded;
```

根据 **DelegateSignatureImpl.inl** 文件中 TDelegate 的定义

```
template <typename InRetValType, typename... ParamTypes, typename UserPolicy>
class TDelegate<InRetValType(ParamTypes...), UserPolicy> : public TDelegateBase<UserPolicy>{};
```

*   模板参数：InRetValType，实例化为 void
*   可变模板参数：ParamTypes，实例化为 class UAnimMontage * 和 bool
*   模板参数：UserPolicy，实例化为 FDefaultDelegateUserPolicy

> 这里说一下，虽然 c++ 语法上支持可变参 ParamTypes 个数为无限多，但由于 UE 自己在 **DelegateCombinations.inl** 文件中使用宏进行了包装，ex：DECLARE_DELEGATE_NineParams，使其最多支持 9 个参数。从实际角度来说 9 个函数参数足够了，如果实在不够可以扩充一下宏或者将多个参数用结构合并在一起。

*   1. 继承的基类`TDelegateBase<FDefaultDelegateUserPolicy>`本身无成员，也没有什么特别的，只是提供了一些方法，调用其基类 UserPolicy::FDelegateExtras 的`GetDelegateInstanceProtected()`方法，得到**委托真正实例对象**指针。实例结构 **XXXXDelegateInstance** 位于我们本章节开头总括图的右侧，通过虚线与 TDelegate 相连。最后再用指针去调用真正的接口，比如 GetHandle() 获取句柄，起着打通委托与实例之间中间件的作用。  
    
*   2. TDelegate 的根基类缺省默认值是 **FDelegateBase**，我们也可以手动指定，比较重要。FDelegateBase 有两个成员，一个是 DelegateSize 指代分配实例的内存大小，一个是 DelegateAllocator 指示内存分配器。这个类主要就是完成对委托实例的内存分配与保存，并且提供了`Unbind`函数用来析构并释放实例的内存。

```
class FDelegateBase
{
    IDelegateInstance* GetDelegateInstanceProtected() const;
    void Unbind();
private:
    friend void* operator new(size_t Size, FDelegateBase& Base);
    void* Allocate(int32 Size);
private:
    FDelegateAllocatorType::ForElementType<FAlignedInlineDelegateType> DelegateAllocator;
    int32 DelegateSize; 
};
inline void* operator new(size_t Size, FDelegateBase& Base)
{
    return Base.Allocate((int32)Size);
}
```

> FDelegateBase 涉及内存的分配与释放及获取与类型强转，在[《由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思》](https://zhuanlan.zhihu.com/p/433019649)中已经做过了详细的解释，可以查阅。

### 3.3 FDefaultDelegateUserPolicy 策略模式

整个委托系统，由于`TDelegate`，`TMulticastDelegate`，`IBaseDelegateInstance`都用运用了策略模式，**FDefaultDelegateUserPolicy** 作为模板的默认参数传入，让这三个类继承 FDefaultDelegateUserPolicy 中的类型重定义。这个模板默认参数有点像 STL 容器的第二个默认模板参数 ---- 内存分配器，部分人根本没有察觉到，我们也可以手动指定，只是通常不会去改。

这个策略类内部有三个 using 重定义 FDelegateInstanceExtras，FDelegateExtras，FMulticastDelegateExtras，我们可以手动替换，但要求必须公有继承于 IDelegateInstance，FDelegateBase，TMulticastDelegateBase，否则 static_assert 对模板手动指定的 UserPolicy 类型转换检测是通不过的，会造成编译失败。

```
struct FDefaultDelegateUserPolicy
{
    using FDelegateInstanceExtras  = IDelegateInstance;
    using FDelegateExtras          = FDelegateBase;
    using FMulticastDelegateExtras = TMulticastDelegateBase<FDefaultDelegateUserPolicy>;
};
```

### 3.4 绑定

在[《由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思》](https://zhuanlan.zhihu.com/p/433019649)中已经做过了详细的解释，但我们还是可以简单介绍一下，以 3.1 的代码为例。

```
// 注册
MontageEndedDelegate.BindUObject(this, &UAbilityTask_PlayMontageAndWait::OnMontageEnded);
```

我们看 TDelegate 模板类的实现，BindUObject 这里分为 const 与非 const 版本，这例非 const 版本。如果用错，static_assert 也检测导致编译失败，所以对应下面的代码，这里模板参较多些，还是嵌套在模板类中，为了帮助新手，简单说一下函数要点。

```
template <typename UserClass, typename... VarTypes>
inline void BindUObject(UserClass* InUserObject, typename TMemFunPtrType<false, UserClass, RetValType (ParamTypes..., VarTypes...)>::Type InFunc, VarTypes... Vars)
{
    static_assert(!TIsConst<UserClass>::Value, "Attempting to bind a delegate with a const object pointer and non-const member function.");

    *this = CreateUObject(InUserObject, InFunc, Vars...);
}
```

> **重要！！！**，3.2 中已经确定了 TDelegate 的类模板参数，明确了委托函数原型是`void(class UAnimMontage*,bool)`，但实际绑了定的是一个成员函数，类型并不相同。  
>   
> 传给 BindUObject 的第一个参数 this 指针，从这个参数可以推断出模板参：UserClass 为 UAbilityTask_PlayMontageAndWait。  
> 可变参 VarTypes... 是除函数声明两个参数外额外绑定的参数，就是 2.2 小节中说的 payload 参数，这是不必有。举例一下：`void ClassA::MyTestFun(class UAnimMontage*,bool,int,float)`如果想将这个函数绑定到我们的委托中，实际多了三个参数，this 指针参数被单独处理了，多了 int 参数转到第一个 VarTypes 中，多了 float 参数转到第二个 VarTypes，且在函数注册时一定要给出 int 与 float 的值，实例化到 tuple 中，如`MontageEndedDelegate.BindUObject(this, &UAbilityTask_PlayMontageAndWait::OnMontageEnded,100,99.f);`。  
> 还有 RetValType 是从 TDelegate 模板类传过来的，本身支持返回类型，但本例是 void；  
> 可变参 ParamTypes... 也是从 TDelegate 模板类传过来的，本例就是 class UAnimMontage * 和 bool  
>   
> TMemFunPtrType 是 **DelegateInstanceInterface.h** 中转为类成员函数指针的重定义，将其重定义为 Type，也比较简单，本身经过模板特化会自动选择带不带 const 版本，我们传入的 false 表示非 const 版本。到此函数用到的细节我们都说了，希望对新人有一定帮助。  

### 3.5 创建

我们分析不完所有的，就还拿我们这个为例分析简要分析，代码会转到下面这里。本函数主要内容就是创建 DelegateInstance 对象并关联上 TDelegate，可以让 TDelegate 调用。

```
template <typename UserClass, typename... VarTypes>
UE_NODISCARD inline static TDelegate<RetValType(ParamTypes...), UserPolicy> CreateUObject(UserClass* InUserObject, typename TMemFunPtrType<false, UserClass, RetValType (ParamTypes..., VarTypes...)>::Type InFunc, VarTypes... Vars)
{
    static_assert(!TIsConst<UserClass>::Value, "Attempting to bind a delegate with a const object pointer and non-const member function.");

    TDelegate<RetValType(ParamTypes...), UserPolicy> Result;
    TBaseUObjectMethodDelegateInstance<false, UserClass, FuncType, UserPolicy, VarTypes...>::Create(Result, InUserObject, InFunc, Vars...);
    return Result;
}
```

> 这个里面模板参数 3.4 小节已经全部解释完毕，就简述功能了，不过这里多个两个：**UserPolicy** 来源于 TDelegate 在模板参，这里就是 FDefaultDelegateUserPolicy。**FuncType** 是 TDelegate 的内部的重定义，也就是 RetValType(ParamTypes...) 函数指针原型。  
> 栈上创建一个 TDelegate 对象 Result，然后转到 DelegateInstance 的创建，这一步将 this 指针，成员函数地址，**扩展的函数参数保存到 Instance 对象中 payload 中**，并藉由两次 new 重载，打通了 TDelegate 对象（Result）与 DelegateInstance 对象联系，最后返回委托。更加细节的创建与转换，还麻烦参考[《由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思》](https://zhuanlan.zhihu.com/p/433019649)，这文章就是针对这个技术点的。  
>   

限于篇幅，我们不可能将所有 Bind 与 Create 都讲一遍，但总体思路是一致的。

### 3.6 执行

到了执行就比较简单了, 总共有两个版本

```
FORCEINLINE RetValType Execute(ParamTypes... Params) const
{
    DelegateInstanceInterfaceType* LocalDelegateInstance = GetDelegateInstanceProtected();
    checkSlow(LocalDelegateInstance != nullptr);
    return LocalDelegateInstance->Execute(Params...);
}

template <
    typename DummyRetValType = RetValType,
    std::enable_if_t<std::is_void<DummyRetValType>::value>* = nullptr
>
inline bool ExecuteIfBound(ParamTypes... Params) const
{
    if (DelegateInstanceInterfaceType* Ptr = GetDelegateInstanceProtected())
    {
        return Ptr->ExecuteIfSafe(Params...);
    }
    return false;
}
```

*   **Execute** 版本比较简单，获取 DelegateInstance 对象的指针，转移到 Instance 的对函数调用，如果函数指针对象无效，可能会触发崩溃，为了安全建议，先用 IsBound() 判断，同时单播这个版本，支持 RetValType 型返回值。
*   **ExecuteIfBound** 版本麻烦一点点，这是一个模板函数，运用了模板的 SFINAE 技术确保返回类型是 void。然后最大的特点是安全，只有指针有效，才调用。同时这个函数将返回值转为 bool 了，成功调用返回 true

### 3.7 内嵌眼花缭乱的模板类

![](https://pic1.zhimg.com/v2-10b887bbbd3fd5ae2ef98ce2014dd824_r.jpg)

估计一些不熟悉 C++ 的看到 TDelegate 类内部这样的代码，估计就晕了，这是什么鬼！！！

关了这部分，我查了一下源码，的确用的不多，我们只有在 Widgets 的模块中，也是宏定义中有用到，看起来也有点复杂, 如下面代码

```
template< class UserClass, typename Var1Type, typename Var2Type >   \
WidgetArgsType& EventName##_UObject( UserClass* InUserObject, typename DelegateName::template TUObjectMethodDelegate_TwoVars_Const< UserClass, Var1Type, Var2Type >::FMethodPtr InFunc, Var1Type Var1, Var2Type Var2 )  \
{ \
    _##EventName = DelegateName::CreateUObject( InUserObject, InFunc, Var1, Var2 ); \
    return *this; \
} \
```

我只能说说个人的理解，这里要提前用到第四章节 DelegateInstance 的知识。从 C++ 设计层面来说，TDelegate 模板类综合看来不过是继承了 FDelegate, 并没有真正保存委托实例。TDelegate 委托虽然有函数原型的签名，但 UE 内部做了签名转换，让我们近乎自由的绑定可调用对象，这**导致我们只知道委托的类型，比如宏中的 DelegateName，根本无法知道它可能绑定的 DelegateInstance 对象的类型**。图片中密密麻麻的类就是做这个的，这样我给一个确定委托后，只要我再补一些扩展参数，就能结合委托本身模板参数，确定得到一个 DelegateInstance 实例的类型或者实例内部定义的类型，`typename DelegateName::template TUObjectMethodDelegate_TwoVars_Const< UserClass, Var1Type, Var2Type >::FMethodPtr`像这样不过是元编程常见的写法罢了。不过话说回来，你非不要这样写，模板参数显示传入，拆开重写一遍也是可以的，只是不够精练，有点 low，达不到 C++ 应有的水平罢了。

四、DelegateInstance 的 C++ 实现
---------------------------

前面说了 TDelegate 的细节，从本质来说，他只是一个空壳，但是**支持多种类型 Instance 并且良好和 Instance 交互**，真正的内容还要看看 Instance。

### 4.1 TBaseUObjectMethodDelegateInstance 的细节

这里以前文中 TBaseUObjectMethodDelegateInstance 为例，如果前文细看了，到这一步也没有什么太多难度了。

```
template <bool bConst, class UserClass, typename WrappedRetValType, typename... ParamTypes, typename UserPolicy, typename... VarTypes>
class TBaseUObjectMethodDelegateInstance<bConst, UserClass, WrappedRetValType(ParamTypes...), UserPolicy, VarTypes...> : public TCommonDelegateInstanceState<WrappedRetValType(ParamTypes...), UserPolicy, VarTypes...>
{
public:
TBaseUObjectMethodDelegateInstance(UserClass* InUserObject, FMethodPtr InMethodPtr, VarTypes... Vars)；
void CreateCopy(FDelegateBase& Base) final ；
RetValType Execute(ParamTypes... Params) const final ；
bool ExecuteIfSafe(ParamTypes... Params) const final ；
FORCEINLINE static void Create(FDelegateBase& Base, UserClass* InUserObject, FMethodPtr InFunc, VarTypes... Vars)
{
  new (Base) UnwrappedThisType(InFunc, Vars...);
}
protected:
    TWeakObjectPtr<UserClass> UserObject;
    FMethodPtr MethodPtr;
}
```

*   TBaseUObjectMethodDelegateInstance 的模板参，我们 3.4 小节，3.5 小节都解析过了，没有什么特别好说的。
*   成员上 UserObject 使用 **TWeakObjectPtr** 保存了 UObject 对象的智能指针弱引用，构造时发生
*   成员 MethodPtr 保存了 UObject 类的函数指针对象，构造时发生，注意这里 FMethodPtr 类型实际是一个重定义：`using FMethodPtr = typename TMemFunPtrType<bConst, UserClass, RetValType(ParamTypes..., VarTypes...)>::Type;`, 这个结构我们 3.4 小节提了，就是一个转换后的签名，表示类成员函数。

成员函数上，也有几个比较重要的：

*   _**Execute**：执行，非安全版，**模拟 C++17 中 std::tuple 及 std::apply(...) 函数**相对比较简单，可以将 tuple 中参数传入并调用函数, 并支持返回值。从语法来上说，它们考虑到了模板内部处理时，object 指针对象可能已经转换 const 版本，所以进行擦除 const 修饰，并使用了 const_cast 将其转为非 const 版本指针。_

> 如果这点你继续看进去，可能领悟到 C++ 模板的精妙，tuple 加 invoke 两个点，还是有技术含量的。

*   _**ExecuteIfSafe**：执行，安全版，相比上面 Execute，多了对弱引用获取对象安全检测，只有检查通过，才会进行函数调用。_
*   **Create**：**一个非常非常重要的静态成员函数**，利用 new 的重载及 TBaseFunctorDelegateInstance 的构造函数，传引用 Base 分配我们 Instance 内存，因为这个 Base 实际就是各种 TDelegate，这样 TDelegate 就和 Instance 打通了。在[《由 UE4 注册 TDelegate 的委托让我陷入了 C++ 的沉思》](https://zhuanlan.zhihu.com/p/433019649)重点讨论过，有兴趣参考。
*   **CreateCopy**：虽是普通成员函数，但实质还是 Create 换个形式，关键还是 new 的重载。

### 4.2 TCommonDelegateInstanceState 的细节

从我们最开始的缩略图看，所有 DelegateInstance 都派生于 **TCommonDelegateInstanceState**，以我个人理解，不得不说老外起名就是好啊，这个 State 太妙了，因为这个类就是为 payload 而生，带状态带缓存服务的，

```
template <typename InRetValType, typename... ParamTypes, typename UserPolicy, typename... VarTypes>
class TCommonDelegateInstanceState<InRetValType(ParamTypes...), UserPolicy, VarTypes...> : IBaseDelegateInstance<InRetValType(ParamTypes...), UserPolicy>
{
public:
    using RetValType = InRetValType;
public:
    explicit TCommonDelegateInstanceState(VarTypes... Vars)
        : Payload(Vars...)
        , Handle (FDelegateHandle::GenerateNewHandle){}
    {
    }

    FDelegateHandle GetHandle() const final
    {
        return Handle;
    }

protected:
    TTuple<VarTypes...> Payload;
    FDelegateHandle Handle;
};
```

这个模板类非常简单，模板参数已经分析了很多遍，不再累述。explicit 显式的构造函数，将我们前面讨论的**额外函数参数**保存到 UE 的 tuple 容器中，也就是 Payload，然后产生 DelegateInstance 实例的句柄，内部就是 uint64 ID。了解 C++ 的都知道，有了 tuple 的助力，那就如虎添翼，支持任意类型参数的保存，还支持转发到函数调用，std::bind 也是供助这个实现。以前写过 stl::tuple 分析，有兴趣参考，UE 的还没有分析过。[《C++ 中 VS2019 下 STL 的 std::tuple 元组深入剖析》](https://zhuanlan.zhihu.com/p/397720700)

### 4.3 IBaseDelegateInstance 与 IDelegateInstance

再向上看一层，也就是 TCommonDelegateInstanceState 的基类 IBaseDelegateInstance

```
template <typename RetType, typename... ArgTypes, typename UserPolicy>
struct IBaseDelegateInstance<RetType(ArgTypes...), UserPolicy> : public UserPolicy::FDelegateInstanceExtras
{
    virtual void CreateCopy(FDelegateBase& Base) = 0;
    virtual RetType Execute(ArgTypes...) const = 0;
    virtual bool ExecuteIfSafe(ArgTypes...) const = 0;
};
```

这个模板类非常简单，没有任何成员，还只是纯虚函数，提供了接口，接口我们也已经讨论过了，模板参数已经分析过了。

**IDelegateInstance** 更是所有 Instance 的虚基类，它不再是模板类，它只是提供接口及类型转换上便利，同`FDelegate`，`TDelegateBase<UserPolicy>`，`TMulticastDelegateBase<UserPolicy>`打交道，用基类指针太方便了，类型擦除方便啊，这是常见的 C++ 设计。

五、TMulticastDelegate 多播
-----------------------

仔细了解了单播委托后，实际再了解多播就非常简单了，我们也不用详细分析了，点出不同点或者一些技术特性就行了。

![](https://pic1.zhimg.com/v2-9b81d8f217c630de7aaaa28ae22d6268_r.jpg)

*   _用宏`DECLARE_MULTICAST_DELEGATE`和`DECLARE_MULTICAST_DELEGATE_<Num>Params`定义, 后面表示函数参数个数_
*   多播定义时函数的返回值必为 void, 否则 static_assert 编译检查失败
*   _多播 TMulticastDelegate 本身相比 TDelegate 来说，继承关系还更简单，那个因为它充分利用了单播已有的结构或信息。结合图示来说，TMulticastDelegate 来身不具有任何新加成员，成员都放到了基类 TMulticastDelegateBase 中了，_
*   多播最重要的成员是`TInvocationListType`类型的数组 InvocationList，我们所有委托都保存在这里。TInvocationListType 是一个 C++ 上的数组的重定义, 数组元素类型就是`TDelegateBase<UserPolicy>`，由前面的单播已知这个结构和 DelegateInstance 关联。

```
using InvocationListType = TArray<TDelegateBase<UserPolicy>, FMulticastInvocationListAllocatorType>;
```

*   多播正如字面意思，就是这个委托可以绑定多个回调函数，我们原来 TDelegate 用 BindXXXXX 开头绑定注册都换成了 AddXXXX 开头的注册，毕竞 TDelegate 只能绑一个用 Bind 词表达准确，TMulticastDelegate 可以绑多个，用 Add 也更符合。
*   随意分析一个 Add 添加绑定:**AddUObject**

```
template <typename UserClass, typename... VarTypes>
inline FDelegateHandle AddUObject(UserClass* InUserObject, typename TMemFunPtrType<false, UserClass, void (ParamTypes..., VarTypes...)>::Type InFunc, VarTypes... Vars)
{
    static_assert(!TIsConst<UserClass>::Value, "Attempting to bind a delegate with a const object pointer and non-const member function.");

    return Add(FDelegate::CreateUObject(InUserObject, InFunc, Vars...));
}
```

> 我们已经知道，委托声明时函数的签名是普通函数，这里绑定的成员函数，所以和单播一样，也要进行一些类型转换, 这里要求成员函数的返回值类型是 void；AddUObject 注册结束后，返回委托实例的句柄。  
>   
> 模板参数：UserClass 可以从 InUserObject 推断，是 Object 所对应的类型，注意这里有 const 与非 const 版本检测  
> 可变模板参数：VarTypes 同 TDelegate 一样，是成员函数额外补的参数，会放入 payload 中，可以有，也可以不添加，不再累述。  
> TMemFunPtrType 前面也讲过了，主要用于转换成成员函数，不再累述。  
>   
> 其中 FDelegate::CreateUObject 这句可能有点让人看不懂，那是因为在模板类 TMulticastDelegate 中有个重定义：`using FDelegate = TDelegate<void(ParamTypes...), UserPolicy>;`, 看到了吗？多播供助已有的单播结构，调用其静态成员函数创建一个 TDelegate 单播委托并返回，这样代码不就是很好的复用了吗！设计的还是非常不错的，模板也用的好。  
> 将生成 TDelegate 传给 TMulticastDelegate 的 Add 函数，Add 函数内部再调用基类的`Super::AddDelegateInstance`，将其添加到 InvocationList 中，这样数组就维护了我们的委托及实例了。

*   执行也比较简单，**Broadcast** 通过循环遍历我们的数组，不过这里 UE 增加了安全执行的调用`(DelegateInstanceInterfaceType*)DelegateInstanceInterface)->ExecuteIfSafe(Params...)`
*   我再提一点 C++ 模板的知识，在模板类的继承中，我们是**无法直接使用模板基类的函数**，可以通过 **using，this->, 或者显式传入参数调用三种方式**，这里 UE 使用了 using 方法, 并且可以添加 public,private 控制权限，我们可以看一点点例子，不知为什么，有时我看到 UE 的模板代码能让我越看越兴奋。

```
public:
    // Make sure TMulticastDelegateBase's public functions are publicly exposed through the TMulticastDelegate API
    using Super::Clear;
    using Super::IsBound;
    using Super::IsBoundToObject;
    using Super::RemoveAll;

private:
    // Make sure TMulticastDelegateBase's protected functions are not accidentally exposed through the TMulticastDelegate API
    using Super::AddDelegateInstance;
    using Super::RemoveDelegateInstance;
    using Super::CompactInvocationList;
```

六、TBaseDynamicDelegate 动态委托
---------------------------

仔细了解了单播与多播委托后，实际再了解动态委托就非常简单了，我们也不用详细分析了，提一些技术点就行了。

*   _动态委托支持单播与多播，对 UObject 支持及蓝图及序列化的支持，可以理解相当于为 UE 本身专门定制的。_ 相比前面介绍的单播与多播两种，**这种委托能在蓝图使用，可以说这是最大的优势吧**，并且绑定利用 UFunction 的序列化技术，只需要函数名称就可以了，相对多了一些查找消耗。
*   _用宏`DECLARE_DYNAMIC_DELEGATE`及`DECLARE_DYNAMIC_MULTICAST_DELEGATE`宏进行定义，分别表示支持动态的单播与多播，同时可以支持含参数个数的宏：`DECLARE_DYNAMIC_DELEGATE_<Num>Param`定义, Num 表示函数参数个数_
*   **TBaseDynamicDelegate** 的基类是`TScriptDelegate<TWeakPtr>`, 本身无成员，接口也比较简单，基本不用分析。内部通过`__Internal_BindDynamic`函数进行注册绑定，但实际被宏`BindDynamic`包装，我们使用宏`BindDynamic`来进行注册。

```
template <typename TWeakPtr = FWeakObjectPtr>
class TScriptDelegate
{
public:
    template <class UObjectTemplate>
    void ProcessDelegate( void* Parameters ) const;
protected:
    TWeakPtr Object;
    FName FunctionName; 
}
```

> TScriptDelegate 默认的成员 Object 是 UObject 的弱引用智能指针，FunctionName 是我们要绑定的函数名称，注意这个函数一定要是 UFunction 标记或者蓝图中的函数，简言之就是该函数能能序列化。当注册绑定时会将 Object 与 FunctionName 进行赋值。  
> 虽然我们的执行在派生类被`Execute()`函数包装了一次，详情见 **Delegate.h** 文件中`FUNC_DECLARE_DYNAMIC_DELEGATE`,`FUNC_DECLARE_DYNAMIC_DELEGATE_RETVAL`两个宏，宏里面的展开式进行了安全检测。不过话说回来，但真正的执行，还是会调用到基类的`ProcessDelegate`函数，执行的时间，会进行查找，所以效率上来说，是前面两种 C++ 版效率低一些。

*   **TBaseDynamicMulticastDelegate** 的基类是`TMulticastScriptDelegate<TWeakPtr>`, 本身无成员，接口也比较简单，基本不用分析。内部通过`__Internal_AddDynamic`和`__Internal_AddUniqueDynamic`函数进行注册绑定, 后面的能够确保唯一性，但实际被宏`AddDynamic`和`AddUniqueDynamic`包装，我们使用宏`AddDynamic`和`AddUniqueDynamic`来进行注册。移除的话内部使用`__Internal_RemoveDynamic`，同样被宏`RemoveDynamic`包装，这才是真正调用的地方。

```
template <class UObjectTemplate>
void ProcessMulticastDelegate(void* Parameters) const
{
public:
    typedef TArray< TScriptDelegate<TWeakPtr> > FInvocationList;

    template <class UObjectTemplate>
    void ProcessDelegate( void* Parameters ) const;
protected:
    mutable FInvocationList InvocationList; 
}
```

> 默认的成员 InvocationList 是`TArray< TScriptDelegate<TWeakPtr> >`是一个数组，做法也等同 TMulticastDelegate，不再累述了。  
> 虽然我们的执行在派生类被`Broadcast()`函数包装了一次，详情见 **Delegate.h** 文件中`FUNC_DECLARE_DYNAMIC_MULTICAST_DELEGATE`宏，宏里面的展开式进行了安全检测。不过话说回来，但真正的执行，还是会调用到基类的`ProcessMulticastDelegate`函数。

*   关于序列化，因为基类 TScriptDelegate，都重载了`<<`操作符。

结语
--

到此我们已经全部分析完毕，可能有纰漏，欢迎联系 rayhunter, 未经许可，禁止转载。

[UE4 反射基础一：揭秘 UBT 生成代码、UObject 注册、UClass 及 CDO 生成](https://zhuanlan.zhihu.com/p/427575094)