#学习如何使用node.js 爬去信息
## 说到爬虫python是最适合的 但是为了学习node cheerio。

```
function getPageNum(callback){
var options = {
host: host,
port: 80,
path: path,
headers: {
'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.117 Safari/537.36'
}
};
var html = ""
http.get(options,function (res) {
res.on('data', function (data) {
html += data;
});
res.on('end', function () {
var $ = cheerio.load(html);
var pagenum=$('.pager').find('a').length;
var pager=$('.pager').find('a').eq(pagenum-2).text();
callback(undefined,pager);
});
}).on('error', function (e) {
callback(e);
})
}
```

```
var $ = cheerio.load(html);
var pagenum=$('.pager').find('a').length;
var pager=$('.pager').find('a').eq(pagenum-2).text();
```
![hahha](http://myhexo.qiniudn.com/yema.png)

```
function gettitle(path2,callback){
     var options = {
        host: host,
        port: 80,
        path: path+path2,
        headers: {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/33.0.1750.117 Safari/537.36'
        }
    };
    var html = ""
    http.get(options,function (res) {
        res.on('data', function (data) {
            html += data;
        });
        res.on('end', function () {
            var $ = cheerio.load(html);
             $('.main').find('table').find('tr').each(function(i,ele){
             	title.push($(this).find('h1').find('a').text().trim());
             });
             callback()
 
        });
    }).on('error', function (e) {
    	callback(e);
        })
}
```

