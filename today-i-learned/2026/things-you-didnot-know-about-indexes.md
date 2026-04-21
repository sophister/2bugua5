# 那些你不知道的数据库索引

![封面图片](./assets/things-you-didnot-know-about-indexexs.png)

**原文链接**：https://jon.chrt.dev/2026/04/15/things-you-didnt-know-about-indexes.html



偶尔看到这篇关于*数据库索引*的文章，作为低价值的**前端页面仔**，还是学到了很多新知识，记录下，以备后用。

>什么？你问我为什么**前端页面仔**也要了解数据库，还包括数据库索引？
>
>不会吧，不会吧，都2026年了，你们公司还没有开始全员AI、全员全栈转型么？
>
>据我了解，现在吹嘘全栈工程师已经落后了，国内某家**价值观公司**的一家子公司，已经开始所谓的合并产品岗（PM）、设计岗（Design)、前端岗(FrontEnd)为一个岗位——**AI产品设计前端工程师(PDFE)** 。把架构岗(Arch)、后端刚(Backend)合并成 **AI架构及后端工程师(ABE)** 。
>
>讲真，轮名词制造、价值观制造方向，国内除了某**遥遥领先公司**，第二就属**价值观公司**。



## 从你已经知道的开始

数据库索引，很像你在学生时代科学课本最后看到的目录索引。

想找关于 Phosphorus（磷）的内容？翻到书的最后，你会看到一个按字母排序的主题列表，以及对应的页码。

```

...
Oxygen .................. 42, 88, 103
Periodic table .......... 12–15
Phosphorus .............. 67, 91
Photosynthesis .......... 54, 72, 110
Potassium ............... 33, 78
...

```

假设我们有一张 Pokémon（宝可梦）的数据库表，大概是这样的¹：

```sql

id   | name       | type_1   | type_2   | generation | is_legendary | base_attack
-----+------------+----------+----------+------------+--------------+-------------
1    | Bulbasaur  | Grass    | Poison   | 1          | false        | 49
4    | Charmander | Fire     | NULL     | 1          | false        | 52
...
25   | Pikachu    | Electric | NULL     | 1          | false        | 55
...
150  | Mewtwo     | Psychic  | NULL     | 1          | true         | 110

```

如果没有索引，要找到 Pikachu，数据库需要逐行读取每一条记录，逐个检查 name 列。

当只有 4 行数据时，这几乎是瞬间完成的；但如果有 1000 万行，就可能成为问题。这种读取方式叫做全表扫描（full table scan），它的速度基本和名字一样（并不快²）。

但如果我们在 name 上加一个索引，就会得到类似于书本索引的结构：

```sql

name          row
------------+-----
Bulbasaur   → 1
Charmander  → 4
Mewtwo      → 150
Pikachu     → 25

```

它是有序的，因此数据库可以对 name 进行二分查找，快速定位到对应的行，而不需要扫描整张表。

在 PostgreSQL 中，这个结构底层是用 B-tree 存储的，但本质和课本中的索引是一样的：一份有序的数据结构，可以快速查找。

那么，我们是不是应该把所有字段都加上索引？


## 索引的代价

尽管索引对查询很有帮助，但“给所有东西加索引”并不是一个好主意，因为它存在一些权衡。

关于索引，需要始终记住一点：

读变快了，写就会变慢。

有了索引之后，每一次 INSERT、UPDATE 或 DELETE，都需要同时更新索引。

当我们插入一条新的 Pokémon 数据时，数据库需要在 name 索引中找到正确的位置并插入进去。而且一个表可以有多个索引，这个成本会乘以索引的数量。

此外，索引本身也是数据结构，需要存储在磁盘上，并被加载到内存中。

如果一张表有 8 个索引，那就意味着缓存中需要维护 9 份结构，而不是 1 份。

还不要忘了查询规划器（query planner）。

当你执行一个查询时，规划器需要选择“成本最低”的执行方式。索引越多，可选方案就越多，规划时间也会增加，在一些非常快的查询中，甚至可能超过执行时间本身。


## 为什么你的索引没有生效

你已经权衡了利弊，决定创建索引。这很好。

但它并没有像你预期的那样工作。你没有看到性能提升，甚至变慢了。

下面是一些常见问题。


### 复合索引对顺序敏感

回到 Pokémon 表，你可能会觉得在 type_1 和 type_2 上建立索引是个不错的选择。

例如：“查询所有 Water + Flying 类型的宝可梦”是一个很合理的需求。

```sql

CREATE INDEX ON pokemon (type_1, type_2);

```

这个索引对于以下查询是有效的：

```sql

SELECT * FROM pokemon WHERE type_1 = 'Water';
SELECT * FROM pokemon WHERE type_1 = 'Water' AND type_2 = 'Flying';

```

但对于这个查询就不太行：

```sql

SELECT * FROM pokemon WHERE type_2 = 'Flying';

```

这让你感到意外吗？

当你创建一个复合索引 (type_1, type_2) 时，数据库实际上会构建一个类似这样的结构：

```

Bug      → Flying   → [Butterfree, Beedrill, ...]
→ Poison   → [Venonat, Spinarak, ...]
→ Water    → [Surskit, ...]
Electric → NULL     → [Pikachu, Raichu, ...]
→ Flying   → [Zapdos, ...]
→ Steel    → [Magnemite, ...]
Fire     → NULL     → [Charmander, Vulpix, ...]
→ Flying   → [Charizard, Moltres, ...]
→ Ground   → [Numel, ...]
Grass    → Poison   → [Bulbasaur, Oddish, ...]
→ ...
Water    → NULL     → [Squirtle, Psyduck, ...]
→ Flying   → [Wingull, Pelipper, ...]
→ Ground   → [Wooper, ...]

```

它首先按照 type_1 排序，然后在每个分组内部再按 type_2 排序。

这使得它非常适合：

- 查询 type_1
- 同时查询 type_1 和 type_2

但对于仅查询 type_2 的情况，它无法按预期工作。

比如 Flying 类型，被分散在 Bug、Electric、Fire 和 Water 下，没有一个可以直接跳转的位置。

因此需要思考一个问题：

常见查询模式是什么？

如果你会像查询 type_1 一样频繁地查询 type_2，那么应该为 type_2 单独创建一个索引。


### 函数会让索引失效

不区分大小写的查询是一个常见需求。

用户不会在意输入的是 “Pikachu”、“pikachu” 还是 “PiKaChU”，他们只想要结果。

于是我们可能会写出这样的查询：

```sql

SELECT * FROM pokemon WHERE lower(name) = 'pikachu';

```

看起来完全没问题。我们已经在 name 上建了索引，这个查询应该很快。

但实际上并不是。

原因是：索引在 name 上，而不是在 lower(name) 上。

对于数据库来说，这两者是完全不同的。

索引中存的是：

Bulbasaur, Charmander, Squirtle, Pikachu

而不是：

bulbasaur, charmander, squirtle, pikachu

因此，当你查询 lower(name) 时，数据库没有对应的有序结构可以使用，只能退回到全表扫描，对每一行执行 lower() 再进行比较。

这适用于任何对列进行函数包裹的情况。

只要数据库在比较表达式的左侧看不到“原始列”，索引就无法使用。

还有一个容易忽略的问题：隐式类型转换。

当你用字符串去比较整数，或者用字符串比较时间戳时，数据库可能会进行隐式转换，其效果等同于在列上应用了函数。

那该怎么解决？

我们可以在表达式上建立索引。

例如，在 lower(name) 上建立索引是完全合法的，这样上面的查询就可以使用索引。

我们后面会再提到这一点。


## 如何避免这些问题？

简短的答案是：不要猜，去测。

怎么测？问数据库本身。

PostgreSQL 提供了一个工具：EXPLAIN。

它可以告诉你数据库打算如何执行一个查询。

在任何 SELECT 前面加上 EXPLAIN，就可以看到执行计划：

```sql

EXPLAIN SELECT * FROM pokemon WHERE name = 'Pikachu';

```

```

Index Scan using pokemon_name_idx on pokemon
Index Cond: (name = 'Pikachu'::text)

```

出现 Index Scan 就说明索引被使用了。

对比之前的问题查询：

```sql

EXPLAIN SELECT * FROM pokemon WHERE lower(name) = 'pikachu';

```
```

Seq Scan on pokemon
Filter: (lower(name) = 'pikachu'::text)

```

Seq Scan 表示全表扫描，没有使用索引。

如果你想看到真实执行时间，而不是规划器的估算，可以使用 EXPLAIN ANALYZE。

它会实际执行查询，并报告真实情况。

有时候结果会出乎你的意料。


## 那些你可能不知道的索引类型

前面提到的单列索引和复合索引已经覆盖了大多数场景，但还有一些其他类型的索引，在特定情况下非常有用。


### 函数索引（Functional indexes）

我们之前已经见过这个查询：

```sql

SELECT * FROM pokemon WHERE lower(name) = 'pikachu';

```

name 上的索引无法使用，因为数据库需要的是 lower(name) 的有序结构。

解决办法是创建这样的索引：

```sql

CREATE INDEX ON pokemon (lower(name));

```

这就是函数索引（也称表达式索引）。

它索引的不是列本身，而是对列应用某个表达式后的结果。

这不仅限于 lower()，你可以对任何确定性的表达式建索引：

```sql

CREATE INDEX ON pokemon (lower(name));
CREATE INDEX ON users ((created_at::date));
CREATE INDEX ON pokemon ((base_attack * 2));

```

只要表达式是确定性的、不可变的即可。

需要注意的是，如果你频繁使用函数索引，也可能是一个信号：

你是否应该直接以这种形式存储数据？

例如，如果你经常用小写进行查询，是否应该直接把数据存为小写？


### 部分索引（Partial indexes）

大多数索引覆盖整张表。

但有时候，你只会查询其中的一小部分数据，对整表建立索引是一种浪费。

在 Pokémon 表中，有一个字段 is_legendary。

目前大约有 1000 种宝可梦，其中只有约 80 种是传说中的（不到 10%）。

假设有一个查询：“显示所有传说宝可梦”。

你可能会创建这样的索引：

```sql

CREATE INDEX ON pokemon (is_legendary, name);

```

这是可行的，但有些浪费。

索引包含了所有 1000 条记录，而查询只关心其中 80 条。

而且由于 is_legendary 是布尔值，索引会先分成 false 和 true 两组，其中 false 部分约有 920 条数据，对这些查询来说是无用的负担。

部分索引只索引满足条件的行：

```sql

CREATE INDEX ON pokemon (name) WHERE is_legendary = true;

```

现在索引只包含 80 条记录，而不是 1000 条。

它更小、查询更快。

查询条件为 `WHERE is_legendary = true` 时可以使用该索引，而不满足条件的查询（例如 false）则会回退到全表扫描，这正是我们希望的行为。

当某个字段的查询大多集中在某个值时，可以考虑使用部分索引。

例如用户表中实现了软删除：

```sql

CREATE INDEX ON users (email) WHERE deleted_at IS NULL;

```

被软删除的数据很少会被查询，但如果不排除它们，会让索引变得臃肿。


### 覆盖索引（Covering indexes）

数据库使用索引定位一行数据时，这只是第一步。

索引只告诉数据库数据在哪里，接下来还需要访问表本身，读取查询所需的其他列。

也就是说，需要两次访问。

但如果索引本身已经包含了查询所需的所有列，就不需要访问表了。

这就是覆盖索引。

在 EXPLAIN 中，它会显示为 Index Only Scan。

例如：

```sql

SELECT name FROM pokemon WHERE is_legendary = true;

```

前面创建的部分索引刚好可以覆盖这个查询。

索引包含 name，并且只包含 is_legendary = true 的数据，而查询正好只需要这些信息。

因此不需要访问表。

我们也可以主动构建覆盖索引。

PostgreSQL 允许使用 INCLUDE 添加额外列：

```sql

CREATE INDEX ON pokemon (name) INCLUDE (base_attack);

```

这样下面的查询就可以完全通过索引完成：

```sql

SELECT name, base_attack FROM pokemon WHERE name = 'Pikachu';

```

为什么不直接把 base_attack 放到索引列里？

因为索引列会参与排序和查找。

如果把 base_attack 加入索引键中，数据库就需要按 name 和 base_attack 排序，这会增加写入成本，而查询并不需要这种排序。

INCLUDE 的作用是：把列带上，但不参与排序。




>编辑（2026-04-15）：对上述内容做一个小修正（感谢 u/therealgaxbo 指出）。写入性能并不是使用 INCLUDE 创建覆盖索引的主要原因，上面的性能提升描述有些夸大。
>
>实际使用 INCLUDE 的原因包括：
>
>
>- 可以包含那些没有合适操作符类的数据类型列（例如 box 类型无法作为 B-tree 索引键，但可以 INCLUDE）
>- 可以在唯一索引中包含额外列，而不改变唯一性约束  
>  例如：
>
>```sql
>
>CREATE UNIQUE INDEX ON users (email) INCLUDE (user_id);
>
>```
>仍然只保证 email 唯一，但可以覆盖需要 user_id 的查询



索引对很多数据库初学者来说，往往是事后才考虑的东西。

即使是经验丰富的工程师，也可能随意创建索引，或者完全忽略它们。

但合理的索引设计，往往决定了数据库性能的上限。


想更深入了解？[《Use The Index, Luke》](https://use-the-index-luke.com/ )这个电子书会教你几乎所有相关知识。

¹ 是的，这张表在范式设计上并不优秀  
² 公平地说，全表扫描在绝对性能上其实非常快。现代数据库每秒可以扫描数百万行。问题不在于慢，而在于它是线性的：数据翻倍，时间也翻倍。而索引查找几乎不会受到数据量增长的影响。

