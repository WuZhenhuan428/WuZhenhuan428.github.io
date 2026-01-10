---
date: 2026-01-10 02:37:00+08:00
title: '初学者对qt运行机制的一些思考'
descpiption: ""
categories:
    - Qt
tags:
    - Qt
weight: 1
---

这里是作为Qt初学者的W

# 背景

虽然接触Qt已经有几天了，但是由于各种事情，直到今天才开始正式写一些Qt代码。

相信和很多人一样，大家认为好用的GUI反而成为了设计的路障。刚接触Qt Creator时认为图形化设计足够高效，但不久就被Designer与文件之间的复杂关系难住。

于是我打算切换到通过纯代码来构建Qt程序。虽然要写更多的代码，但是心智上的负担更小了。

# 实践

于是我通过搜索资料，~~抄~~写了一个dialog类型的demo
> 顺带一提，官网的Tutorial全是炫技，Documentation又是平铺式的列出库的内容，想找一个简单易懂的入门内容还真不容易^^;

main.cpp没有改动：

```cpp
#include "dialog.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Dialog w;
    w.show();
    return a.exec();
}
```

dialog.h:

```cpp
#ifndef DIALOG_H
#define DIALOG_H

// Widgets
#include <QDialog>
#include <QCheckBox>
#include <QRadioButton>
#include <QPushButton>
#include <QPlainTextEdit>

// Layout
#include <QHBoxLayout>
#include <QVBoxLayout>

// Text Editor
#include <QPalette>

class Dialog : public QDialog
{
    Q_OBJECT

public:
    Dialog(QWidget *parent = nullptr);
    ~Dialog();

private slots:
    void onCheckBoxBold(bool checked);
    void onCheckBoxUnderline(bool checked);
    void onCheckBoxItalic(bool checked);
    void setTextColor();

private:
    // Widgets declaration
    QCheckBox* CheckBoxBold;
    QCheckBox* CheckBoxUnderline;
    QCheckBox* CheckBoxItalic;

    QRadioButton* RadioButtonRed;
    QRadioButton* RadioButtonGreen;
    QRadioButton* RadioButtonBlue;

    QPlainTextEdit* TextEditor;

    QPushButton* PushButtonOK;
    QPushButton* PushButtonCancel;
    QPushButton* PushButtonExit;

    // Functions prototype declaration
    void InitUI();
    void InitSignalSlots();
};

#endif // DIALOG_H
```

dialog.cpp内容如下，没有细分出更多的文件。

```cpp
#include "dialog.h"

Dialog::Dialog(QWidget *parent)
    :QDialog(parent)
{
    InitUI();
    InitSignalSlots();
};

Dialog::~Dialog() {}

void Dialog::InitUI()
{
    // Create layout
    CheckBoxBold = new QCheckBox("Bold");
    CheckBoxUnderline = new QCheckBox("underline");
    CheckBoxItalic = new QCheckBox("Italic");

    RadioButtonRed = new QRadioButton("Red");
    RadioButtonGreen = new QRadioButton("Green");
    RadioButtonBlue = new QRadioButton("Blue");

    TextEditor = new QPlainTextEdit;
    TextEditor->setPlainText("Default text.");

    PushButtonOK = new QPushButton("OK");
    PushButtonCancel = new QPushButton("Cancel");
    PushButtonExit = new QPushButton("Exit");

    QHBoxLayout* HLay1 = new QHBoxLayout;
    HLay1->addWidget(CheckBoxUnderline);
    HLay1->addWidget(CheckBoxBold);
    HLay1->addWidget(CheckBoxItalic);

    QHBoxLayout* HLay2 = new QHBoxLayout;
    HLay2->addWidget(RadioButtonRed);
    HLay2->addWidget(RadioButtonGreen);
    HLay2->addWidget(RadioButtonBlue);

    QHBoxLayout* HLay3 = new QHBoxLayout;
    HLay3->addWidget(PushButtonOK);
    HLay3->addWidget(PushButtonCancel);
    HLay3->addStretch();
    HLay3->addWidget(PushButtonExit);

    QVBoxLayout* VLay1 = new QVBoxLayout;
    VLay1->addLayout(HLay1);
    VLay1->addLayout(HLay2);
    VLay1->addWidget(TextEditor);
    VLay1->addLayout(HLay3);

    setLayout(VLay1);
}

void Dialog::InitSignalSlots()
{
    // Connect checkbox
    connect(CheckBoxBold, SIGNAL(clicked(bool)), this, SLOT(onCheckBoxBold(bool)));
    connect(CheckBoxUnderline, SIGNAL(clicked(bool)), this, SLOT(onCheckBoxUnderline(bool)));
    connect(CheckBoxItalic, SIGNAL(clicked(bool)), this, SLOT(onCheckBoxItalic(bool)));

    // Connect radio button
    connect(RadioButtonRed, SIGNAL(clicked()), this, SLOT(setTextColor()));
    connect(RadioButtonGreen, SIGNAL(clicked()), this, SLOT(setTextColor()));
    connect(RadioButtonBlue, SIGNAL(clicked()), this, SLOT(setTextColor()));

    // Connect button
    connect(PushButtonOK, SIGNAL(clicked()), this, SLOT(close()));
    connect(PushButtonCancel, SIGNAL(clicked()), this, SLOT(close()));
    connect(PushButtonExit, SIGNAL(clicked()), this, SLOT(close()));
}

// Slot function implementation
void Dialog::onCheckBoxBold(bool checked)
{
    QFont font = TextEditor->font();
    font.setBold(checked);
    TextEditor->setFont(font);
};

void Dialog::onCheckBoxUnderline(bool checked)
{
    QFont font = TextEditor->font();
    font.setUnderline(checked);
    TextEditor->setFont(font);
};

void Dialog::onCheckBoxItalic(bool checked)
{
    QFont font = TextEditor->font();
    font.setItalic(checked);
    TextEditor->setFont(font);
};

void Dialog::setTextColor()
{
    QPalette palette = TextEditor->palette();

    if(RadioButtonRed->isChecked())
    {
        palette.setColor(QPalette::Text, Qt::red);
    }
    else if(RadioButtonGreen->isChecked())
    {
        palette.setColor(QPalette::Text, Qt::green);
    }
    else if(RadioButtonBlue->isChecked())
    {
        palette.setColor(QPalette::Text, Qt::blue);
    }
    // Write back
    TextEditor->setPalette(palette);
};
```

代码本身不复杂，整体流程就是在Dialog类中创建了各种对象，在构造函数中进行布局和信号槽连接，并实现了信号的触发事件。真正值得思考的是关于Qt的运行机制。

# 思考

与平常接触的嵌入式C掌控全局的开发方式不同，开发Qt程序更像是在FPGA上部署Verilog。

以下是我做的不完全类比：
|          | Qt           | FPGA                  |
| :---:    | :---:        | :---:                 |
| 运行环境 | QApplication | FPGA架构+开发工具联合 |
| 层次     | QObject      | 层次信息              |
| 子模块   | 头文件库     | IP核                  |
| 约束关系 | 设置布局     | 连线/时序约束         |
| 通讯方式 | 信号槽       | 信号线/标志位         |
| 逻辑描述 | 逻辑代码     | Verilog代码           |

类似之处：
1. 同样是基础设施，QApplication和FPGA架构+EDA工具链一样，用户无需（极少）关注内部的实现，只需要在既定的框架中添加自己的内容即可。可以说是提供了运行环境。
2. 引用各种库的过程类似于调用IP核，无需关注内部实现，只需要关注接口实现、行为和使用约束。另一方面，头文件和IO名称都可以直接反映出大量的信息。
3. 在确定好需要调用的模块后，通过信号槽/连线的方式约束逻辑块的行为，无需设计额外的接口获取逻辑块内部的信息。
4. 类似于逻辑设计不关心具体的连线，Qt的widget组件不关心自己的大小，而是由布局和窗口大小进行约束。
5. Qt C++源文件则类似于HDL，只是在行为上描述了要做什么/不能做什么，具体的实现由工具链决定。

不同之处：
1. Qt的信号是动态的，可能来自函数调用、事件投递或者消息队列，而数字设计的信号路径是固定的，有固定的时序模型。
2. Qt的`connect`是动态的，而数字设计的`netlist`是固定的。
3. Qt可以通过反馈改变系统本身，相当于FPGA改变了连线规则，这在硬件中是不可能的。

# 总结

对于设计Qt程序的人来说，不是嵌入式开发那样的掌控者，更像是一个协作者。通过描述规则，让Qt框架执行流程并产生对应的内容，同时能接受来自外部的刺激信号，因此有了与外界交互的能力。

# 碎碎念

正确的类比能明显减轻心智负担（笑）

为了写博客而熬夜不是长久之计

工夫红茶没有毛尖好喝