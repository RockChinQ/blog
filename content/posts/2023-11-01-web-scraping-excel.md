---
title: "爬虫 | 爬取新闻站的内容并存到 Excel 中"
date: 2023-11-01
draft: false
---

\\n

代码开源在：<https://gitee.com/RockChin/SampleCodes/tree/main/2023/11/NewsPageCrawler>

\\n\\n\\n\\n

### 直接测试 {#直接测试 .wp-block-heading}

\\n\\n\\n\\n

安装依赖：

\\n\\n\\n\\n

``` wp-block-code
pip install pandas openpyxl requests fake-useragent beautifulsoup4
```

\\n\\n\\n\\n

将上方的链接中的 crawler.py 的内容保存到本地文件，执行：

\\n\\n\\n\\n

``` wp-block-code
python crawler.py
```

\\n\\n\\n\\n

程序将爬取 https://news.tjut.edu.cn/yw1.htm 上每条新闻的
标题、发布日期、每张图片链接 并保存到 news.xlsx

\\n\\n\\n\\n

### 实现原理 {#实现原理 .wp-block-heading}

\\n\\n\\n\\n

首先确定目标网站 https://news.tjut.edu.cn/yw1.htm ，
到页面上右键"检查元素"

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/image.png){.wp-image-160
style="aspect-ratio:0.7890625;width:265px;height:auto"}

\\n\\n\\n\\n

挨个展开，选中包含了新闻标题的表格元素：

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/image-1.png)

\\n\\n\\n\\n

右键 "复制selector"

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/%E6%88%AA%E5%B1%8F2023-11-01-20.26.27.png)

\\n\\n\\n\\n

就是这个：main \> div.page2.box \> div.w720.fl \> table

\\n\\n\\n\\n

现在开始写代码：

\\n\\n\\n\\n

``` wp-block-code
# 引入re模块 正则表达式提取时间\nimport re\n\n# 引入requests用于发送http请求 下载网页源码\nimport requests\n\n# 引入fake_useragent用于生成随机User-Agent 以实现反反爬\nfrom fake_useragent import UserAgent\n\n# 引入BeautifulSoup用于解析网页源码\nfrom bs4 import BeautifulSoup\n\n# 引入pandas用于处理并保存数据到Excel\nimport pandas as pd\n\n# 此爬虫爬取天津理工大学的某个新闻板块内容：https://news.tjut.edu.cn/yw1.htm\n# 并逐一爬取每个新闻详情页中的标题、时间、配图\n# 最后将数据保存到Excel中\nmain_page_url = 'https://news.tjut.edu.cn/yw1.htm'\n\nheaders = {\n    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",\n    "Accept-Encoding":"gzip, deflate, br",\n    "Accept-Language":"zh-CN,zh;q=0.9",\n    "User-Agent": UserAgent().random\n}\n\n# 发送http请求，获取网页源码\nresponse = requests.get(main_page_url, headers=headers, timeout=10)\n\nresponse.encoding = "utf-8"\n\n# 解析网页源码\nmain_page_soup = BeautifulSoup(response.text, 'html.parser')
```

\\n\\n\\n\\n

这里直接使用 requests
来发送请求，并且带上了一些请求头，这些不重要，只有一些反爬策略做得很严格的网站才会去校验这种东西。

\\n\\n\\n\\n

然后拿到了网页的源码（HTML），最后放到BeautifulSoup4里进行解析。

\\n\\n\\n\\n

接下来就按照前面获得的元素的路径来获取元素对象：

\\n\\n\\n\\n

``` wp-block-code
# 获取新闻列表元素\nmain_page_news_list = main_page_soup.select_one("#main > div.page2.box > div.w720.fl > table")\n\n# 提取所有tr元素\ntrs = main_page_news_list.select("tr")
```

\\n\\n\\n\\n

在网页中继续展开刚刚的table元素，也可以看到里面有很多tr，tr下包含了新闻的标题和详细页链接。

\\n\\n\\n\\n

而其中只有id为line开头的才是真正的新闻链接，其他的是分割线：

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/image-2.png)

\\n\\n\\n\\n

所以写个筛选：

\\n\\n\\n\\n

``` wp-block-code
\n# 删除分割线, 只保留id为line开头的tr元素, 这些才是新闻标题\nfiltered_trs = []\n\nfor tr in trs:\n    if tr is not None and tr.get("id") is not None and tr.get("id").startswith("line"):\n        filtered_trs.append(tr)\n\nprint("新闻条数：", len(filtered_trs))
```

\\n\\n\\n\\n

接下来就可以遍历每个新闻详细页然后获取所需信息（标题、时间、配图链接们）

\\n\\n\\n\\n

``` wp-block-code
\n# 爬取每个新闻详情页的标题、时间、配图\nnews_list = []\n"""\n数据结构:\n[\n    {\n        "title": "xxx",\n        "time": "xxx",\n        "images: [\n            "xxx",\n            "xxx"\n        ]\n    }\n]\n"""\n\nroot_url = "https://news.tjut.edu.cn/"\n\nfor tr in filtered_trs:\n    # 这里就是处理每个新闻详细页，下方讲解
```

\\n\\n\\n\\n

可以观察到，新闻列表中tr的第二个td元素内为有效的链接元素：

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/%E6%88%AA%E5%B1%8F2023-11-01-20.35.02.png)

\\n\\n\\n\\n

取出来，而新闻详情页链接就是https://news.tjut.edu.cn/拼接上这个a元素的href值，用requests下载页面源代码：

\\n\\n\\n\\n

``` wp-block-code
\n    # 获取tr内所有td元素\n    tds = tr.select("td")\n\n    if tds is None or len(tds) < 4:\n        continue\n\n    # 获取新闻详情页url\n    detail_url = root_url + tds[1].select_one("a").get("href")\n\n    # print(detail_url)\n\n    # 发送http请求，获取网页源码\n    response = requests.get(detail_url, headers=headers, timeout=10)\n\n    response.encoding = "utf-8"\n\n    # 解析网页源码\n    detail_page_soup = BeautifulSoup(response.text, 'html.parser')
```

\\n\\n\\n\\n

接下来就依葫芦画瓢了，前往新闻详情页，找到包含标题、时间的元素，这里使用find函数：查找class为指定值的div元素，因为整个html中只有这一个div元素具有这个class名称，所以可以直接找到需要的：

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/%E6%88%AA%E5%B1%8F2023-11-01-20.39.10.png)

\\n\\n\\n\\n

``` wp-block-code
\n    # 获取新闻标题\n    # 元素后面.text即可获取其中的纯文本\n    title = detail_page_soup.find("div", class_="main_contit").select_one("h1").text\n\n    # 获取新闻时间\n    sub_title = detail_page_soup.find("div", class_="main_contit").select_one("div").text\n    time_str = re.findall(r"\\d{4}-\\d{2}-\\d{2}", sub_title)[0]
```

\\n\\n\\n\\n

新闻时间，我们使用了一个正则表达式来提取，正则表达式教程可以看看：<https://www.runoob.com/regexp/regexp-tutorial.html>

\\n\\n\\n\\n

新闻的所有配图，就在正文这个class=\"main_conDiv\"的元素中查找所有的img元素即可：

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/image-3.png)

\\n\\n\\n\\n

``` wp-block-code
\n    # 获取新闻配图\n    imgs_elements = detail_page_soup.find("div", class_="main_conDiv").select("img")\n\n    imgs = []\n    \n    # 同样是https://news.tjut.edu.cn/拼接上图片的src属性就是图片的访问链接\n    for img_element in imgs_elements:\n        imgs.append(root_url+img_element.get("src"))
```

\\n\\n\\n\\n

存到news_list列表中：

\\n\\n\\n\\n

``` wp-block-code
\n    news_list.append({\n        "title": title,\n        "time": time_str,\n        "images": imgs\n    })
```

\\n\\n\\n\\n

现在我们取得了所有新闻的标题、时间和配图列表，跳出循环了，下一步是保存到excel

\\n\\n\\n\\n

直接使用pandas的导出到excel到功能，把每一个新闻都包装成一个DataFrame，拼接到整个DataFrame中，最后导出到news.xlsx

\\n\\n\\n\\n

``` wp-block-code
\n# 初始化pandas的DataFrame\ndf = pd.DataFrame(columns=['新闻标题', '发布日期'])\n\n# 遍历每一条新闻数据\nfor new in news_list:\n    data = {\n        '新闻标题': [new['title']],\n        '发布日期': [new['time']]\n    }\n\n    # 插入每一个图片链接\n    for i in range(len(new['images'])):\n        data['图片链接{}'.format(i+1)] = [new['images'][i]]\n\n    # 将每一条新闻数据转换为DataFrame\n    df = pd.concat([df, pd.DataFrame(data)])\n\n# 保存数据到Excel\ndf.to_excel('news.xlsx', index=False)
```

\\n\\n\\n\\n

![](https://pub-c7c365991905453481dc9f8834619665.r2.dev/blog/wp-uploads/2023/11/%E6%88%AA%E5%B1%8F2023-11-01-20.46.15.png)

\\n
