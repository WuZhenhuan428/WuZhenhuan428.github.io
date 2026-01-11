---
date: 2026-01-12 03:14:00+08:00
title: "关于信号槽机制和前后端分离的一些想法"
descpiption: ""
categories:
    - Qt
tags:
    - Qt
weight: 1
---

# Qt的信息传输机制
Qt的信息传输通过信号和槽的连接来实现，发射方和接收方彼此不知道也不应该直到对方的存在。作为GUI库，这样做无疑能够保证前后端最大程度的解耦。

# 类之间的引用关系
以音乐播放器为例，假设`main`函数下有`MainWindow`和`Player`两个类，其中`MainWindow`引用了`Player`，即：

```cpp
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Player player;
    MainWindow w(&player);
    /*
        TODO
    */
    w.show();
    return a.exec();
}
```
在这种情况下，前端`MainWindow`向后端传输信号是理所当然的，可以直接在`MainWindow`类中通过connect完成。

而在为了保证不反向引用，则需要回到顶层模块，也就是`main`函数中对两个信号进行连接，即在`TODO`中写入如下代码：
```cpp
QObject::connect(&w, &MainWindow::filepathChanged, &player, &Player::read);
```

另一方面，出于方便考虑在`MainWindow`类中引用例如`Player`类，但是`Player`类最好只用于在`connect`中调动槽函数，而不是直接调用内部的方法。

# 总结和思考

前后端以及模块之间的解耦是必要的，Qt更是直接基于这种思想来设计信号传输机制。在上述例子中，需要保证信号的方向只能是`UI->逻辑`。

换句话说，下级模块只应该通过接受的信号进行工作，而永远不应该知道有上级模块的存在。

# 每日碎碎念

放弃ErgoDox的想法了。打算设计一个简单的直列键盘，均衡一下双手的负载，把右边多出来的两排符号放到左右手区域之间也许会有不错的效果。此外电路板的面积相比ErgoDox会小不少。

Qt Creator即使不使用Designer，只作为IDE来使用体验也不错。

这几天作息完全混乱了。没想到编程竟然有着让人彻夜难眠程度的能力（笑）
