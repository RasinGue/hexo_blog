---
title: 重返Gamemaker Studio 入门篇1
date: 2021-02-23 19:35:55
tags: Gamemaker Studio
---

> 空有雄心壮志，不如脚踏实地从简单做起。

经历了三年空窗期，虽说实际情况产生了不可计量的变化，也和当时想做游戏的简单初衷相去甚远。但愿望如果不开始付诸实际，只会停留在空想。因此还是准备从简单做起，不想因为复杂的实现和编程花费太多时间，而把重心侧重在玩法和构思上面，GameMaker Studio 2 貌似是一个比较折中的选择。闲话少叙，从一个简单的平台冒险类游戏下手，基本能够达到目前构思游戏demo的制作需求。本系列的目的是在跟着教程实践的基础上，对游戏制作过程的一个简单记录，仅供自己回顾，不做教程目的。

------

在线文档: [GameMaker Studio 2 Manual](https://manual-en.yoyogames.com/#t=Content.htm)
官方社区: [GameMaker Community](https://forum.yoyogames.com/index.php)
知识库：[GameMaker Knowledge Base](https://help.yoyogames.com/hc/en-us/categories/204246668-GameMaker-Studio-2)

本系列参考的教程：[GameMaker Studio 2: Complete Platformer Tutorial](https://www.youtube.com/watch?v=izNXbMdu348&list=PLPRT_JORnIupqWsjRpJZjG07N01Wsw_GJ)

------

## 界面介绍

在打开游戏IDE之后，我们可以看到几个区域。如图所示，中间部分是工作区，所有基本的元素编辑和代码撰写都在这里。右边是素材区，已经被多个文件夹详细划分。左上是菜单栏和工具栏，最左一栏为房间设置栏，之后再详细讨论其使用方法。

![IDE](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210223200012.png)

工作区的操作同大多数游戏一样，通过滚轮滑动调整上下，点击中键进行移动，按住`Ctrl`键加上滚轮滑动可以控制工作区的界面缩放，也可以使用 `Ctrl + +` 和 `Ctrl + -` 达到相同效果。

再进行工作的时候，可以使用快捷键 `F12` 将工作区最大化，方便编辑。

## 创建贴图

可以直接在工作区里点击 `右键→Assets→Create Sprite`；或者在素材区对应的文件夹里如前者一般创建。唯一的区别是，直接在工作区里创建的文件存储在根目录，而对应文件夹创建可以更加分门别类，当然也可以重新拖放文件达到相同效果。同理，其他需要创建的东西也可以这样操作。

以下就是一个空的贴图文件配置。

![sprite](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210223200815.png)

贴图可以根据我们的需要进行命名，命名一般需要规则，如玩家静止贴图用 `sPlayer`, 玩家跑动贴图用 `sPlayerR`，玩家跳跃贴图用 `sPlayerA`。下方的 `Size` 需要注意，一般是 `64*64` 的像素大小，也可以点击按钮进行调整。右边的 `Edit Image` 可以直接在编辑器里面编辑。我个人觉得还是使用更完备的外部编辑器画好再导入进来，比如 `Aseprite` 或 `Photoshop`等等。下面的 `Import` 按钮可以导入图片。导入图片可以是单张贴图或者多帧动画。注意这里导入的动画实际上是拼贴好的 `png` 文件，文件名以 `_strip(num)` 结尾，其中 `num` 是实际上帧的数量。这样导入的图片将会自动被切割成 `num` 数量的帧。如下图所示，一张拼贴好的4帧 `sPlayer_r_strip4.png` 文件：

![strip_4](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210223202502.png)

左下角的 `Collision Mask` 是用于检测游戏对象碰撞的大小，可以通过手动自行设置或者自动检测来调整。

右侧是动画界面，右上角是动画帧率。注意这里的帧率**仅用于显示**，而**不是**指游戏内的**实际帧率**。右边的坐标指定该贴图的圆心位置，一般可以直接使用 `Middle Centre` 让系统自动指定。

这里我们创建了两个基本的贴图：`sPlayer`和 `sWall`，并将素材导入。

![](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210223214139.png)

## 创建对象

贴图只是纯粹的图片，不能执行游戏中的指令和效果，因此就需要创建对象并绑定贴图。首先，我们先创建一个玩家对象 `oPlayer`，并将刚才创建好的 `sPlayer` 进行绑定。

![oPlayer](https://raw.githubusercontent.com/rasin-tsukuba/blog-images/master/img/20210223214535.png)

下方的 `Events` 可以用来管理对象相关的所有事件。点击并创建新的事件。首先，`Create` 事件对应在对象创建时，我们需要初始化一些简单的变量：

```
hsp = 0    //水平速度
vsp = 0    //垂直速度
grv = 0.3  //重力
walksp = 4 //行走速度
```

紧接着，我们创建 `Step`事件。`Step` 事件管理每个时间片里面需要执行的内容，因此代码会较为复杂。首先定义以下按键：

```
//Get Player Input
key_left = keyboard_check(vk_left) || keyboard_check(ord("A")) //方向键左或者键A
key_right = keyboard_check(vk_right)|| keyboard_check(ord("D")); //方向键右或者键D
key_jump = keyboard_check_pressed(vk_space)|| keyboard_check_pressed(ord("W")); //空格键或键W，仅需点击即可
```

这样就定义好了基本的行动按键。接下来，对应的按键要加上其逻辑才能有实际意义。我们继续定义一下运动的状态：

```
var move = key_right - key_left // `var` 用于声明局部变量
hsp = move * walksp // 水平速度等于行动方向乘行走速度
vsp = vsp + grv;  // 垂直速度等于现有垂直速度加上重力速度
```

首先定义了一个临时变量 `move`。`move` 在这里是一个布尔值，当 `key_right`为 `True` 且 `key_left` 为 `False` 时，`move`值为 `1` ；当 `key_right`为 `False` 且 `key_left` 为 `True` 时，`move` 值为 `-1` 。这样就可以将作为一个方向的标识。

正如我们在 `Create` 事件中定义的几个基础的速度变量，在 `Step` 事件中需要对行进速度进行实时更改。对于水平速度如此定义无可厚非，但垂直速度并没有按照物理学的实际情况算上加速度，只是简单的通过在每个时间片减去重力的速度达到下落的效果。

## 碰撞检测

如果不做碰撞检测，那么人物在水平行进和下落的时候就不会被墙阻挡住，就没有办法进行游戏。因此要对人物和墙体之间做碰撞检测。先从跳跃开始：

```
// Jump
if (place_meeting(x, y+1, oWall)) && (key_jump){
    vsp = -7
}
```

`place_meeting` 函数用于检查某个对象实例的 `Collision Mask` 与另一个实例或对象的所有实例的碰撞检测。因此这个函数可以检测玩家对象和所有的墙体，而不需要对单个墙体进行设定。由于房间坐标的左上顶点是 `(0, 0)`，因此垂直的负方向是向上，因此向上跳跃的速度设为负值。触发跳跃的条件是当玩家位于墙对象上分且按下跳跃。

水平移动要算上玩家的移动速度。因此在 `place_meeting` 函数中需要注意：

```
if (place_meeting(x+hsp, y, oWall)){ //加上玩家的速度 也就是下一个时间片原本的位移
    while (!place_meeting(x+sign(hsp), y, oWall)){ //当还没碰到墙时要继续移动，这里用方向表示更简洁
        x = x + sign(hsp) //继续移动到墙边
    }
    hsp = 0 //将移动速度设为0
}

x = x + hsp; //更改玩家横坐标位置
```

同理，垂直移动也可以用类似的代码来实现：

```
//Vertical Collision
if (place_meeting(x, y+vsp, oWall)){
	
	while (!place_meeting(x, y+sign(vsp), oWall)){
		y = y + sign(vsp);
	}
	vsp = 0;
}
y = y + vsp;
```