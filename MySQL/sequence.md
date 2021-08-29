# Sequence

若插入数据数据时想要实现某列能默认自动填充且自增的功能，就需要序列来实现。

**使用自定义函数+触发器来实现**

1. 新建序列表（用于为需要添加序列的字段准备）

   ```sql
   drop table if exists sequence;   
   create table sequence (       
   seq_name        VARCHAR(50) NOT NULL, -- 序列名称       
   current_val     INT         NOT NULL, -- 当前值       
   increment_val   INT         NOT NULL    DEFAULT 1, -- 步长(跨度)       
   PRIMARY KEY (seq_name)   );
   ```

2. 新增序列数据

   ```sql
   INSERT INTO sequence VALUES ('seq_test1_num1', '0', '1');
   INSERT INTO sequence VALUES ('seq_test1_num2', '0', '2');
   ```

   > 新增后数据如下：
   
   ![inesrt_sequence_value1](https://i.loli.net/2021/08/13/hkpbR1tU3JxmzAG.png)

3. 创建currval函数用于获取序列当前值(v_seq_name 参数值 代表序列名称)

   ```sql
   create function currval(v_seq_name VARCHAR(50))   
   returns integer(11) 
   begin
    declare value integer;
    set value = 0;
    select current_val into value  from sequence where seq_name = v_seq_name;
      return value;
   end;
   ```

4. 创建nextval 函数用于获取序列下一个值(v_seq_name 参数值 代表序列名称)

   ```sql
   delimiter #
   create function nextval (v_seq_name VARCHAR(50)) returns integer
   begin
       update sequence set current_val = current_val + increment_val  where seq_name = v_seq_name;
   	return currval(v_seq_name);
   end;
    
   或者：
    
   create function nextval (v_seq_name VARCHAR(50)) returns integer(11) 
   begin
       update sequence set current_val = current_val + increment_val  where seq_name = v_seq_name;
   	return currval(v_seq_name);
   end;
   ```

5. **新建表 平时你自己用的表（即需要创建序列的表）**

   ```sql
   DROP TABLE IF EXISTS `test1`;
   CREATE TABLE `test1` (
     `name` varchar(255) NOT NULL,
     `value` double(255,0) DEFAULT NULL,
     `num1` int(11) DEFAULT NULL,
     `num2` int(11) DEFAULT NULL,
     PRIMARY KEY (`name`)
   );
   ```

6. **新建触发器 插入新纪录前给自增字段赋值实现字段自增效果**

   ```sql
   CREATE TRIGGER `TRI_test1_num1` BEFORE INSERT ON `test1` FOR EACH ROW BEGIN
   set NEW.num1 = nextval('seq_test1_num1');
   set NEW.num2 = nextval('seq_test1_num2');
   END
   ```

7. 最后插入数据测试自增效果

   ```sql
   INSERT INTO test1 (name, value) VALUES ('1', '111');
   INSERT INTO test1 (name, value) VALUES ('2', '222');
   INSERT INTO test1 (name, value) VALUES ('3', '333');
   INSERT INTO test1 (name, value) VALUES ('4', '444');
   ```

   > 此时未插入num1和num2字段的值，但触发器会被触发且执行定义的函数为num1和num2填充值
   >
   > 结果如下：
   
   ![inesrt_sequence_value2](https://i.loli.net/2021/08/13/yIY4xRLFUBfVahl.png)

