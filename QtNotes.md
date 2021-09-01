
#### 写入注册表

```makedown
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QSettings>
#include <QLabel>

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    
    
    //输出键值
    QLabel *label = new QLabel(this);
    label->setGeometry(QRect(50,50,200,25));
    //实例QSettings
    QSettings *reg = new QSettings("HKEY_CURRENT_USER\\Software\\Qt01",
    QSettings::NativeFormat);
    //判断value是否为空 不为空则输出
    if(reg->value("键名001") != "")
    {
    label->setText("键名001::"+reg->value("键名001").toString());
    }
    //删除QSettings
    delete reg;
}

MainWindow::~MainWindow()
{
    delete ui;
}


```


#### 查找注册表

```makedown
#include <QSettings>
#include <QLabel>

//输出键值
QLabel *label = new QLabel(this);
label->setGeometry(QRect(50,50,200,25));
//实例QSettings
//参数1：如果没有按照章节Qt01进行，则注册表中没有Qt01。
QSettings *reg = new QSettings("HKEY_CURRENT_USER\\Software\\Qt01",
QSettings::NativeFormat);
//判断value是否为空，不为空则输出
if(reg->value("键名001") != "")
{
label->setText("键名001::"+reg->value("键名001").toString());
}
//删除QSettings
delete reg;

```












