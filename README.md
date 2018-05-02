##### 参考AlloyTouch的QQ看点demo写的我的页面
![我的页面.gif](https://github.com/MiuMiu-S/QQkandian-by-AlloyTouch/blob/master/mine_page.gif)
 <br>
更新后ios11.3系统要加上下面这句
document.addEventListener("touchmove", function (evt) {
    evt.preventDefault();
}, {passive: false});
