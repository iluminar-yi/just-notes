SQL Query Basics
--
* SQL其实是最接近人类语言的一种Query Language，很多语句就算是直接翻译也是可以理解的"人话"，例如：
    ```sql
    -- SQL里的命令不区分大小写——"SELECT"和"select"是没有区别的
    -- 假设tblProductSales只有四列：OrderID, BuyerName, ProductID, Quantity
    select OrderID, BuyerName, ProductID, Quantity
    from tblProductSales;
    ```
    就可以翻译成：从`tblProductSales`表格选择（所有的）订单ID、买家姓名、产品ID和购买数量返回
* `select *`就是"选择所有（列/Columns）"的意思，所以这在你不知道表格的结构却想看一下数据的时候有用。况且当你看到数据的时候你就知道都有什么列了
    ```sql
    select * from someoneElsesTable;
    -- 此时根据不同的数据库系统可能支持不同的限制返回行数（而不是返回所有行/Row）的方法
    select * from someoneElsesTable limit 5;
    select top 5 * from someoneElsesTable;
    ```
* 有些数据库支持用`as`来给列起别名/Alias的
    ```sql
    select OrderID as oid, BuyerName as name, ProductID as pid, Quantity as q
    from tblProductSales;
    ```
    这样返回的数据里只有`oid`, `name`, `pid`, `q`四列
* 这在使用[聚合函数](#聚合函数/Aggregate Functions)的时候可以起名字用，不这么做的话不同的数据库会使用不同的默认名称
    ```sql
    select max(Quantity) as MaxPurchaseQuantityInHistory
    from tblProductSales;
    ```
    翻译：从`tblProductSales`表格选择最多的购买数称为`MaxPurchaseQuantityInHistory`返回
* 这种情况下，这些都可以合着用
    ```sql
    -- 例如想研究在购买数量多于1的那些订单里，买家都多买了多少个
    -- ExtraQuantity指多于1份的部分
    select OrderID as oid, Quantity - 1 as ExtraQuantity, *
    from tblProductSales;
    ```
  这样返回的数据里只有`oid`, `ExtraQuantity`, `OrderID`, `BuyerName`, 四列
* 某一列的名称有空格或者其他保留字符/Reserved Words的时候则需要Escape这些字符，说白了就是带上引号，如`` `Column Name` ``
#### `where`语句/Clauses
* `where`的作用其实和英语（更偏口语）的"where"是一样的，例如：I tend to delete all the emails where I don't find the content safe for work.
当然，这句话用更好的语法来说，例如：I tend to delete all the emails of which I don't find the content safe for work. 但是这里的"where"不局限于
是否是东西，还是地点，还是人，所以总体而言更加通用。
    ```sql
    select OrderID, BuyerName, ProductID, Quantity
    from tblProductSales
    where Quantity > 1;
    ```
    翻译为：从`tblProductSales`表格选择所有订单ID、买家姓名、产品ID和购买数量，当购买数量大于1的时候/从`tblProductSales`表格选择所有购买数量大于1的时候的订单ID、买家姓名、产品ID和购买数量返回

#### 操作符/Operators
* 加减乘除大于小于大于等于小于等于和等于应该不用说了吧：`+`, `-`, `*`, `/`, `>`, `<`, `>=`, `<=`, `=`
* 不等于是`!=`，这个和Java一样
* 但是如果是要查一个数值是不是`null`的时候要用`is not`：`where BuyerName is not null`

#### 聚合函数/Aggregate Functions
* 聚合这个词看上去很厉害，但其实叫这个名字的原因很简单：为了把分散的数据集合成更少的数据，例如求和/Sum函数就是把很多数字变成一个数字的函数
* 一些例子：`sum`, `avg`, `max`, `min`, `count`
* 经常要配合`Group By`一起使用，为什么呢？

#### Group/归类，集合
* `Group`其实就是归类的过程，所以`Group By`就是"根据...来归类"

假设数据是这样的：

| OrderID                              | BuyerName | ProductID    | Quantity |
|--------------------------------------|-----------|--------------|----------|
| 9a661dd9-eadc-4c31-8b01-710e6fd2c55f | Alex      | AProduct01   | 2        |
| 59c34a99-3a13-4c39-be56-af52de4c642f | Alex      | BProduct2020 | 1        |
| 96313bf4-fb10-4e21-bf35-64ad5e8e5fc8 | Mary      | AProduct01   | 10       |
| 69de362a-b487-49b9-b459-7cadf23b8969 | Alex      | AProduct01   | 1        |
| 3f8a5683-1f8c-4317-9ce1-23a93c15bd81 | Chris     | CProductV01  | 2        |
| 99adca2d-bf19-4b03-b304-226e63dbf2e8 | Alex      | BProduct2020 | 1        |

* 那当我们根据BuyerName归类/`group by BuyerName`时，我们在数据库内存中产生了类似这样的归类结果：
```
{
  Alex: [
    (Alex, AProduct01, 2),
    (Alex, BProduct2020, 1),
    (Alex, AProduct01, 1),
    (Alex, BProduct2020, 1),
  ],
  Mary: [
    (Mary, AProduct01, 10),
  ],
  Chris: [
    (Chris, CProductV01, 2),
  ],
}
```
* 而当我们根据ProductID归类/`group by ProductID`时，我们在数据库内存中产生了类似这样的归类结果：
```
{
  AProduct01: [
    (Alex, AProduct01, 2),
    (Mary, AProduct01, 10),
    (Alex, AProduct01, 1),
  ],
  BProduct2020: [
    (Alex, BProduct2020, 1),
    (Alex, BProduct2020, 1),
  ],
  CProductV01: [
    (Chris, CProductV01, 2),
  ],
}
```
* 因此当我们想知道"每个用户分别购买了多少次"的时候，我们需要：
    ```sql
    select BuyerName, count(1) as NumPurchases
    from tblProductSales
    group by BuyerName;
    -- 或者
    select BuyerName, count(*) as NumPurchases
    from tblProductSales
    group by BuyerName;
    -- 或者
    select BuyerName, count(OrderID) as NumPurchases
    from tblProductSales
    group by BuyerName;
    ```
  这个query的意思便是：先根据`tblProductSales`表格的买家名称进行归类，然后在每个归好的类里分别数有多少条记录，并在数完后选择买家名字和数完的结果将其命名为"NumPurchases"
  
  为什么有三个写法？`count(OrderID)`是指数数多少个不同的`OrderID`。因为每一个`OrderID`都是独特的，所以可以数出每一个人有多少条数据。而`count(1)`和`count(*)`都是约定俗成
  的数行数的方法
* 同样的，当我们想知道"每个产品平均购买数量是多少"的时候，我们需要：
    ```sql
    select ProductID, avg(Quantity) as AverageQuantity
    from tblProductSales
    group by ProductID;
    ```
  
#### Having/"具有/满足...的条件时"
* 唯一存在目的是因为`where`里不能有聚合函数——也就是，不能`where count(1) > 1`只能`having count(1) > 1`
* 所以它只会和`group by`一起出现
  
#### Order/排列
* `order by`虽然和`group by`一样都有`by`但是`order by`其实可以在没有使用`order by`的情况下使用
* 在不指明排列顺序的情况下默认`asc`/ascending/上升/从小到大，同时也可以设置成`desc`/descending/下降/从大到小
* 用的时候写作`order by someColumn asc`或者`order by someColumn desc`

#### 联接/Join
* `join`其实是个概念，不是每一个联接语句都需要用到`join`
* `join`的作用是把两个及以上的表格根据某种要求链接在一起
* 完整的`join`定义是要利用集合的知识，但为了快速理解，直接用例子：
    假设我们有`tblProductSales`和`tblProductInventory`:
    
    | OrderID                              | BuyerName | ProductID    | Quantity |
    |--------------------------------------|-----------|--------------|----------|
    | 9a661dd9-eadc-4c31-8b01-710e6fd2c55f | Alex      | AProduct01   | 2        |
    | 59c34a99-3a13-4c39-be56-af52de4c642f | Alex      | BProduct2020 | 1        |
    | 96313bf4-fb10-4e21-bf35-64ad5e8e5fc8 | Mary      | AProduct01   | 10       |
    | 69de362a-b487-49b9-b459-7cadf23b8969 | Alex      | AProduct01   | 1        |
    | 3f8a5683-1f8c-4317-9ce1-23a93c15bd81 | Chris     | CProductV01  | 2        |
    | 99adca2d-bf19-4b03-b304-226e63dbf2e8 | Alex      | BProduct2020 | 1        |
    
    | ProductID    | UnitPrice  |
    |--------------|------------|
    | AProduct01   | 3.15       |
    | BProduct2020 | 10.00      |
    | CProductV01  | 2.18       |
    | DProductV2.3 | 9.22       |
    
    当我们想把两个表格的内容合在一起展示的时候，我们就`join`了这两个表格。
* 通常我们用到的是inner join，这是`join`的默认模式，也是不需要特意写出`join`的情况
    假设我们想显示在显示所有订单的同时显示单价和总价，那么我们需要：
    ```sql
    select t1.*, t2.UnitPrice, t1.Quantity * t2.UnitPrice as TotalPrice
    from tblProductSales t1 join tblProductInventory t2
    on t1.ProductID = t2.ProductID;
    ```
    等同于：
    ```sql
    select t1.*, t2.UnitPrice, t1.Quantity * t2.UnitPrice as TotalPrice
    from tblProductSales t1 inner join tblProductInventory t2
    on t1.ProductID = t2.ProductID;
    ```
    还等同于：
    ```sql
    select t1.*, t2.UnitPrice, t1.Quantity * t2.UnitPrice as TotalPrice
    from tblProductSales t1, tblProductInventory t2
    where t1.ProductID = t2.ProductID;
    ```
    翻译为：从`tblProductSales`表格（称为"t1")和`tblProductInventory`表格（称为"t2")里，仅当t1里的产品ID和t2里的产品ID一致时，选择t1里的所有列，和t2的单价，和t1的数量和t2的单价相乘后称为总价返回
    
    当然，这是一种省事的写法，这么写的话`tblProductSales`的列会在前面，不好看，为了好看我们就得自己定义列的顺序：
    ```sql
    select t1.OrderID, t1.BuyerName, t1.ProductID, t2.UnitPrice, t1.Quantity, t1.Quantity * t2.UnitPrice as TotalPrice
    from tblProductSales t1 join tblProductInventory t2
    on t1.ProductID = t2.ProductID;
    ```
* 除了`inner join`以外，还有`left join`/`left outer join`，`right join`/`right outer join`，和`outer join`/`full outer join`/`full join`
* 其中`outer join`就是`left join`和`right join`同时做的操作
* `left join`指的是不管右边的表格里有没有对应的数据，都要在最终结果里确保有左边（指定列）的所有数据，右边不存在的话右边的列的数据都会是`null`
* 同样的`right join`指的是不管左面有没有对应的数据，都要在最终结果里确保有右边（指定列）的所有数据，左边不存在的话右边的列的数据都会是`null`，例如：
    ```sql
    select t1.*, t2.UnitPrice, t1.Quantity * t2.UnitPrice as TotalPrice
    from tblProductSales t1 right join tblProductInventory t2
    on t1.ProductID = t2.ProductID;
    ```
    因为没有人购买过`DProductV2.3`，所以左边会因为没有对应的数据而出现`null`

    | OrderID                              | BuyerName | ProductID    | Quantity | UnitPrice    | TotalPrice |
    |--------------------------------------|-----------|--------------|----------|--------------|------------|
    | 9a661dd9-eadc-4c31-8b01-710e6fd2c55f | Alex      | AProduct01   | 2        | 3.15         | 6.30       |
    | 59c34a99-3a13-4c39-be56-af52de4c642f | Alex      | BProduct2020 | 1        | 10.00        | 10.00      |
    | 96313bf4-fb10-4e21-bf35-64ad5e8e5fc8 | Mary      | AProduct01   | 10       | 3.15         | 31.50      |
    | 69de362a-b487-49b9-b459-7cadf23b8969 | Alex      | AProduct01   | 1        | 3.15         | 3.15       |
    | 3f8a5683-1f8c-4317-9ce1-23a93c15bd81 | Chris     | CProductV01  | 2        | 2.18         | 4.36       |
    | 99adca2d-bf19-4b03-b304-226e63dbf2e8 | Alex      | BProduct2020 | 1        | 10.00        | 10.00      |
    | null                                 | null      | null         | null     | 9.22         | null       |

    其中`TotalPrice`也是`null`因为`null`加减乘除数字得`null`。同时，之所以没有看到`DProductV2.3`是因为我们选择的是t1里的`ProductID`而没有选t2里的。
    
    所以，如果要找没有被买过的产品的话，我们可以：
    ```sql
    select t2.ProductID
    from tblProductSales t1 right join tblProductInventory t2
    on t1.ProductID = t2.ProductID
    where t1.ProductID is null;
    ```
    首先，t1和t2用`ProductID``right join`完了以后在来自t1的`ProductID`还能是`null`的话说明在t1里没有对应数据。此时的t2里的`ProductID`便是没有被购买过的产品ID
