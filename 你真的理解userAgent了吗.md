> 最近在做数据分析，发现navigator.useAgent字段包含了很多奇怪的信息，包括每个浏览器都有Mozilla/5.0字段，而且既有Chrome又有Safari，原因是什么？所以进行了一下分析，在这里进行下分享～
>

目录

[1.userAgent为什么有这么多相似的字段](#title1)

[2.各大浏览器userAgent解析](#title2)

[3.最全userAgent代码判断附上](#title3)



# 1.userAgent为什么有这么多相似的字段

userAgent是我们经常会用到的字段，里面包含了很多信息，先来看看常见的浏览器，比如mac下的Chrome，userAgent是：

```javascript
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36
```

 再比如，FireFox的userAgent是：

```javascript
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.3 Safari/605.1.15
```

Chrome的渲染引擎不是Blink吗？为什么有KHTML、Safari、还有Gecko？

FireFox的渲染引擎不是Gecko吗？为什么有KHTML、Safari？



这个其实要从浏览器的历史说起。

​    最初的主角叫NCSA Mosaic，简称Mosaic（马赛克），是1992年末位于伊利诺伊大学厄巴纳-香槟分校的国家超级计算机应用中心（National Center for Supercomputing Applications，简称NCSA）开发，并于1993年发布的一款浏览器。它自称“NCSA_Mosaic/2.0（Windows 3.1）”，Mosaic可以同时展示文字和图片，从此浏览器变得有趣多了。

​    然而很快就出现了另一个浏览器，这就是著名的Mozilla，中文名称摩斯拉。一说 Mozilla = Mosaic + Killer，意为Mosaic杀手，也有说法是 Mozilla = Mosaic & Godzilla，意为马赛克和哥斯拉，而Mozilla最初的吉祥物是只绿色大蜥蜴，后来更改为红色暴龙，跟哥斯拉长得一样。

​    但Mosaic对此非常不高兴，于是后来Mozilla更名为Netscape，也就是网景。Netscape自称“Mozilla/1.0(Win3.1)”，事情开始变得更加有趣。网景支持框架（frame），由于大家的喜欢框架变得流行起来，但是Mosaic不支持框架，于是网站管理员探测user agent，对Mozilla浏览器发送含有框架的页面，对非Mozilla浏览器发送没有框架的页面。

​    后来网景拿微软寻开心，称微软的Windows是“没有调试过的硬件驱动程序”。微软很生气，后果很严重。此后微软开发了自己的浏览器，这就是Internet Explorer，并希望它可以成为Netscape Killer。IE同样支持框架，但它不是Mozilla，所以它总是收不到含有框架的页面。微软很郁闷很快就沉不住气了，它不想等到所有的网站管理员都了解IE并且给IE发送含有框架的页面，它选择宣布IE是兼容Mozilla，并且模仿Netscape称IE为“Mozilla/1.22(compatible; MSIE 2.0; Windows 95)”，于是IE可以收到含有框架的页面了，所有微软的人都嗨皮了，但是网站管理员开始晕了。

​    因为微软将IE和Windows捆绑销售，并且把IE做得比Netscape更好，于是第一次浏览器血腥大战爆发了，结果是Netscape以失败退出历史舞台，微软更加嗨皮。但没想到Netscape居然以Mozilla的名义重生了，并且开发了Gecko，这次它自称为“Mozilla/5.0(Windows; U; Windows NT 5.0; en-US; rv:1.1) Gecko/20020826”。
​    Gecko是一款渲染引擎并且很出色。Mozilla后来变成了Firefox，并自称“Mozilla/5.0 (Windows; U; Windows NT 5.1; sv-SE; rv:1.7.5) Gecko/20041108 Firefox/1.0”。Firefox性能很出色，Gecko也开始攻城略地，其他新的浏览器使用了它的代码，并且将它们自己称为“Mozilla/5.0 (Macintosh; U; PPC Mac OS X Mach-O; en-US; rv:1.7.2) Gecko/20040825 Camino/0.8.1”，以及“Mozilla/5.0 (Windows; U; Windows NT 5.1; de; rv:1.8.1.8) Gecko/20071008 SeaMonkey/1.0”，每一个都将自己装作Mozilla，而它们全都使用Gecko。

​    Gecko很出色，而IE完全跟不上它，因此user agent探测规则变了，使用Gecko的浏览器被发送了更好的代码，而其他浏览器则没有这种待遇。Linux的追随者对此很难过，因为他们编写了Konqueror，它的引擎是KHTML，他们认为KHTML和Gecko一样出色，但却因为不是Gecko而得不到好的页面，于是Konqueror为得到更好的页面开始将自己伪装成“like Gecko”，并自称为“Mozilla/5.0 (compatible; Konqueror/3.2; FreeBSD) (KHTML, like Gecko)”。自此user agent变得更加混乱。

​    这时更有Opera跳出来说“毫无疑问，我们应该让用户来决定他们想让我们伪装成哪个浏览器。”于是Opera干脆创建了菜单项让用户自主选择让Opera浏览器变成“Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; en) Opera 9.51”，或者“Mozilla/5.0 (Windows NT 6.0; U; en; rv:1.8.1) Gecko/20061208 Firefox/2.0.0 Opera 9.51”， 或者“Opera/9.51 (Windows NT 5.1; U; en)”。

​    后来苹果开发了Safari浏览器，并使用KHTML作为渲染引擎，但苹果加入了许多新的特性，于是苹果从KHTML另辟分支称之为WebKit，但它又不想抛弃那些为KHTML编写的页面，于是Safari自称为“Mozilla/5.0 (Macintosh; U; PPC Mac OS X; de-de) AppleWebKit/85.7 (KHTML, like Gecko) Safari/85.5”，这进一步加剧了user agent的混乱局面。
​    因为微软十分忌惮Firefox，于是IE重装上阵，这次它自称为“Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0) ”，并且渲染效果同样出色，但是需要网站管理员的指令它这么做才行。

​    再后来，谷歌开发了Chrome浏览器，Chrome使用Webkit作为渲染引擎，和Safari之前一样，它想要那些为Safari编写的页面，于是它伪装成了Safari。于是Chrome使用WebKit，并将自己伪装成Safari，WebKit伪装成KHTML，KHTML伪装成Gecko，最后所有的浏览器都伪装成了Mozilla，这就是为什么所有的浏览器User-Agent里都有Mozilla。Chrome自称为“Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/525.13 (KHTML, like Gecko) Chrome/0.2.149.27 Safari/525.13”。

​    因为以上这段历史，现在的User-Agent字符串变得一团糟，几乎根本无法彰显它最初的意义。追根溯源，微软可以说是这一切的始作俑者，但后来每一个人都在试图假扮别人，最终把User-Agent搞得混乱不堪。

​    一句话结论：因为网站开发者可能会因为你是某浏览器（这里是 Mozilla），所以输出一些特殊功能的程序代码（这里指好的特殊功能），所以当其它浏览器也支持这种好功能时，就试图去模仿 Mozilla 浏览器让网站输出跟 Mozilla 一样的内容，而不是输出被阉割功能的程序代码。大家都为了让网站输出最好的内容，都试图假装自己是 Mozilla，一个已经不存在的浏览器……

附各大浏览器诞生年表：
1993年1月23日：Mosaic
1994年12月：Netscape
1994年：Opera
1995年8月16日：Internet Explorer
1996年10月14日：Kongqueror
2003年1月7日：Safari
2008年9月2日：Chrome

注：故事部分转自现代简明魔法。



所以，总结更新下浏览器的引擎：

Chrome：渲染引擎是Blink。Chrome早期的时候，使用的是与Safari一样的用的是WebKit。而WebKit的基础，是KDE的开放源代码KHTML。

FireFox：Mozilla旗下的一个产品，渲染引擎是Gecko。

Safari：渲染引擎是Webkit。

Opera：早期的渲染引擎是Presto，2013年的时候被Blink取代。

IE：IE分为IE浏览器及Edge浏览器。IE的渲染引擎是Trident。Edge一开始的渲染引擎是EdgeHTML，该引擎是Trident的一个分支，2018年12月，微软发表声明Edge将会重新以Chromium为基础建置浏览器，表示之后将会使用Blink排版引擎，并终止EdgeHTML的开发。

具体文献：

[EdgeHtml](https://zh.m.wikipedia.org/wiki/EdgeHTML)
[Belfiore, Joe, Microsoft Edge: Making the web better through more open source collaboration](https://blogs.windows.com/windowsexperience/2018/12/06/microsoft-edge-making-the-web-better-through-more-open-source-collaboration/), Microsoft, 2018-12-06
[Microsoft Edge and Chromium Open Source: Our Intent](https://github.com/MicrosoftEdge/MSEdge/blob/7d69268e85e198cee1c2b452d888ac5b9e5995ca/README.md). Microsoft Edge Team. 6 December 2018

所以，我们就理解为什么各大浏览器是这样的userAgent了。



# 2.各大浏览器userAgent解析

userAgent的语法为：

```javascript
User-Agent: <product> / <product-version> <comment>
```

 大部分的浏览器userAgent为：

```javascript
User-Agent: Mozilla/5.0 (<system-information>) <platform> (<platform-details>) <extensions>
```

 

### Firefox

firefox的userAgent包含这4个部分：

Mozilla/5.0 (platform; rv:geckoversion) Gecko/geckotrail Firefox/firefoxversion

前面历史中提到，基本每个浏览器都包含了Mozilla/5.0字段。

1. platform：描述了浏览器运行的平台，包括 Windows, Mac, Linux, Android等。platform也可以用多个分号隔开。

2. rv:geckoversion：表示Gecko的发布版本。在最近的firefox版本中，firefoxversion和geckoversion一致。

3. Gecko/geckotrail：表示当前浏览器的渲染引擎是Gecko。在现在的opera，可以看到，geckotrial已经固定为20100101

4. Firefox/firefoxversion：表示当前浏览器是Firefox，firefoxversion表示版本号。

 例如：

Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:86.0) Gecko/20100101 Firefox/86.0

**所以，检测firefox的核心，就是检测有没有Firefox字段。**



### Chrome

Chrome的渲染引擎是Blink。因为兼容的原因，它添加了KHTML, like Gecko and Safari字段。

例如：

```javascript
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36
```

**所以，检测chrome的核心，就是检测有没有Chrome字段。**

另外，Chrome提了[user agent client hint](https://web.dev/user-agent-client-hints/)来替代UA，在Chrome 84版本中就开始可以用了，使用例子如下：

```javascript
// Log the brand data
console.log(navigator.userAgentData.brands);

// Log the mobile indicator
console.log(navigator.userAgentData.mobile);

// Log the full user-agent data
navigator
  .userAgentData.getHighEntropyValues(
    ["architecture", "model", "platform", "platformVersion",
     "uaFullVersion"])
  .then(ua => { console.log(ua) });
```

 ![637104c8c30da9dcd7491cf970503_w1722_h706.png](http://km.oa.com/files/photos/pictures/18e/637104c8c30da9dcd7491cf970503_w1722_h706.png)

 

### Opera

Opera现在也是是用Blink渲染引擎了，所以它的userAgent与Chrome类似。但是在最后加上了"OPR/<version>"。

例如：

```javascript
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36 OPR/74.0.3911.160
```

 比较旧的Opera的浏览器一般为：

```javascript
Opera/9.80 (Macintosh; Intel Mac OS X; U; en) Presto/2.2.15 Version/10.00
Opera/9.60 (Windows NT 6.0; U; en) Presto/2.1.1
```

**所以，检测Opera的核心，就是检测有OPR字段或者Opera字段**

 

### Safari

例如：

```javascript
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.3 Safari/605.1.15
```

可以看到，Safari和Chrome的UA是类似的，区别是Safari没有Chrome字段而Chrome两个都有。

**所以，检测Safari的核心，就是检测有Safari字段，同时没有Chrome字段。**



### IE

IE edge：

```javascript
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362
```

IE11：

```javascript
Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko
```

IE10：

```javascript
Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; Trident/6.0)
```

IE9：

```javascript
Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)
```

 IE8：

```javascript
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0)
```

 从上面可以看到，**检测IE8-11的核心是检测是否有MSIE或Trident字段，检测IE Edge的核心是检测是否有edge字段**



# 3.最全userAgent代码判断附上

最后的话附上浏览器识别代码：

```javascript
export function getBrowser() {
  const { userAgent } = navigator;
  const rMsie = /(msie\s|trident.*rv:)([\w.]+)/;
  const rEdge = /(edge)\/([\w.]+)/;
  const rFirefox = /(firefox)\/([\w.]+)/;
  const rOpera = /(opera).+version\/([\w.]+)/;
  const rChrome = /(chrome)\/([\w.]+)/;
  const rSafari = /version\/([\w.]+).*(safari)/;
  let browser = "other";
  let version = "0";
  let brand = "other";
  const ua = userAgent.toLowerCase();
  const { vendor } = navigator;
  function uaMatch(ua: string) {
    let match = rMsie.exec(ua);
    if (match != null) {
      return { browser: "ie", version: match[2] || "0" };
    }
    match = rEdge.exec(ua);
    if (match != null) {
      return { browser: "edge", version: match[2] || "0" };
    }
    match = rChrome.exec(ua);
    if (match != null && /Google/.test(vendor)) {
      return { browser: match[1] || "", version: match[2] || "0" };
    }
    match = rSafari.exec(ua);
    if (match != null && /Apple Computer/.test(vendor)) {
      return { browser: match[2] || "", version: match[1] || "0" };
    }
    match = rFirefox.exec(ua);
    if (match != null) {
      return { browser: match[1] || "", version: match[2] || "0" };
    }
    match = rOpera.exec(ua);
    if (match != null) {
      return { browser: match[1] || "", version: match[2] || "0" };
    }
    return { browser: "other", version: "0" };
  }

  try {
    const browserMatch = uaMatch(ua);
    ({ browser, version } = browserMatch);

    if (/qqbrowser/.test(ua)) {
      brand = "qq";
    } else if (/se/.test(ua) && /metasr/.test(ua)) {
      brand = "sougou";
    } else if (/360se/.test(ua)) {
      brand = "360";
    } else if (/ucweb/.test(ua)) {
      brand = "uc";
    } else if (/2345explorer/.test(ua)) {
      brand = "2345";
    } else if (/lbbrowser/.test(ua)) {
      brand = "liebao";
    } else if (/maxthon/.test(ua)) {
      brand = "maxthon";
    } else {
      brand = browser;
    }
  } catch (e) {
    console.error(` getBrowser error: ${e}`);
  }
  return `${brand}_${browser}_${version}`;
}
```

 

另外，github有优秀且更加全面的开源代码，建议参考下这里的开源项目会更全：https://github.com/mumuy/browser/blob/master/Browser.js







参考文章：

https://www.jianshu.com/p/eeb42ee1da4c

https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent