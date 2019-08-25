## 在LINQ中，委托通常被视为数据管道的一部分，接受输入并返回结果，或者判断某项是否符合当前的筛选器等等。

```

// 下面的代码通过编译
Expression<Func<int, int>> expr = x => x + 1;


// 下面的代码编译不通过
Expression<Func<int, int, int>> expr2 = (x, y) => { return x + y; };
Expression<Action<int>> expr3 = x => {  };
```
- 但是别想象的太美好，这种方式只能创建最简单的表达式树，复杂点的编译器就不认识了。

- 右边是一个Lambda表达式，而左边是一个表达式树。为什么可以直接赋值呢？这个就要多亏我们的Expression<TDelegate>泛型类了。
### 而Expression<TDelegate>是直接继承自LambdaExpression的，我们来看一下Expression的构造函数：
```
internal Expression(Expression body, string name, bool tailCall, ReadOnlyCollection<ParameterExpression> parameters)
    : base(typeof(TDelegate), name, body, tailCall, parameters)
{
}
```

实际上这个构造函数什么也没有做，只是把相关的参数传给了父类，也就是LambdaExpression，由它把我们表达式的主体，名称，以及参数保存着。

```
Expression<Func<int, int>> expr = x => x + 1;
Console.WriteLine(expr.ToString());  // x=> (x + 1)
 
var lambdaExpr = expr as LambdaExpression;
Console.WriteLine(lambdaExpr.Body);   // (x + 1)
Console.WriteLine(lambdaExpr.ReturnType.ToString());  // System.Int32
 
foreach (var parameter in lambdaExpr.Parameters)
{
    Console.WriteLine("Name:{0}, Type:{1}, ",parameter.Name,parameter.Type.ToString());
}
 
//Name:x, Type:System.Int32
```


