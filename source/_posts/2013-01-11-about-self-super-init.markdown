---
layout: post
title: "about self = [super init]"
date: 2013-01-11 15:06
comments: true
categories: iOS
---

先看一下官方文档给出的初始化示例代码：

    - (id)init {  
       self = [super init];  // Call a designated initializer here.  
       if (self != nil) {  
             // 省略其他细节  
       }  
       return self;  
    }  


容易让人困惑的地方在于，将父类初始化之后，将其返回的对象指针覆盖当前对象的指针。
这种方式令人费解，目前暂时找不到官方解释这么做的原因。  
<!--more-->

官方文档 有解释。
我们先分以下几种情况分别分析：（假设superSelf是[super init]的返回值）

    superSelf == nil
    
此时父类初始化失败，self随之被赋值为nil并返回，表现正常。

    superSelf == self
    
大部分类的初始化都是这个结果。此时赋值没有任何影响。 

    superSelf != self
    
这种情况正是大部分人疑惑的地方。执行 self = [super init] 之后，我们创建的对象将会被重定向到另外一块内存上。接下来重点解释这种情况。

首先，出现父类指针跟子类指针不一样的情况其实就是父类的init方法返回的对象跟原先创建的对象不一样，分为以下几种：  
1 单件。  
此时如果执行 self = [super init] 将使所有子类指向这个单件的内存。
这不仅使所有子类互相修改数据，甚至访问子类自己增加的变量的时候，可能会崩溃。
建议不要继承单件、或者保证单件的子类也是单件。  
2 ClassClusters（类簇），初始化方法返回了不同的子类。  
以下是对象初始化后，返回不同的子类的例子：

    NSString *str1 = [NSString alloc];  
    NSString *str2 = [str1 initWithString:@"hello"];  
    
上面的str1和str2是不一样的对象，存在于不同的内存块中，在GNUStep里面，str1是GSPlaceholderString对象，str2是GSCInlineString对象。
3 共享。  
先看例子：  
     
    NSNumber *n1 = [[NSNumber alloc] initWithInt:1];  
    NSNumber *n2 = [[NSNumber alloc] initWithInt:1];  

以上的n1和n2，指向了同一块内存！
由于NSNumber是创建之后就不能修改的对象，所以Foundation在这里做了一些优化，相同数值的NSNumber对象将共享同一块内存。
4 父类可能在初始化中释放了当前的对象并创建了新的内存区域。  
这时，子类需要将self指向新的内存区域才能正常工作。所以一定要执行self = [super init];


###总结：
在初始化方法中使用self = [super init]语句是Objective-C的标准做法。
一般情况下都要用以上语句来防止父类改变对象的内存地址导致self指针指向无效内存。
在父类是单件、类簇或者有共享资源的时候，必须依照实际情况考虑是否加上这行代码。
总之，当需要继承父类的时候，调用父类的init之前，必须知道父类的init方法的工作方式。