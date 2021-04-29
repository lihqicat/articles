> 最近项目中使用了装饰器模式的思想，虽然装饰器听起来很简单，学习了一些资料之后，发现装饰器设计模式是一种在项目中使用频率很高的一种设计思想。本文希望通过以点带面的形式，对装饰器如何优雅的在项目中应用做一个总结～

# 装饰器概念
在开发过程中，很多时候我们不想要类的功能一开始就很庞大，一次性包含很多职责（毕竟程序员一直恪守着封装抽象bababa等概念）。这个时候我们可以使用装饰器模式。动态的给某个对象添加一些职责，并且不会影响从这个类派生的其他对象。

在传统的面向对象开发中，给对象添加功能时，我们通常会采用继承的方式，继承的方式目的是为了复用，但是随之而来也带来一些问题：

（1）父类和子类存在强耦合的关系，当父类改变时，子类也需要改变；

（2）子类需要知道父类中的细节，至少需要知道接口名，从而进行复用或复写，这样其实破坏了封装性；

（3）继承的方式，可能会创建出大量子类。比如现在有BBA三种类型的汽车，构造了一个汽车基类，三个三种类型的汽车。现在需要给汽车装上雾灯、前大灯、导航仪、刮雨器，如果采用继承的方式，那么就要构建3*4个类。但是如果把雾灯、前大灯、导航仪、刮雨器动态地添加到汽车上，那么只需要增加4个类。这种采用动态添加职责的方式就是装饰器。

装饰器的目的就是在不改变原来类的基础上，为其在运行期间动态的添加职责。



# 用AOP装饰函数
这里为什么要讨论到AOP呢？我们后面来回答这个疑问。

我们先回顾一下OOP（Object Oriented Programing），OOP作为面向对象编程模式，取得了巨大的成功应用，主要思想是封装、继承和多态。

而AOP是一种新的编程方式，它和OOP不同，OOP把系统看成多个对象的交互，AOP把系统分为不同的关注点，或者称之为切面（Aspect）。

AOP（Aspect Oriented Programing），意为面向切面编程，通过预编译方式和动态代理的方式实现程序功能时统一维护的一种技术。 主要就是把与业务逻辑无关的功能抽离开来，这些与业务逻辑无关的功能常见有数据统计、函数执行时间、权限控制、异常处理等。把这些功能抽离出来后，再在通过动态切入的方式注入业务逻辑中。这样做的好处是可以保证业务内部的高内聚和业务模块之间的低耦合，从而可以方便复用与业务逻辑无关的模块。


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/208c2558984443b4880f08562e15da4b~tplv-k3u1fbpfcp-watermark.image)


这里为什么要把装饰器和AOP结合起来讨论呢？

从上面的描述中，会发现AOP和装饰器模式概念很像，但装饰器注重的是装饰，而AOP注重的是横切面统一维护。两者其实最终都是要解决同一个问题，就是提高编程的低耦合与高可复用性。

说了这么多，还是有点抽象，我们来看看实际例子，如何使用AOP在项目中应用。

# AOP的实现（ES5版）
AOP是Java Spring中最重要的功能之一，使用过的同学一定知道，其内分为3种通知，before（前置通知）、after（后置通知）、arround（环绕通知）。我们来模拟AOP中before和after的实现。

## before（前置通知）
顾名思义，就是在函数调用之前执行。

```
Function.prototype.before = function (beforefn) {
  var __self = this;
  return function () {
    beforefn.apply(this, arguments);
    return __self.apply(this, arguments);
  };
};
```

## after（后置通知）
与before相反，在函数调用之后执行
```javascript
Function.prototype.after = function (afterfn) {
  var __self = this;
  return function () {
    var ret = __self.apply(this, arguments);
    afterfn.apply(this, arguments);
    return ret;
  };
};
```

## 日志记录上报例子
接下来我们以数据上报为例子，讲讲如何用AOP来装饰函数。

比如页面有一个分享按钮，点开之后会弹出分享框分享给某人，点击完毕后，需要进行数据上报。这个时候，通常的方式，我们会这样去实现：
```javascript
<html>
<button tag="share" id="share">点击打开分享框</button> 
<script>
    var showShare = function () {
        console.log("打开分享框");
        log("打开分享框");
    };
    var log = function (tag) {
        console.log("上报标签为: " + getAttribute("tag"));
    };
    var getAttribute = function () {
        return document.getElementById('button') && document.getElementById('button').getAttribute('tag');
    }
    document.getElementById("button").onclick = showShare;
</script> 
</html>
```
 可以看到showShare函数中，既负责打开弹出，又负责上报统计，这是两个层面的内容，耦合在了一起，使用AOP分离后，代码如下：
```javascript
<html>
<button tag="login" id="button">点击打开分享框</button>
<script>
    var showShare = function () {
        console.log("打开分享框");
    };
    var log = function () {
        console.log("上报标签为: " + getAttribute("tag"));
    };
    var getAttribute = function () {
        return document.getElementById('button') && document.getElementById('button').getAttribute('tag');
    }
    showShare = showShare.after(log); // 打开分享框之后上报数据
    document.getElementById("button").onclick = showShare;
</script> 
</html>
```
数据上报功能在行数后执行，当然，如果希望在函数前执行，则调用：
```javascript
showShare = showShare.before(log); // 打开分享框之后上报数据
```
可以看到，讲不同职责的功能严格分离开来，避免耦合正是AOP要实现的目标，也是装饰器的效果。



我们再举一个例子：ajax请求，是每个项目必不可少的，那么怎么用AOP来实现呢？

## 使用AOP解耦ajax参数
在项目中使用ajax请求，通常，我们可能会封装一个统一的请求库，例如：
```javascript
var ajax = function (type, url, param) {
  console.dir(param);
  // 发送 ajax 请求的代码略
};
ajax("get", "http:// xxx.com/userinfo", { name: "sven" });
```
ajax函数在项目中一直运转良好，和后台的合作也一直非常愉快，直到有一天，我们的网站收到了CSRF攻击，解决CSRF攻击最好的方式就是给HTTP请求加上一个token参数。现在的任务是要给每个ajax函数都加上token，这个时候我们第一个想到的肯定是需要去修改下ajax的函数：
```javascript
var ajax = function (type, url, param) {
  param = param || {};
  Param.Token = getToken(); // 发送 ajax 请求的代码略...
};
```
虽然解决了这个问题，但是我们ajax库就变得不可复用了，每个ajax请求都带上了token，虽然在我的项目中没有问题，但是如果要移植到其他项目，或者进行开源，token就显得冗余了，也许另外一个项目不需要token，也有可能生成的token方式不一样。

为了解决这个问题，我们还是要把ajax还原为一个纯净的函数。
```javascript
var ajax = function (type, url, param) {
  console.log(param);
};
```
 然后，使用AOP的before进行装饰，把token参数加到param中。
```javascript
var getToken = function () {
  return "Token";
};
ajax = ajax.before(function (type, url, param) {
  param.Token = getToken();
});
ajax("get", "http:// xxx.com/userinfo", { name: "sven" });
```
可以看到，用AOP的方式给ajax动态加上了token参数，使ajax保持了请求功能的单一性，提高了ajax的复用性，下次如果要在其他项目中使用到，就可以不用做任何修改了。

 

# AOP的实现（ES7版）
前面其实是使用ES5的语法模拟了AOP的实现，在JS新一代的语言中，已经实现了装饰器的语法糖，Vscode的源码中更是使用了装饰器来实现了依赖注入的模式。那么，AOP与装饰器的结合，又是如何使用的呢？

## 日志记录上报例子
### log装饰器实现
```javascript
export function log(target, name, decriptor) {
  var _origin = decriptor.value;
  decriptor.value = function () {
    console.log(`Calling ${name} with `, arguments);
    return _origin.apply(null, arguments);
  };

  return decriptor;
}
```
### 调用装饰器
```javascript
import { log } from "./log";
class Person {
  @log
  say(nick) {
    return `hi ${nick}`;
  }
}

var person = new Person();
person.say("小明");
```

# 使用AOP解耦ajax参数
## ajax参数装饰器
```javascript
function getToken() {
  return "token";
}
// 给ajax添加token参数
export function decorateToken(target, name, descriptor) {
  let method = descriptor.value;
  descriptor.value = function (...args) {
    args[2].token = getToken();
    method.apply(this, args);
  };
}
```
## 调用装饰器
```javascript
class Util {
  @decorateToken
  static ajax(type, url, param) {
    console.log("param is ", param);
  }
}

Util.ajax("get", "http://baidu.com", { name: "test" });
```
可以看到给ajax添加上了参数：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f78418efbfac4874b97ad52fce5139e9~tplv-k3u1fbpfcp-watermark.image)



# 总结
装饰器模式，随着ES7和Typescript语法糖的支持，越来越渗透到前端领域的开发中，JS这门语言也越来越庞大，以前只有在Java或py等用的AOP，也可以通过装饰器模式，应用到了JS中。

通过对装饰器的研究，挺有感触，很多时候我们会大谈某某设计模式，但是在代码中的应用其实很少，而当项目逐渐庞大时，耦合度也越来越高，历史包袱越来越重，我们更加难以迈出一步做重构。

本文通过列举了日志上报、ajax请求的例子，展示了装饰器其实是一个非常常用且nice的设计模式，有时候一个不起眼的功能，通过稍微的"装饰"（例如应用某种设计模式），会变得优雅与纯净。编写优雅的代码，是一个程序员需要一直追求的目标与超越。





参考文章：

https://juejin.im/post/6844903838172839943

《Javascript设计模式与开发实战》