# edge扩展开发记录




本文介绍了个人制作edge浏览器扩展"my new tab"的过程，主要讲述edge浏览器扩展开发的基本逻辑。

<!--more-->

## 1、为什么要自己做一个？
我的需求其实可以在市面上的一些自定义新标签页的扩展满足了，我使用这些扩展基于四个需求，

### 1.1、 方便切换搜索引擎的搜索工具，
这点几乎是原生新标签页所做不到的,目前的浏览器大都只支持一种搜索引擎，切换也需要跑到设置页面进行一同修改，实为不便。
### 1.2、 提供每日热搜的标题及对应的跳转链接。
那么，第二点并不是想象中那样好满足，目前某新标签页内提供的b站热门其实是每日热门，而不是综合热门，每日热门是可以轻松通过买量上去的，例证就是米哈游的pv、op、ep等视频在刚发布当天就可以上每日热门，但是之后很难上到综合热门。这种信息茧房让我感到不适。
### 1.3、 保护隐私
这很好理解，用户端根本无法确知自己的搜索记录是否会被记录，当然，经过本人本次项目的开发及审核经验，本人认为新标签页有较低概率收集用户隐私。当然，如果是图标隐私的话，就不好说了，因为图标信息、壁纸是存在服务器的，所以可以实现用户信息同步。泄露图标信息倒是没啥，但是每次打开浏览器都要加载并且尝试从网站获取图标和壁纸确实对网速和后台造成了负荷，这是本人所无法接受的。
### 1.4、 定制化
个人认为别人做的标签页为了迎合大众多少都会加入许多我用不到的模块，这些模块同样让人生厌。而且整体的美观度也无法根据我自己的想法随便改。

综上我的需求可以归结为搜索、热门、导航。

## 2、edge浏览器扩展的源码组织
edge跟chrome同源，所以扩展函数什么的也是通用的，这里展示一下一个扩展的基本结构：
```
- icons/
  - icon64.png
- newpage/
  - style.css
  - index.html
  - script.js
- background.js
- manifest.json
```
如上所展示的，扩展的主要文件由四个文件构成，`manifest.json、background.js、index.html、script.js`,其中`background.js`并不是必要的。
### 2.1、manifest文件
manifest文件主要用于声明扩展的作者版本号等信息，除开一些无关紧要的信息，manifest还声明了重要的安全隐私相关的权限信息，比如跨域访问、允许访问的域名、闹钟事件等，本次开发使用到的热搜显示模块就使用到了以上三个功能，这里说明一下。至于index.html，是扩展的图形化操作界面，有一些扩展再点击后会弹出一个小窗口，这个小窗口也是使用html编写的，一般叫popup.html。本项目只需要本地编写一个页面然后重定向为主页即可。此外，浏览器的CORS禁止当前网页的脚本跨域访问，但是如果实现在manifest文件里面声明需要访问的网站就可以轻松跨域，这算是扩展的优势，因为普通的脚本即使临时禁用CORS也能fetch，但是无法读取获得的内容。顺带一提，比起python爬虫获取json文件，js脚本的运行速度要快上不少，个人认为可能是浏览器的环境在浏览器启动后就做好了，但是python脚本却需要先启用雨滴桌面，然后桌面唤起vbs脚本，然后vbs脚本建立终端窗口，在终端窗口在建立python环境，这个流程之长确实会影响爬虫的效率，更何况python这种语言也不是什么高效率的语言，在python脚本里面还要import对应的库，确实不如直接js脚本的fetch方法来的快。

此外，为了避免fetch时把自己的cookie等信息一块发出去，在fetch使用以下命令
```
fetch('https://example.com/data', {
  method: 'GET',
  credentials: 'omit'  // 不发送 cookies 和 HTTP 认证信息
})
```

### 2.2、html文件及对应的js脚本
关于前端的基础知识这里没办法完整且详尽地记录，这里只写几个遇到的问题，问题基本都是因为js脚本是运行在浏览器扩展的环境下面，所以寻常网页脚本可以做的事在这里并不能完全自如的做出来，比如爬虫遇到gbk编码返回的信息，js脚本就无法安装库来解析，或者说搞起来很麻烦。（至少目前我还没整明白，我看别人都是扔到后端去做，但是一个完全本地运行的扩展如果为了解析gbk字符串就要用后端的话属实违背我的本意了）

还有就是在爬虫的使用上，我是想使用background脚本先获取内容，解析为json之后存在浏览器的存储里面（最大大小支持5M，在本项目的使用场景里面足够了），遇到的问题是，js脚本无法再浏览器环境下使用dompraser
```
const url = 'https://www.52pojie.cn/forum.php?mod=guide&view=hot';

async function fetchData() {
  try {
    const response = await fetch(url);
    const text = await response.text();

    // 解析HTML
    const parser = new DOMParser();
    const doc = parser.parseFromString(text, 'text/html');

    // 提取数据
    const items = Array.from(doc.querySelectorAll('.commonbox .subject')).map(element => {
      const title = element.querySelector('a')?.textContent.trim();
      const summary = element.querySelector('.excerpt')?.textContent.trim() || 'No summary available';
      return { title, summary };
    });

    // 保存到本地存储
    chrome.storage.local.set({ data: items });

  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

// 每天更新一次数据
const updateInterval = 24 * 60 * 60 * 1000; // 24小时
fetchData();
setInterval(fetchData, updateInterval);

```

所以不得不采用字符串匹配的方法：
```
const url = 'https://www.52pojie.cn/forum.php?mod=guide&view=hot';

async function fetchData() {
  try {
    const response = await fetch(url);
    const text = await response.text();

    // 使用正则表达式解析数据
    const items = [];
    const regex = /<a href=".*?" class="s xst">(.*?)<\/a>.*?<div class="excerpt">(.*?)<\/div>/gs;
    let match;

    while ((match = regex.exec(text)) !== null) {
      const title = match[1].trim();
      const summary = match[2].trim();
      items.push({ title, summary });
    }

    // 保存到本地存储
    chrome.storage.local.set({ data: items });

  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

// 每天更新一次数据
const updateInterval = 24 * 60 * 60 * 1000; // 24小时
fetchData();
setInterval(fetchData, updateInterval);

```

### 2.3、扩展的更新逻辑
扩展所有的图标、壁纸全都存在本地，毫无疑问可以加快反应速度，在断网的状态也能保持优雅，但是对应断网状态下，热搜的刷新逻辑就成了问题，具体来说，有两个问题，

1、定时刷新的话，错过闹钟时间的话是不是就不刷新了？

2、在断网状态下扩展在后台疯狂请求刷新怎么办？

问题1的解法是使用chrome的alarm，只要在manifest声明该权限后就可以设置闹钟，平时background脚本沉睡，等到时间就会起来干活，如果错过闹钟，那么下次启动的时候就会自动执行

问题2的解法是加入刷新按键，html通过按键触发网页的js脚本，网页js脚本会在后台与background脚本通信进而触发刷新

目前的问题在于如果刚打开浏览器的时候没网，过一会网络连上了，那么就要手动刷新了，这是比较麻烦的，但是相应的，这个脚本基本每天只会执行一次，具有极高的效率，相比网上其他人做的脚本可以说更适合本人使用了。

最后，展示一下成品
![成品界面](https://pic.imgdb.cn/item/66d07081d9c307b7e930f5a4.png)

## 3、之后的发展方向

1、界面美化

目前的界面还是太丑了，最丑的就是搜索栏、左一块右一块，跟edge浏览器的新界面一样丑，两个泡泡连着，就像烫破的伤口

2、加入更多的订阅源

52破解的gbk编码太奇葩了，对于不支持gbk处理的扩展js脚本简直天克，之后看看有没有别的中转源进行爬虫。实在不行看有没有别的替代源也行。不过我不觉得我有太多精力在满足需求后还继续完善下去也就是了。
