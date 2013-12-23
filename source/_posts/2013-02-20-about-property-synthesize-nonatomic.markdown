---
layout: post
title: "Objective-C 的一些关键字"
date: 2013-02-20 15:16
comments: true
categories: iOS
---

使用 @property 配合 @synthesize 可以让编译器自动实现 getter/setter 方法，使用的时候也很方便，可以直接使用 “对象.属性” 的方法调用；如果我们想要 “对象.方法” 的方式来调用一个方法并获取到方法的返回值，那就需要使用 @property 配合 @dynamic 了。  

使用 @dynamic 关键字是告诉编译器由我们自己来实现访问方法。如果使用的是 @synthesize，那么这个工作编译器就会帮你实现了。  

readonly 此标记说明属性是只读的，默认的标记是读写，如果你指定了只读，在 @implementation 中只需要一个读取器。或者如果你使用 @synthesize 关键字，也是有读取器方法被解析。而且如果你试图使用点操作符为属性赋值，你将得到一个编译错误。  

readwrite 此标记说明属性会被当成读写的，这也是默认属性。设置器和读取器都需要在 @implementation 中实现。如果使用 @synthesize 关键字，读取器和设置器都会被解析。

nonatomic：非原子性访问，对属性赋值的时候不加锁，多线程并发访问会提高性能。如果不加此属性，则默认是两个访问方法都为原子型事务访问。  

atomic 和 nonatomic 用来决定编译器生成的 getter 和 setter 是否为原子操作。  
<!--more-->
atomic :  
设置成员变量的 @property 属性时，默认为 atomic，提供多线程安全。
在多线程环境下，原子操作是必要的，否则有可能引起错误的结果。加了 atomic，setter 函数会变成下面这样：  

    {% codeblock lang:objc %}
    {lock}
      if (property != newValue) { 
           [property release]; 
           property = [newValue retain]; 
       }
    {unlock}
    {% endcodeblock %}
    
nonatomic :  
禁止多线程，变量保护，提高性能。

atomic 是 ObjC 使用的一种线程保护技术，基本上来讲，是防止在写未完成的时候被另外一个线程读取，造成数据错误。而这种机制是耗费系统资源的，所以在 iPhone 这种小型设备上，如果没有使用多线程间的通讯编程，那么 nonatomic 是一个非常好的选择。  

指出访问器不是原子操作，而默认地，访问器是原子操作。这也就是说，在多线程环境下，解析的访问器提供一个对属性的安全访问，从获取器得到的返回值或者通过设置器设置的值可以一次完成，即便是别的线程也正在对其进行访问。如果你不指定  nonatomic ，在自己管理内存的环境中，解析的访问器保留并自动释放返回的值，如果指定了 nonatomic ，那么访问器只是简单地返回这个值。

assign :  
简单赋值，不更改索引计数。
对基础数据类型 （例如 NSInteger，CGFloat）和 C 数据类型（int, float, double, char 等）适用简单数据类型

此标记说明设置器直接进行赋值，这也是默认值。在使用垃圾收集的应用程序中，如果你要一个属性使用assign，且这个类符合NSCopying 协议，你就要明确指出这个标记，而不是简单地使用默认值，否则的话，你将得到一个编译警告。这再次向编译器说明你确实需要赋值，即使它是可拷贝的。


copy :  
建立一个索引计数为1的对象，然后释放旧对象。
对 NSString 它指出，在赋值时使用传入值的一份拷贝。拷贝工作由 copy 方法执行，此属性只对那些实行了 NSCopying 协议的对象类型有效。更深入的讨论，请参
考“复制”部分。

retain :  
释放旧的对象，将旧对象的值赋予输入对象，再提高输入对象的索引计数为 1。  
对其他 NSObject 和其子类
对参数进行 release 旧值，再 retain 新值
指定 retain 会在赋值时唤醒传入值的 retain 消息。此属性只能用于 Objective-C 对象类型，而不能用于 Core Foundation 对象。(原因很明显，retain 会增加对象的引用计数，而基本数据类型或者 Core Foundation 对象都没有引用计数)。  
注意: 把对象添加到数组中时，引用计数将增加对象的引用次数 +1。

retain 的实际语法为 :

    {% codeblock lang:objc %}
    - (void)setName:(NSString *)newName
    { 
        if (name != newName) { 
            [name release]; 
            name = [newName retain]; 
        // name’s retain count has been bumped up by 1 
        } 
    }
    {% endcodeblock %}
    
copy 与 retain :  
copy 其实是建立了一个相同的对象，而 retain 不是 :
比如一个 NSString 对象，地址为 0×1111，内容为 @"STR"
copy 到另外一个 NSString 之后，地址为 0×2222，内容相同，新的对象 retain 为 1，旧有对象没有变化；
retain 到另外一个 NSString 之后，地址相同（建立一个指针，指针拷贝），内容当然相同，这个对象的 retain 值 +1；
也就是说，retain 是指针拷贝，copy 是内容拷贝。


retain 的 set 方法应该是浅复制，copy 的 set 方法应该是深复制了
copy 另一个用法：  
copy 是内容的拷贝，对于像 NSString，的确是这样.
但是, 如果是 copy 的是一个 NSArray 呢? 比如, 
NSArray *array = [NSArray arrayWithObjects:@"hello", @"world", @"baby"];
NSArray *array2 = [array copy]; 
这个时候，系统的确是为 array2 开辟了一块内存空间，但是我们要认识到的是，array2 中的每个元素，只是 copy 了指向array中相对应元素的指针。这便是所谓的"浅复制".

assign 与 retain：  
1. 接触过 C，那么假设你用 malloc 分配了一块内存，并且把它的地址赋值给了指针 a，后来你希望指针 b 也共享这块内存，于是你又把 a 赋值给（assign）了 b。此时 a 和 b 指向同一块内存，请问当 a 不再需要这块内存，能否直接释放它？答案是否定的，因为 a 并不知道 b 是否还在使用这块内存，如果 a 释放了，那么 b 在使用这块内存的时候会引起程序 crash 掉。
2. 了解到 1 中 assign 的问题，那么如何解决？最简单的一个方法就是使用引用计数（reference counting），还是上面的那个例子，我们给那块内存设一个引用计数，当内存被分配并且赋值给 a 时，引用计数是 1。当把 a 赋值给 b 时引用计数增加到 2。这时如果 a 不再使用这块内存，它只需要把引用计数减 1，表明自己不再拥有这块内存。b 不再使用这块内存时也把引用计数减 1。当引用计数变为0的时候，代表该内存不再被任何指针所引用，系统可以把它直接释放掉。

总结：上面两点其实就是 assign 和 retain 的区别，assign 就是直接赋值，从而可能引起 1 中的问题，当数据为 int, float 等原生类型时，可以使用assign。retain 就如 2 中所述，使用了引用计数，retain 引起引用计数加 1, release 引起引用计数减1，当引用计数为0时，dealloc 函数被调用，内存被回收。

    NSString *pt = [[NSString alloc] initWithString:@"abc"];
    
上面一段代码会执行以下两个动作
1. 在堆上分配一段内存用来存储 @"abc"  比如：内存地址为：0X1111 内容为 "abc"。
2. 在栈上分配一段内存用来存储 pt 比如：地址为：0Xaaaa 内容自然为 0X1111。  

下面分别看下assign retain copy assign 的情况：  

    NSString *newPt = [pt assing];  
    
此时 newPt 和 pt 完全相同 地址都是 0Xaaaa  内容为 0X1111  即 newPt 只是 pt 的别名，对任何一个操作就等于对另一个操作。 因此retainCount 不需要增加。  
retain 的情况：

    NSString *newPt = [pt retain];  
    
此时 newPt 的地址不再为 0Xaaaa，可能为 0Xaabb 但是内容依然为 0X1111。 因此 newPt 和 pt 都可以管理 "abc" 所在的内存。因此 retainCount 需要增加1。  
copy 的情况：

    NSString *newPt = [pt copy];

此时会在堆上重新开辟一段内存存放 @"abc" 比如 0X1122 内容为 @"abc" 同时会在栈上为 newPt 分配空间。比如地址：0Xaacc 内容为0X1122 因此 retainCount 增加 1 供 newPt 来管理 0X1122 这段内存



***
看了这么多也许大家有点晕， 现在进行实际的代码演示：

    @property (nonatomic, assign) int number;
    
这里定义了一个 int 类型的属性， 那么这个 int 是简单数据类型，本身可以认为就是原子访问，所以用 nonatomic,  不需要进行引用计数，所以用 assign。 适用于所有简单数据类型。  

    @property (nonatomic, copy) NSString *myString;
    
这里定义了一个 NSString 类型的属性，不需要原子操作，所以用 nonatomic.
为什么需要 copy，而不是 retain 呢！ 因为如果对 myString 赋值原字符串是一个可变的字符串(NSMutableString)对象的话，用retain的话，当原字符串改变的时候你的myString属性也会跟着变掉。我想你不希望看到这个现象。 实际上博主测试， 如果原来的字符串是 NSString 的话，也只是retain一下，并不会copy副本。  

    @property (nonatomic, retain) UIView *myView;

这里定义了一个 UIView 类型的属性，不需要原子操作，所以用 nonatomic。
当对 myView 赋值的时候原来的 UIView 对象 retainCount 会加 1。  

    // 接口文件
    @interface MyClass : NSObject
    @property (nonatomic, assign) int number;
    @property (nonatomic, copy) NSString  *myString;
    @property (nonatomic, retain) UIView *myView;
    @end  
    
    // 实现文件
    @implementation MyClass
    @synthesize number;
    @synthesize myString;
    @synthesize myView;
    // 释放内存
    - (void) dealloc
    {
        [myString release];   // copy 的属性需要 release;
        [myView release];     // retain 的属性需要 release;
        [super dealloc];      // 传回父对象
    }
    @end
假如你有一段代码创建了一个 MyClass 对象:  

    MyClass *instance  = [MyClass alloc] init];
    
    // number 赋值，没什么可说的，简单数据类型就这样
    instance.number = 1;
    // 创建一个可变字符串
    NSMutableString *string = [NSMutableString stringWithString:@"hello"];
    instance.myString = string;                   // 对 myString 赋值
    [string appendString:@" world!"];           // 往 string 追加文本 
    NSLog(@”%@”, string);                           // 此处 string 已经改变，输出为"hello world!"
    NSLog(@”%@”, instance.myString);         // 输出 myString，你会发现此处输出仍然为 "hello" 因为 myString 在 string 改变之前已经 copy 了一份副本
    UIView *view = [[UIView alloc] init];
    NSLog(@”retainCount = %d”, view.retainCount);
    // 输出 view 的引用计数，此时为 1
    instance.myView = view;      // 对 myView 属性赋值
    NSLog(@”retainCount = %d”, view.retainCount);
    // 再次输出 view 的引用计数，此时为 2，因为 myView 对 view 进行了一次 retain。 
    [view release];
    // 此处虽然 view 被 release 释放掉了，但 myView 对 view 进行了一次 retain，那么 myView 保留的 UIView 的对象指针仍然有效。
    [instance release] ;