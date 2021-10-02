### Python笔记



#### 1、Excel处理与绘图

```markdown
#!/usr/bin/python
# coding:utf-8

import numpy as np
import matplotlib.pyplot as plt
import os
import xlwt




for i in range(20):
    # i = 2
    i = 57+i
    fileName = "CCD数据整理/CCDTEST_1/"+"第" + str(i) + "个位置"
    path = ''

    # 创建一个workbook 设置编码
    workbook = xlwt.Workbook(encoding='utf-8')
    # 创建一个worksheet
    worksheet = workbook.add_sheet('My Worksheet')

    # 写入excel
    # 参数对应 行, 列, 值
    worksheet.write(0, 0, label='N.O')
    worksheet.write(0, 1, label='Length')
    worksheet.write(0, 2, label='Low')
    worksheet.write(0, 3, label='High')
    worksheet.write(0, 4, label='CCD')

    index = 1
    # #txt file/
    data = np.array(0)

    # 判断结果
    if not os.path.exists('./' + fileName + '/'):
        os.mkdir('./' + fileName)

    with open("./"+fileName+".txt", "r") as f:
        for line in f.readlines():
            line = line.strip('\n')
            # print(line)
            lineData = line.split('，')
            # print(lineData)
            # lineData.remove(len(lineData))
            if(lineData != ['']):
                lineData.pop(len(lineData)-1)
                # lineData[len(lineData)]
                lineArray = np.array(lineData)
                vector = lineArray.astype(float)
                avg = np.average(vector)
                AVG = np.linspace(avg, avg, vector.size)
                # print(np.max(vector))
                avgMinMax = np.max(vector) + np.min(vector)
                # AVGMinMax = np.linspace(avgMinMax/2.0, avgMinMax/2.0, vector.size)
                AVGMinMax = np.linspace(np.max(vector) -150, np.max(vector) -150, vector.size)
                x = np.arange(0, vector.size, 1)
                y = vector

                plt.plot(x, y, 'b.', linewidth=2)
                plt.plot(x, AVG, 'g--', linewidth=2)
                plt.plot(x, AVGMinMax, 'r-', linewidth=2)
                plt.grid(True)

                High = np.sum(y>AVGMinMax)
                Low = np.sum(y <= AVGMinMax)

                num = 4
                if(index%4):
                    num = index%4
                # print(num)

                path = './'+fileName+'/'+str(index)+'__'+str(num)+'_Sum='+str(len(y))+"High_Low_ " +str(High)+'_'+str(Low)+ '.png'
                plt.savefig(path)

                plt.close()

                worksheet.write(index, 0, label=str(index))
                worksheet.write(index, 1, label=str(len(y)))
                worksheet.write(index, 2, label=str(Low))
                worksheet.write(index, 3, label=str(High))
                worksheet.write(index, 4, label=str(num))
                index = index + 1
                # 保存
                workbook.save(fileName+'.xls')
                plt.show()
            else:
                # print(lineData)
                print('**')

```

