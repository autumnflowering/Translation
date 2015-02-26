<title>Core Data Progamming Guide</title>

原文最后修订日期：2014/7/15

# Introduction to Core Data Programming Guide #

## Organization of This Document ##

以下文章解释 Core Data Framework 解决的问题、提供的解决方案、其基本功能，以及用它执行的常见任务：

- Technology Overview 描述 Core Data 是什么，以及你为什么会选择它。
- Core Data Basics 描述 Core Data 的基本技术架构。
- Managed Object Models 描述 managed object model 的特性。
- Using a Managed Object Model 描述如何在程序中使用 managed object model.
- Managed Objects 描述 managed object (NSManagedObject 类) 的特性，以及如何、为何实现一个自定义的类来表示一个实体。
- Managed Object Accessor Methods 描述怎样为自定义的 managed object 编写 accessor 方法。- Creating and Deleting Managed Objects 描述怎样正确地以编程方式实例化及删除 managed object.
- Fetching Managed Objects 描述怎样获取 managed object, 以及怎样高效地获取。- Using Managed Objects 描述操纵 managed object 的问题。- Object Lifetime Management 描述内存管理的方面。- Relationships and Fetched Properties 描述关系、怎样为关系建模，以及如何在 managed objects 间操纵关系。该文章也描述了 fetched properties, which are like weak unidirectional relationships.- Non-Standard Persistent Attributes 描述怎样在非标准值类型上使用 attributes （如颜色和 C 语言结构体）。- Managed Object Validation 描述验证的类型、怎样实现验证方法，以及何时使用验证。- Faulting and Uniquing 描述 Core Data 如何限制 object graph 的规模，以及如何保证 managed object context 中的 managed object 的唯一性。
- Using Persistent Stores 描述怎样创建持久性 store、怎样把一种存储类型迁移到另一种类型，以及管理存储的元数据。- Core Data and Cocoa Bindings 描述 Core Data 怎样集成并利用 Cocoa bindings.- Change Management 描述创建多个 managed object contexts 或多个 persistence stacks 时会遇到的问题。
- Persistent Store Features 描述不同存储类型的特性，以及如何配置 SQLite 存储的行为。- Concurrency with Core Data 描述怎样在 Core Data 程序中使用并发编程。- Core Data Performance 描述怎样高效地使用 Core Data.- Troubleshooting Core Data 描述开发者常犯的错误以及如何纠正。
- Efficiently Importing Data 描述怎样把数据导入到 Core Data 程序。
- Core Data FAQ- Glossary## See Also ##You should also refer to:
- Core Data Starting Point 
- Core Data Tutorial for iOS 
- Core Data Utility Tutorial 
- Core Data Model Editor Help 
- Core Data Snippets

# Technology Overview #

本文描述 Core Data 提供的基本特性，以及适合采用此技术的原因。

## Core Data Features ##

Core Data framework 为与对象生命周期和 object graph 管理相关的任务提供了通用、自动化的解决方案。其特性包括：

- Change tracking and undo support. Core Data provides built-in management of undo and redo beyond basic text editing.- Relationship maintenance. Core Data manages change propagation, including maintaining the consistency of relationships among objects.- Futures (faulting). Core Data can reduce the memory overhead of your program by lazily loading objects. It also supports partially materialized futures, and copy-on-write data sharing.- Automatic validation of property values. Core Data’s managed objects extend the standard key-value coding validation methods that ensure that individual values lie within acceptable ranges so that combinations of values make sense.- Schema migration. Dealing with a change to your application’s schema can be difficult, in terms of both development effort and runtime resources. Core Data’s schema migration tools simplify the task of coping with schema changes, and in some cases allow you to perform extremely efficient in-place schema migration.- Optional integration with the application’s controller layer to support user interface synchronization. Core Data provides the NSFetchedResultsController object on iOS, and integrates with Cocoa Bindings on OS X.- Full, automatic, support for key-value coding and key-value observing. In addition to synthesizing key-value coding and key-value observing compliant accessor methods for attributes, Core Data synthesizes the appropriate collection accessors for to-many relationships.
- Grouping, filtering, and organizing data in memory and in the user interface.
- Automatic support for storing objects in external data repositories.
- Sophisticated query compilation. Instead of writing SQL, you can create complex queries by associating an NSPredicate object with a fetch request. NSPredicate provides support for basic functions, correlated subqueries, and other advanced SQL. With Core Data, it also supports proper Unicode, locale-aware searching, sorting, and regular expressions.- Merge policies. Core Data provides built in version tracking and optimistic locking to support automatic multi-writer conflict resolution.

## Why Should You Use Core Data ##

使用 Core Data 作为支撑的 model 层，代码行数可减少 50% - 70%. 这利益于 Core Data 提供的上述特性，你无须自己实现、测试、或优化这些特性。

Core Data 有一个成熟的代码库，其质量由单元测试保证，每天被众多程序所使用。经历数个版本后，该框架已高度优化，它利用了 model 和运行时特性所提供的信息，这些信息通常是 application-level 代码所不能提供的。除了优越的安全性和错误处理，相比其他解决方案，它还提供了最好的内存伸缩性。即使你上穷碧落下黄泉苦苦追寻，也未必能获得 Core Data 免费为你提供的良好性能。

除了 Core Data 自身提供的优势外，它还与 OS X 工具链良好集成。Model design 工具可使你以图形方式创建自己的 schema, 快速而简单。你可以使用 Instruments 程序提供的模板来衡量 Core Data 的性能，以及调试各种问题。在 OS X 桌面中，Core Data 还与 Interface Builder 集成，以在 model 中创建 UI.

## What Core Data Is Not ##

- Core Data 不是一个关系数据库或关系数据库管理系统 (RDBMS). Core Data 可使用 SQLite 作为一种持久 store, 但它不是一个数据库。为了强调这一点，你可以使用一个 in-memory store, 使用 Core Data 跟踪和管理变更，但即便这样，也完全可以不把任何数据保存到文件中。
- Core Data is not a silver bullet. 它不能使你免于编写代码。- Core Data does not rely on Cocoa bindings. Core Data 与 Cocoa bindings 良好地集成在一起并使用了相同的技术——二者合用可极大地减少代码量——但没有 bindings 也完全可以使用 Core Data. 你可以编写一个没有 UI 的 Core Data 程序（参阅 Core Data Utility 教程）。
# Core Data Basics #

## Basic Core Data Architecture ##

The **managed object context** (or **context** for short) serves as your gateway to an underlying collection of framework objects—collectively known as the **persistence stack**—that mediate between the objects in your application and external data stores. At the bottom of the stack are **persistent object stores**.

### Managed Objects and Contexts ###

可把 managed object context 想像成一个智能 scratch pad. 从持久 store 中撷取对象时，把这些对象的临时副本放在 scratch pad 中，这样它们就形成了一个 object graph（或 object graph 集合）。此后就可以随意修改这些对象了。实际保存这些变化之前，持久 store 不会被修改。

Core Data framework 中的 model 对象，谓之 **managed objects**. 所以有 managed objects 都必须注册到 managed object context, 通过这个 context 向 object graph 中增删对象、跟踪对对象及关系所作的更改。通过跟踪变更，context 可以支持 undo 和 redo, 还可保证修改对象的关系时，object graph 的完整性得以维护。保存变更时，context 保证对象处于有效状态。一个程序可以有多个 managed object context. Persistent store 中的一个对象，在给定一个 context 中至多可以有一个相应的 managed object（参阅 Faulting and Uniquing）。从另一个角度理解，persistent store 中的一个对象可以在多个 context 中被同时编辑，但每个 context 都有自己相应于该源对象的 managed object, 且被独立地编辑。这在保存时可能导致数据不一致，Core Data 提供了许多方法来处理这个问题。

### Fetch Requests ###

要使用 managed object context 撷取数据，需创建一个 fetch request. 后者是一个对象，指定了需要什么数据，如“所有雇员”或“市场部的所有雇员，按薪水从高到低排列”。

一个 fetch request 有 3 部分：

- 最少需指定实体的名字（by implication, 一次只能撷取一种实体）。
- 也可以包含一个 predicate 对象，指定所撷取对象应满足的条件。
- 也可以包含一组 sort descriptor 对象，指定所撷取对象的排序。

把 fetch request 发送给 managed object context, 后者从其持久 store 关联的数据源中返回匹配的结果（也可能没有匹配的结果）。由于所有的对象都必须注册到 managed object context, 故 fetch 返回的对象会被自动注册到 fetch 时所用的 context. 又由于持久 store 中的一个对象可对应到给定 context 中的至多一个 managed object, 故若 context 中已存在要 fetch 的对象，则为 fetch request 返回此对象。

The framework tries to be as efficient as possible. Core Data is *demand driven*, so you don't create more objects than you actually need. The graph does not have to represent all the objects in the persistent store. Simply specifying a persistent store does not bring any data objects into the managed object context. When you fetch a subset of the objects from the persistent store, you only get the objects you asked for. If you follow a relationship to an object that hasn't been fetched, it is fetched automatically for you. If you stop using an object, by default it will be deallocated. (This is of course not the same as removing it from the graph.)
### Persistent Store Coordinator ###

Persistence stack 中间有一个 persistent store coordinator.

实际上，一个 persistent store coordinator （也）定义了一个栈。Coordinator 被设计为向 managed object contexts 呈现一个 façade, 这样一组持久 store 就表现为一个聚合的 store 了。Managed object context 就可以基于 coordinator 覆盖的所有数据 store 而创建一个 object graph 了。一个 coordinator 仅可关联到一个 managed object model. If you want to put different entities into different stores, you must partition your model entities by defining configurations within the managed object models.

### Persistent Stores ###

一个持久 store 关联到一个文件（或其他外部数据源），并最终负责 store 中的数据与 managed object context 中相应对象的映射。正常情况下，你与持久 store 的唯一互动是指定一个新外部数据 store 的位置（如打开或关闭文档时），与 Core Data 的其他大多互动都是通过 managed object context 的。

程序代码——尤其是与 managed object 相关的逻辑代码——不要假设数据可能位于哪个持久 store 中。这样如果程序抽象得当，不必修改代码即可更改文件格式。

> Your application code—and in particular the application logic associated with managed objects—should not make any assumptions about the persistent store in which data may reside. Core Data provides native support for several file formats. You can choose which to use depending on the needs of your application. If at some stage you decide to choose a different file format, your application architecture remains unchanged. Moreover, if your application is suitably abstracted, then you will be able to take advantage of later enhancements to the framework without any additional effort. For example—even if the initial implementation is able to fetch records only from the local file system—if an application makes no assumptions about where it gets its data from, then if at some later stage support is added for a new type of remote persistent store, it should be able to use this new type with no code revisions.

重要：尽管 Core Data 支持 SQLite 作为一种持久 store 类型，但它并不是可以管理任意的 SQLite 数据库。要使用 SQLite 数据库，必须让 Core Data 自己创建并管理数据库。### Persistent Documents ###

可以编程方式创建并配置 persistence stack, 但多数情况下，你只是想创建一个基于文档的、可读写文件的程序。NSPersistentDocument 是 NSDocument 的一个子类，可用以简易地利用 Core Data framework. By default, an NSPersistentDocument instance creates its own ready-to-use persistence stack, 包括一个 managed object context 和一个持久 store. 这种情况下，一个文档和一个外部数据 store 之间存在一个一一映射。 There is in this case a one-to-one mapping between a document and an external data store.

NSPersistentDocument 类提供了访问文档的 managed object context 的方法，也实现了 NSDocument 的标准方法以使用 Core Data framework 读写文件。默认地，你无须编写额外代码即可处理对象持久化。A persistent document’s undo functionality is integrated with the managed object context.

## Managed Objects and the Managed Object Model ##

为了既管理 object graph 又支持对象持久化，Core Data 需要对它所操作的对象的丰富描述。A managed object model is a schema that provides a description of the managed objects, or entities, used by your application. 通常使用 Xcode's Data Model Design 工具以图形方式创建 managed object model, 也可以在运行时以编程方式构建 model.

The model is composed of a collection of entity description objects that each provide metadata about an entity, including the entity's name, the name of the class that represents it in your application (this does not have to be the same as its name), and its attributes and relationships. The attributes and relationships in turn are represented by attribute and relationship description objects.

Managed object 必须是 NSManagedObject 或其子类的实例，可表示一个实体。它使用一个私有的内部 store 来维护其属性及实现 managed object 所需的所有基本行为。Managed object 有一个对实体描述的引用，通过这个引用可获得关于它自己的元数据，包括它所表示的实体的名字、attribute 信息及关系。可子类化 NSManagedObject 以实现额外的行为。

# Managed Object Models #

Core Data 的许多功能依赖于描述实体及其属性和关系的那个 schema. Schema 由 managed object model 表示——NSManagedObjectModel 的实例。一般来说，此 model 越丰富，Core Data 就能越好地支持你的程序。

## Features of a Managed Object Model ##

NSManagedObjectModel 类的实例，描述了一个 schema ——程序中实体的集合。

### Entities ###

A model contains NSEntityDescription objects that represent the model's entities. Two important features of an entity are its name, and the name of the class used to represent the entity at runtime. 注意区分实体和用以表示实体的类，managed object 是实体的实例。An NSEntityDescription object may have NSAttributeDescription and NSRelationshipDescription objects that represent the properties of the entity in the schema. An entity may also have fetched properties, represented by instances of NSFetchedPropertyDescription, and the model may have fetch request templates, represented by instances of NSFetchRequest.In a model, entities may be arranged in an inheritance hierarchy, and entities may be specified as abstract.

#### Entity Inheritance ####

实体继承与类继承相似。可把一些相似实体的共同属性重构成一个 super-entity.

In many cases, you also implement a custom class to correspond to the entity from which classes representing the sub-entities also inherit. Rather than implementing business logic common to all the entities several times over, you implement them in one place and they are inherited by the subclasses.

若用 Xcode 的数据建模工具创建 model, 可选中实体后在 Data Model Inspector 中指定其 parent.若用代码创建 entity inheritance hierarchy, 则必须自上而下构建。不能直接设置某个实体的 super-entity, 只能设置某个实体的 sub-entities （使用 setSubentities: 方法）。故若要为某个实体设置 super-entity, 必须为 super-entities 指定一组 sub-entities 并将该实体包含在内。
#### Abstract Entities ####

可将实体指定为抽象的，即不创建其实例。多个实体都是某个通用实体的专门化、而该通用实体自己又不需要实例化时，通常会使用抽象实体。如绘图程序中将 Graphic 实体定义为抽象的，而其 sub-entities 可以是圆、线、矩形等。
### Properties ###

#### Attributes ####

#### Relationships ####
### Fetch Request Templates ###

### User Info Dictionaries ###

### Configurations ###


# Using a Managed Object Model #

# Managed Objects #

# Managed Object Accessor Methods #

# Creating and Deleting Managed Objects #

# Fetching Managed Objects #

# Using Managed Objects #

# Object Lifetime Management #

# Relationships and Fetched Properties #

# Non-Standard Persistent Attributes #

# Managed Object Validation #

# Faulting and Uniquing #

# Using Persistent Stores #

# Core Data and Cocoa Bindings #

# Change Management #

# Persistent Store Features #

# Concurrency with Core Data #

# Core Data Performance #

# Troubleshooting Core Data #

# Efficiently Importing Data #

# Core Data FAQ #
