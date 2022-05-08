# sparkSqlTask
## 题目一：为 Spark SQL 添加一条自定义命令
* SHOW VERSION
* 显示当前 Spark 版本和 Java 版本

核心代码
```scala
```

执行截图


## 题目二：构建 SQL 满足如下要求（通过 set spark.sql.planChangeLog.level=WARN，确认执行）
### 构建一条 SQL，同时 apply 下面三条优化规则：
* CombineFilters
* CollapseProject
* BooleanSimplification

执行sql
```sql
select a11, a2+1 as a21 from (select a1+1 as a11, a2 from chaicq0 where a1 > 1 and true) where a11 > 1;
```

部分执行结果
```
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.PushDownPredicates ===
 Project [a11#17, (a2#11 + 1) AS a21#18]                     Project [a11#17, (a2#11 + 1) AS a21#18]
!+- Filter (a11#17 > 1)                                      +- Project [(a1#10 + 1) AS a11#17, a2#11]
!   +- Project [(a1#10 + 1) AS a11#17, a2#11]                   +- Filter (((a1#10 > 1) AND true) AND ((a1#10 + 1) > 1))
!      +- Filter ((a1#10 > 1) AND true)                            +- Relation default.chaicq0[a1#10,a2#11] parquet
!         +- Relation default.chaicq0[a1#10,a2#11] parquet

22/05/08 16:10:59 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.CollapseProject ===
!Project [a11#17, (a2#11 + 1) AS a21#18]                       Project [(a1#10 + 1) AS a11#17, (a2#11 + 1) AS a21#18]
!+- Project [(a1#10 + 1) AS a11#17, a2#11]                     +- Filter (((a1#10 > 1) AND true) AND ((a1#10 + 1) > 1))
!   +- Filter (((a1#10 > 1) AND true) AND ((a1#10 + 1) > 1))      +- Relation default.chaicq0[a1#10,a2#11] parquet
!      +- Relation default.chaicq0[a1#10,a2#11] parquet

22/05/08 16:10:59 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.BooleanSimplification ===
 Project [(a1#10 + 1) AS a11#17, (a2#11 + 1) AS a21#18]     Project [(a1#10 + 1) AS a11#17, (a2#11 + 1) AS a21#18]
!+- Filter (((a1#10 > 1) AND true) AND ((a1#10 + 1) > 1))   +- Filter ((a1#10 > 1) AND ((a1#10 + 1) > 1))
    +- Relation default.chaicq0[a1#10,a2#11] parquet           +- Relation default.chaicq0[a1#10,a2#11] parquet

```


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

