---
title: 软件开发 SOLID 原则
date: 2026-11-02 00:48:35
tags: [SOLID, 软件工程]
categories: [技术]
keywords: [SOLID, 软件工程, 设计原则]
description: 在 AI 时代重新认识软件工程中的 SOLID 五大设计原则（SRP、OCP、LSP、DIP、ISP）以及 DIP、IoC 与 DI 的区别。
---

> 在AI日渐强大的2026年，我发现有必要再认真学习一下SOLID这五条原则，因为人基本上不怎么写代码了，所以学会review代码就更重要了，在软件工程中SOLID 这一套经过时间沉淀的规则就有其存在的必要了。


## 认识一下SOLID

### 单一职责原则

SRP 单一职责原则 (Single Responsibility Principle) 指的是一个类或者模块应该只负责一个功能，而不是多个功能。

### 开放封闭原则

OCP 开放封闭原则 (Open Closed Principle) 指的是一个软件实体（类、模块、函数等）应该对扩展开放，对修改关闭。

### 里氏替换原则

LSP 里氏替换原则 (Liskov Substitution Principle) 指的是子类应该能够替换父类，而不影响程序的正确性。

> 这个很好理解，基本上只要是在面向对象语言中，子类都应该能够替换父类。

### 依赖倒置原则

DIP 依赖倒置原则 (Dependency Inversion Principle) 指的是高层模块不应该依赖于低层模块，两者都应该依赖于抽象。

> 例如，查询用户接口本身需要依赖于底层的数据库查询接口，而不是直接依赖于数据库

> 此外依赖注入不是依赖倒置, 依赖倒置是一种设计原则, 而依赖注入是一种实现方式。
> 还有IoC, 控制反转, 和依赖倒置也不是一个东西, IoC 体现的是谁负责依赖的注入

他们的关系如下: 
> DIP 说的是源码依赖指向哪里（静态结构），IoC 说的是谁调用谁、谁掌握控制权（运行时流程）。二者正交，DI 是把它们连起来的施工手段。
> DIP 只有两条（Martin, 1996）：高层不依赖低层，两者都依赖抽象；抽象不依赖细节，细节依赖抽象。
> IoC 是更泛的词：回调、事件循环、模板方法、框架生命周期、插件、DI 全算。共同点是「别打给我们，我们会打给你」——main 不再驱动一切，框架反过来调你。

为了区分上述概念

1. 违反 DIP：字段类型就是具体类

```text
+------------------------+
|  OrderService          |   high-level policy
+------------------------+
|  - repo : MySqlUserRepo|
+------------------------+
|  + placeOrder() : void |
+------------------------+
             |
             | uses
             v
+------------------------+
|  MySqlUserRepo         |   low-level detail
+------------------------+
|  + find(id) : User     |
|  + save(u : User): void|
+------------------------+
             |
             v
        MySQL driver
```

2. 符合 DIP：接口归高层，实现反向依赖

```
  +------------------------+
  |  OrderService          |   domain (high-level)
  +------------------------+
  |  - repo : UserRepo     |
  +------------------------+
  |  + placeOrder() : void |
  +------------------------+
               |
               | uses
               v
  +------------------------+
  |    <<interface>>       |
  |       UserRepo         |   domain (interface owned by high level)
  +------------------------+
  |  + find(id) : User     |
  |  + save(u : User): void|
  +------------------------+
               ^
               :
               : implements
               :
  +------------------------+
  |  MySqlUserRepo         |   infra (low-level)
  +------------------------+
```

3. 倒置的是依赖方向，不是调用方向

```
     运行时调用流                  编译期依赖流
     ------------                  ------------
     OrderService                  OrderService
          |                             |
          | calls                       | uses
          v                             v
     MySqlUserRepo                    UserRepo
                                        ^
                                        |
                                        | implements
                                        |
                                   MySqlUserRepo
```
同一次协作里两条箭头方向相反，这才是 inversion 的含义。

4. 加容器：只填空隙，不改结构
```
  +---------------------------------------+
  |  Container / main()                   |   composition root
  |  bind(UserRepo => MySqlUserRepo)      |
  +---------------------------------------+
           |                     |
    inject |                     | new
           v                     v
  +----------------+   +------------------+
  |  OrderService  |   |  MySqlUserRepo   |
  +----------------+   +------------------+
           |                     |
           | uses                | implements
           +----------+----------+
                      |
                      v
         +-------------------------+
         |      <<interface>>      |
         |        UserRepo         |
         +-------------------------+
```

### 接口隔离原则

ISP 接口隔离原则 (Interface Segregation Principle) 指的是客户端不应该依赖于它不需要的接口。

> 处理这种文件的最简单方法就是拆分大接口成为一个个的小接口
