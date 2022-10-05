---
title: HTML 实现自定义弹窗
date: 2020-03-25 18:51:50
abstract: "关于实现 HTML 自定义弹窗的一种实现方式"
tags: 
    - "HTML"
    - "CSS"
    - "JavaScript"
categories:
    - "Blog"
---

## 问题描述

&#160; &#160; &#160; &#160; 在编程实践过程中，需要实现在 Web 页面实现一个“制定闹钟”的功能，而且我希望这个功能能够以弹窗的方式实现。由于毫无 Web 端开发的经验，所以难免地查了类似“HTML实现自定义弹窗”、“JS实现定闹钟功能”等等关键词。由日常的搜索引擎经验可知，这时候得到的检索结果很多，所以本篇博客的意义在于帮助解决这样的问题，即 <font clolr="red">HTML如何实现简易弹窗</font>，当然，这里采用的样例问题就是 <font color="red">实现一个弹窗实现定闹钟功能</font>。

## 解决途径

&#160; &#160; &#160; &#160; 在学习 HTML 的过程中，页面部件的显示位置是由 HTML 文件的布局来控制的，所以该如何看待“弹窗”？<font color="red">不妨把它看做是一个部件(这样来设计实现它的布局自然直接使用HTML来实现即可)。</font> 那如何让这个“部件”拥有“弹窗的效果”？<font color="red">把待显示部件放到页面最前方即可。</font>那如何让弹窗在该弹出来的时候弹出来，用户点击关闭之后即可关闭呢？<font color="red">当该部件不需要显示的时候的设置为“none”，在需要它显示的时候，设置为“block”即可。</font>

&#160; &#160; &#160; &#160; 由此，已经捋清了实现的途径（原理）。在这样的实现原理下，就需要说一下实现的细节问题了。

&#160; &#160; &#160; &#160; HTML 对于部件来说，在该部件的 div 标签下实现布局、class 标签下指定 css 样式即可。同时，css 内可以设定样式 `z-index` 值，这就有点像是空间直角坐标系一般：将屏幕横向看做 x 轴、纵向看做 y 轴，这样就可以确定屏幕上的每个像素点；那么引入 z 轴呢？即可实现部件堆叠的效果！这也就是 `z-index` 的作用，`z-index` 的值越大，堆叠的部件就越排在上面。也就是说，要实现的弹窗，让它的 `z-index` 值足够大，就可以满足弹窗的需求了。

&#160; &#160; &#160; &#160; 然而，常常看到的弹窗，会使得页面其余部分变为浅色（或是灰色），这个效果该如何实现？只需要再添加一层“遮罩层”即可！可以让遮罩层的颜色设置为想要的颜色（比如灰色），这样，在弹窗出现的时候，弹窗“堆在最上层”，遮罩层“堆在相邻层”，原页面“堆在最下层”，这样层次分明，自然就可以实现弹窗的效果了。这时的弹窗，本质上还是 HTML 部件，自然布局就可以用 HTML 来写了。

## 实现代码

&#160; &#160; &#160; &#160; 首先，设置 HTML 部件，让它拥有目标实现的效果。

```html
<!-- 定义弹窗 -->
<!-- 原理: 将显示形式设置为none, 在真正需要的时候再将其设置为block -->
<div id="dialogmask" class="dialogmask opacity"></div>
<div id="dialog" class="box" style="display: none">　
    <div id="dialog_title" class="dialogtitle">
        <label style="padding-left: 10px">制定闹钟</label>
        <label style="float: right;padding-right: 10px;" onclick="close_pop_window()">[关闭]</label>
    </div>
    <div class="container">
        <div class="alarmbox" id="alarm_box" value="当前时间"></div>
        <!--<div class="time fl" id="alarm_time"></div>-->
        <div class="clear"></div>
        <div class="set">
            <input type="text" name="hour" id="hour" value="" placeholder="时" />
            <input type="text" name="min" id="min" value="" placeholder="分" />
            <input type="text" name="sec" id="sec" value="" placeholder="秒" />
            <button type="button" id="btn">设置</button>
            <div class="clear"></div>
        </div>
    </div>
    <script type="text/javascript" src="static/js/alarm.js"></script>
</div>
```

&#160; &#160; &#160; &#160; 然后，实现弹窗出现 / 关闭的触发条件。这里是使用按钮点击，出现窗体，并点击窗体“关闭”按钮来关闭窗体。

```js
function show_pop_window(func_code, info) {
    $("#dialog").css({display: "block"}); //通过Jquery的css()更改样式
    $("#dialogmask").css({display: "block"});
    $("#logcontent").html(info);
    if (func_code === 4) {
        $("#dialog_title").css({background: "BlueViolet"});
    } else {
        $("#dialog_title").css({background: "#00CC00"});
    }
}

function close_pop_window() {
    $("#dialog").css({display: "none"});
    $("#dialogmask").css({display: "none"});
}
```
&#160; &#160; &#160; &#160; 然后你就会拥有一个类似这样的弹窗：

![弹窗效果展示](example.png)

&#160; &#160; &#160; &#160; 当然，如果你需要一个闹钟，你可以这样实现：

```js
//获取元素
// var oBox = document.querySelector(".alarmbox");
// var oTime = document.querySelector(".time");
var oBox = document.getElementById("alarm_box");

var oHour = document.getElementById("hour");
var oMin = document.getElementById("min");
var oSec = document.getElementById("sec");
var oBtn = document.getElementById("btn");

//设置定时器
var timer;
timer = setInterval(function() { clock(); }, 500);

// 必须定义在此处，以避免被回收
var uh, um, us;

function clock() {
    var curr_date = new Date();
    var curr_hour = curr_date.getHours();
    var curr_minute = curr_date.getMinutes();
    var curr_second = curr_date.getSeconds();

    //修改时分秒的格式
    var sh = (curr_hour < 10) ? '0'+curr_hour : ''+curr_hour;
    var sm = (curr_minute < 10) ? '0'+curr_minute : ''+curr_minute;
    var ss = (curr_second < 10) ? '0'+curr_second : ''+curr_second;
    //修改innertext
    oBox.innerText = "当前时间:\n" + sh + " : " + sm + " : " + ss;

    oBtn.onclick = function() {
        uh = parseInt(oHour.value); uh = isNaN(uh) ? '00' : (uh < 10) ? '0'+uh : ''+uh;
        um = parseInt(oMin.value); um = isNaN(um) ? '00' : (um < 10) ? '0'+um : ''+um;
        us = parseInt(oSec.value); us = isNaN(us) ? '00' : (us < 10) ? '0'+us : ''+us;
    }

    if ((sh == uh) && (sm == um) && (ss == us)) {
        var alarm_music = document.getElementById("alarm_music");
        alert("TIME IS UP!!!");
        oHour.value = '';
        oMin.value = '';
        oSec.value = '';
        alarm_music.play();
    }
}
clock();
```
&#160; &#160; &#160; &#160; 当然，其中需要注意的就是 JS 对象的回收，恰好其中两个部分内容形成了对比：

&#160; &#160; &#160; &#160; 第一个部件是显示时间的部件，这个部件的内容会随着时间的变化而变化，那么显示内容的字符串就不能是一个局部变量！因为上一个时刻显示的字符串在下一秒就会变化，若它是一个局部变量，下一个时刻就被回收掉，从而造成显示异常。

&#160; &#160; &#160; &#160; 第二个部件就是音频播放部件。这里的 alarm_music 是一个 HTML 的 audio 标签部件，用于存储音频，在闹钟计时结束时播放。但音频每次播放之后，这个变量就播放结束了，即再对它调用 play 方法也无法继续播放。所以就可以将这个变量设置为局部变量，这样就可以在每次播放时创造一个变量实现音乐播放，达到预期效果。当然，也可以设置 audio 为 loop 的，这样的话就需要用一个按钮来设置让音乐停止播放（我没尝试，但逻辑应当如此）。

&#160; &#160; &#160; &#160; 当然，这些理解起来并不困难，属于“编译原理”的基础内容，重点在于思考，而不是单纯地把这些不符合预期的东西简单粗暴地归类为莫名 Bug。

&#160; &#160; &#160; &#160; 此外，这里还要说一下：<font color="red">Safari浏览器不允许音频在未经过用户交互的情况下自动播放，所以上述实现中，在Safari下闹钟计时结束后，是不能播放音乐的。</font>然而目前Chrome暂无此限制，若需要解决Safari该限制问题，可以换一种实现思路，即在与用户交互时占用播放，在真正需要播放声音的时候，才播放音乐。

&#160; &#160; &#160; &#160; 最后，补充一下 CSS 样式：
```css
.container {
    width: 80%;
    height: 60%;
    margin: 0 auto;
    margin-top: 50px;
    text-align: center;
    background-color: pink;
    border: 2px solid #808080;
    display: block;
    padding: 50px;
}

.dialogmask { /* 弹窗所用 */
    position: fixed;
    top: 0px;
    height: 100%;
    width: 100%;
    z-index: 80;
    display: none;
}

.opacity { /* 弹窗所用 遮罩浑浊处理*/
    opacity: 0.3;
    filter: alpha(opacity=30);
    /* background-color: #000; */
}

.box { /* 弹窗所用 */
    overflow: hidden;
    position: absolute;
    width: 60%;
    height: 70%;
    z-index: 100; /*值越大，和其他层层叠时越在上面*/
    left: 20%;
    top: 15%;
    background-color: #fff;
    /* border: 2px solid rgb(0, 153, 153);*/
    border: 2px solid #666;
    border-radius: 10px;
    box-shadow: 10px 10px 10px #888888;
}

.alarmbox {
    display: inline-block;
    text-align: center;
    color: cornflowerblue;
    font-size: 50px;
}

.dialogtitle {
    width: 100%;
    height: 30px;
    line-height: 30px;
    position: absolute;
    font-size: 18px;
    top: 0px;
    background-color: lightgrey;
}

.dialogcontent {
    padding-top: 20px;
    OVERFLOW: scroll;
    height: calc(100% - 20px);
    height: -webkit-calc(100% - 20px);
}

.logcontent {
    padding: 10px;
}
 
.time {
    display: inline-block;
    width: 100px;
    margin: 0 auto;
    text-align: center;
    color: cornflowerblue;
    font-size: 30px;
    margin-top: 10px;
}

.fl {
    float: left;
}

.fr {
    float: right;
}

.set {
    width: 100%;
    color: gray;
    text-align: center;
    margin-top: 80px;
}

.container input {
    width: 60px;
    padding: 4px 8px;
    background-color: rgba(246, 192, 242, 0.7);
    outline: none;
}

.container button {
    margin-top: 6px;
    border: 2px solid #666;
    border-radius: 5px;
    background-color: pink;
    display: inline-block;
}
```
