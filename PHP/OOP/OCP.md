# 开闭原则OCP

> 来源：[source](https://zhuanlan.zhihu.com/p/24269134)

## 1、官方定义

开闭原则，英文缩写**OCP**，全称`Open Closed Principle`。

原始定义：Software entities (classes, modules, functions) should be open for extension but closed for modification。

字面翻译：软件实体（包括类、模块、功能等）应该对扩展开放，但是对修改关闭。

## 2、自己理解

### **2.1、原理解释**

- 对扩展开放。模块对扩展开放，就意味着需求变化时，可以对模块扩展，使其具有满足那些改变的新行为。换句话说，模块通过扩展的方式去应对需求的变化。


- 对修改关闭。模块对修改关闭，表示当需求变化时，关闭对模块源代码的修改，当然这里的“关闭”应该是尽可能不修改的意思，也就是说，*应该尽量在不修改源代码的基础上面扩展组件*。

### **2.2、为什么要“开”和“闭”**

一般情况，我们接到需求变更的通知，通常方式可能就是修改模块的源代码，然而修改已经存在的源代码是存在很大风险的，尤其是项目上线运行一段时间后，开发人员发生变化，这种风险可能就更大。所以，为了避免这种风险，在面对需求变更时，我们一般不修改源代码，即所谓的对修改关闭。不允许修改源代码，我们如何应对需求变更呢？答案就是我们下面要说的对扩展开放。

通过扩展去应对需求变化，就要求我们必须要面向接口编程，或者说面向抽象编程。所有参数类型、引用传递的对象必须使用抽象（接口或者抽象类）的方式定义，不能使用实现类的方式定义；通过抽象去界定扩展，比如我们定义了一个接口A的参数，那么我们的扩展只能是接口A的实现类。**总的来说，开闭原则提高系统的可维护性和代码的重用性。**