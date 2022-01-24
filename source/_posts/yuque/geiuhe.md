---
title: Qt 开发经验总结
urlname: geiuhe
date: '2022-01-20 02:51:28 +0800'
categories:
  - 编程
  - Qt
tags:
  - Qt
  - 经验总结
---

记录一些 Qt 开发中的经验总结，可能包含代码设计，踩过的坑等等。

<!-- more -->

### 控件布局和数据渲染分离。

例如对于一个电话列表窗口，在构造函数中进行控件布局但不初始化数据，然后调用 public 的 setPhones 之类接口的进行更新。
​

```cpp
class PhoneTable: public QWidget
{
    Q_OBJECT
public:
    explicit PhoneTable(QWidget *parent = nullptr)
        : QWidget(parent) {
        // 控件容器布局，不渲染具体数据
        QHBoxLayout *mainLay = new QHBoxLayout;
        setLayout(mainLay);
    }

    PhoneTable(const QList<int> &phones, QWidget *parent = nullptr)
        : PhoneTable(parent) {
        // 在这里通过 setPhones 来渲染
        setPhones(phones);
    };
    ~PhoneTable(){};

    // 国际化，更新数据都可以直接调用了
    void setPhones(const QList<int> &phones);
};


class PhoneDialog: public QWidget
{
    Q_OBJECT
public:
    explicit PhoneDialog(QWidget *parent = nullptr)
        : QWidget(parent) {
        PhoneTable *table = new PhoneTable;
        QHBoxLayout *mainLay = new QHBoxLayout;
        mainLay->addWidget(table);
        setLayout(mainLay);

        // maybe do something
        table->setPhones({
                             11111111,
                             22222222,
                             33333333
                         });
    };
};
```

这样做的好处是之后做国际化或者上层控件更新数据的话方便更新该控件。
​

### 数据更新，但控件 UI 没更新

很大可能是调用了某个控件的 exec，然后通过对应的信号来更新数据，这种情况应当在 exec 后补上该控件的 repaint 调用。
​

```cpp
class PlainTextDialog: public QDialog
{
    Q_OBJECT
public:
    explicit PlainTextDialog(QWidget *parent = nullptr);

signals:
    void valueChanged();
};


// 实际调用
QPushButton button("Button");
PlainTextDialog dialog;

QObject::connect(&dialog, &PlainTextDialog::valueChanged,
&button, [&]{
   button.setText("new text"); // UI 没有更新，因为被 exec 阻塞了
});

dialog.exec();
button.repaint(); // UI 更新了
```
