---
title: Android MVP架构 Google Samples
tags: [MVP,架构,Android]
grammar_cjkRuby: true
categories: [Android]
date: 2017-01-14
---

[原始出处](https://github.com/googlesamples/android-architecture)


### MVP
![][1]

### MVP with Loade
![enter description here][2]

### MVVM with DataBinding
![enter description here][3]

### Clear Architecture
![enter description here][4]


It's based on the MVP, adding a domain layer between the presentation layer and repositories, splitting the app in three layers:

- **MVP:** Model View Presenter pattern from the base sample.
- **Domain:** Holds all business logic. The domain layer starts with classes named use cases or interactors used by the application presenters. These use cases represent all the possible actions a developer can perform from the presentation layer.
- **Repository:** Repository pattern from the base sample.


**Belefit**

The big difference with base MVP  is the use of the Domain layer and use cases. Moving the domain layer from the presenters will help to avoid code repetition on presenters (e.g. Task filters).

Use cases define the operations that the app needs. This increases readability since the names of the classes make the purpose obvious (see tasks/domain/usecase/).

Use cases are good for operation reuse over our domain code.

### MVP with Dagger2

![enter description here][5]
Components从根本上来说就是一个注入器，也可以说是@Inject和@Module的桥梁。 Components可以提供所有定义了的类型的实例，比如：我们必须用@Component注解一个接口然后列出所有的@Modules组成该组件，如 果缺失了任何一块都会在编译的时候报错。@Component接口定义了对象提供者（module）和对象之间的联系，也表述了一种依赖关系。

[Google官方MVP+Dagger2架构详解](http://www.jianshu.com/p/01d3c014b0b1)

### MVP with Loader & ContentProvider

![enter description here][6]


### MVP with RxJava

### Deal With Tablet



Tablet mode

![Diagram for tablet][7]

There are two new important classes:

TasksTabletPresenter acts as the presenter for every view. It communicates with the list, detail and edit views through their respective presenters.

TasksMvpTabletController is in charge of the creation of the MVP views and presenters for tablet. It also handles navigation.

The entry point for the list-detail view is TasksActivity. On a tablet, TaskDetailActivity and AddEditActivity are never used.

Phone mode

![Diagram for phone][8]

On a phone, TasksTabletPresenter and TasksMvpTabletController are not used but each feature uses the same view and presenter as in tablet mode.


### MVP with Conductor

Based on the MVP  and uses the Conductor framework to **refactor to a single Activity architecture**.

Activity and Fragment are replaced with Controller classes, which have a simpler lifecycle. 

![Diagram][9]


  [1]: ./images/1484099154702.jpg "MVP"
  [2]: ./images/1484099351203.jpg "MVP with Loader"
  [3]: ./images/1484105170638.jpg "MVVM with DataBinding"
  [4]: ./images/1484288974172.jpg "Clear Architecture"
  [5]: ./images/1484294470126.jpg "Dagger2流程"
  [6]: ./images/1484296368111.jpg "MVP with Loader & ContentProvider"
  [7]: ./images/1484298681906.jpg "Tablet mode"
  [8]: ./images/1484298695203.jpg "Phone mode"
  [9]: ./images/1484299244005.jpg "MVP with Conductor"