---
layout: post
title: "iOS应用开发最佳实践系列一：编写高质量的Objective-C代码「转载」"
date: 2012-12-20 14:47
comments: true
categories: iOS
---

「文章出处: http://www.cnblogs.com/xdream86/p/3309345.html」  


###点标记语法

属性和幂等方法（多次调用和一次调用返回的结果相同）使用点标记语法访问，其他的情况使用方括号标记语法。  
良好的风格： 

    view.backgroundColor = [UIColor orangeColor];
    [UIApplication sharedApplication].delegate;
    
不良的风格：

    [view setBackgroundColor:[UIColor orangeColor]];
    UIApplication.sharedApplication.delegate;
    
<!--more-->

###间距
二元运算符和参数之间需要放置一个空格，一元运算符、强制类型转换和参数之间不放置空格。关键字之后圆括号之前需要放置一个空格。  

    void *ptr = &value + 10 * 3;    
    NewType a = (NewType)b;
    
    for (int i = 0; i < 10; i++) {
        doCoolThings();
    }
    
数组和字典类型的字面值的方括号两边各放置一个空格。  

    NSArray *theShit = @[ @1, @2, @3 ];
    
字典字面值的键和冒号之间没有空格，冒号和值之间有一个空格。  

    NSDictionary *keyedShit = @{ GHDidCreateStyleGuide: @YES };

C函数声明中，左括号的前面不保留空格，并且函数名应该像类一样带有命名空间标识。  
良好的风格：  

    void RNCWesomeFunction(BOOL hasSomeArgs);  
    
长的字面值应被拆分为多行。  
良好的风格：  
    
    NSArray *theShit = @[
        @"Got some long string objects in here.",
        [AndSomeModelObjects too],
        @"Moar strings."
    ];


    NSDictionary *keyedShit = @{
        @"this.key": @"corresponds to this value",
        @"otherKey": @"remoteData.payload",
        @"some": @"more",
        @"JSON": @"keys",
        @"and": @"stuff",
    };
    
每一行代码使用4个空格缩进。不使用tab缩进。下图是在Xcode的Preferences进行缩进设置的截图。  


方法签名以及其他关键字（if/else/switch/while等）后面跟随的左花括号总是和语句出现于同一行，而右花括号独占一行。  
良好的风格：  

    if (user.isHappy) {
    //Do something
    }

    else {
    //Do something else
    }
    
如果一个方法内有多个功能区域，可以使用空行分隔功能区域。  
每一行代码不要超过100个字符。  
每一个方法之前都有一个99字符宽的注释行，注释行相对于使用空行更能提高代码的辨识度，当一行代码很长的时候，注释行也起到了越界检测的作用。注释行：  
    //////////////////////////////////////////////////////////////////////////////////////////////////  
    
###条件语句  
所有的逻辑块必须使用花括号包围，即使条件体只需编写一行代码也必须使用花括号。  
良好的风格做法：
    

    
    
    
