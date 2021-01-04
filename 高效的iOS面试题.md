### Runtime

#### 介绍下runtime的内存模型（isa、对象、类、metaclass、结构体的存储信息等）

- 对象：OC中的对象指向一个objc_object指针类型，typedef struct objc_object *id; 从它的结构体中可以看出，它包括一个isa指针，指向的是该对象的类对象，一个对象实例就是通过isa指针找到它的Class的，而这个Class中存储的就是这个实例的方法列表，属性列表，成员变量列表等相关的信息。

  ``` c
  struct objc_object {
      Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
  };
  ```

- 类：在OC中的类是用Class来表示的，实际上它指向的是一个objc_class的指针类型，typedef struct objc_class *Class;

  ```c
  struct objc_class {
      Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
  
  #if !__OBJC2__
      Class _Nullable super_class                              OBJC2_UNAVAILABLE;
      const char * _Nonnull name                               OBJC2_UNAVAILABLE;
      long version                                             OBJC2_UNAVAILABLE;
      long info                                                OBJC2_UNAVAILABLE;
      long instance_size                                       OBJC2_UNAVAILABLE;
      struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
      struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
      struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
      struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
  #endif
  
  } OBJC2_UNAVAILABLE;
  ```

- OC的Class类型包括如下数据（即：元数据metadata）：super_class（父类类对象）；name（类对象的名称）；version、info（版本和相关信息）；instance_size（实例内存大小）；ivars（实例变量列表）；methodLists（方法列表）；cache（缓存）；protocols（实现的协议列表）；

- 当然也包括一个isa指针，这说明Class也是一个对象类型，所以我们称之为类对象，这个的isa指向的是元类对象（metaclass），元类中保存了创建类对象（Class）的类方法的全部信息。

- OC对象、类、元类之间的关系

![image-20201226161222552](/Users/chanli/Library/Application Support/typora-user-images/image-20201226161222552.png)

#### 为什么要设计metaclass

- 类对象、元类对象能够**复用消息发送流程机制**；`objc_msgSend(void /* id self, SEL op, ... */ )`
- 职责分离，**单一职责原则**，不用所有的类方法都加到同一个类里

#### class_copyIvarList & class_copyPropertyList区别

> property 正常使用会生成对应的实例变量，所以 Ivar 可以查到

- class_copyIvarList 获取类对象中的所有实例变量信息，从 `class_ro_t` 中获取
- class_copyPropertyList 获取类对象中的属性信息， `class_rw_t` 的 properties，先后输出了 category / extension / baseClass 的属性，而且仅输出当前类的属性信息，而不会向上去找 superClass 中定义的属性。
- 参考如下代码

```c
Ivar *
class_copyIvarList(Class cls, unsigned int *outCount)
{
    const ivar_list_t *ivars;
    Ivar *result = nil;
    unsigned int count = 0;

    if (!cls) {
        if (outCount) *outCount = 0;
        return nil;
    }

    mutex_locker_t lock(runtimeLock);

    ASSERT(cls->isRealized());
    
    if ((ivars = cls->data()->ro()->ivars)  &&  ivars->count) {
        result = (Ivar *)malloc((ivars->count+1) * sizeof(Ivar));
        
        for (auto& ivar : *ivars) {
            if (!ivar.offset) continue;  // anonymous bitfield
            result[count++] = &ivar;
        }
        result[count] = nil;
    }
    
    if (outCount) *outCount = count;
    return result;
}

objc_property_t *
class_copyPropertyList(Class cls, unsigned int *outCount)
{
    if (!cls) {
        if (outCount) *outCount = 0;
        return nil;
    }

    mutex_locker_t lock(runtimeLock);

    checkIsKnownClass(cls);
    ASSERT(cls->isRealized());
    
    auto rw = cls->data();

    property_t **result = nil;
    auto const properties = rw->properties();
    unsigned int count = properties.count();
    if (count > 0) {
        result = (property_t **)malloc((count + 1) * sizeof(property_t *));

        count = 0;
        for (auto& prop : properties) {
            result[count++] = &prop;
        }
        result[count] = nil;
    }

    if (outCount) *outCount = count;
    return (objc_property_t *)result;
}
```

#### class_rw_t 和 class_ro_t 的区别

- class_rw_t

  - class_ro_t
  - Protocols
  - MethodList
  - Properties
  
- class_rw_t 中的 properties 属性按顺序包含分类/扩展/基类中的属性。

  ```c
  struct class_ro_t {
      uint32_t flags;
      uint32_t instanceStart;
      uint32_t instanceSize;
  #ifdef __LP64__
      uint32_t reserved;
  #endif
  
      const uint8_t * ivarLayout;
      
      const char * name;
      method_list_t * baseMethodList;
      protocol_list_t * baseProtocols;
      const ivar_list_t * ivars;
  
      const uint8_t * weakIvarLayout;
      property_list_t *baseProperties;
  
      method_list_t *baseMethods() const {
          return baseMethodList;
      }
  };
  
  struct class_rw_t {
      // Be warned that Symbolication knows the layout of this structure.
      uint32_t flags;
      uint32_t version;
  
      const class_ro_t *ro;
  
      method_array_t methods;
      property_array_t properties;
      protocol_array_t protocols;
  
      Class firstSubclass;
      Class nextSiblingClass;
  
      char *demangledName;
  
  #if SUPPORT_INDEXED_ISA
      uint32_t index;
  #endif
  }
  ```

#### category如何被加载的,两个category的load方法的加载顺序，两个category的同名方法的加载顺序

- +load方法是 images 加载的时候调用，其主类和所有分类的 +load 都会被调用，**优先级是先调用主类**。且如果有继承链，加载顺序是先基类的 +load，接着是父类，最后是子类（当类或分类被添加到runtime时被调用的）

- category的+load则是按照编译顺序来的，先编译的先调用，后编译的后调用，可在Xcode的BuildePhase中查看。

  子类+load方法等父类先执行完+load方法才执行

  分类+load方法会在它的主类+load方法之后执行

#### initialize && load

- initialize： **类第一次被使用的时候会被调用**，底层实现有个逻辑先判断父类是否被初始化过，没有则先调用父类，然后调用当前类的 initialize 方法。
  - 一个类 A 存在多个 category，且 category中各自实现了 initialize 方法，这时候走的是**消息发送流程，也就是说 initialize 方法只会被调用一次，也就是最后编译的那个 category 中的 initialize 方法。**
- 如果 +load 方法中调用了其他类：比如B的某个方法，其实是走的消息发送流程，由于B没有被初始化过，则会调用其 initialize 方法，但此刻B的 +load 方法可能还没有被系统调用过。

**不管是 load 还是 initialize 方法都是 runtime 底层自动调用的，如果开发者自己手动进行了 [super load] 或者 [super initialize] 方法，实际上是走消息发送流程，这里也涉及了一个调用流程，需要引起注意。**

#### category & extension 区别，能给NSObject添加Extension吗，结果如何

- 不可以为系统类添加扩展

