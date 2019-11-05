[论高可靠性系统中软件容错技术的应用](https://blog.csdn.net/feitianxuxue/article/details/78918768)

[.net saga](https://www.cnblogs.com/hankuikui/p/9953214.html)
# DDD驱动设计笔记

## 一. 概述

- 领域（Domain）其实就是一个组织所要做的整个事情，已经这个事情下所包含的一切内容。这是一个范围概念，而且是面向业务的（注意这里不是面向技术的，更不是面向数据库的持久化的），每个组织都有自己的人员、自己的工作业务范围和做事方式，当你为该组织开发软件的时候，你面对的就是这个组织的领域。
- 如何定义一个领域 这个是更简单的一个问题，在领域设计中，有两个方法：战略设计和战术设计，其实我个人感觉可以定义两步走，这两个是有先后之分的

   1. 战略设计中定义了，一个领域就是一个问题空间，我们在业务中所遇到的所有的问题与挑战；
   2. 在战术设计中，一个领域就算一个解决问题空间，用来解决在问题空间的所有问题；

   所以，其实一个领域就是一个我们建立的一个解决方案，一个项目，在我们的问答项目中，整个解决方案就是一个问答领域。

- 子域 / 核心子领域 / 通用子领域

    什么是子域（SubDomain）呢？这个很好理解，就是在整个领域中，我们如何对其进行拆分，然后满足我们的业务逻辑。一个子域可能是一个 dll ，一个命名空间的形式存在。

- 核心子领域 / 通用子领域 / 支撑子领域

   1. 我们再来分析一下，我们的问答领域中的有哪些内容，首先：肯定有消息发布子领域，这个也是上边说到的，这个毋庸置疑，一个问答系统，消息发布是肯定的（这里说明下：发布问题，回答问题，讨论问题等都属于一个消息的发布，这个应该理解），而且这个子领域是缺少它不可的，这个就是我们的核心子领域。

   2. 比如日志记录，数据操作痕迹记录（哪个管理员修改了哪些数据），这些子领域贯穿着我们整个领域系统，被其他领域共用，我们称之为 通用子领域

   3. 站内的即时消息，wiki百科，通知提醒，活动跟踪，等等，这些都不是我们的核心子域，因为没有这些，我们依然可以进行问答，但是这些确是支撑着我们核心子域的相关功能，我们就把这些命名为 支撑子领

- 隔离内核

   以前我们写逻辑，就算直接在控制器里，判断当前用户权限，但是现在我们是通过一个中间件，判断 Token 所包含的用户Role 是否有这个权限，再进行下一步，只不过在DDD中，把这一块单拿出来形成了一个安全子领域了。

## 二. 限界上下文 —— 领域模型的边界

- 限界上下文是显式的，有语义的

   1. 限界上下文（Bounded Context）定义了每个模型的应用范围，在每个Bounded Context中确保领域模型的一致性。
   2. 不同的限界上下文中，领域模型可以不用保证一致性。通常我们根据团队的组织、软件系统的每个部分的用法及物理表现（如组件划分，数据库模式）来设置模型的边界。

*在电商系统中，销售子域是核心域，商品子域和物流子域为支撑子域。在这三个子域中，都要和商品打交道。如果把商品抽象为Product对象的话，按我们一般的常规思路（抛开子域的划分）来说，不管是商品销售还是发货，我们都可以共用同一个Product对象。*
 *但在DDD中，在商品子域和销售子域中，可以共享这个Product对象，但在物流子域，就有点大材小用。为什么呢？因为毕竟物流子域关注的是商品的发货处理和物流跟踪。针对发货流程而言，我只关心商品的数量、大小、重量等规格，而不必了解商品的价格等其他信息。所以说物流子域应该关注的是货物的发货处理而不是商品。*
 *那为什么我们之前的开发思路会共用同一个Product对象呢？*

**答案很简单，没有进行领域的划分。把整个项目一概而论，统一建模导致的结果.**

 *在DDD的思想下，当划分子域之后，每个子域都对应有各自的上下文。在销售子域和商品子域所在的上下文语境中，商品就是商品，无二义性。在物流子域的上下文语境中，我们也可以说商品的发货处理，但这时的商品就特指货物了。确定了真实面目之后，我想我们也会不由自主的抽象一个新的Cargo对象来处理物流相关的业务。这也是DDD带来的好处，让我们更清晰的建模。*

- 定义限界上下文

    在我们上边的子域定义中，我们出现了三个子域，我这里同时也定义了三个限界上下文（这里说下，两者不是一对一的关系），总体来说，我们不应该按技术架构或者开发任务来创建限界上下文，***应该按照语义的边界来考虑。*** 我们的实践是，考虑产品所讲的通用语言，从中提取一些术语称之为概念对象，寻找对象之间的联系；或者从需求里提取一些动词，观察动词和对象之间的关系；我们将紧耦合的各自圈在一起，观察他们内在的联系，从而形成对应的界限上下文。形成之后，我们可以尝试用语言来描述下界限上下文的职责，看它是否清晰、准确、简洁和完整。简言之，限界上下文应该从需求出发，按领域划分。

- 上下文都包含哪些内容
    一个限界上下文不是只有领域模型，当然这个是必不可少的，它总体来说是一个系统，一个应用程序，或者一个业务服务，它里边会有实体，值对象，领域事件（一个个方法事件组成，比如用户注册，修改密码，验证信息等等都是该上下文中的领域事件），在我们的身份和访问上下文中，是这样定义的
    ![身份与访问上下文](https://img2018.cnblogs.com/blog/1468246/201810/1468246-20181025153435326-2063024321.png "身份与访问上下文")

是从战术技术上来解释说明战略中的领域概念，你想一下，我们如何在代码中直接体现领域的概念？当然没办法，领域是一个通过语言，领域专家和技术人员都能看懂的一套逻辑，而代码中的上下文才是实实在在的通过技术来实现。

- DDD中的实体是什么
许多对象不是由它们的属性来定义，而是通过一系列的连续性（continuity）和标识（identity）来从根本上定义的。只要一个对象在生命周期中能够保持连续性，并且独立于它的属性（即使这些属性对系统用户非常重要），那它就是一个实体。
对于实体Entity，实体核心是用唯一的标识符来定义，而不是通过属性来定义。即即使属性完全相同也可能是两个不同的对象。同时实体本身有状态的，实体又演进的生命周期，实体本身会体现出相关的业务行为，业务行为会实体属性或状态造成影响和改变。
如果从值对象本身无状态，不可变，并且不分配具体的标识层面来看。那么值对象可以仅仅理解为实际的Entity对象的一个属性结合而已。该值对象附属在一个实际的实体对象上面。值对象本身不存在一个独立的生命周期，也一般不会产生独立的行为。

- 为什么要使用实体
当我们需要考虑一个对象的个性特征，或者要区分不同对象的时候，我们就需要一个实体这个领域概念，一个实体是一个唯一的东西，并且可以长时间相当长的一段时间内持续的变化，但是无论我们做了多少变化，这个的实体对象可能也已经变化的很多了，但是因为他们都一个相同的身份标识，所有还是同一个实体。很简单，就好像一个学生，无论手机号，姓名，年龄，邮箱，是否毕业等等，全部变化了，因为唯一标识的原因，我们就可以认为，变化前后的所有对象，都是同一个实体。随着对象的改变，我们可能会一直跟踪变化过程，什么时候，什么人，发生了什么变化：就比如学生因为学习太好，学校研究通过，提前毕业，更新状态为已毕业等。
这个时候我们发现了，实体的两大特性：

1. 有唯一的标识，不受状态属性的影响。
2. 可变性特征，状态信息一直可以变化。

- 值对象 —— 不变性
值对象虽然有时候和实体特别想象，看上边的学校家庭信息就可得知，但是它却有着自己独有的好处，值对象很常见：比如数字，字符串，日期时间，甚至一个人的信息，邮寄地址等等，当然还有更复杂的值对象，这些都是反映 通用语言 概念的值对象。
我们应该尽量使用值对象来建模，而不是实体对象，你可能很想不通，即使上边的学生的家庭地址信息，你一定要单放一个数据库表，构建实体模型，在设计的时候我们应该也要更偏向作为一个值对象容器，而不是子实体容器，因为这样我们可以对值对象很好的创建，测试，使用，优化和维护。

当你决定一个领域概念是否是一个值对象的时候，你需要考虑它是否有以下特性：

1. 它描述了领域中的一个东西
2. 可以作为一个不变量。
3. 当它被改变时，可以用另一个值对象替换。
4. 可以和别的值对象进行相等性比较。

实体与值对象的区别：

1. 实体拥有标识，而值对象没有。
2. 相等性测试方式不同。实体根据标识判等，而值对象根据内部所有属性值判等。
3. 实体允许变化，值对象不允许变化。
4. 持久化的映射方式不同。实体采用单表继承、类表继承和具体表继承来映射类层次结构，而值对象使用嵌入值或序列化大对象方式映射。

在DDD领域驱动设计第一次被提出的时候，聚合的概念就随之而来了，在之前的文章中，我们说到了领域和子领域的划分，也说了限界上下文的定义，这些都是和我们平时以数据模型为中心所不同的概念，可能理解起来不是很容易，但是至少我们有了这个影子，想象着一个大的领域项目，根据业务来拆分成了多个子领域与上下文，可能不同的上下文中甚至有相似的概念，举个栗子就是，订单上下文有商品，物流上下文有货物，库存上下文有存货等等等等，这时候你会发现，其实他们都是指的同一个东西，只不过在不同的上下文中被人为的赋予了不同的概念，有的是实体（库存），有的是值对象（订单），但是它们又不是一个概念，因为他们属于不同的子领域。
![orderitem](https://img2018.cnblogs.com/blog/1468246/201811/1468246-20181106154755619-516926705.png "order item")

这个时候，既然我们从大的方面已经对限界上下文进行分离整合，与之而来的肯定是领域模型的分离（我们肯定不能把每一个表放在一起，也不会把他们都一个个并列排开），那既然有分离肯定就是有聚合，这个时候，聚合就出来了，其实DDD提出聚合的概念是为了保证领域内对象之间的一致性问题，因为我们从上边也看到了，在不同的地方会存在调用关系，当然主要还是子领域内部相互调用，

比如创建一个订单，必然会生成订单详情，订单详情肯定会有商品信息，我们在修改商品信息的时候，肯定就不能影响到这个订单详情中的商品信息。再比如：用户在下单的时候，会选择一个地址作为邮寄地址，如果该用户立刻下另一个订单，并对自己个人中心的地址进行修改，肯定就不能影响刚刚下单的邮寄地址信息。

这个时候，聚合就有很强的作用，通过值对象保证了对象之间的一致性。我们平时在开发的时候，虽然没有用到DDD，肯定也是经常用到聚合，就比如上边的问题，撇开DDD不谈，就平时来说，你肯定不会把商品 id 直接绑定到订单详情表中，为外键的，不然会死得很惨。这个时候其实我们就有一些聚合的概念了，因为什么呢，下单的时候，我们关注订单领域模型，修改商品的时候，我们关注商品领域模型，这些就是我们说到的聚合，当然一个上下文会有很多聚合，而且聚合要尽可能的细分，那如何正确的区分聚合，以及以什么为基准，请往下看。

1. 哪些实体或值对象在一起才能够有效的表达一个领域概念。

   比如：订单模型中，必须有订单详情，物流信息等实体或者值对象，这样才能完整的表达一个订单的领域概念，就比如文章开头中提到的那个Code栗子中，OrderItem、Goods、Address等

2. 确定好聚合以后，要确定聚合根.

   比如：订单模型中，订单表就是整个聚合的聚合根。

```C#
    /// <summary>
    /// 聚合根 Order
    /// </summary>
    public class Order : AggregateRoot
    {
        public Guid Id;
        public string OrderNo;
        public Address Address;//值对象
        public List<OrderItem> Items;//实体集合
        //...
    }
```

## ***聚合中的实体和值对象应该具有相同的生命周期，并应该属于一个业务场景。***

-------------------------------------------------------------------------------------------------------------------
![120901748](https://img2018.cnblogs.com/blog/1468246/201811/1468246-20181106170934289-120901748.png "120901748")

这样初看是没有什么问题，很正常呀，发帖子是发回复的聚合根，回复必须有一个帖子，不然无效，看似合理的地方却有不合理。

比如，当我要对一个帖子发表回复时，我取出当前帖子信息，嗯，这个很对，但是，如果我对回复进行回复的时候，那就不好了，我每次还是都要取出整个带有很多回复的帖子，然后往里面增加回复，然后保存整个帖子，因为聚合的一致性要求我们必须这么做。无论是在场景还是在并发的情况下这是不行的。

如果帖子和回复在一个聚合内，聚合意味着“修改数据的一个最小单元”，聚合内的所有对象要看成是一个整体最小单元进行保存。这么要求是因为聚合的意义是维护聚合内的不变性，数据一致性；
仔细分析我们会发现帖子和回复之间没有数据一致性要求。所以不需要设计在同一个聚合内。

从场景的角度，我们有发表帖子，发表回复，这两个不同的场景，发表帖子创建的是帖子，而发表回复创建的是回复。但是订单就不一样，我们有创建订单，修改订单这两个场景。这两个场景都是围绕这订单这个聚合展开的。

所以我们应该把回复实体也单独作为一个聚合根来处理：

![论坛发帖和回复](https://img2018.cnblogs.com/blog/1468246/201811/1468246-20181106171952131-1427193675.png "论坛发帖和回复")

## ***聚合根、实体、值对象的区别？***

- 从标识的角度：

聚合根具有全局的唯一标识，而实体只有在聚合内部有唯一的本地标识，值对象没有唯一标识，不存在这个值对象或那个值对象的说法；

- 从是否只读的角度：

聚合根除了唯一标识外，其他所有状态信息都理论上可变；实体是可变的；值对象是只读的；

- 从生命周期的角度：

聚合根有独立的生命周期，实体的生命周期从属于其所属的聚合，实体完全由其所属的聚合根负责管理维护；值对象无生命周期可言，因为只是一个值；

## ***聚聚合根、实体、值对象对象之间如何建立关联***

- 聚合根到聚合根：通过ID关联；

- 聚合根到其内部的实体，直接对象引用；

- 聚合根到值对象，直接对象引用；

- 实体对其他对象的引用规则：1）能引用其所属聚合内的聚合根、实体、值对象；2）能引用外部聚合根，但推荐以ID的方式关联，另外也可以关联某个外部聚合内的实体，但必须是ID关联，否则就出现同一个实体的引用被两个聚合根持有，这是不允许的，一个实体的引用只能被其所属的聚合根持有；

- 值对象对其他对象的引用规则：只需确保值对象是只读的即可，推荐值对象的所有属性都尽量是值对象；

## ***如何识别聚合与聚合根？***

明确含义：一个Bounded Context（界定的上下文）可能包含多个聚合，每个聚合都有一个根实体，叫做聚合根；

识别顺序：先找出哪些实体可能是聚合根，再逐个分析每个聚合根的边界，即该聚合根应该聚合哪些实体或值对象；最后再划分Bounded Context；

聚合边界确定法则：根据不变性约束规则（Invariant）。不变性规则有两类：1）聚合边界内必须具有哪些信息，如果没有这些信息就不能称为一个有效的聚合；2）聚合内的某些对象的状态必须满足某个业务规则；

1. 一个聚合只有一个聚合根，聚合根是可以独立存在的，聚合中其他实体或值对象依赖与聚合根。

2. 只有聚合根才能被外部访问到，聚合根维护聚合的内部一致性。

- 优点
其实整篇文章都是在说的聚合的优点，这里简单再概况下：

 聚合的出现，很大程度上，帮助了DDD领域驱动设计的全部普及，试想一下，如果没有聚合和聚合根的思维，单单来说DDD，总感觉不是很舒服，而且领域驱动设计所分的子领域和限界上下文都是从更高的一个层面上来区分
