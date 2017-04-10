# 小卖铺 Catalog
有一百多个product吧，货真价实，有图有真相，有条形码，用来测试和demo还是可以的。

### 怎么来的
其实淘宝有卖的，动不动就是几十万的数据，太大，太重了。
首先从淘宝卖家获取获取一些测试图片(当然也有一些测试数据，但大多数卖家只能提供图与数据不匹配的测试数据，毕竟没有没有买人家的)。
由于条码数据太混乱了，很多东西，根本就没图没真相，筛选一些完整的数据，不是很容易。
api.jisuapi.com 这个网站有一些免费的barcode查询API可以试用。从上面抓了一下数据下来。


### Step1
从图片获取barcode，病从remote AIP 获取商品详情。商品详情以json文件的形式存到本地data目录（把xxxxx成你自己的apikey，可以到api.jisuapi.com免费申请）
```shell
$ find . -name "*.jpg" | awk -F '\/|\\.' '{print "curl -s -o ./data/" $5 ".json \"http://api.jisuapi.com/barcode2/query?appkey=xxxxx&barcode=" $5 "\"" }' > 
$ batch_fetch.sh
$ sh batch_fetch.sh
```

### Step2
由于一些product可能在remote api中也不存在，所以需要移除 status 为201的json数据
```shell
$ find ./data -name "*.json" | awk '{print "bar_status=$(cat " $1 " | jq  -r \".status\") && [ \"$bar_status\" = \"210\" ] && rm -rf " $1 " && echo \"" $1 "\" >> remove_list.txt"}' >batch_clean_json.sh
$ sh batch_clean_json.sh
```
##Step3 
同时清理status 为201的本地图片
```shell
$ cat remove_list.txt | awk -F '\/|\\.' '{print "find ./pic -name \"" $4 ".jpg\">> remove_list_pic.txt"}'
$ cat remove_list_pic.txt | awk '{print "rm -rf "$1}' >batch_clean_pic.sh
$ sh batch_clean_pic.sh
```
### Step4
从图片获取，获取本地的图片信息和barcode
```shell
$ find . -name "*.jpg" | awk -F '\/|\\.' '{print $5 ", " $4 ", "$0}' > catalog.csv
```
