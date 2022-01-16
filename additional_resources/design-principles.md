# 设计原则

## 常见设计原则的简要概述

---

## [SOLID](https://en.wikipedia.org/wiki/SOLID)

- [单一功能原则(Single Responsibility Principle, SRP)](https://en.wikipedia.org/wiki/Single-responsibility_principle)：
一个类应该只有一个功能，也就是说，只有对软件规范的一个部分的改变才能够影响到该类的规范。
- [开闭原则(Open/Closed Principle, OCP)](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)：
“软件实体......应该是对扩展开放的，对修改封闭的。”
- [里氏替换原则(Liskov Substitution Principle, LSP)](https://en.wikipedia.org/wiki/Liskov_substitution_principle)：
“程序中的对象应该可以用其子类型的实例来替换，而不改变该程序的正确性。”
- [接口隔离原则(Interface Segregation Principle, ISP)](https://en.wikipedia.org/wiki/Interface_segregation_principle)：
“许多客户端特定的接口比一个通用的接口要好。”
- [依赖反转原则(Dependency Inversion Principle, DIP)](https://en.wikipedia.org/wiki/Dependency_inversion_principle)：
“应该依靠抽象，而不是具体的实例。”

## [DRY (Don’t Repeat Yourself)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)

“每一项知识都必须在一个系统内有一个单一的、明确的、权威的表述。”

## [KISS原则](https://en.wikipedia.org/wiki/KISS_principle)

大多数系统在保持简单而不是复杂的情况下运行效果最好；因此，简单性应该是设计的一个关键目标，应该避免不必要的复杂性。

## [得墨忒耳定律(Law of Demeter, LoD)](https://en.wikipedia.org/wiki/Law_of_Demeter)

根据“信息隐藏”的原则，一个特定的对象应该尽可能少地假设其他事物（包括其子组件）的结构或属性。

## [契约式设计(Design by contract, DbC)](https://en.wikipedia.org/wiki/Design_by_contract)

软件设计者应该为软件组件定义正式的、精确的和可验证的接口规范，这些规范使用前置条件、后置条件和不变量扩展了抽象数据类型的普通定义。

## [封装](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming))

将数据与操作该数据的方法捆绑在一起，或限制对对象某些组件的直接访问。
封装是用来将结构化数据对象的值或状态隐藏在一个类中，防止未经授权的各方直接访问它们。

## [命令查询分离原则(Command-Query-Separation, CQS)](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation)

“函数不应产生抽象的副作用......只有命令（程序）才允许产生副作用。” - Bertrand Meyer: Object-Oriented Software Construction

## [最小惊讶原则(Principle of least astonishment, POLA)](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)

系统中的一个组件的行为应该是大多数用户所期望的行为。
这种行为不应该使用户感到惊讶或意外。

## Linguistic-Modular-Units

“模块必须与所使用的语言中的语法单位相对应。” - Bertrand Meyer: Object-Oriented Software Construction

## Self-Documentation

“模块的设计者应努力使所有关于模块的信息成为模块本身的一部分。” - Bertrand Meyer: Object-Oriented Software Construction

## Uniform-Access

“一个模块所提供的所有服务都应该通过一个统一的符号来提供，这并不违背它们的实现方式，即不管是通过存储还是通过计算来实现的。” - Bertrand Meyer: Object-Oriented Software Construction

## Single-Choice

“每当一个软件系统必须支持一组备选方案时，系统中有且仅有一个模块知道它们的详尽清单。” - Bertrand Meyer: Object-Oriented Software Construction

## Persistence-Closure

“每当一个存储机制存储一个对象时，它必须同时存储该对象的附属物。每当一个检索机制检索一个先前存储的对象时，它也必须检索该对象尚未被检索的任何附属物。” - Bertrand Meyer: Object-Oriented Software Construction

> Latest commit 9834f57 on 25 Aug 2021