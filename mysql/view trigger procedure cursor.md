**数据表**

*结构*

```mysql
CREATE TABLE `orders` (
  `id` int(11) NOT NULL,
  `amount` decimal(10,0) NOT NULL DEFAULT '0',
  `time` datetime NOT NULL,
  `user_id` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
*数据*

```mysql
INSERT INTO `orders` (`id`, `amount`, `time`, `user_id`)
     VALUES
     	(1,40,'2016-04-01 15:25:33',1),
     	(2,100,'2016-04-02 15:25:47',4),
     	(3,100,'2017-04-03 15:26:02',2),
     	(4,100,'2016-04-04 15:26:14',4),
     	(5,100,'2016-04-05 15:26:40',4),
     	(6,100,'2016-04-06 15:26:54',2),
     	(7,100,'2016-05-07 15:25:47',3),
     	(8,100,'2016-04-19 15:26:02',2),
     	(9,100,'2016-04-08 15:26:14',4),
     	(10,100,'2016-04-30 15:26:40',4),
     	(11,100,'2016-04-09 15:26:54',2),
     	(12,100,'2016-04-20 15:25:47',3),
     	(13,100,'2016-06-19 15:26:02',2),
     	(14,100,'2016-04-10 15:26:14',4),
     	(15,100,'2016-04-11 15:26:40',4);
```

## 1.视图
___

```mysql
-- 创建视图
create view orders_view1 as select * from orders;
-- 查看视图结构
desc orders_view1;
-- 查看视图创建语句
show create view orders_view1;
-- 查看视图内容
select * from order_view1;
-- 删除视图
drop view orders_view1;
```

**with check option**

> 这里可以理解为 with check option 的作用就是多了一个 check 的功能，即检查的功能，也就是说插入的数据必须满足该视图的条件，才允许被操作。

```mysql
create view orders_view2 as select * from orders where amount > 50 and id > 50 with check option;
update orders_view2 set amount = 40 where id = 60;
```
**执行以上命令之后，mysql报错**
`ERROR 1369 (HY000): CHECK OPTION failed 'test.orders_view2'`

## 2.触发器
___

```mysql
-- 创建触发器
create trigger orders_trigger_after_insert after insert on orders for each row select 'recode added' into @aa;
select @aa;
-- 或者
create trigger orders_trigger_after_insert after insert on orders for each row
begin
update orders set amount = amount + 1 where id = 1;
-- 代码区域
end;

insert into orders(id,amount,`time`,user_id) values(16, 30, '2016-04-01 15:25:33', 1);
```

## 3.存储过程
___
*语法*

```
create procedure 存储过程名(参数列表)
begin
	存储过程体
end
call 存储过程名(参数列表)
```

```mysql
delimiter //
create procedure orders_procedure(in x int) -- in 表示输入
begin
	select * from orders where id = x;
end //
delimiter ;
-- 执行
call orders_procedure(1);
```

## 4.游标
___
*语法*

```
DECLARE test_cursor CURSOR FOR 结果集; //声明游标
OPEN test_cursor; //打开游标
FETCH test_cursor INTO 变量名; //捕获数据
CLOSE test_cursor; //关闭游标
DECLARE CONTINUE HANDLER FOR NOT FOUND; //结果集查询不到数据自动跳出
```

```mysql
delimiter //
create procedure res(out count int)
begin
    declare o int default 0;
    declare blag int default 1;
    declare order_cursor cursor for select id from orders;
    declare continue handler for not found set blag = 0;
    set count = 100;
    open order_cursor;
    read_loop:loop
        fetch order_cursor into o;
        if blag = 0 then
            leave read_loop;
        end if;

        if o = 67 then
            set count = count + o;
        end if;
    end loop read_loop;
    close order_cursor;
end //

delimiter ;

call res(@count);
select @count;
```
