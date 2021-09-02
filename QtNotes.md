
#### 1、写入注册表

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


#### 2、查找注册表

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


#### 3、修改IE浏览器的默认主页


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



#### 4、进程管理器

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

#### 5、线程QThread应用

mainwindow.h

```makedown
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QPushButton>
#include <QThread>
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
    QPushButton *starThread;
    QThread *thread;
private slots:
    //执行线程
    void startFun();
};

#endif // MAINWINDOW_H
```


mainwindow.cpp

```makedown
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QDebug>

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    //执行线程1
    starThread = new QPushButton(this);
    starThread->setText("执行新线程");
    starThread->setGeometry(QRect(30,50,80,25));
    connect(starThread,SIGNAL(clicked()),this,SLOT(startFun()));

}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::startFun()
{
    //实例线程
    thread = new QThread();
    //启动
    thread->start();
    int i=0;
        while(true)
        {
            //挂起1秒
            thread->sleep(1);
            i++;
            qDebug() <<"print："<<i;
            if(i > 5)
            {
                break;
            }
        }
    qDebug() << "结束";
}

```



#### 6、线程QRunnable应用


QRunnable类
QRunnable类是所有可运行对象的基类。
QRunnable 类是一个接口，用于表示需要执行的任务或代码段，由您重新实现的 run() 函数表示。
您可以使用 QThreadPool 在单独的线程中执行您的代码。 如果 autoDelete() 返回 true（默认值），QThreadPool 会自动删除 QRunnable。 使用 setAutoDelete() 更改自动删除标志。
QThreadPool 通过在 run() 函数中调用 QThreadPool::tryStart(this) 支持多次执行相同的 QRunnable。 如果启用了 autoDelete，则当最后一个线程退出 run 函数时，QRunnable 将被删除。 当启用 autoDelete 时，使用相同的 QRunnable 多次调用 QThreadPool::start() 会产生竞争条件，因此不推荐使用。

pure virtual void QRunnable::run()
在你的子类中实现这个纯虚函数。
void QRunnable::setAutoDelete(bool autoDelete)
如果 autoDelete 为真，则启用自动删除；否则自动删除被禁用。
如果开启了自动删除，QThreadPool会在调用run()后自动删除这个runnable；否则，所有权仍属于应用程序员。
请注意，必须在调用 QThreadPool::start() 之前设置此标志。在 QThreadPool::start() 之后调用此函数会导致未定义的行为。


mainwindow.h
```makedown
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QPushButton>
#include <QProgressBar>

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
    QPushButton *startButton;
    QProgressBar *progressBar;
    QProgressBar *progressBar2;
private slots:
    void startFun();
};

#endif // MAINWINDOW_H

```


mainwindow.cpp

```makedown
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QProgressBar>
#include <QThreadPool>
#include <runnable.h>

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);


    //第一个QProgressBar
    progressBar = new QProgressBar(this);
    progressBar->setGeometry(QRect(50,50,200,16));
    //初始值
    progressBar->setValue(0);
    //范围值
    progressBar->setRange(0,100000-1);
    //第二个QProgressBar
    progressBar2 = new QProgressBar(this);
    progressBar2->setGeometry(QRect(50,100,200,16));
    //初始值
    progressBar2->setValue(0);
    //范围值
    progressBar2->setRange(0,100000-1);
    //实例按钮
    startButton = new QPushButton(this);
    startButton->setGeometry(QRect(260,45,80,25));
    startButton->setText("执行");
    connect(startButton,SIGNAL(clicked()),this,SLOT(startFun()));
}

MainWindow::~MainWindow()
{
    delete ui;
}

//开始执行
void MainWindow::startFun()
{
    //实例runnable类
    runnable *hInst = new runnable(progressBar);
    //实例第二个
    hInst = new runnable(progressBar2);
    //进程池
    QThreadPool::globalInstance()->start(hInst);
}

```


runnable.h

```makedown
#ifndef RUNNABLE_H
#define RUNNABLE_H

#include <QRunnable>
#include <QProgressBar>
class runnable : public QRunnable
{
    public:
    runnable(QProgressBar *progressBar);
    virtual ~runnable();
    void run();
    private:
    QProgressBar *m_ProgressBar;
};

#endif // RUNNABLE_H

```

runnable.cpp

```makedown
#include <runnable.h>
#include <QProgressBar>
runnable::runnable(QProgressBar *progressBar):
QRunnable(), m_ProgressBar(progressBar)
{
    for(int i=0;i<100000;i++)
    {
        progressBar->setValue(i);
    }
}
runnable::~runnable(){}

void runnable::run(){}

```
















