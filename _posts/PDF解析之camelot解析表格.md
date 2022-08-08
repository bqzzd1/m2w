---
title: camelot解析pdf表格
tags: 
- 数据清洗
- 爬虫
- 好用的第三方
categories:
- 爬虫

---

> ### 一、概述

Camelot 是一个 Python 工具，用于将 PDF 文件中的表格数据提取出来。

具体而言，用户可以像使用 Pandas 那样打开 PDF 文件，然后利用这个工具提取表格数据，最后再指定输出的形式（如 csv 文件）。

项目地址：[https://github.com/camelot-dev/camelot](https://link.zhihu.com/?target=https%3A//github.com/camelot-dev/camelot)

> ### 二、**安装方法**

- 项目作者提供了三种安装方法。首先，你可以使用 Conda 进行安装，这是最简单的。

  ```python
  conda install -c conda-forge camelot-py
  ```

  最流行的安装方法是使用 pip 安装。

  ```python
  pip install camelot-py[cv]
  ```

  还可以从项目中克隆代码，并使用源码安装。

  ```python
  git clone https://www.github.com/camelot-dev/camelot
  cd camelot
  pip install ".[cv]"
  ```

注意：

如果后面导入camelot库包的时候，出现错误，可能是缺少`Ghostscript`包，用pip 安装即可。

另外还需设置系统环境变量，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808172054.png)

设置后若还显示错误，可以将gsdll64.dll文件直接放进项目目录即可。



> ### 三、**代码示例**

##### 1. 快速入门使用

项目提供的 PDF 文件如图所示，假设用户需要提取这些文字之间的表格 2-1 中的信息。

![Python-Camelot：三行代码轻松提取PDF表格数据](http://www.zzvips.com/uploads/allimg/211118/220114M16-1.jpg)

*PDF 文件。我们需要提取表格 2-1。*

使用 Camelot 提取表格数据的代码如下：

```python
>>> import camelot
>>> tables = camelot.read_pdf('foo.pdf') #类似于Pandas打开CSV文件的形式
>>> tables[0].df # get a pandas DataFrame!
>>> tables.export('foo.csv', f='csv', compress=True) # json, excel, html, sqlite，可指定输出格式
>>> tables[0].to_csv('foo.csv') # to_json, to_excel, to_html, to_sqlite， 导出数据为文件
>>> tables
<TableList n=1>
>>> tables[0]
<Table shape=(7, 7)> # 获得输出的格式
>>> tables[0].parsing_report
{
    'accuracy': 99.02,
    'whitespace': 12.24,
    'order': 1,
    'page': 1
}
```

以下为输出的结果，对于合并的单元格，Camelot 在抽取后做了空行处理，这是一个稳妥的方法。

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808170440.png)

#### 2. 详细说明

上面的例子的说明： 1、创建一个表格对象

```markdown
>>> tables = camelot.read_pdf('foo.pdf')
>>> tables
<TableList n=1>   # 只检测到一个表格
```



默认情况下，Camelot仅使用PDF的`第一页`来提取表。要指定多个页面，可以使用pages关键字参数：

```markdown
>>> camelot.read_pdf('your.pdf', pages='1,2,3')
```



也可以使用命令行执行相同的操作

```markdown
camelot --pages 1,2,3 lattice your.pdf
```



该pages关键字参数接受页面页码的逗号分隔的字符串。还可以指定页面范围 - 例如，pages=1,4-10,20-30或pages=1,4-10,20-end。

注意：

如果pdf文件是加密的表格，需要加入password参数，值为解密密码

```markdown
>>> tables = camelot.read_pdf('foo.pdf', password='userpass')
>>>tables
<TableList n=1>
```

命令行：

```markdown
camelot --password userpass lattice foo.pdf
```

目前，Camelot仅支持使用ASCII密码和算法代码1或2加密的PDF 。如果无法读取PDF，则抛出异常。这可能是由于未提供密码，密码不正确或加密算法不受支持。

------

2、查看表的形状（行和列），通过表格索引查看 我们可以使用其索引访问每个表。从上面的代码片段中，我们可以看到该tables对象只有一个表，因为n=1。让我们使用索引访问该表0并查看它shape。

```markdown
>>> tables[0]
<Table shape=(7, 7)>
```



------

3、打印解析报告。

```markdown
>>> print tables[0].parsing_report
{
    'accuracy': 99.02,
    'whitespace': 12.24,
    'order': 1,
    'page': 1
}
```



从解析的参数的评价标准可以得出，其准确性是很好的，空白较少，这意味着表格最有可能被正确提取。可以使用table对象的df属性将表作为pandas DataFrame访问。

------

4、打印出提取表格中的内容 数据格式是pandas DataFrane，因此用df访问

```markdown
>>> tables[0].df
```

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808171127.png)

5、导出表格中的内容 可以使用其to_csv()方法将表导出为CSV文件。或者可以使用to_json()，或方法表分别导出为JSON格式，Excel，HTML文件或SQLite数据库。方法如下：

- to_csv()

- to_json()

- to_excel()

- to_html()

- to_sqlite()

  ```markdown
  >>> tables[0].to_csv('foo.csv')
  ```

  

------

上面的1~5 步骤都可以通过命令行直接完成。

```markdown
>>> tables = camelot.read_pdf('foo.pdf')
>>> tables
<TableList n=1>   # 只检测到一个表格0
```



使用的pdf例子的传送门：-->[here](https://www.shouxicto.com/?url=aHR0cHM6Ly9jYW1lbG90LXB5LnJlYWR0aGVkb2NzLmlvL2VuL21hc3Rlci9fc3RhdGljL3BkZi9mb28ucGRm)

### 3. camelot两种表格解析（提取）方法

##### 1、流解析（stream）

Stream可用于解析在`单元格之间具有空格`的表，以模拟表结构。它建立在PDFMiner的功能之上，即使用边距将页面上的字符分组为单词和句子。

1. PDF页面上的单词根据其y轴重叠分组为文本行。
2. 计算文本，然后用于猜测PDF页面上有趣的表区域。您可以阅读[Anssi Nurminen的硕士论文](https://www.shouxicto.com/?url=aHR0cDovL2RzcGFjZS5jYy50dXQuZmkvZHB1Yi9iaXRzdHJlYW0vaGFuZGxlLzEyMzQ1Njc4OS8yMTUyMC9OdXJtaW5lbi5wZGY/c2VxdWVuY2U9Mw==)，以了解有关此表检测技术的更多信息。[见第20,35和40页]
3. 然后猜测每个表区域内的列数。这是通过计算每个文本行中的单词数模式来完成的。基于此模式，选择每个文本行中的单词以计算列x范围的列表。
4. 然后使用位于当前列x范围内/外的单词来扩展当前列列表。
5. 最后，使用文本行的y范围和列x范围形成表格，并且页面上找到的单词基于其x和y坐标分配给表格的单元格。

##### 2、格子解析（lattice）

格子本质上更具有`确定性`，并且它不依赖于猜测。它可用于解析在单元格之间划分了行的表，并且它可以自动解析页面上存在的多个表。

它首先使用`ghostscript`将`PDF页面`转换为`图像`，然后通过使用`OpenCV`应用`一组形态变换（侵蚀和膨胀）`来处理它以获得`水平和垂直线段`。

下面介绍处理的步骤： 1、检测分割线段

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808171317.png)

2、通过`重叠检测`到的线段并叠加"和"它们像素强度来检测线的`交叉点`。

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808171336.png)

3、通过再次`重叠检测`到的线段来计算`表边界`，这次是“或”它们的像素强度。

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808171353.png)

4、由于PDF页面的尺寸及其图像不同，因此检测到的表格边界，线条交叉点和线段将缩放并转换为PDF页面的坐标空间，并创建表格的表示。

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808171408.png)

5、使用线段和线交叉检测生成单元。

6、最后，页面上找到的单词将根据其x和y坐标分配给表格的单元格

> ### 四、高级使用

1. 处理背景线
1. 处理背景线
2. 可视调试
3. 指定表区域
4. 指定列表分隔符
5. 沿分隔符拆分文本
6. 标记上标和下表
7. 从文本中删除字符
8. 改善猜测的表区域
9. 改进猜测的表行
10. 检测短线
11. 在生成单元格中移动文本
12. 复制跨越单元格中的文本
13. 调整布局生成

> ### 五、命令行的使用

用 `camelot --help` 查看Camelot 参数使用:

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220808171610.png)

参数说明：

| 参数   | 完成参数          | 参数功能                                                     |
| :----- | :---------------- | :----------------------------------------------------------- |
| -q     | --quiet TEXT      | 不输出日志和警告                                             |
| -p     | --pages TEXT      | 页数范围，可以用都好分割符，或者页码范围，例如： 1,3,4 or 1,4-end |
| -pw    | --password TEXT   | 解密密码                                                     |
| -o     | -output TEXT      | 输出路径                                                     |
| -f     | --format [csv     | json                                                         |
| -z     | --zip             | 创建                                                         |
| -split | --split_text      | 跨多个单元格拆分文本。                                       |
| -flag  | --flag_size       | 根据字体大小标记文本。有用的检测超/下标。                    |
| -strip | --strip_text TEXT | 将字符串赋值给单元格之前，表格中的字符应该被删除             |
| -M     | --margins         | 字符边缘、线边缘、单词边缘                                   |