# sparkSqlTask
## 题目一：为 Spark SQL 添加一条自定义命令
* SHOW VERSION
* 显示当前 Spark 版本和 Java 版本

核心代码
```scala
```

执行截图


## 题目二：构建 SQL 满足如下要求
### 构建一条 SQL，同时 apply 下面三条优化规则：
* CombineFilters
* CollapseProject
* BooleanSimplification

执行sql
```sql
```

执行截图


### 构建一条 SQL，同时 apply 下面五条优化规则：
* ConstantFolding
* PushDownPredicates
* ReplaceDistinctWithAggregate
* ReplaceExceptWithAntiJoin
* FoldablePropagation

执行sql
```sql
```

执行截图



### 题目三：实现自定义优化规则（静默规则）
* 静默规则，通过 set spark.sql.planChangeLog.level=WARN，确认执行到就行

核心代码
```scala
```

执行命令
```shell
```

执行截图

