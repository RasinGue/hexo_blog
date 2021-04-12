---
title: 重返Gamemaker Studio 入门篇2
date: 2021-03-23 19:35:55
tags: Gamemaker Studio
---

在线文档: [GameMaker Studio 2 Manual](https://manual-en.yoyogames.com/#t=Content.htm)
官方社区: [GameMaker Community](https://forum.yoyogames.com/index.php)
知识库：[GameMaker Knowledge Base](https://help.yoyogames.com/hc/en-us/categories/204246668-GameMaker-Studio-2)

本系列参考的教程：[GameMaker Studio 2: Complete Platformer Tutorial](https://www.youtube.com/watch?v=izNXbMdu348&list=PLPRT_JORnIupqWsjRpJZjG07N01Wsw_GJ)

------

## 添加枪械

人物将会携带枪支进行射击。首先要添加枪械的贴图和对象。由于枪械是一个长矩形，并且根据常识，人物应该在枪械的后半部把持着。因此，将枪械的原点设在偏后半部的中心位置比较妥当。

![Gun](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210227201918.png)

再者，枪械一般位于胸部位置，而玩家贴图是Q版一比一身材，因此将其置于身体中心再向下一些的位置。我们在 `oGun` 对象的 `BeginStep` 事件中添加代码：

```
x = oPlayer.x
y = oPlayer.y + 9 //较为合适的位置
```

我们希望枪械的前端将会指向鼠标的方向，并且鼠标在人物左右的时候，枪械也会自动跟随鼠标翻转：

```
image_angle = point_direction(x, y, mouse_x, mouse_y) //鼠标的坐标
if (image_angle > 90) and (image_angle < 270){ //逆时针计算角度，零度为正下方
    image_yscale = -1
}
else{
    image_yscale = 1
}
```

## 添加子弹

子弹是一个两帧动画，首先是一个圆形，之后变成一个带着小尾巴的圆。我们希望子弹在发射出去之后，保持第二帧动画不变，直到碰到墙壁消失。那么可以通过 `oBullet`对象的 `Animation End` 事件代码实现：

```
image_speed = 0 //当动画结束时，将动画速度设为0，即不再播放
image_index = 1 //将动画帧数设为第二帧
```

![Bullet](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210227201857.png)

接下来，在 `Post Draw` 事件中，我们添加碰到墙体后子弹销毁的代码：

```
if (place_meeting(x, y, oWall))
    instance_destroy() //该实例销毁
```

这里的 `Post Draw` 事件，是指在绘制一旦结束之后就立即检测。官方文档的解释是：

> The **Post Draw** event is triggered after the standard draw events, but before the Draw GUI events. Like the Pre Draw event, it is based on the size of the screen buffer size, and is placed before the Draw GUI events to enable you to perform post-processing effects and other things on a full screen basis simply and easily without interfering with any HUD/GUI elements that you may have in your game.

论坛中网友的解释是

> the Post Draw event is the moment in between the Draw events and the image being sent to the screen.

简单点理解就是，游戏引擎在绘制之后，还没有把绘制完成的东西送到显示屏上就立即执行的事件。

## 枪械射击与后坐力

枪械连击中间需要加入一个延迟，并且枪械在发射子弹的过程中最好还带有一个后坐力的效果。首先我们先在 `oGun` 的 `Create` 事件中声明一下相关变量:

```
firingdelay = 1 // 发射间隔
recoil = 0 // 后坐力
```

依旧是在 `Begin Step` 事件中，我们添加对于射击延迟和后坐力的一个检测：

```
firingdelay = firingdelay - 1 // 每个时间片减少1计时
recoil = max(0, recoil - 1) // 后坐力取0和当前值减1的最大值

if (mouse_check_button(mb_left)) && (fd<0){ //当鼠标左键点击并且延迟计时小于0时
    recoil = 4  //设定后坐力
    fd = 5  //设定设计间隔
    
    with (instance_create_layer(x, y, "Bullets", oBullet)){ //with 语句
        speed = 25 // 将对应创建的新bullet对象赋予速度
        direction = other.image_angle + random_range(-3, 3) // other 指with中指代的对象，这里加入一个随机数使得子弹不会单纯往一个方向设计，而是在一个小的随机范围内发射
        image_angle = direction //重新设定子弹行进的方向
    }	
}

x = x - lengthdir_x(recoil, image_angle) //往发射角度的反向延长线上后退
y = y - lengthdir_y(recoil, image_angle) 
```

官方文档中对于 `with` 语句的解释是：

> In a number of cases you want to do a lot more than just change a single variable within those other instances, and may want to perform more complex code actions that require multiple functions and lines of code.

> Once the expression has set the scope of the with, the statement will then be executed for each of the indicated instances, as if that instance is the current (self) instance.

也就是说，`with` 可以再指定一个新对象，这个新对象在 `with` 语句中用 `other` 来指代。

这时候要记得在 `Room1` 中添加两个新层 `Bullets` 和 `Gun`，这两层应置于 `Player`上方。
