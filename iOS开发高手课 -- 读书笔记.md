## iOS开发高手课 -- 读书笔记

### 01 | iOS知识体系

![img](https://static001.geekbang.org/resource/image/ec/1f/ec339916b408ae3c86a5ee237ae3dc1f.png)

### 02 | App启动速度优化与监控

一般情况下，App 的启动分为冷启动和热启动。

- 冷启动是指， App 点击启动前，它的进程不在系统里，需要系统新创建一个进程分配给它启动的情况。这是一次完整的启动过程。
- 热启动是指 ，App 在冷启动后用户将 App 退后台，在 App 的进程还在系统里的情况下，用户重新启动进入 App 的过程，这个过程做的事情非常少。

#### App 的启动主要包括三个阶段：

1. main() 函数执行前
2. main() 函数执行后
3. 首屏渲染完成后

##### main() 函数执行前

- **加载可执行文件**（App 的.o 文件的集合）可执行文件是指Mach-O格式的文件；
- **加载动态链接库**，进行 rebase 指针调整和 bind 符号绑定；
- **Objc 运行时的初始处理**，包括 Objc 相关类的注册、category 注册、selector 唯一性检查等；
- **初始化**，包括了执行 +load() 方法、attribute((constructor)) 修饰的函数的调用
- **创建 C++ 静态全局变量**

**启动速度优化TODO：**

- **减少动态库加载**。每个库本身都有依赖关系，苹果公司建议使用更少的动态库，并且建议在使用动态库的数量较多时，尽量将多个动态库进行合并。数量上，苹果公司最多可以支持 6 个非系统动态库合并为一个。
- **减少加载启动后不会去使用的类或者方法**。
- **+load() 方法里的内容可以放到首屏渲染完成后再执行，或使用 +initialize() 方法替换掉**。因为，在一个 +load() 方法里，进行运行时方法替换操作会带来 4 毫秒的消耗。不要小看这 4 毫秒，积少成多，执行 +load() 方法对启动速度的影响会越来越大。
- **控制 C++ 全局变量的数量**。

##### main() 函数执行后

指的是从 main() 函数执行开始，到 appDelegate 的 didFinishLaunchingWithOptions 方法里首屏渲染相关方法执行完成。

首页的业务代码都是要在这个阶段，也就是首屏渲染前执行的，主要包括了：

- **首屏初始化所需配置文件的读写操作；**
- **首屏列表大数据的读取；**
- **首屏渲染的大量计算等。**

##### 首屏渲染完成后

这个阶段，主要完成的是，非首屏其他业务服务模块的初始化、监听的注册、配置文件的读取等。这个阶段就是从渲染完成时开始，到 **didFinishLaunchingWithOptions 方法作用域结束时结束**。

#### 功能级别的启动优化

功能级别的启动优化，就是要从 main() 函数执行后这个阶段下手。

优化的思路是： main() 函数开始执行后到首屏渲染完成前只处理首屏相关的业务，其他非首屏业务的初始化、监听注册、配置文件读取等都放到首屏渲染完成后去做。

#### 方法级别的启动优化

检查首屏渲染完成前主线程上有哪些耗时方法，将没必要的耗时方法滞后或者异步执行。通常情况下，耗时较长的方法主要发生在计算大量数据的情况下，具体的**表现就是加载、编辑、存储图片和文件等资源**。

#### 查看耗时

##### 查看Main()调用前花费的总时间

在Product->Scheme->Edit Scheme->Run->Arguments->Environment Variables->`DYLD_PRINT_STATISTICS`设置为YES，就可以在控制台中查看main函数执行前总共花费的多长时间。

##### 查看加载了多少动态库

在Product->Scheme->Edit Scheme->Run->Diagnostics->Logging->勾选Dynamic Library Loads，就可以在控制台中查看本项目中加载的所有动态库（包括系统的和自己的）

#### App 启动速度的监控手段

**第一种方法是：定时抓取主线程上的方法调用堆栈，计算一段时间里各个方法的耗时。**

一般将这个定时间隔设置为 0.01 秒

Xcode 工具套件里自带的 Time Profiler ，采用的就是这种方式。

**第二种方法是：对 objc_msgSend 方法进行 hook 来掌握所有方法的执行耗时。**

hook 方法的意思是，在原方法开始执行时换成执行其他你指定的方法，或者在原有方法执行前后执行你指定的方法，来达到掌握和改变指定方法的目的。

hook objc_msgSend 这种方式的优点是非常精确，而缺点是只能针对 Objective-C 的方法。

#### 做一个方法级别启动耗时检查工具

Objective-C 里每个对象都会指向一个类，每个类都会有一个方法列表，方法列表里的每个方法都是由 selector、函数指针和 metadata 组成的。

objc_msgSend 就是在运行时根据对象和方法的 selector 去找到对应的函数指针，然后执行。

objc_msgSend 方法执行的逻辑是：**先获取对象对应类的信息，再获取方法的缓存，根据方法的 selector 查找函数指针，经过异常错误处理后，最后跳到对应函数的实现。**

###### hook objc_msgSend

[Fishhook](https://github.com/facebook/fishhook):可以在 iOS 上运行的 Mach-O 二进制文件中动态地重新绑定符号 

objc_msgSend 是用汇编语言实现的，所以还需要从汇编层面多加点料。

需要先实现两个方法 pushCallRecord 和 popCallRecord，来分别记录 objc_msgSend 方法调用前后的时间，然后相减就能够得到方法的执行耗时。

https://github.com/ming1016/GCDFetchFeed

### 03 | Auto Layout

Auto Layout 用到的布局算法是 **Cassowary**

#### Auto Layout 的生命周期

Auto Layout 不只有布局算法 Cassowary，还包含了布局在运行时的生命周期等一整套布局引擎系统，用来统一管理布局的创建、更新和销毁。

这一整套布局引擎系统叫作 **Layout Engine** ，是 Auto Layout 的核心，主导着整个界面布局。

每个视图在得到自己的布局之前，Layout Engine 会将**视图、约束、优先级、固定大小**通过计算转换成最终的大小和位置。在 Layout Engine 里，每当约束发生变化，就会触发 Deffered Layout Pass，完成后进入监听约束变化的状态。当再次监听到约束变化，即进入下一轮循环中。Layout Engine 在碰到约束变化后会重新计算布局，获取到布局后调用 superview.setNeedLayout()，然后进入 Deferred Layout Pass。

Deferred Layout Pass 的主要作用是做容错处理。如果有些视图在更新约束时没有确定或缺失布局声明的话，会先在这里做容错处理。

接下来，Layout Engine 会从上到下调用 layoutSubviews() ，通过 Cassowary 算法计算各个子视图的位置，算出来后将子视图的 frame 从 Layout Engine 里拷贝出来。

### 04 | 架构怎么设计更合理

简单架构向大型项目架构演进中，就需要解决三个问题，即：**模块粒度应该如何划分？如何分层？多团队如何协作？**

