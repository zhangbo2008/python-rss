# RSSHub

> 🍰 万物皆可 RSS

RSSHub 是一个轻量、易于扩展的 RSS 生成器，可以给任何奇奇怪怪的内容生成 RSS 订阅源

本项目是[原RSSHub](https://github.com/DIYgod/RSSHub)的Python实现。


**其实用Python写爬虫要比JS更方便:p**

DEMO地址：https://pyrsshub.vercel.app


## 交流

Discord Server： [https://discord.gg/4BZBZuyx7p](https://discord.gg/4BZBZuyx7p)

## RSS过滤

你可以通过以下查询字符串来过滤RSS的内容：

- include_title: 搜索标题
- include_description: 搜索描述
- exclude_title: 排除标题
- exclude_description: 排除描述
- limit: 限制条数

## 贡献 RSS 方法

1. fork这份仓库
2. 在spiders文件夹下创建新的爬虫目录和脚本，编写爬虫，参考我的[爬虫教程](https://juejin.cn/post/6953881777756700709)
3. 在blueprints的main.py中添加对应的路由（按照之前路由的格式）
4. 在templates中的main目录下的feeds.html上写上说明文档，同样可参照格式写
5. 提pr

## 部署

### 本地测试

首先确保安装了[pipenv](https://github.com/pypa/pipenv)

``` bash
git clone https://github.com/alphardex/RSSHub-python
cd RSSHub-python
pipenv install --dev
pipenv shell
flask run
```

### 生产环境

``` bash
gunicorn main:app -b 0.0.0.0:5000
```

### 部署到 deta.dev

[![Deploy](https://button.deta.dev/1/svg)](https://go.deta.dev/deploy?repo=https://github.com/hillerliao/rsshub-python)

或  

安装 [Deta CLI](https://docs.deta.sh/docs/cli/install/)；  
在终端运行`deta login`；
在项目根目录运行`deta new --python pyrsshub`；  
将 `pyrsshub` 目录下的 `.deta` 文件夹移到根目录；
运行`deta deploy`；
获取网址 `https://<micro_name>.deta.dev/`；
更新`deta update`

### 部署到 Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fhillerliao%2Frsshub-python)

### Docker 部署

制作镜像文件 `docker build -t pyrsshub:latest .`

创建docker容器 `docker run -dit --name pyrsshub -p 8080:80 pyrsshub:latest`






2024-12-10,22点14

整体架构:

ios---->rss---->数据库.(mongodb)



小鱼的架构:
  a用户订阅了 a1,a2,c 频道(频道就是小林说视频, 峰哥带你勇闯天涯频道)
  b用户订阅了 b1,b2,c 频道




  爬虫爬a1,a2,b1,b2,c. 增量更新入库,  然后再分发给a,b的通知.


  增量更新: 比如保存a1channel : 从2020年到今天的所有视频.
  用户搜索时候,跟他最后登录时间比较.把时间区域的发给他.

follow-rss:
  无后端, 无数据库.
  没有订阅功能.
  每次用户登录的时候, 现爬虫. 爬完信息直接给用户. 都不保存.
  不就是,不及时. 峰哥发了视频, 你只能登录时候才进行爬虫,才知道更新了.

收费:
  普通用户只能订阅30个频道.
  付费的订阅50个.

# python-rss


# 2024-12-12,20点35 理解代码 蓝图:
@bp.route('/aisixiang/search/<string:category>/<string:keywords>')
def aisixiang_search(category='', keywords=''):
    from rsshub.spiders.aisixiang.search import ctx
    return render_template('main/atom.xml', **filter_content(ctx(category, keywords)))



核心是ctx函数:

from urllib.parse import quote, unquote
from rsshub.utils import fetch, DEFAULT_HEADERS


domain = 'https://www.aisixiang.com'


def parse(post):
    item = {}
    item['description'] = item['title'] = post.css('a::text').getall()[-1]
    item['link'] = f"{domain}{post.css('a::attr(href)').getall()[-1]}"
    item['pubDate'] = post.css('span::text').extract_first()
    return item


def ctx(category='', keywords=''):
    keywords = unquote(keywords,encoding='utf-8')   # 加减引号
    keywords_gbk = quote(keywords, encoding='gbk')
    url = f"{domain}/data/search.php?keyWords={keywords_gbk}&searchfield={category}"   # 拼成url
    tree = fetch(url, headers=DEFAULT_HEADERS) # fetch来请求网页
    posts = tree.css('.search_list').css('li')
    return {
        'title': f'{keywords} - {category}搜索 - 爱思想',
        'link': url,
        'description': f'{keywords} - {category}搜索 - 爱思想',
        'author': 'hillerliao',
        'items': list(map(parse, posts))
    }



# 上述代码的debug流程:

首先浏览器get 这个地址/aisixiang/search/<string:category>/<string:keywords>
通过地址传入两个参数category和keywords
然后走ctx(category, keywords) 函数.









