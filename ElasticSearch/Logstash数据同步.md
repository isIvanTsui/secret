<h1 align="center">Logstash数据同步</h1>

1. 将下载好的 `mysql-connector-java.8.22.jar` 拷贝到 `**lib/mysql/**` 下

   ![在这里插入图片描述](https://raw.githubusercontent.com/isIvanTsui/img/master/20210407165625599.png)

2. 进入 `config` 目录，新建 `logstash.conf`文件，并修改内容如下：

   ```
   input {
     beats {
       port => 5044
     }
     # 可以在 logstash 控制台输入相对应的参数，来改变 output 的行为
     stdin {}
     jdbc {
     	type => "jdbc"
     	# 数据库连接地址，我的是 MySQL 8.0 的，所以连接必须带上时区
     	jdbc_connection_string => "jdbc:mysql://连接地址:3306/数据库名称?characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai"
     	# 数据库连接账号密码
   	jdbc_user => "root"
       jdbc_password => "root"
       # MySQL依赖包路径，名称一定要对应；
       jdbc_driver_library => "../lib/mysql/mysql-connector-java-8.0.22.jar"
       # the name of the driver class for mysql
       jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
       # 数据库重连尝试次数
       connection_retry_attempts => "3"
       # 判断数据库连接是否可用，默认false不开启
       jdbc_validate_connection => "true"
       # 数据库连接可用校验超时时间，默认3600S
       jdbc_validation_timeout => "3600"
       # 开启分页查询（默认false不开启）；
       jdbc_paging_enabled => "true"
       # 单次分页查询条数（默认100000,若字段较多且更新频率较高，建议调低此值）；
       jdbc_page_size => "500"
       # statement为查询数据sql，如果sql较复杂，建议配通过statement_filepath配置sql文件的存放路径；
       # sql_last_value为内置的变量，存放上次查询结果中最后一条数据tracking_column的值，此处即为ModifyTime；
       # 这个你需要自己多尝试，执行 sql 文件
       #statement_filepath => "../lib/mysql/jdbc.sql"
   	# 查询语句，高级一点的就是增加查询条件
       statement => "select * from `xxx`"
       # 是否将字段名转换为小写，默认true（如果有数据序列化、反序列化需求，建议改为false）；
       lowercase_column_names => false
       # Value can be any of: fatal,error,warn,info,debug，默认info；
       sql_log_level => warn
       # 是否记录上次执行结果，true表示会将上次执行结果的tracking_column字段的值保存到last_run_metadata_path指定的文件中；
       record_last_run => true
       # 需要记录查询结果某字段的值时，此字段为true，否则默认tracking_column为timestamp的值；
       use_column_value => true
       # 需要记录的字段，用于增量同步，需是数据库字段
       tracking_column => "ModifyTime"
       # Value can be any of: numeric,timestamp，Default value is "numeric"
       tracking_column_type => timestamp
       # record_last_run上次数据存放位置；
       last_run_metadata_path => "mysql/last_id.txt"
       # 是否清除last_run_metadata_path的记录，需要增量同步时此字段必须为false；
       clean_run => false
       # 同步频率(分 时 天 月 年)，默认每分钟同步一次； 定时任务中的 corn 表达式
       schedule => "* * * * *"
     }
   }
   
   output {
     elasticsearch {
     	# 作为数组可以存储集群数据
       hosts => ["http://localhost:9200"]
       # 索引名字
       index => "blog"
       # index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
       # 数据的唯一索引，就是你查询的表的主键或者一个唯一 ID，自动替换为 ES 的 _id 字段
       document_id => "%{blog_id}"
     }
     stdout {
   	  codec => json_lines
     }
   }
   ```

   

3. 重启`Logstash`

   可以看到 `MySQL` 数据库中的内容已经同步过来了

   ![在这里插入图片描述](https://raw.githubusercontent.com/isIvanTsui/img/master/20210407171126323.png)