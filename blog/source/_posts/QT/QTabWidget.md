---
title: 用于示范QTabWidget的用法
date: 2019-12-17 21:33:33
tags:
categories: QT
---

## QTabWidget重要函数
```
public:
void setTabText(int, QString);  /* 设置页面的名字. */
void setTabToolTip(QString); /* 设置页面的提示信息. */
void setTabEnabled(bool); /* 设置页面是否被激活. */
void setTabPosition(QTabPosition::South); /* 设置页面名字的位置.*/ 
void setTabsClosable(bool); /* 设置页面关闭按钮. */
int currentIndex(); /* 返回当前页面的下标，从0开始. */
int count(); /* 返回页面的数量. */
void clear(); /* 清空所有页面. */
void removeTab(int); /* 删除页面. */ 
void setMoveable(bool); /* 设置页面是否可被拖拽移动. */ 
void setCurrentIndex(int); /* 设置当前显示的页面. */

signals:
void tabCloseRequested(int). /* 当点击第参数个选项卡的关闭按钮的时候，发出信号. */
void tabBarClicked(int). /* 当点击第参数个选项卡的时候，发出信号. */
void currentChanged(int). /* 当改变第参数个选项卡的时候，发出信号. */
void tabBarDoubleClicked(int). /* 当双击第参数个选项卡的时候，发出信号 */

QTabWidget设置setTabsClosable(true)后所有加进来的tab上都会出现关闭按钮，然后利用QTabWidget的tabCloseRequested(int)信号实现tab的关闭。
```
## 使用示例
### main.cpp
```
#include "Widget.h"
#include <QApplication>
 
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;
    w.show();
    return a.exec();
}
```
### Widget.h
```
#ifndef WIDGET_H
#define WIDGET_H
 
#include <QWidget>
#include <QTabWidget>
 
class Widget : public QWidget
{
    Q_OBJECT
 
    QTabWidget m_tabWidget;
 
protected slots:
    void onTabCurrentChanged(int index);
    void onTabCloseRequested(int index);
        public:
    Widget(QWidget *parent = 0);
    ~Widget();
};
 
#endif
```
### Widget.cpp
```
#include "Widget.h"
#include <QPlainTextEdit>
#include <QLabel>
#include <QPushButton>
#include <QVBoxLayout>
#include <QDebug>
 
Widget::Widget(QWidget *parent)
            : QWidget(parent)
{
    m_tabWidget.setParent(this);
    m_tabWidget.move(12,13);
    m_tabWidget.resize(500,500);
    m_tabWidget.setTabPosition(QTabWidget::South); //标签的位置
    m_tabWidget.setTabShape(QTabWidget::Triangular); //标签的外观
    m_tabWidget.setTabsClosable(true); //可关闭标签
 
    QPlainTextEdit* edit = new QPlainTextEdit(&m_tabWidget);
    edit->insertPlainText("lst Tab Page");
 
    m_tabWidget.addTab(edit, "1st");
 
    QWidget* widget = new QWidget(&m_tabWidget);
    QVBoxLayout* layout = new QVBoxLayout();
    QLabel* lbl = new QLabel(widget);
    QPushButton* btn = new QPushButton(widget);
 
    lbl->setText("2nd Tab Page");
    lbl->setAlignment(Qt::AlignCenter);
 
    btn->setText("2nd Tab Page");
 
    layout->addWidget(lbl);
    layout->addWidget(btn);
 
    widget->setLayout(layout);
 
    m_tabWidget.addTab(widget, "2nd");
 
    m_tabWidget.setCurrentIndex(1); //设置第2个标签页为当前页面
 
    connect(&m_tabWidget, SIGNAL(currentChanged(int)), this, SLOT(onTabCurrentChanged(int)));
    connect(&m_tabWidget, SIGNAL(tabCloseRequested(int)), this, SLOT(onTabCloseRequested(int)));
}
 
void Widget::onTabCurrentChanged(int index)
{
    qDebug() << "Page change to: " << index;
}
 
void Widget::onTabCloseRequested(int index)
{
    m_tabWidget.removeTab(index);
}
 
Widget::~Widget()
{
 
}
```