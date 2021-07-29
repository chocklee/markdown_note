## UI相关

### UITableView相关

#### 一、重用机制

##### 1、iOS如何实现cell的重用机制？

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f978581901cd47e6acaf657150cb3848~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- A1-A7使用相同的identifier，当tableView向上滑动时，A1滑出页面后，就被放入了重用池。
- 当A7即将展示时，首先会在重用池中查看是否有相同的identifier的cell可以被重用，如果有则直接取出，若无则创建一个新的cell。

#### 二、数据源同步问题

- 当数据源在主线程中有删除操作，同时在子线程上又有加载更多数据的操作时，机会出现数据源同步问题。

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c70fb7316314be189f8fc93a89e080c~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

##### 1、数据源同步解决方案

###### a、并发访问、数据拷贝

- 子线程返回主线程的数据中，仍然包含删除的这一条数据。

  <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8de059013e442259cb2100ad0d2e3b8~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 在主线程进行删除操作时，将操作记录下来。之后再子线程同步数据时，同步删除操作。

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d02c0e9a2c541fdae68f92e956b82be~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

###### b、串行访问

- 将子线程的数据同步和主线程的删除操作全部放入一个串行队列中执行
- 删除动作可能会有延时

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82272c95868046e2923d0874538b93db~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

### 事件传递&视图响应

#### 一、UIView和CALayer

##### 1、UIView和CALayer的关系和区别？

###### a、关系

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbb3bbf2a63e41ef8033829d53af1aa7~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- UIView对象中的layer指向一个CALayer变量
- UIView对象中的backgroundColor属性，是CALayer同名属性的封装
- UIView展示部分是由CALayer中的contents来决定的。contents对应的backing store其实是一个bitmap的位图

###### b、区别

- UIView为其提供内容，以及负责处理触摸等事件，参与响应链
- CALayer负责显示内容contents

##### 2、为什么UIView负责触摸事件，CALayer负责显示？

- 设计模式，==单一职责==原则

#### 二、事件传递与视图响应链

##### 1、当点击View C2区域，系统是如何找到响应视图的？

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1d45052f68f460ab4a5b81d1e311e5d~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

######a、事件传递的流程

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf0ca20b2ade4d47942d1785629c271b~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 当用户点击屏幕，事件会被==UIApplication==接受，并传递给==UIWindow==
- ==UIWindow==调用==hitTest==函数，在==hitTest==内调用==pointInside==判断事件是否在该视图内
- 若为false，则返回该视图，事件传递流程结束
- 若为true，则可==倒序遍历==该视图的==子视图==，并调用==子视图==的==hitTest==函数
- 找到最终==hitTest==为==true==的==子视图==，并依次返回，事件传递流程结束

###### b、hitTest系统内部实现

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75ec3b79151347a68744ec95aa933d5e~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 在当前子视图调用==hitTest==函数前，需要将当前坐标转换为==子视图==中的坐标

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ae91e65f51b4ef1b473a295cac94a81~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

##### 2、如何只让方形图片的圆形区域接受事件响应？

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a9bf859e35b4d828d82b8ab88f515a1~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 重写视图的==pointInside==函数，使得点击区域在圆形范围内返回true，否则返回false

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ff427594ec54f569a716feffb781bcd~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

##### 3、视图响应流程

###### a、事件的响应是通过响应链来传递的

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dc62693c9d14c228c15c88047bf555d~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- UIView通过继承UIResponder，拥有以下方法

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3de99b6292ae4e6e84f789aff50e0169~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

###### b、事件传递之后由谁来响应？

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f750c21389f242fead9bfaf029d74a91~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 如果响应视图无法处理响应事件，则响应事件会通过==响应链==传递给==父视图==尝试处理，直到传递给==UIApplication==
- ==如果传递给UIApplication依然没有处理响应事件，则事件将被忽略==

### 图像显示原理

#### 一、图像显示流程

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6447508839224adb9c06755bafa64fb1~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- CPU和GPU是通过==事件总线==链接在一起的
- CPU输出的==位图==，在适当时机由事件总线上传给GPU
- GPU会对位图进行渲染，然后将结果放入==帧缓冲区==
- 视频控制器通过==Vsync信号==，在指定时间（16.7ms）之前，从==帧缓冲区==中提取屏幕显示内容，然后显示在显示器上

#### 二、UI视图显示过程

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e6cdf899c87481ebf875d02644eff2c~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 当创建一个UIView对象，它的显示部分由CALayer来控制
- CALayer有一个contents属性，就是最终绘制到屏幕上的==位图==
- 在绘制contents内容时，系统会回调==drawRect:==函数，我们可以在此函数中增加绘制内容
- 绘制好的位图通过==Core Animation==框架，最终经由GPU当中的==OpenGL==渲染管线，渲染在屏幕上

#### 三、CPU工作过程

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c982d22124d5471eb436fc5d7d3516dc~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

##### 1、Layout

- UI布局（frame设置）
- 文本计算（size计算）

##### 2、Display

- 绘制（==drawRect:==）

##### 3、Prepare

- 图片编解码

##### 4、Commit

- 提交位图

#### 四、GPU渲染管线过程

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/151886f2bc2843bcbcbc9c3c2a81f8b5~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

### 卡顿&掉帧的原因

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c6f837a47284ee19cafdebc94c8dc27~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 按照每秒==60FPS==的刷新率，每隔==16.7ms==就会有一次==Vsync==信号
- 在16.7ms内，需要CPU和GPU协同产生这一帧的画面，并在下一次==Vsync==信号来临时，显示这一帧画面
- 如果CPU和GPU的工作时长超过16.7ms，那么当Vsync信号来临时，无法提供这一帧的画面，就会出现掉帧的现象
- 上一帧没有显示的画面，会在下一针Vsync信号来临时显示

#### 一、滑动优化方案

##### 1、CPU

- 对象创建、调整、销毁可以放在子线程
- 预排版（布局计算、文本计算）操作，可以放在子线程操作
- 预渲染（文本等==异步绘制==、==图片编解码==等）操作，降低CPU的耗时

##### 2、GPU

- 避免==离屏渲染==，降低纹理渲染的耗时
- 如果==视图层级复杂==，GPU在视图合成时会做大量的计算。可以通过==异步绘制==等机制，减少视图层级，减轻GPU的压力

### 绘制原理&异步绘制

#### 一、UIView的绘制原理

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/124c9a9bbc70499b8eb22d7a30d55eb3~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 当调用==setNeedsDispaly==函数，实际是调用==view.layer==的==setNeedsDisplay==函数
- 该函数会将==layer==标记，在==runloop==即将结束时，调用==CALayer==的==display==函数，进入当前视图的真正绘制
- 在==CALayer==的==display==函数中，会判断它的代理是否响应==displayLayer:==函数，如果YES，则可进行==异步绘制==，否则进入==系统绘制流程==

#### 二、系统的绘制流程

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff3af82c73454ea585e1ca028f62e2df~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- ==layer==会创建一个==backing store==，在==drawRect:==函数中可以拿到这个上下文。
- ==layer==会判断是否有代理，如果有代理，在系统内部绘制完成后，会调用==UIView drawRect:==，允许在系统绘制基础上，进行增添修改。
- 最终由==CALayer==上传==backing store==到==GPU==。

#### 三、异步绘制

- 如果layer存在代理，则由代理执行==display:==函数生成位图，并设置该==bitmap==作为==layer.contents==属性

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1940943ab3464846bf4f7d18d9cd216e~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

### 离屏渲染

- ==在屏渲染（on-screen rendering）==意为当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示屏幕缓冲区中进行
- ==离屏渲染（off-screen rendering）==意为离屏渲染，指的是GPU在当前屏幕缓冲区以外==新开辟==一个缓冲区进行渲染操作

#### 一、什么场景会触发离屏渲染？

- 圆角（需要和maskToBounds一起使用）
- 图层蒙版
- 阴影
- 光栅栏

#### 二、为什么要避免离屏渲染

- 创建新的渲染==缓冲区==，会有内存上的开销
- 多通道渲染管线，最终需要合成，会涉及==上下文切换==，增加GPU的开销
- 总结：离屏渲染会增加GPU的处理时间，这样可能导致CPU+GPU的总处理时间超过16.7ms，从而出现掉帧卡顿的现象

### UI视图面试总结

- 系统的UI事件传递机制是怎样的？
  - 考察==hitTest==和==pointInside==内部实现

- 使UITableView滚动更流畅的方案或思路都有哪些？
  - CPU方面，在==子线程进行对象的创建、调整、销毁、预排版、图片异步绘制==

- 什么是离屏渲染？
  - 在当前屏幕缓存区外，新开辟一个缓冲区进行渲染

- UIView 和 CALayer之间的关系是怎样的？

  - UIView负责==事件传递==和==事件响应==

  - CALayer负责UI==视图显示==

  - 使用到设计模式六大设计原则中的==单一职责==原则

## Objective-C语言特性相关

### 分类

#### 一、使用分类做了哪些事？

- 声明私有方法
- 分解体积庞大的类文件
- 把Framework的私有方法公开

#### 二、分类的特点

- 运行时决议
- 可以为系统类增加分类

#### 三、分类可以添加哪些内容？

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbf2ae9087e84fa3bdb99be856fa243f~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 实例方法
- 类方法
- 协议
- 属性（只声明了setter和getter方法，并没有添加成员变量）

#### 四、分类源码解读

- 分类添加的方法可以“覆盖”原类方法

- 同名分类方法谁能生效取决于编译顺序

- 名字相同的分类会引起编译报错

  

- ==实例方法==存在于==类对象==中，==类方法==存在于==元类对象==中
- 对象在调用方法的时候，会有以下顺序：
  - 调用实例方法：实例通过==isa==找到==类对象==，然后查看是否有方法，有就调用
  - 调用类方法：类对象通过==isa==找到==元类对象==，然后查看是否有方法，有就调用
- 将==Category==中的方法copy到方法列表的==最前边==，==属性==和==协议==也是同样的方式
- 此时，如果再调用类的方法就会从方法列表中从前往后查询，而如果Category中有相同的方法，就会直接使用Category中的方法
- 在运行时，后加载的Category会合并在先加载的Category的前面

##### 面试题

- Category的实现原理
  - Category编译之后的底层结构是==struct category_t==，里面存储着分类的对象方法、类方法、属性、协议信息
  - 在运行时，runtime会将category的数据，合并到类信息中心（类对象、元类对象）
- Category和Class Extension的区别是什么？
  - Class Extension在编译的时候，它的数据就已经包含在类信息中
  - Category是在运行时，才会将数据合并到类信息中

### 关联对象

#### 一、能否给分类添加“成员变量”？

- 可以通过“关联对象”技术，为分类添加成员变量

##### 成员变量被添加到哪了？

- 关联对象并不是存储在被关联对象本身内存中
- 关联对象存储在全局的统一的一个AssociationsManager中

#### 二、关联对象源码解读

##### 关联对象的本质

关联对象由AssociationsManager管理并在AssociationsHashMap存储。==所以对象的关联内容都在同一个全局容器中==。

- Category中可以添加属性、协议、方法等, 但是并不能添加==成员变量==
- 如果要给Category添加成员变量，就需要使用runtime提供的==关联对象==

- 添加关联对象

  ```objective-c
  void objc_setAssociatedObject(id object, const void * key, id value, objc_AssociationPolicy policy)
  ```

- 获得关联对象

  ```objective-c
  id objc_getAssociatedObject(id object, const void * key)
  ```

- 移除所有的关联对象

  ```objective-c
  void objc_removeAssociatedObjects(id object)
  ```

- objc_AssociationPolicy

  | **objc_AssociationPolicy**        | 对应的修饰符      |
  | :-------------------------------- | :---------------- |
  | OBJC_ASSOCIATION_ASSIGN           | assign            |
  | OBJC_ASSOCIATION_RETAIN_NONATOMIC | strong, nonatomic |
  | OBJC_ASSOCIATION_COPY_NONATOMIC   | copy, nonatomic   |
  | OBJC_ASSOCIATION_RETAIN           | strong, atomic    |
  | OBJC_ASSOCIATION_COPY             | copy, atomic      |

##### 关联对象的原理

- 实现关联对象技术的核心对象有四个
  - AssociationsManager
  - AssociationsHashMap
  - ObjectAssociationMap
  - ObjcAssociation

<img src="https://user-gold-cdn.xitu.io/2019/1/21/1686f4f6435eeb00?imageslim" alt="img" style="zoom:75%;" />

<img src="https://user-gold-cdn.xitu.io/2019/1/21/1686f4ff4ca2cd0c?imageslim" alt="img" style="zoom:75%;" />

### 扩展

#### 一、一般扩展做什么？

- 声明私有属性
- 声明私有方法
- 声明私有成员变量

#### 二、分类和扩展的区别？

- 分类是==运行时决议==，扩展是==编译时决议==
- 分类可以有声明有实现。扩展只以声明的形式存在，多数情况下寄生于宿主类的.m文件中
- 可以为系统类添加分类，不能为系统类添加扩展

### 代理

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/538d213db6974b2c848364b825e4c064~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 准确的说是一种软件的==设计模式==
- iOS当中以==@protocol==形式体现
- 传递方式是一对一

#### 一、 如何规避循环引用？

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6288e6e5436e40ecbde0013858fa1037~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

### 通知

- 是使用==观察者模式==来实现的用于跨层传递消息的机制
- 传递方式是==一对多==

##### 通知和代理的区别？

- 通知是用观察者模式实现的，代理是代理模式实现的。通知的传递方式是一对多，代理的传递方式是一对一

#### 一、如何实现通知机制？

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62215dcc9f984656be0a1eb9836cc05f~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 全局字典==Notification_Map==，以==notificationName==为key，以数组==Observers_List==为value
- ==Observers_List==保存所有声明相同的==notificationName==对象
- 对象以Observer的形式保存，其中包括执行的函数，对象本身等

### KVO

##### 什么是KVO？

- Key-Value Observing，用于监听某个对象属性值的改变
- KVO是Objective-C对观察者设计模式的又一实现
- Apple使用了isa混写计数（isa-swizzling）来实现KVO

![image-20201226160704490](/Users/chanli/Library/Application Support/typora-user-images/image-20201223155430559.png)

- 使用setter方法改变值KVO才会生效
- 使用setValue:forKey:改变值KVO才会生效
- 成员变量直接修改需==手动添加==KVO的两个方法才会生效

#### 一、添加KVO的对象的isa指针指向何处

- 添加了KOV的对象，它的isa指针发生了改变，指向系统动态生成的子类==NSKVONotifying_Xxx==
- 对象调用方法的过程：
  - 首先通过isa指针，找到类对象
  - 在类对象中查找方法，如果方法存在就会调用

##### 1、使用KVO监听的Person对象

<img src="https://user-gold-cdn.xitu.io/2018/8/13/1653290088b83a96?imageslim" alt="img" style="zoom:75%;" />

##### 2、未使用KVO监听的Person对象

<img src="https://user-gold-cdn.xitu.io/2018/8/13/165328f211c6ef8e?imageslim" alt="img" style="zoom:75%;" />

#### 二、验证NSKVONotifying_Xxx中的setter的方法实现

- 添加KVO之后，调被监听属性的setter方法时调用了Foundation框架中的==_NSSet*ValueAndNotify==函数（*表示类型）

#### 三、探索NSKVONotifying_Xxx类对象的isa指针指向何处

- NSKVONotifying_Xxx类对象的isa指针指向NSKVONotifying_Xxx的元类对象

#### 四、_NSSet*ValueAndNotify的内部实现

```objective-c
[self willChangeValueForKey:@"age"];
... // 原来的setter实现
[self didChangeValueForKey:@"age"];
```

- 调用顺序
  - 调用==willChangeValueForKey:==
  - 调用原来的setter实现
  - 调用==didChangeValueForKey:==
    - didChangeValueForKey: 内部会调用 ==observeValueForKeyPath:ofObject:change:context:==

#### 五、面试题

##### 1、iOS用什么方式实现对一个对象的KVO？（KVO的本质是什么）

- 利用Runtime API动态生成一个子类，并且让实例对象的isa指针指向这个子类
- 当修改实例对象的属性时，会调用Foundation的_NSSet*ValueAndNotify函数
  - willChangeValueForKey:
  - 父类原来的setter方法
  - didChangeValueForKey:
    - 内部会触发监听器Observer的监听方法 ==observeValueForKeyPath:ofObject:change:context:==

##### 2、如果直接修改对象的成员变量，是否会触发监听器的（observeValueForKeyPath:ofObject:change:context:）方法？

- 直接修改对象的成员变量，而不调用setter方法，将不会触发观察者的==observeValueForKeyPath:ofObject:change:context:==方法

##### 3、如何受到触发KVO？

- 在修改成员变量的前后添加willChangeValueForKey:和didChangeValueForKey:方法，就可以手动触发KVO
- willChangeValueForKey:和didChangeValueForKey:这两个方法必须同时出现，如果只有一个，将不会触发KVO

### KVC

- Key-Value-Coding，键值编码，可以通过Key来访问某个属性

- 常见API有：

  ```objective-c
  - (void)setValue:(id)value forKeyPath:(NSString *)keyPath; // 设置实例中的自定义属性对象中的属性值
  - (void)setValue:(id)value forKey:(NSString *)key; // 设置实例中的属性值
  - (id)valueForKeyPath:(NSString *)keyPath; // 获取实例中的自定义属性对象中的属性值
  - (id)valueForKey:(NSString *)key; // 获取实例中的属性值
  ```

#### 一、KVC赋值时，方法和属性的顺序

- 使用KVC给一个对象赋值时，会有以下方法和属性的调用顺序
  - 查看 ==setKey:== 方法是否存在，如果存在直接调用，如果不存在进入下一步
  - 查看 ==_setKey:== 方法是否存在，如果存在直接调用，如果不存在进入下一步
  - 查看 ==+ (BOOL)accessInstanceVariablesDirectly== 方法的返回值，默认返回YES
    - YES：可以访问成员变量，进入下一步
    - NO：不可以访问成员变量，同时调用 ==- (void)setValue:(id)value forUndefinedKey:(NSString *)key== 方法，如果方法不存在会抛出异常
  - 调用成员变量：_key, _isKey, key, isKey
    - 调用顺序，从左到右，只有发现存在成员变量，就不会再调用后续变量
    - 如果没有成员变量，会调用 ==- (void)setValue:(id)value forUndefinedKey:(NSString *)key== 方法，如果方法不存在会抛出异常

<img src="https://user-gold-cdn.xitu.io/2018/8/16/1653e63385b66420?imageslim" alt="img" style="zoom:80%;" />

#### 二、KVC取值时，方法和成员变量的调用顺序

- KVC取值时，方法和成员变量的调用顺序如下：
  - 判断是否有这几个方法：==getKey，key，isKey，_key==
    - 从左到右，如果有方法，直接调用，取值结束
    - 如果没有进入下一步
  - 调用 ==+ (BOOL)accessInstanceVariablesDirectly== 查看是否可以访问成员变量，默认YES
    - YES：可以访问成员变量，进入下一步
    - NO：不可以访问成员变量，判断是否实现 ==- (id)valueForUndefinedKey:(NSString *)key== 方法，实现时调用，未实现会抛出异常
  - 判断是否有这几个成员变量：==_key，  _isKey，key，isKey==
    - 从左到右，如果有成员变量，直接访问，取值结束
    - 如果没有这几个成员变量，直接进入下一步
  - 判断是否实现 ==- (id)valueForUndefinedKey:(NSString *)key== 方法，实现时调用，未实现会抛出异常

<img src="https://user-gold-cdn.xitu.io/2018/8/16/1654345519a1f4a3?imageslim" alt="img" style="zoom:80%;" />

#### 三、KVC相关面试题

##### KVC赋值时，会触发KVO吗？

使用KVC给属性或成员变量赋值时，都会触发KVO。系统会自动调用 ==willChangeValueForKey:== 和 ==didChangeValueForKey:== 两个方法

### 属性关键字

- 读写权限
  - readonly
  - readwrite
- 原子性
  - atomic
  - nonatomic
- 引用计数
  - retain/strong
  - assign/unsafe_unretained
  - weak
  - copy

#### 一、atomic是否是线程安全的？

- atomic修饰的对象，系统会对它的set、get方法进行加锁
- 如果atomic修饰一个数组，那么对数组赋值set和获取get，是可以保证线程安全的
- 如果对数组进行添加元素和删除元素操作，则不在atomic的操作范围内，是线程不安全的

#### 二、assign和weak的区别？

- assign
  - 修饰基本数据类型，如int、bool等
  - 修饰对象类型时，不改变其引用计数
  - 会产生悬垂指针  
- weak
  - 不改变被修饰对象的引用计数
  - 所指对象在被释放之后会自动置为nil

#### 三、浅拷贝和深拷贝的区别？

##### 1、引用计数

- 在iOS中，使用引用计数来管理OC对象的内存
  - 一个新创建的OC对象引用计数默认是==1==，当引用计数减为==0==，OC对象就会销毁，释放其占用的内存空间
  - 调用==retain==会让OC对象的引用计数==+1==，调用==release==会让==OC==对象的引用计数==-1==
- 内存管理的经验总结
  - 当调用==alloc==、==new==、==copy==、==mutableCopy==方法返回了一个对象，在不需要这个对象时，要调用==release==或者==autorelease==来释放它
  - 想拥有某个对象，就让它的引用计数==+1==；不想再拥有某个对象，就让它的引用计数==-1==

##### 2、copy和mutableCopy

- 通过==copy==和==mutableCopy==, 可以生成一个副本, 与源代码分隔开, 两者之间互不干扰
- 深拷贝: 产生一个新的副本, 与源对象相互独立
- 浅拷贝: 指针拷贝, 指向源对象
- 不可变对象的copy是浅拷贝、mutableCopy是深拷贝，可变对象的copy和mutableCopy都是深拷贝

<img src="https://user-gold-cdn.xitu.io/2019/3/22/169a0ff5604cb8ec?imageslim" alt="img" style="zoom:50%;" />

###### 自定义对象的拷贝

- 自定义对象的拷贝需要实现==NSCopying==协议 （实现==copyWithZone:==方法）

##### 3、引用计数的存储

- 可以通过==retainCount==方法, 查看==引用计数==的存储

##### 4、weak的原理是什么？



### Objective-C语言面试总结

- MRC下如何重写retain修饰变量的setter方法？

  ```objective-c
  @property （nonatomic, retain）id obj;
  
  - (void)setObj:(id)obj {
    if (_obj != nil) {
      [_obj release];
      _obj = [obj retain];
    }
  }
  ```

- 请简述分类的实现原理? 

- KVO的实现原理？

- 能否为分类添加成员变量？

## Runtime相关

### Runtime数据结构

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eeb4d65bd98345f09f2e7b8116ec93f2~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

#### 一、objc_object

<img src="/Users/chanli/Library/Application Support/typora-user-images/image-20201226161120418.png" alt="image-20201226161120418" style="zoom:50%;" />

> Objective-C的对象、类主要是基于C\C++的什么数据结构实现的?
>
> 结构体: Objective-C中类的属性多样, 只有结构体能承载

- NSObject的底层实现如下:

  <img src="https://user-gold-cdn.xitu.io/2018/6/16/1640716c77136d88?imageslim" alt="img" style="zoom:70%;" />

- ==Class== 类型是一个指针，所以 ==NSObject_IMPL== 结构体中只包含一个指针变量
- 在 arm64 架构中，一个地址占8个字节，所以isa指针占用8个字节的内存空间

##### 一个NSObject对象占用多少内存？

- ==系统分配了16个字节给NSObject对象（通过malloc_size函数获得）==
- ==但NSObject对象只使用了8个字节的内存空间（64bit环境下，可以通过class_getInstanceSize函数获得）==

> ==class_getInstanceSize==函数获取到的, 是类型中所有成员变量一共占用内存的大小。
>
> 实际上==class_getInstanceSize==获取到的大小, 是==内存对齐==之后的大小, 即==所有成员变量中, 占用内存最大的那个成员变量的倍数== 
>
> ==Person==内的==NSObject_IMPL==有一个==isa==占用==8==个字节, ==_age==（int）占用==4==个字节, 所以==class_getInstanceSize==获取到的是==8==的倍数, 即==16==个字节

> OC中给实例对象分配空间时（由==malloc_size==函数获取）, 是按照16, 32, 48, 64, 80, 96...按照==16==的倍数递增的

##### LLDB命令

- ==memory read/数量格式字节数 内存地址==：读取内存，有缩略写法：==x==

```
格式: 
x: 16进制       f: 浮点数       d: 十进制

字节大小:
b: byte: 1字节          h: half word: 2字节
w: word: 4字节          g: giant word: 8字节
```

- ==memory write 内存地址 数值==：修改内存中的值

#### 二、objc_class、cache_t、class_data_bits_t、class_rw_t、class_ro_t、method_t

##### 1、Class的结构

==objc_class继承自objc_object==

![img](https://user-gold-cdn.xitu.io/2019/3/10/1696620b0ff82f7a?imageslim)

- ==class_data_bits_t== 主要是对 ==class_rw_t== 的封装
- ==class_data_bits_t bits==编译后指向==ro==，在动态加载的过程中创建了==rw==，此时的指向顺序是==bits->rw->ro==

##### 2、class_rw_t

- ==class_rw_t== 代表了类相关的读写信息、对 ==class_ro_t== 的封装

- ==class_rw_t== 里的==methods==、==properties==、==protocols==是二维数组，是可读可写的，包含了==类的初始内容、分类内容==

![img](https://user-gold-cdn.xitu.io/2019/3/10/1696620b8e888ca1?imageslim)

##### 3、class_ro_t

- ==class_ro_t== 代表了类的相关只读信息

- ==class_ro_t== 里面的 ==baseMethodList、baseProtocols、ivars、baseProperties== 是一维数组，是只读的，包含了==类的初始内容==

![img](https://user-gold-cdn.xitu.io/2019/3/10/1696620c0a442e40?imageslim)

##### 4、method_t

-  ==method_t== 是对==方法/函数==的封装，method_t结构如下：

![img](https://user-gold-cdn.xitu.io/2019/3/10/169662b359ea00cd?imageslim)

- ==IMP==代表函数的具体实现

![img](https://user-gold-cdn.xitu.io/2019/3/10/169662b3cf9b4855?imageslim)

- ==SEL==代表==方法/函数名==，一般叫选择器，底层结构跟 ==char ==* 类似
  - 可以通过==@selector()==和==sel_registerName()==获得
  - 可以通过==sel_getName()==和==NSStringFromSelector()==转成字符串
  - 不同类中相同名字的方法，所对应的的方法选择器是相同的

![img](https://user-gold-cdn.xitu.io/2019/3/10/169662b4897e4edc?imageslim)

- ==types==包含了函数返回值、参数编码的字符串

![img](https://user-gold-cdn.xitu.io/2019/3/10/169662b576f80a8b?imageslim)

##### 5、cache_t  方法缓存

- 用于快速查找方法执行函数
- 是可增量扩展的哈希表结构

- Class 内部结构中有个方法缓存（cache_t），用==散列表（哈希表）==来缓存曾经调用过得方法，可以提高方法的查找速度

![img](https://user-gold-cdn.xitu.io/2019/3/10/1696630d08922f06?imageslim)

- 调用方法时的查找过程

> 以对象方法为例：
>
> 通过isa指针知道到Class
>
> 先从cache中查找，如果有方法直接调用
>
> 如果没有找到方法，在自身的Clss->bits->rw中查找方法，如果找到方法直接调用，并将方法缓存到cache中
>
> 如果没有找到方法，会通过superclass找到父类，从父类->bits->rw中查找方法
>
> 如果在父类中找到方法，就直接调用，同时将方法缓存在自己（不是父类的）的cache中
>
> 如果一直找不到，就进入下一阶段

#### 三、isa指针、isa指向

##### 1、isa

- 分为==指针型isa==和==非指针型isa==
- 指针型isa的==值==代表Class的地址，isa的==值的部分==代表Class的地址

- OC中，每一个对象都有一个isa指针
- 实例对象的isa指向类对象，类对象的isa指向元类对象，元类对象的isa指向基类的元类对象，基类元类对象的isa指向基类元类对象本身

<img src="https://user-gold-cdn.xitu.io/2019/3/9/16961ad586a1565d?imageslim" alt="img" style="zoom:70%;" />

- 想要通过isa获取类对象和元类对象的地址，需要使用 ==isa & ISA_MASK==

<img src="https://user-gold-cdn.xitu.io/2019/3/9/16961ab15f3bc7b6?imageslim" alt="img" style="zoom:67%;" />

### 对象、类对象、元类对象

#### 一、OC对象的分类

- instance对象（实例对象）
- class对象（类对象）
- meta-class对象（元类对象）

##### 1、instance

- instance对象就是通过类==alloc==出来的对象，每次调用alloc都会产生新的instance对象
- instance对象在内存中存储的信息包括
  - isa指针
  - 其他==成员变量==

##### 2、class

- class类对象，可以通过==alloc==创建出instance对象

- 有三种方式获取一个类对象

  ```objective-c
  1. - (Class)class
  2. + (Class)class
  3. object_getClass(实例对象)
    
  NSObject *obj1 = [[NSObject alloc] init];
  // - (Class)class
  Class objectClass1 = [obj1 class];
  // + (Class)class
  Class objectClass2 = [NSObject class];
  // object_getClass(实例对象)
  Class objectClass3 = object_getClass(obj1);
  ```

- objectClass1~objectClass3都是NSObject的class对象

- 它们都是同一个对象，每个类在内存中有且只有一个class对象，指向同一块内存地址。

- class对象在内存中存储的信息主要包括

  - isa指针
  - superclass指针
  - 类的属性信息（@property）
  - 类的==实例方法==信息（instance method）
  - 类的==协议==信息（protocol）
  - 类的==成员变量==信息（ivar，描述成员变量的类型和名字, 而不是如同实例一般具体的值）
  - ...

  ![img](https://user-gold-cdn.xitu.io/2018/8/3/165006cee5b69c5c?imageslim)

##### 3、meta-class

- 每一个类在内存中有且只有一个==meta-class==对象
- 可以通过运行时的 ==object_getClass(类对象)== 方法获取类的元类型

```objective-c
Class objectMetaClass = object_getClass([NSObject class]);
```

- meta-class对象和class对象的内存结构是一样的，但用途不一样，主要包括：

  - isa指针
  - superclass指针
  - 类的 ==类方法== 信息（class method）

  ![img](https://user-gold-cdn.xitu.io/2018/8/3/16500740738f6b77?imageslim)

> 注意：元类对象和类对象拥有相同的结构

![img](https://user-gold-cdn.xitu.io/2018/8/3/16500784e1cfae91?imageslim)

> 注意：==- (Class)class== 和 ==+ (Class)class== 方法只能获取class对象，不能获取meta-class对象，meta-class对象只能通过Runtime的 ==object_getClass(类对象)== 获取

- 可以通过 Runtime 的 ==class_isMetaClass(对象)== 函数，来判断对象是否是==元类型==

- 下面是==objc_getClass==、==object_getClass==、==- (Class)class、+ (Class)class==的区别

```objc
 1.Class objc_getClass(const char *aClassName)
 1> 传入字符串类名
 2> 返回对应的类对象
 
 2.Class object_getClass(id obj)
 1> 传入的obj可能是instance对象、class对象、meta-class对象
 2> 返回值
 a) 如果是instance对象，返回class对象
 b) 如果是class对象，返回meta-class对象
 c) 如果是meta-class对象，返回NSObject（基类）的meta-class对象
 
 3.- (Class)class、+ (Class)class
 1> 返回的就是类对象
 
 - (Class) {
     return self->isa;
 }
 
 + (Class) {
     return self;
 }
```

#### 二、isa和superclass

##### 1、isa

- instance的isa指向class
  - 当调用对象方法时，通过instance的isa找到class，最后找到对象方法的实现进行调用
- class的isa指向meta-class
  - 当调用类方法时，通过class的isa找到meta-class，最后找到类方法的实现进行调用
- meta-class的isa指向基类的meta-class

##### 2、class对象的superclass指针

- 类对象的superclass指针，指向父类的类对象

> 注意：class的superclass会指向父类的class对象，最后指向的是NSObject的class对象，而NSObject的类对象中的superclass指针，指向nil
>
> 如果在发现NSObject的class对象中也没有找到要调用的方法时，就会报错 unrecognized selector sent to instance

##### 3、meta-class中的superclass指针

- 与类对象的superclass指针类似，meta-class中的superclass指针指向父类的meta-class

> 注意：基类NSObject的meta-class对象的superclass最终指向的是NSObject的class对象，而不是指向nil

### 消息传递

- 一共分为三个阶段：==消息发送、动态方法解析、消息转发==

#### 消息发送（objc_msgSend）

- OC中调用方法，会使用 ==objc_msgSend== 函数给==消息接受者==发送消息，所以OC调用方法的过程被称为==消息机制==
- ==消息发送==流程如下图

![img](https://user-gold-cdn.xitu.io/2019/3/12/16970cab5aa493ba?imageslim)

- 缓存查找
- 当前类中查找
  - 对于==已排序好==的列表，采用==二分法查找==算法查找方法对应执行函数
  - 对于==没有排序==的列表，采用==一般遍历查找==方法对应执行函数
- 父类逐级查找

#### 动态方法解析

- 当消息发送过程中，没有找到需要调用的方法时，会进入==动态方法解析==阶段
- ==动态方法解析==流程如下图：

![img](https://user-gold-cdn.xitu.io/2019/3/13/169757a89302fcdd?imageslim)

- 当消息接受者存在，且没有动态解析过时，才会进行动态解析
- 在动态解析过程中，会根据==类对象==和==元类对象==进行判断，分别处理
- 类对象调用 ==+ resolveInstanceMethod:== 方法，元类对象调用 ==+ resolveClassMethod:== 方法

#### 消息转发

##### 消息转发的实现过程

- 当动态方法解析也没有找到需要调用的方法实现时，就会进入到消息转发阶段，调用 ==_objc_msgForward_impcache== 函数
- 而在代码中，实际调用的是 ==forwardingTargetForSelector:== 方法
- 如果没有实现 ==forwardingTargetForSelector:== 方法，程序会继续调用 ==- methodSignatureForSelector:== 方法
- 在 ==- methodSignatureForSelector:== 方法中设置好==方法编码==后，就会调用 ==- forwardInvocation:== 方法
- ==NSInvocation==类中封装了方法的调用，通过打印可以看到==方法的调用者==和==方法名==

##### 消息转发的流程图

![img](https://user-gold-cdn.xitu.io/2019/3/13/1697654a50076d27?imageslim)

### Method-Swizzling

```objective-c
#import <Foundation/Foundation.h>

@interface RuntimeObject : NSObject

- (void)test;
- (void)otherTest;

@end

#import "RuntimeObject.h"
#import <objc/runtime.h>
@implementation RuntimeObject

+ (void)load {
    // 获取test方法
    Method test = class_getInstanceMethod(self, @selector(test));
    // 获取otherTest方法
    Method otherTest = class_getInstanceMethod(self, @selector(otherTest));
    // 交换两个方法的实现
    method_exchangeImplementations(test, otherTest);
}

- (void)test {
    NSLog(@"test");
}

- (void)otherTest {
    // 实际上是调用test的具体实现
    [self otherTest];
    NSLog(@"otherTest");
}
```

### 动态添加方法

- 使用 ==class_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types)== 函数动态为类添加方法

### @dynamic

- @dynamic
  - 动态运行时语言将函数决议推迟到运行时
  - 编译时语言在编译期进行函数决议
- @dynamic的作用：告诉编译器不要生成setter和getter方法，同时不要生成成员变量，等到运行时再添加方法实现

### Runtime实战

- 能否向编译后的类中增加实例变量？

  不能向编译后得到的类中增加实例变量；能向运行时创建的类中添加实例变量；

  1.因为编译后的类已经注册在 runtime 中,类结构体中的 objc_ivar_list  实例变量的链表和 instance_size 实例变量的内存大小已经确定，同时runtime会调用  class_setvarlayout 或 class_setWeaklvarLayout 来处理strong weak 引用.所以不能向存在的类中添加实例变量。
   2.运行时创建的类是可以添加实例变量，调用class_addIvar函数. 但是的在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前,原因同上.

- [obj foo]和objc_msgSend()函数之间有什么关系？

- runtime如何通过Selector找到对应的IMP地址的？

## 内存管理相关

### 内存管理方案

- 散列表
  - 弱引用表
  - 引用计数表

#### 1、散列表结构

​	<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76c605f650e8462cbdc0eb588d2cbdb4~tplv-k3u1fbpfcp-watermark.image?imageslim" alt="img" style="zoom:67%;" />

#### 2、SideTable结构

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18a8382d619642a2a221c4069d422757~tplv-k3u1fbpfcp-watermark.image?imageslim" alt="img" style="zoom:67%;" />

#### 3、为什么不是一个SideTable，而是SideTables？

- 因为修改数据时会加锁，如果所有对象的表放在同一个SideTable，那么加锁会太频繁，效率会受影响

  <img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4587b6371cba48d7a4e24478f5979130~tplv-k3u1fbpfcp-watermark.image?imageslim" alt="img" style="zoom:50%;" />

- 因为有多个SideTables，所以可以异步操作多张表，彼此之间影响会降低

  <img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d23825151e349b086ae21437bf2ba21~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

#### 4、怎样实现快速分流（怎样通过一个对象快速定位它所在的表）？

- SideTables的本质是一张Hash表
- Hash表查找的时间复杂度为O(1)
- 通过对象地址与Hash表的count取模，获取目标值下标索引

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e36bc2ce5f04f42bcd33da3f96d3266~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

#### 5、自旋锁 Spinlock_t

#### 6、引用计数表Size_t

- 引用计数表是通过Hash表来实现的
- Hash表查找的时间复杂度为O(1)

#### 7、弱引用表weak_table_t

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7c6d7dbe1354592b064d70c6abd515b~tplv-k3u1fbpfcp-watermark.image?imageslim" alt="img" style="zoom:50%;" />

### 弱引用管理

#### 1、弱引用变量如何被添加到弱引用表中？

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac290893f30c4e318533d5fca8e7ceb9~tplv-k3u1fbpfcp-watermark.image?imageslim" alt="img" style="zoom:50%;" />

#### 2、当一个对象被释放，weak变量是如何处理的？

- 根据对象，查找到弱引用表，提取弱引用数组
- 遍历其中所以弱引用指针，并置为nil

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd00da90a8104dd381ad3b232233e381~tplv-k3u1fbpfcp-watermark.image?imageslim" alt="img" style="zoom:50%;" />

### 自动释放池

#### 一、@autoreleasepool编译后的代码

![img](https://user-gold-cdn.xitu.io/2019/3/22/169a589c34d4980f?imageslim)

- 一个自动释放池会自动销毁，所以main函数中的代码可以模拟为下面的代码

![img](https://user-gold-cdn.xitu.io/2019/3/22/169a58ba4cebcab8?imageslim)

- autoreleasepool的实现原理是怎样的？
- autoreleasepool为何可以嵌套使用？

### 循环引用

- 自循环引用

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d2b4f0773d1474d9b6adc8a18f36441~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 相互循环引用

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f92f9df0cc8e49c09b123fae412c1fe8~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

- 多循环引用

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a357791f7a0b4890a4b70307c4996cda~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

### 内存管理面试总结

- 什么是ARC?
- 为什么weak指针指向的对象在废弃之后会被自动置为nil
- autoreleasepool是如何实现的？
- 什么是循环引用？你遇到过哪些循环引用？是如何解决的？

## Block相关

### Block介绍

什么是Block?

- Block是将==函数==及其==执行上下文==封装起来的==对象==

什么是Block调用？

- Block调用即使==函数调用==

### 截获变量

- 局部变量

  - 基本数据类型 - 对于基本数据类型的局部变量==捕获其值==

  ```objc
  - (void)method {
      int multiplier = 6;
      int(^Block)(int) = ^int(int num) {
          return num * multiplier;
      };
      multiplier = 4;
      NSLog(@"result is %d", Block(2)); // result is 12
  }
  ```

  - 对象类型 - 对于对象类型的局部变量连同==所有权修饰符一起捕获==

- 静态局部变量 - 以==指针形式捕获==局部静态变量

  ```objc
  - (void)method {
      static int multiplier = 6;
      int(^Block)(int) = ^int(int num) {
          return num * multiplier;
      };
      multiplier = 4;
      NSLog(@"result is %d", Block(2)); // result is 8
  }
  ```

- 全局变量 - 不捕获，可以在Block内部直接修改其值

- 静态全局变量 - 不捕获，可以在Block内部直接修改其值

### __block修饰符

- 一般情况下，对被截获变量进行==赋值==操作需添加==__block修饰符==

#### 对变量进行赋值时

##### 需要__block修饰符

- 局部变量
  - 基本数据类型
  - 对象类型

##### 不需要__block修饰符

- 静态局部变量
- 全局变量
- 静态全局变量

==__block修饰的变量变成了对象==

```objc
- (void)method {
    __block int multiplier = 6;
    int(^Block)(int) = ^int(int num) {
        return num * multiplier;
    };
    multiplier = 4;
    NSLog(@"result is %d", Block(2)); // result is 8
}
```

### Block的内存管理

> 总结:
>  block在栈上时, 并不会对__block变量产生强引用
>  当block被copy到堆时
>  1.会调用block内部的copy函数
>  2.copy函数内部会调用 _Block_object_assign 函数
>
>  3. _Block_object_assign函数会对__block变量形成强引用

#### Block的类型

![img](https://user-gold-cdn.xitu.io/2019/3/7/16958491986ab5b8?imageslim)

- _NSConcrete==Global==Block 全局block
  - 内部没有使用局部变量的block
- _NSConcrete==Stack==Block 栈block
  - 内部使用了局部变量的block
- _NSConcrete==Malloc==Block 堆block
  - 栈block调用copy后就是堆block，通过copy将block从栈区复制到了堆区
  - 全局block调用copy后类型不变，还在数据区
  - 堆block调用copy后类型不变（不会生成新的block，原有引用计数加1）

#### Block的Copy操作

- ==在ARC环境下, 编译器会根据情况自动将栈上的block复制到堆上==

![image-20201226160704490](/Users/chanli/Library/Application Support/typora-user-images/image-20201226160704490.png)

##### 栈上Block的Copy

<img src="/Users/chanli/Library/Application Support/typora-user-images/image-20201226161551792.png" alt="image-20201226161551792" style="zoom:67%;" />

##### 栈上__block变量的copy

- __forwariding指针存在的意义？

![image-20201226161808604](/Users/chanli/Library/Application Support/typora-user-images/image-20201226161808604.png)

- 当block在栈上时, 通过==__forwarding==指针拿到的是栈中的==__block==结构体

- 当block在堆上时，通过==__forwarding==指针拿到的是堆中的==__block==结构体

  <img src="https://user-gold-cdn.xitu.io/2019/3/8/1695cfc1b6c1e6a0?imageslim" alt="img" style="zoom:70%;" />

#### __forwariding总结

```objective-c
{
	__block int multiplier = 10;
	_blk = ^int(int num) {
		return num * multiplier;
	};
	multiplier = 6;
	[self executeBlock];
}

- (void)executeBlock {
	int result = _blk(4);
	NSLog(@"result is %d", result); // result is 24
}
```

不论在任何内存位置，都可以顺利的访问同一个__block变量。

### Block的循环引用

下面这段代码会产生循环引用吗？

```objc
{
	__block MCBlock *blockSelf = self;
	_blk = ^int(int num) {
		// var = 2
		return num * blockSelf.var;
	}
	_blk(3);
}
```

- ==在MCR下==，__block不会将修饰的对象类型的局部变量进行强引用，所以不会产生循环引用
- ==在ARC下==，__block会将修饰的对象类型的局部变量进行强引用，产生循环引用，引起内存泄漏

解决方案：

```objective-c
{
	__block MCBlock *blockSelf = self;
	_blk = ^int(int num) {
		// var = 2
		int result = num * blockSelf.var;
		blockSelf = nil;
		return result;
	}
	_blk(3);
}
```

> 总结:
>  不论在ARC还是MRC下,栈中的block不会对捕获到的==对象类型auto变量==进行强引用(引用计数+1), 只会在copy到堆中时, 会对==对象类型auto变量==进行强引用
>  ARC下, 被__weak修饰的==对象类型auto变量==, 在block复制到堆中时不会进行强引用

## 多线程相关

### GCD

#### 1、同步/异步 & 串行/并发

a、