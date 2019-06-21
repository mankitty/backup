---
title: ubuntu16.04 QT环境配置
date: 2019-06-20 15:33:33
tags:
categories: QT
---
## QT安装（Qt Library）
qt下载镜像链接**http://download.qt.io/archive/qt/**，选择合适的版本下载，我这里选择的是http://download.qt.io/archive/qt/5.6/5.6.3/qt-opensource-linux-x64-5.6.3.run
## 安装QT
1. sudo chmod +x qt-opensource-linux-x64-5.6.3.run
2. sudo ./qt-opensource-linux-x64-5.6.3.run
3. qtchooser -install qt5.6 /opt/Qt5.6.3/5.6.3/gcc_64/bin/qmake
4. export QT_SELECT=qt5.6