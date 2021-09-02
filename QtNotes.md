
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


#### 修改IE浏览器的默认主页


```makedown
#include <QSettings>
//实例QSettings
QSettings *reg = new
QSettings("HKEY_CURRENT_USER\\Software\\Microsoft\\Internet
Explorer\\Main",
QSettings::NativeFormat);
//判断value是否为空，不为空则输出
if(reg->value("Start Page") != "")
{
//IE默认主页修改为：百度首页
reg->setValue("Start Page","http://www.baidu.com");
}
//删除QSettings
delete reg;

```



#### 进程管理器

mainwindow.h

```makedown
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QTableView>
#include <QStandardItemModel>
#include <QDebug>
#include "windows.h" //1顺序不可颠倒
#include "tlhelp32.h" //2
#include <QPushButton>
#include <QMessageBox>
#include <QProcess>

namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private:
    Ui::MainWindow *ui;

private:
    QTableView *tableView;
    QStandardItemModel *model;
    QString cPid;
private slots:
    //QTableView行事件
    void sendContent(QModelIndex);
    //点击删除事件
    void deletePro();
};

#endif // MAINWINDOW_H

```

mainwindow.cpp

```makedown
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);


    //实例QListView
    tableView = new QTableView(this);
    //QListView位置
    tableView->setGeometry(QRect(10,10,340,430));
    //实例QPushButton
    QPushButton *button = new QPushButton(this);
    button->setGeometry(QRect(260,450,80,25));
    button->setText("结束进程");
    connect(button,SIGNAL(clicked()),this,SLOT(deletePro()));
    //实例数据模型
    model = new QStandardItemModel();
    model->setHorizontalHeaderItem(0,new QStandardItem("进程名"));
    model->setHorizontalHeaderItem(1,new QStandardItem("PID"));
    //获取系统快照句柄，可以获取进程、线程、模块、进程使用的堆的句柄
    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hProcessSnap == FALSE )
    {
    qDebug() << "调用失败";
    return;
    }
    PROCESSENTRY32 pe32;
    pe32.dwSize = sizeof(PROCESSENTRY32);
    BOOL bRet = Process32First(hProcessSnap, &pe32);
    int i = 0;
    while (bRet)
    {
    i++;
    //前面2行不要
    if(i > 1)
    {
    //获取进程名
    QString pName = QString::fromWCharArray(pe32.szExeFile);
    //追加数据模型第一列
    model->setItem(i-2,0,new QStandardItem(pName));
    //获取PID号
    QString pid = QString::number(pe32.th32ProcessID);
    //追加数据模型第二列
    model->setItem(i-2,1,new QStandardItem(pid));}
    bRet = Process32Next(hProcessSnap, &pe32);
    }
    //清理hProcessSnap
    ::CloseHandle(hProcessSnap);

    //绑定数据
    tableView->setModel(model);
    //列宽
    tableView->setColumnWidth(0,190);
    tableView->setColumnWidth(1,130);
    //选取整行
    tableView->setSelectionBehavior(QAbstractItemView::SelectRows);
    //不可修改
    tableView->setEditTriggers(QAbstractItemView::NoEditTriggers);
    //行单击事件
    connect(tableView,SIGNAL(clicked(QModelIndex)),SLOT(sendContent(QModelIndex)));
    //纵向头隐藏
    tableView->verticalHeader()->setVisible(false);
    //关闭滚动条
    //tableView->setVerticalScrollBarPolicy(Qt::ScrollBarAlwaysOff);
    //隔行变色
    tableView->setAlternatingRowColors(true);
}

MainWindow::~MainWindow()
{
    delete ui;
}


//QTableView点击行事件
void MainWindow::sendContent(QModelIndex)
{
    //获取点击行索引
    int index = tableView->currentIndex().row();
    //将PID值赋予QString类型保存
    cPid = model->data(model->index(index,1)).toString();
}

//点击结束进程
void MainWindow::deletePro()
{
    if(cPid == "")
    {
        QMessageBox::warning(this,"警告",tr("请选择要结束的进程!!!"));
    }
    else
    {
        //Taskkill/pid 3184
        //结束进程命令
        QProcess process(0);
        QString dos = "Taskkill";
        QStringList list;
        list.append("/pid");
        list.append(cPid);
        //拼接参数，因为QProcess 不支持中文与空格
        process.start(dos,list);
        QMessageBox::warning(this,"警告",tr("成功结束!!!"));
    }
}

```







