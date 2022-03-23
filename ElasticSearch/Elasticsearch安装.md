<h1 align="center">Elasticsearch的安装</h1>

## ElasticSearch的安装

1. 下载：

   下载地址：https://www.elastic.co/cn/downloads/

   历史版本下载：https://www.elastic.co/cn/downloads/past-releases/

2. 运行：

   ![img](https://raw.githubusercontent.com/isIvanTsui/img/master/20201124212858.png)

   ![img](https://raw.githubusercontent.com/isIvanTsui/img/master/20201124212847.png)

   ![image-20220311093930611](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220311093930611.png)



-----

## 安装可视化界面elasticsearch-head

1. 下载：

   https://github.com/mobz/elasticsearch-head

   它是一个`vue`项目

2. 安装依赖包：

   ```
   cd elasticsearch-head
   # 安装依赖
   npm install
   ```

3. 修改`ElasticSearch`的`elasticsearch.yaml`文件来允许跨域请求：

   **开启跨域（在elasticsearch解压目录config下elasticsearch.yml中添加）**

   ```
   # 开启跨域
   http.cors.enabled: true
   # 所有人访问
   http.cors.allow-origin: "*"
   ```

   ![image-20220311094307951](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220311094307951.png)

4. 运行`elasticsearch-head`项目

   

   ```
   # 输入运行命令
   npm run start
   ```

   ![image-20220311094542636](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220311094542636.png)

   访问：http://localhost:9100/

   ![image-20220311094612983](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220311094612983.png)

-----



## 安装索引分析引擎`Kibana`

1. 下载：

   https://www.elastic.co/cn/downloads/

   历史版本下载：https://www.elastic.co/cn/downloads/past-releases/

2. 安装

   解压即可（尽量将ElasticSearch相关工具放在统一目录下）

   ![img](https://raw.githubusercontent.com/isIvanTsui/img/master/20201124224049.png)

3. 启动

   ![img](https://raw.githubusercontent.com/isIvanTsui/img/master/20201124224155.png)

   ![img](https://raw.githubusercontent.com/isIvanTsui/img/master/20201124224502.png)

   

4. 使用

   http://localhost:5601/

   先建立索引和添加文档，然后在`Kibana`里导入索引

   ![importIndex](https://raw.githubusercontent.com/isIvanTsui/img/master/importIndex.gif)

   **移除索引**

   ![removeIndex](https://raw.githubusercontent.com/isIvanTsui/img/master/removeIndex.gif)

   > 注意：这里的移除索引并不是真正的把索引从`ElasticSearch`引擎里删除掉，而只是移除了在`Kibana`里导入的索引

   


## Logstash

1. 下载：

   https://www.elastic.co/cn/downloads/logstash

   历史版本下载：https://www.elastic.co/downloads/past-releases#logstash

2. 安装

   解压即可（尽量将ElasticSearch相关工具放在统一目录下）

   ![image-20220323100832204](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220323100832204.png)

3. 启动

   - `config`文件夹下新建`logstash.conf`文件，并编写如下类容：

     ```
     input {
      tcp {
      mode => "server"
      host => "127.0.0.1"
      port => 4560
      codec => json_lines
      }
     }
     output {
       elasticsearch {
         hosts => "127.0.0.1:9200"
         index => "springboot-logstash-%{+YYYY.MM.dd}"
       }
       stdout{  
           codec=>rubydebug  
       }
     }
     ```

     ![image-20220323132427691](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220323132427691.png)
   
   - 执行`logstash -e "input { stdin { } } output { elasticsearch { hosts => ['127.0.0.1:9200'] } }"`命令，查看索引是否建立成功：
   
     ![img](https://raw.githubusercontent.com/isIvanTsui/img/master/l)
   
   - 在`bin`目录下打开`cmd`执行`logstash -f ../config/logstash.conf`命令
   
     ![image-20220323101700388](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20220323101700388.png)
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   