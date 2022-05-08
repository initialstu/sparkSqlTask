# sparkSqlTask
## 题目一：为 Spark SQL 添加一条自定义命令
* SHOW VERSION
* 显示当前 Spark 版本和 Java 版本

核心代码
* SqlBase.g4
```
```

* 增加ShowVersionCommand类
```scala
```

* SparkSqlParser.scala增加visitShowVersion方法
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
select distinct a11, a2+1 as a21, 'abc' as a33 from (select a1+1 as a11, a2 from chaicq0 where a1 > 1 and 1=1) where a11 > 1 except select b1, b2, 'cba' as b3 from chaicq1;
```

部分执行结果
```
22/05/08 16:53:24 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ReplaceExceptWithAntiJoin ===
!Except false                                                      Distinct
!:- Distinct                                                       +- Join LeftAnti, (((a11#30 <=> b1#23) AND (a21#31 <=> b2#24)) AND (a33#32 <=> b3#33))
!:  +- Project [a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]         :- Distinct
!:     +- Filter (a11#30 > 1)                                         :  +- Project [a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]
!:        +- Project [(a1#10 + 1) AS a11#30, a2#11]                   :     +- Filter (a11#30 > 1)
!:           +- Filter ((a1#10 > 1) AND (1 = 1))                      :        +- Project [(a1#10 + 1) AS a11#30, a2#11]
!:              +- Relation default.chaicq0[a1#10,a2#11] parquet      :           +- Filter ((a1#10 > 1) AND (1 = 1))
!+- Project [b1#23, b2#24, cba AS b3#33]                              :              +- Relation default.chaicq0[a1#10,a2#11] parquet
!   +- Relation default.chaicq1[b1#23,b2#24] parquet                  +- Project [b1#23, b2#24, cba AS b3#33]
!                                                                        +- Relation default.chaicq1[b1#23,b2#24] parquet

22/05/08 16:53:24 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ReplaceDistinctWithAggregate ===
!Distinct                                                                                 Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]
 +- Join LeftAnti, (((a11#30 <=> b1#23) AND (a21#31 <=> b2#24)) AND (a33#32 <=> b3#33))   +- Join LeftAnti, (((a11#30 <=> b1#23) AND (a21#31 <=> b2#24)) AND (a33#32 <=> b3#33))
!   :- Distinct                                                                              :- Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]
    :  +- Project [a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]                             :  +- Project [a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]
    :     +- Filter (a11#30 > 1)                                                             :     +- Filter (a11#30 > 1)
    :        +- Project [(a1#10 + 1) AS a11#30, a2#11]                                       :        +- Project [(a1#10 + 1) AS a11#30, a2#11]
    :           +- Filter ((a1#10 > 1) AND (1 = 1))                                          :           +- Filter ((a1#10 > 1) AND (1 = 1))
    :              +- Relation default.chaicq0[a1#10,a2#11] parquet                          :              +- Relation default.chaicq0[a1#10,a2#11] parquet
    +- Project [b1#23, b2#24, cba AS b3#33]                                                  +- Project [b1#23, b2#24, cba AS b3#33]
       +- Relation default.chaicq1[b1#23,b2#24] parquet                                         +- Relation default.chaicq1[b1#23,b2#24] parquet

22/05/08 16:53:24 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.PushDownPredicates ===
 Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]                             Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]
 +- Join LeftAnti, (((a11#30 <=> b1#23) AND (a21#31 <=> b2#24)) AND (a33#32 <=> b3#33))   +- Join LeftAnti, (((a11#30 <=> b1#23) AND (a21#31 <=> b2#24)) AND (a33#32 <=> b3#33))
    :- Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]                          :- Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]
    :  +- Project [a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]                             :  +- Project [a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]
!   :     +- Filter (a11#30 > 1)                                                             :     +- Project [(a1#10 + 1) AS a11#30, a2#11]
!   :        +- Project [(a1#10 + 1) AS a11#30, a2#11]                                       :        +- Filter (((a1#10 > 1) AND (1 = 1)) AND ((a1#10 + 1) > 1))
!   :           +- Filter ((a1#10 > 1) AND (1 = 1))                                          :           +- Relation default.chaicq0[a1#10,a2#11] parquet
!   :              +- Relation default.chaicq0[a1#10,a2#11] parquet                          +- Project [b1#23, b2#24, cba AS b3#33]
!   +- Project [b1#23, b2#24, cba AS b3#33]                                                     +- Relation default.chaicq1[b1#23,b2#24] parquet
!      +- Relation default.chaicq1[b1#23,b2#24] parquet

22/05/08 16:53:24 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.FoldablePropagation ===
!Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]                                          Aggregate [a11#30, a21#31, abc], [a11#30, a21#31, abc AS a33#32]
!+- Aggregate [a11#30, a21#31, a33#32], [a11#30, a21#31, a33#32]                                       +- Aggregate [a11#30, a21#31, abc], [a11#30, a21#31, abc AS a33#32]
    +- Project [(a1#10 + 1) AS a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]                              +- Project [(a1#10 + 1) AS a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]
!      +- Join LeftAnti, ((((a1#10 + 1) <=> b1#23) AND ((a2#11 + 1) <=> b2#24)) AND (abc <=> b3#33))         +- Join LeftAnti, ((((a1#10 + 1) <=> b1#23) AND ((a2#11 + 1) <=> b2#24)) AND (abc <=> cba))
          :- Filter (((a1#10 > 1) AND (1 = 1)) AND ((a1#10 + 1) > 1))                                           :- Filter (((a1#10 > 1) AND (1 = 1)) AND ((a1#10 + 1) > 1))
          :  +- Relation default.chaicq0[a1#10,a2#11] parquet                                                   :  +- Relation default.chaicq0[a1#10,a2#11] parquet
          +- Project [b1#23, b2#24, cba AS b3#33]                                                               +- Project [b1#23, b2#24, cba AS b3#33]
             +- Relation default.chaicq1[b1#23,b2#24] parquet                                                      +- Relation default.chaicq1[b1#23,b2#24] parquet

22/05/08 16:53:24 WARN [main] PlanChangeLogger:
=== Applying Rule org.apache.spark.sql.catalyst.optimizer.ConstantFolding ===
 Aggregate [a11#30, a21#31, abc], [a11#30, a21#31, abc AS a33#32]                                    Aggregate [a11#30, a21#31, abc], [a11#30, a21#31, abc AS a33#32]
 +- Aggregate [a11#30, a21#31, abc], [a11#30, a21#31, abc AS a33#32]                                 +- Aggregate [a11#30, a21#31, abc], [a11#30, a21#31, abc AS a33#32]
    +- Project [(a1#10 + 1) AS a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]                            +- Project [(a1#10 + 1) AS a11#30, (a2#11 + 1) AS a21#31, abc AS a33#32]
!      +- Join LeftAnti, ((((a1#10 + 1) <=> b1#23) AND ((a2#11 + 1) <=> b2#24)) AND (abc <=> cba))         +- Join LeftAnti, ((((a1#10 + 1) <=> b1#23) AND ((a2#11 + 1) <=> b2#24)) AND false)
!         :- Filter (((a1#10 > 1) AND (1 = 1)) AND ((a1#10 + 1) > 1))                                         :- Filter (((a1#10 > 1) AND true) AND ((a1#10 + 1) > 1))
          :  +- Relation default.chaicq0[a1#10,a2#11] parquet                                                 :  +- Relation default.chaicq0[a1#10,a2#11] parquet
          +- Project [b1#23, b2#24, cba AS b3#33]                                                             +- Project [b1#23, b2#24, cba AS b3#33]
             +- Relation default.chaicq1[b1#23,b2#24] parquet                                                    +- Relation default.chaicq1[b1#23,b2#24] parquet

```



### 题目三：实现自定义优化规则（静默规则）
* 静默规则，通过 set spark.sql.planChangeLog.level=WARN，确认执行到就行

核心代码
```scala
```

执行命令
```shell
spark-sql --jars /home/student3/chaicq/spark-sql-job/SparkJob-1.0-SNAPSHOT.jar --conf spark.sql.extensions=org.bigdata.sparkjob.MySparkSessionExtension
```
```sql
select a1*1 from chaicq0;
```

部分执行结果
```
22/05/08 17:56:31 WARN [main] MySparkRule: running MySparkRule
22/05/08 17:56:31 WARN [main] PlanChangeLogger:
=== Applying Rule org.bigdata.sparkjob.MySparkRule ===
!Project [(a1#0 * 1) AS (a1 * 1)#14]              Project [a1#0 AS (a1 * 1)#14]
 +- Relation default.chaicq0[a1#0,a2#1] parquet   +- Relation default.chaicq0[a1#0,a2#1] parquet


```

