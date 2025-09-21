# 10 Rules for Best Parctice in Coding that You May Break

## 1. The code shall read like English without comments.

## 2. Thou shalt take time to name things well.

## 3. Thy classes and functions shall **do one thing** at a time

## 4. Thou shalt use as less state as possible.
- Derived state may cause Cache Invalidation
  
## 5. Thou shalt never refactor before the 3rd repititon.

## 6. Thou shalt not use **Static** and **Global State**
- Bugs。使用静态变量或全局静态变量意味着代码库里的任何位置都可以访问修改它。这也就意味着你会忘记在哪里更新过它，从而导致Bugs。
- Poor Unit Testability。很难设置一个孤立的测试环境，因为静态变量的生命周期是全局，测试之间需要手动重置。
- Code Comprehension。因为全局特性，变量在何时更新，变量当前的状态是什么变得难以理解。
- Multithreading。Unity中多线程用的相对较少。当多个线程读写同一份静态变量的时候很容易出现死锁，dirty read, read after write等问题。
- 当你需要第二个的时候。

## 7. Thy classes shalt be **Highly Cohesive**
高内聚

## 8. Thy classes shalt be **Loosely Couple**
低耦合

## 9. Thou shalt use composition over inheritance

- 一个典型的继承可能存在的问题就是多继承。想象一个`Animal` super class, 它有一个` live() `方法，然后一个哺乳动物类，和一个卵生动物类，继承自 `Animal` 。哺乳动物添加了一个新的` MakeMilk() `方法，卵生动物添加了` LayEgg() `方法。然后实例化奶牛实例，属于哺乳动物，实例化蛇实例，属于卵生动物。到目前为止一切都很好，但是如果出现一个新的生物，它既可以产奶，又能下蛋，你想要它同时继承哺乳动物类和卵生动物。此时，` live() `方法变得困惑，不知道应该通过哺乳动物调用，还是通过卵生动物调用。如果卵生动物覆写了自己的`  live() `方法，会进一步增加困惑。继承树结构就变得不再是树。
- 组件化的思想就是，对于每一个需要的功能，我们设计对应的组件，然后对于每一个实例，我们添加实例需要的组件，从而实现完整的功能。还是上面这个例子，我们可以设计两个组件，分别是 MilkMakingComponent 和 EggLayingComponent, 组件内部分别有` MakeMilk() `方法和` LayEgg() `方法。当然奶牛，蛇等也需要有一个类用来实例化，但是它们不再通过继承获得方法，而是通过添加需要的组件。组件化提高了代码的灵活性和复用性。
- 当然，组件和继承不是两者只能选一的关系，实际开发中可以即使用继承又使用组件。不过推荐多使用组件，少使用继承。Unity 的 Prefab 和Prefab Variant System 可以提供继承的视角，同时功能通过添加不同的组件实现。

## 10. Thou shalt follow the **Rule of Demeter**
- 先看一个显然 break the law 的例子
```C#
enemy.GetCurrentTarget().GetHealthComponent().TakeDamage()
```
- 分析一下为什么这段代码不好。简单来说就是它隐式地产生了一个依赖链。包含这行代码的类声明了enemy变量，因此明确依赖于 Enemy 类。它可能有一个接口，其中包含方法 GetCurrentTarget()，该方法可能返回敌人的目标，该目标可能是玩家、NPC，甚至可能是 null。到这里一切正常。直到当它继续调用方法 GetHealthComponent()，然后调用方法 TakeDamage()的时候，事情变得糟糕。这一系列方法调用给类添加了隐式依赖。现在，这个类不仅依赖于 Enemy，还依赖于 Target 和 HealthComponent。如果调用链中的任何一个发生变化，我们就会遇到问题。
- 我们往往不想给一个类添加很多依赖项，而这是最简单的不经意间就添加依赖的写法。我们想要做的是让这个类承担更多责任，而不是链式调用方法。我们可以暴露更高级别给Enemy类，例如创建一个方法`DoDamageToCurrentTarget()`，直接对当前目标造成伤害。核心思想是，这个类应该尽有可能少地依赖项，且只包含需要的依赖项，并避免依赖项的依赖。