# OOP
- "is a"关系和"has a"关系

假设有两个类：Computer和Employee。明显地，这两个类之间不存在"is a"的关系，即Employee不是计算机，它们之间没有继承关系的必要。因此不可能产生代码重用性。但这两个类之间是"has a"关系，即是支持的关系。例如，Employee"has a"Computer。明显地是一种支持关系。这种支持关系落实到代码中，就是在Employee中创建Computer的对象，调用其方法，到达完成某种运算和操作的目的。
