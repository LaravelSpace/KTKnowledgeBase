```json

```

---

## 构建商品搜索系统

搜索背后的实现，可以非常简单，简单就用一个SQL，LIKE一下就能实现；也可以很复杂，复杂到需要一个大型公司数千人一起合作去完成。但是二者的区别也仅仅是，搜索速度的快慢以及搜出来的内容好坏而已。这里，我们借助ES(Elasticsearch)来快速、低成本地构建一个体验还不错的搜索系统。

### 倒排索引机制

搜索的核心需求是全文匹配，对于全文匹配，数据库的索引是根本派不上用场的，那只能全表扫描。全表扫描已经非常慢了，这还不算，还需要在每条记录上做全文匹配，也就是一个字一个字的比对，这个速度就更慢了。所以，使用数据来做搜索，性能上完全没法满足要求。

假设我们有这样两个商品：

![](E:\Workspace\KTKnowledgeBase\Image\GeekBang\HouDuanCunChu\SouSuoXiTong_img01.jpg)

为了能够支持快速地全文搜索，ES中对于文本采用了一种特殊的索引：倒排索引（InvertedIndex）。当我们往ES写入商品记录的时候，ES会先对需要搜索的字段，也就是商品标题进行分词。分词就是把一段连续的文本按照语义拆分成多个单词。然后ES按照单词来给商品记录做索引，就形成了上面那个表一样的倒排索引。

![](E:\Workspace\KTKnowledgeBase\Image\GeekBang\HouDuanCunChu\SouSuoXiTong_img02.jpg)

当我们搜索关键字苹果手机的时候，ES会对关键字也进行分词，苹果手机被分为苹果和手机。然后，在倒排索引中去搜索每个关键字分词，搜索结果是：苹果=666,888；手机=888。

注意，**整个搜索过程中，我们没有做过任何文本的模糊匹配**。ES的存储引擎存储倒排索引时，肯定不是像我们上面表格中展示那样存成一个二维表，实际上它的物理存储结构和MySQL的InnoDB的索引是差不多的，都是一颗查找树。

### 如何在ES中构建商品的索引

虽然ES是为搜索而生的，但本质上，它仍然是一个存储系统。在ES里面，数据的逻辑结构类似于MongoDB，每条数据称为一个DOCUMENT，简称DOC。DOC就是一个JSON对象，DOC中的每个JSON字段，在ES中称为FIELD，把一组具有相同字段的DOC存放在一起，存放它们的逻辑容器叫INDEX，这些DOC的JSON结构称为MAPPING。这里面最不好理解的就是这个INDEX，它实际上类似于MySQL中表的概念，而不是我们通常理解的用于查找数据的索引。

ES是一个用Java开发的服务端程序，除了Java以外就没有什么外部依赖了，首先参考官方文档安装ES，然后还需要安装中文文本分词插件，安装完成后，需要重启ES。你可以使用下面的命令安装分词插件。

```shell
$ elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.0/elasticsearch-analysis-ik-7.6.0.zip
```

为了能实现商品搜索，我们需要先把商品信息存放到ES中，首先我们先定义存放在ES中商品的数据结构，也就是MAPPING。我们这个MAPPING只要两个字段就够了，sku_id就是商品ID，title保存商品的标题，当用户在搜索商品的时候，我们在ES中来匹配商品标题，返回符合条件商品的sku_id列表。

接下来我们使用上面这个MAPPING创建INDEX，类似于MySQL中创建一个表。

```
curl -X PUT "localhost:9200/sku" -H 'Content-Type: application/json' -d '{
        "mappings": {
                "properties": {
                        "sku_id": {
                                "type": "long"
                        },
                        "title": {
                                "type": "text",
                                "analyzer": "ik_max_word",
                                "search_analyzer": "ik_max_word"
                        }
                }
        }
}'
{"acknowledged":true,"shards_acknowledged":true,"index":"sku"}
```

这里面，使用PUT方法创建一个INDEX，INDEX的名称是“sku”，直接写在请求的URL中。请求的BODY是一个JSON对象，内容就是我们上面定义的MAPPING，也就是数据结构。这里面需要注意一下，由于我们要在title这个字段上进行全文搜索，所以我们把数据类型定义为text，并指定使用我们刚刚安装的中文分词插件IK作为这个字段的分词器。

创建好INDEX之后，就可以往INDEX中写入商品数据，插入数据需要使用HTTPPOST方法。然后就可以直接进行商品搜索了，搜索使用HTTPGET方法。

### 搜索框的搜索提示功能

我们在电商的搜索框中搜索商品时，它都有一个搜索提示的功能，比如我输入苹果还没有点击搜索按钮的时候，搜索框下面会提示苹果手机、苹果电脑，这个功能也可以用ES实现。

首先用户每输入一个字都可能会发请求查询搜索框中的搜索推荐。所以搜索推荐的请求量远高于搜索框中的搜索。ES针对这种情况提供了suggestion api，并提供的专门的数据结构应对搜索推荐，性能高于match，但它应用起来也有局限性，就是只能做前缀匹配。再结合pinyin分词器可以做到输入拼音字母就提示中文。

如果想做非前缀匹配，可以考虑Ngram。不过Ngram有些复杂，需要开发者自定义分析器。比如有个网址`www.geekbang.com`，用户可能记不清具体网址了，只记得网址中有2个e，此时用户输入ee两个字母也是可以在搜索框提示出这个网址的。