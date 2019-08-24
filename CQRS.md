## CQRS（Command & Query Responsibility Segregation）命令查询职责分离，和 REST 同属于架构风格，如果单纯理解 CQRS，是比较容易的，另一种方式解释就是，一个方法要么是执行某种动作的命令，要么是返回数据的查询，命令的体现是对系统状态的修改，而查询则不会，职责的分离更加有利于领域模型的提炼，系统的灵活性和可扩展性也得到进一步加强。

![CQRS](https://images0.cnblogs.com/blog2015/435188/201504/211440100462493.png "CQRS")
