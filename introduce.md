## AlloyTouch实战--60行代码搞定QQ看点资料卡 
link:https://github.com/AlloyTeam/AlloyTouch/wiki/kandian

<h1 align=center>
 AlloyTouch实战--60行代码搞定QQ看点资料卡
</h1>

## 先验货
![](http://images2015.cnblogs.com/blog/105416/201612/105416-20161229100016070-1865978262.png)

- 访问DEMO你也可以[点击这里](http://alloyteam.github.io/AlloyTouch//refresh/infinite/kandian.html)
- 源代码可以[点击这里](https://github.com/AlloyTeam/AlloyTouch/blob/master/refresh/infinite/kandian.html#L915-L978)

如你体验所见，流程的滚动的同时还能支持头部的动画？不断地加载新数据还能做到流畅的滑动！怎么做得的？使用AlloyTouch CSS 0.2.0及以上版本便可！


## 头部动画

![](http://images2015.cnblogs.com/blog/105416/201612/105416-20161229100113867-627323058.gif)


## 加载更多

![](http://images2015.cnblogs.com/blog/105416/201612/105416-20161229100100961-1863388318.gif)


## 实现代码

```js
var infoList = document.getElementById("infoList"),
    mockHTML = infoList.innerHTML,
    scroller = document.getElementById("scroller"),
    header = document.getElementById("header"),
    userLogo = document.getElementById("user-logo-wrapper"),
    loading = false,
    alloyTouch = null;

Transform(scroller, true);
Transform(header);
header.originY = -70;
header.translateY = -70;
Transform(userLogo);

alloyTouch = new AlloyTouch({
    touch: "#wrapper",
    vertical: true,
    target: scroller,
    property: "translateY",
    maxSpeed: 2,
    outFactor: 0.1,
    min: 160 * -20 + window.innerHeight - 202 - 50,
    max: 0,
    lockDirection: false,
    touchStart: function () {
        reastMin();
    },
    lockDirection: false,
    change: function (v) {
        if (v <= this.min + 5 && !loading) {
            loading = true;
            loadMore();
        }
        if (v < 0) {
            if (v < -140) v = -140;
            var scaleY = (240 + v) / 240;
            header.scaleY = scaleY;
            userLogo.scaleX = userLogo.scaleY = scaleY;
            userLogo.translateY = v / 1.7;
        } else {
            var scaleY = 1 + v / 240;
            header.scaleY = scaleY;
            userLogo.scaleX = userLogo.scaleY = scaleY;
            userLogo.translateY = v / 1.7;
        }
    }
})

function loadMore() {
    setTimeout(function () {
        infoList.innerHTML += mockHTML;
        loading = false;
        reastMin();
    }, 500)
}

function reastMin() {
    alloyTouch.min = -1 * parseInt(getComputedStyle(scroller).height) + window.innerHeight - 202;
}

document.addEventListener("touchmove", function (evt) {
    evt.preventDefault();
}, false);
```

就这么多代码。当然你要引用一个transformjs和alloy_touch.css.js。先看这一堆：

```js
Transform(scroller, true);
Transform(header);
header.originY = -70;
header.translateY = -70;
Transform(userLogo);
```
* Transform(xxx)是什么意思？
> 赋予xxx transformation能力
* 第一个scroller加上true代表关闭透视投影，为什么要关闭透视投影？
> 因为scroller里面是有文本，防止文本在IOS中模糊的情况。
* header是顶部的那个蓝色的区域。为什么要设置originY和translateY？为什么要设置为-70？
> 因为header的高度为140px，用户向上滚动的过程中，需要对其进行scaleY变换。通常我们的做法是设置CSS3 transform-origin为 center top。而使用transformjs之后，可以抛弃transform-origin，使用originY或者originX属性便可。originY 设置为 -70，相对于高度为140的header来说就是center top。

再看这一堆：
```js
alloyTouch = new AlloyTouch({
    touch: "#wrapper",
    vertical: true,
    target: scroller,
    property: "translateY",
    maxSpeed: 2,
    outFactor: 0.1,
    lockDirection: false,
    min: 160 * -20 + window.innerHeight - 202 - 50,
    max: 0,
    touchStart: function () {
        resetMin();
    },
    lockDirection: false,
	...
    ...
    ...
})
...
...
function resetMin() {
    alloyTouch.min = -1 * parseInt(getComputedStyle(scroller).height) + window.innerHeight - 202;
}
```
使用AlloyTouch最关键的一点就是计算min和max的值。min和max决定了可以滚到哪里，到了哪里会进行回弹等等。这里max是0毫无疑问。
* 但是min那一堆加减乘除是什么东西？
> 这里首次加载是20行数据，每一行高度大概160，主意是大概， window.innerHeight是视窗的高度，202px是滚动的容器的padding top的值，50px是用来留给显示**加载更多**的...
![](http://images2015.cnblogs.com/blog/105416/201612/105416-20161229100051132-1273608491.png)

如上图所示，主要是需要求???的高度。
* 那么怎么解决大概160*20的问题？
> touchStart的时候reastMin。resetMin会去通过getComputedStyle计算整个scroller的高度。
* maxSpeed是干什么用的？
> 用来限制滚动的最大速度，个人感觉调整到2挺舒适，这个可以根据场景和被运动的属性灵活配置。
* outFactor是干什么用的？
> 用来设置超出min或者max进行拖拽的运动比例系数，系数越小，超出min和max越难拖动，也就是受到的阻力越大。
* lockDirection是干什么用的？
> lockDirection默认值是true。代表用户起手时候是横向的，而你监听的是竖直方向的touch，这样的话是不会触发运动。只有起手和监听对应上才会有触摸运动。这里把lockDirection设置成false就没有这个限制，不管用户起手的direction，都会有触摸运动。

再看AlloyTouch注入的change事件！头部动效核心的一个配置函数：

```js
change: function (v) {
    if (v <= this.min + 5 && !loading) {
        loading = true;
        loadMore();
    }
    if (v < 0) {
        if (v < -140) v = -140;
        var scaleY = (240 + v) / 240;
        header.scaleY = scaleY;
        userLogo.scaleX = userLogo.scaleY = scaleY;
        userLogo.translateY = v / 1.7;
    } else {
        var scaleY = 1 + v / 240;
        header.scaleY = scaleY;
        userLogo.scaleX = userLogo.scaleY = scaleY;
        userLogo.translateY = v / 1.7;
    }
}
```

v代表当前被运动对象的被运动属性的当前的值，根据这个v去做一些效果和加载更多。
* 什么时候加载更多？
> 当滚动你能看到加载更多的时候去加载更多
* 什么时候能看到加载更多？
> v <= this.min + 5。 可以看到change回调里可以拿到this，也就是AlloyTouch对象的实例，当v等于this.min代表滚到了底部，所以这里加上5代表快要滚动底部已经看到了加载更多。就去执行loadMore函数。
* loading是干什么用的？
> 防止重复loadMore用得，因为change执行得很频繁，所以这里会通过loading的状态去锁上。
* 下面一堆设置scaleX、scaleY、translateY以及一堆数字是怎么来的？
> 慢慢调试得出的最佳效果~~反正就是根据v的数值映射到 header和用户头像的transform属性上，这里就不一一讲了。

再看loadMore：
```js
function loadMore() {
    setTimeout(function () {
        infoList.innerHTML += mockHTML;
        loading = false;
        reastMin();
    }, 500)
}
```
这里使用了一段假的HTML去模拟AJAX异步请求以及数据转HTML的过程，整个耗时500ms，500ms后会去：
- 插入HTML
- 重置loading状态
- 重置AlloyTouch的min

最后：
```js
document.addEventListener("touchmove", function (evt) {
    evt.preventDefault();
}, false);
```
阻止掉整个document的默认事件，不会把整个页面拖下来，在手Q里的话，你就看不到网址和X5内核提供技术支持了。

## 开始AlloyTouch

Github：[https://github.com/AlloyTeam/AlloyTouch](https://github.com/AlloyTeam/AlloyTouch)

任何意见和建议欢迎[new issue](https://github.com/AlloyTeam/AlloyTouch/issues)，AlloyTouch团队会第一时间反馈。
