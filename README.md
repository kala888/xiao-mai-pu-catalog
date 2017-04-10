#小卖铺 Catalog
有一百多个product吧，一些真实杂货，有图有真相，有条形码，用来测试。

#怎么来的
从淘宝卖家获取获取测试图片(当然也有一些测试数据，但是图与数据不匹配，毕竟没有没有买人家的，有些不厚道)。 
由于条码数据太混乱了，很多东西，根本就没图没真相，筛选一些完整的数据，不是很容易。
api.jisuapi.com 这个网站有一些免费的barcode查询API可以试用。从上面抓了一下数据下来。

##Step1 
从图片获取，获取图片信息和barcode
find . -name "*.jpg" | awk -F '\/|\\.' '{print $5 ", " $4 ", "$0}' > catalog.csv

##Step2
根据barcode 获取 商品信息，存到本地data目录，吧xxxxx成你自己的apikey，请到api.jisuapi.com免费申请
find . -name "*.jpg" | awk -F '\/|\\.' '{print "curl -s -o ./data/" $5 ".json \"http://api.jisuapi.com/barcode2/query?appkey=xxxxx&barcode=" $5 "\"" }' > batch_fetch.sh
sh batch_fetch.sh

##Step3 移除 status 为201的json数据
find ./data -name "*.json" | awk '{print "bar_status=$(cat " $1 " | jq  -r \".status\") && [ \"$bar_status\" = \"210\" ] && rm -rf " $1 " && echo \"" $1 "\" >> remove_list.txt"}' >batch_clean_json.sh
sh batch_clean_json.sh

##Step3 移除 status 为201的json数据
cat remove_list.txt | awk -F '\/|\\.' '{print "find ./pic -name \"" $4 ".jpg\">> remove_list_pic.txt"}' >remove.sh
cat remove_list_pic.txt | awk '{print "rm -rf "$1}' >remove_list.sh
remove_list.sh

