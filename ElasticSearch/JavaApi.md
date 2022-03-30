<h1 align="center">JavaApi方式操作Elasticsearch</h1>

**完整版的项目所在地址：**

https://github.com/isIvanTsui/EsDemo

## 索引

```java
/**
 * 索引的CRUD
 *
 * @author Ivan
 * @date 2022/03/10
 */
@SpringBootTest
public class IndexTests {
    @Autowired
    private RestHighLevelClient client;

    /**
     * 创建索引
     */
    @Test
    public void createIndex() throws IOException {
        CreateIndexResponse response = client.indices().create(new CreateIndexRequest("ivan"), RequestOptions.DEFAULT);
        //是否创建成功
        System.out.println("索引是否创建成功:" + response.isAcknowledged());
        // 查看返回对象
        System.out.println(response);
    }

    /**
     * 索引是否存在
     */
    @Test
    public void IndexIsExists() throws IOException {
        boolean exists = client.indices().exists(new GetIndexRequest("ivan"), RequestOptions.DEFAULT);
        System.out.println("索引存在吗：" + exists);
    }

    /**
     * 删除索引
     */
    @Test
    public void deleteIndex() throws IOException {
        AcknowledgedResponse response = client.indices().delete(new DeleteIndexRequest("ivan"), RequestOptions.DEFAULT);
        System.out.println("删除成功:" + response.isAcknowledged());
        System.out.println(response);
    }
}
```

## 文档

```java
/**
 * 文档的CRUD
 *
 * @author Ivan
 * @date 2022/03/10
 */
@SpringBootTest
public class DocTests {
    @Autowired
    private RestHighLevelClient client;

    /**
     * 添加文档
     */
    @Test
    public void addDoc() throws IOException {
        User user = new User(1, "张三", 18);
        IndexRequest request = new IndexRequest("ivan");
        request.id(user.getUid() + "");
        request.timeout(TimeValue.timeValueMillis(1000));// request.timeout("1s")
        request.source(JSONUtil.toJsonStr(user), XContentType.JSON);
        // 将我们的数据放入请求中
        IndexResponse response = client.index(request, RequestOptions.DEFAULT);
        System.out.println(response.status());
        System.out.println(response);
    }

    /**
     * 批量添加文档
     */
    @Test
    public void addDocByBatch() throws IOException {
        ArrayList<User> users = new ArrayList<>();
        users.add(new User(1, "张学友", 28));
        users.add(new User(2, "刘德华", 24));
        users.add(new User(3, "黎明", 38));
        users.add(new User(4, "周杰伦", 48));
        users.add(new User(5, "林俊杰", 22));
        users.add(new User(6, "周润发", 20));
        users.add(new User(7, "黄晓明", 17));
        users.add(new User(8, "古天乐", 33));
        BulkRequest request = new BulkRequest();
        for (User user : users) {
            request.add(new IndexRequest("user").id(user.getUid() + "")
                    .source(JSONUtil.toJsonStr(user), XContentType.JSON));
        }
        BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
        System.out.println(responses.status());
        System.out.println(responses);
    }

    /**
     * 获取文档
     */
    @Test
    public void getDoc() throws IOException {
        GetResponse response = client.get(new GetRequest("ivan", "1"), RequestOptions.DEFAULT);
        System.out.println(response.getSourceAsString());
        System.out.println(response);
    }

    /**
     * 文档是否存在
     */
    @Test
    public void docIsExists() throws IOException {
        GetRequest request = new GetRequest("ivan", "1");
        // 不获取返回的 _source的上下文了
        request.fetchSourceContext(new FetchSourceContext(false));
        request.storedFields("_none_");
        boolean exists = client.exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    /**
     * 更新文档
     */
    @Test
    public void updateDoc() throws IOException {
        User user = new User(1, "李四", 20);
        UpdateRequest request = new UpdateRequest("ivan", "1");
        request.doc(JSONUtil.toJsonStr(user), XContentType.JSON);
        UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
        System.out.println(response.status());
        System.out.println(response);
    }

    /**
     * 删除文档
     */
    @Test
    public void deleteDoc() throws IOException {
        DeleteResponse response = client.delete(new DeleteRequest("ivan", "1"), RequestOptions.DEFAULT);
        System.out.println(response.status());
        System.out.println(response);
    }
}
```

## 映射

```java
/**
 * 创建索引映射
 * 自定义创建索引
 *
 * @throws IOException ioexception
 */
@Test
public void createIndexAndMapping() throws IOException {
    //先创建索引
    CreateIndexRequest request = new CreateIndexRequest("ivan");
    request.settings(Settings.builder().put("index.number_of_shards", 3) // 分片数
            .put("index.number_of_replicas", 2) // 副本数
            .put("analysis.analyzer.default.tokenizer", "ik_smart") // 默认分词器
    );
    CreateIndexResponse re = client.indices().create(request, RequestOptions.DEFAULT);
    System.out.println("索引创建成功了吗：" + re.isAcknowledged());

    //再创建文档映射
    XContentBuilder builder = XContentFactory.jsonBuilder()
            .startObject()
            .startObject("properties")
            /*cid字段*/
            .startObject("cid")
            .field("type", "long")
            .field("store", true)
            .endObject()

            /*title字段*/
            .startObject("title")
            //字段类型
            .field("type", "text")
            //是否存储
            .field("store", true)
            //插入时分词
            .field("analyzer", "ik_smart")
            //搜索时分词
            .field("search_analyzer", "ik_max_word")
            .endObject()

            /*contents 字段*/
            .startObject("contents")
            .field("type", "text")
            .field("store", true)
            .field("analyzer", "ik_smart")
            .field("search_analyzer", "ik_max_word")
            .endObject()

            //sortCode字段
            .startObject("sortCode")
            .field("type", "long")
            .field("store", true)
            .endObject()

            /*searchName 字段*/
            .startObject("searchName")
            .field("type", "text")
            .field("store", true)
            .field("analyzer", "ik_smart")
            .field("search_analyzer", "ik_max_word")
            .endObject()

            .endObject()
            .endObject();
    AcknowledgedResponse response = client.indices().putMapping(new PutMappingRequest("ivan").source(builder), RequestOptions.DEFAULT);
    System.out.println("文档映射创建成功了吗：" + response.isAcknowledged());
}
```

## 创建索引-添加Mapping-添加文档-查询-高亮

```java

import cn.hutool.core.util.StrUtil;
import cn.hutool.json.JSONUtil;
import com.ivan.search.mapper.CprMainMapper;
import com.ivan.search.vo.Content;
import com.ivan.search.vo.CprVo;
import com.ivan.search.vo.SearchOne;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.PutMappingRequest;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.common.xcontent.XContentBuilder;
import org.elasticsearch.common.xcontent.XContentFactory;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * 创建正式的全文检索索引
 *
 * @author cuiyingfan
 * @date 2022/03/11
 */
@SpringBootTest
public class FullTextSearchIndex {
    @Autowired
    private RestHighLevelClient client;

    @Resource
    private CprMainMapper cprMainMapper;

    /**
     * 创建索引映射
     * 自定义创建索引
     *
     * @throws IOException ioexception
     */
    @Test
    public void createIndexAndMapping() throws IOException {
        //先创建索引
        CreateIndexRequest request = new CreateIndexRequest("ivan");
        request.settings(Settings.builder().put("index.number_of_shards", 3) // 分片数
                .put("index.number_of_replicas", 2) // 副本数
                .put("analysis.analyzer.default.tokenizer", "ik_smart") // 默认分词器
        );
        CreateIndexResponse re = client.indices().create(request, RequestOptions.DEFAULT);
        System.out.println("索引创建成功了吗：" + re.isAcknowledged());

        //再创建文档映射
        XContentBuilder builder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("properties")
                /*cid字段*/
                .startObject("cid")
                .field("type", "long")
                .field("store", true)
                .endObject()

                /*title字段*/
                .startObject("title")
                //字段类型
                .field("type", "text")
                //是否存储
                .field("store", true)
                //插入时分词
                .field("analyzer", "ik_smart")
                //搜索时分词
                .field("search_analyzer", "ik_max_word")
                .endObject()

                /*contents 字段*/
                .startObject("contents")
                .field("type", "text")
                .field("store", true)
                .field("analyzer", "ik_smart")
                .field("search_analyzer", "ik_max_word")
                .endObject()

                //sortCode字段
                .startObject("sortCode")
                .field("type", "long")
                .field("store", true)
                .endObject()

                /*searchName 字段*/
                .startObject("searchName")
                .field("type", "text")
                .field("store", true)
                .field("analyzer", "ik_smart")
                .field("search_analyzer", "ik_max_word")
                .endObject()

                .endObject()
                .endObject();
        AcknowledgedResponse response = client.indices().putMapping(new PutMappingRequest("ivan").source(builder), RequestOptions.DEFAULT);
        System.out.println("文档映射创建成功了吗：" + response.isAcknowledged());
    }

    /**
     * 添加文档
     */
    @Test
    public void addDoc() throws IOException {
        //查询出来是一个1对多的结构
        List<CprVo> drugs = cprMainMapper.getDrugs();
        //将它转换为我们需要的结构
        ArrayList<SearchOne> list = new ArrayList<>();
        for (CprVo drug : drugs) {
            SearchOne searchOne = new SearchOne(drug.getCid(), drug.getTitle(), drug.getSortCode(), drug.getSearchName());
            StringBuilder builder = new StringBuilder();
            for (Content content : drug.getContents()) {
                builder.append(content.getSubtitle() + content.getContent());
            }
            searchOne.setContents(builder.toString());
            list.add(searchOne);
        }
        BulkRequest request = new BulkRequest();
        for (SearchOne one : list) {
            request.add(new IndexRequest("ivan").id(one.getCid() + "")
                    .source(JSONUtil.toJsonStr(one), XContentType.JSON));
        }
        BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
        System.out.println(responses.status());
        System.out.println(responses);
    }

    /**
     * 完整测试搜索
     */
    @Test
    public void FullTestSearch() throws IOException {
        String index = "ivan";
        String keyword = "柴胡";
        // 搜索请求
        SearchRequest searchRequest;
        if (StrUtil.isEmpty(index)) {
            searchRequest = new SearchRequest();
        } else {
            searchRequest = new SearchRequest(index);
        }
        // 条件构造
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        // 第几页
        searchSourceBuilder.from(0);
        // 每页多少条数据
        searchSourceBuilder.size(1000);
        // 配置高亮
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("title").field("contents");
        highlightBuilder.preTags("<span style='color:red'>");
        highlightBuilder.postTags("</span>");
        searchSourceBuilder.highlighter(highlightBuilder);
        // 精确查询
//        QueryBuilders.termQuery();
        // 匹配所有
//        QueryBuilders.matchAllQuery();
        // 最细粒度划分：ik_max_word，最粗粒度划分：ik_smart
        searchSourceBuilder.query(QueryBuilders.multiMatchQuery(keyword, "title", "contents").analyzer("ik_max_word"));
//        searchSourceBuilder.query(QueryBuilders.matchQuery("content", keyWord));
        searchSourceBuilder.timeout(TimeValue.timeValueSeconds(10));

        searchRequest.source(searchSourceBuilder);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        List<Map<String, Object>> results = new ArrayList<>();
        for (SearchHit searchHit : searchResponse.getHits().getHits()) {
            Map<String, HighlightField> highlightFieldMap = searchHit.getHighlightFields();
            HighlightField title = highlightFieldMap.get("title");
            HighlightField description = highlightFieldMap.get("contents");
            // 原来的结果
            Map<String, Object> sourceMap = searchHit.getSourceAsMap();
            // 解析高亮字段，替换掉原来的字段
            if (title != null) {
                Text[] fragments = title.getFragments();
                StringBuilder n_title = new StringBuilder();
                for (Text text : fragments) {
                    n_title.append(text);
                }
                sourceMap.put("title", n_title.toString());
            }
            if (description != null) {
                Text[] fragments = description.getFragments();
                StringBuilder n_description = new StringBuilder();
                for (Text text : fragments) {
                    n_description.append(text);
                }
                sourceMap.put("contents", n_description.toString());
            }
            results.add(sourceMap);
        }
        System.out.println(results);
    }


}
```
