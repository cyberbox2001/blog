### CQRS（Command & Query Responsibility Segregation）命令查询职责分离，和 REST 同属于架构风格，如果单纯理解 CQRS，是比较容易的，另一种方式解释就是，一个方法要么是执行某种动作的命令，要么是返回数据的查询，命令的体现是对系统状态的修改，而查询则不会，职责的分离更加有利于领域模型的提炼，系统的灵活性和可扩展性也得到进一步加强。



### CQRS

## Commands
- 发送命令是唯一改变系统状态的方式
- Commands对所有系统的变更都要负责
- 如果没有command，系统的状态就一直保持不变
- Command不应该返回任何值
- 我使用两个类来实现：Command和CommandHandler。Command仅仅是被CommandHandler是用来作为一些操作的输入参数。在我这里，command仅仅是领域模型中调用的一系列特定操作。

## Query
- query相当于读操作
- 通过读取系统的状态，过滤，聚合，转换数据，最后以某种特定的格式传输
- 可以被执行多次并且不会影响到系统的状态
- 我通常在一个类中使用多个execute(…）方法来实现，但我现在却认为将Query和QueryHandler、QueryExecutor分开来讲或许是对的。

## Model
model代表了数据的容器，而domain model则包含了复杂的商业逻辑

## CQRS != Event Sourcing（事件溯源）
Event Sourcing 的的定义很简单：我们的领域产生的事件即表明了系统中发生过的变化。如果我们从刚开始就记录了整个系统并进行回放，我们就会拿到当初系统的状态记录。可以想象下银行账户，从刚开始的空账户，通过回放每个单一事务得到最终（也就是目前）的收支情况。所以，只要我们存储了所有的事件，就能得到系统当前的状态。






![CQRS](https://images0.cnblogs.com/blog2015/435188/201504/211440100462493.png "CQRS")
