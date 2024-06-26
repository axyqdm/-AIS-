## 船舶轨迹数据预处理及分析

#### 介绍

此项目是对AIS数据进行预处理以及分析，一共包含三部分。（注意：使用时请将csv文件下到一个文件夹下，或者自行修改代码，本项目用的数据集为美国 MarineCadastre.gov 网站上提供的。）

- 第一部分：数据清洗及轨迹提取
- 第二部分：特征相关性与轨迹平稳性分析
- 第三部分：轨迹修复

#### 一、数据清洗及轨迹提取

##### 1.数据清洗（数据清洗.py 箱状图.py）

在 AIS 信号的发送、传输、接收过程中，数据难免会出现中断或缺失，且在现实环境中的 AIS 数据会有冗余、异常等情况，所以首先对数据进行清洗工作。

数据清洗过程及理由如下：

（1）首先将 AIS 数据按 MMSI 和时间升幂排序。将原始数据按 MMSI 和时间先后顺序有利于进行后面的数据清洗。

（2）删除 MMSI 不为 9 位的数据。排完序后发现有的船舶的 MMSI 不为 9 位数，因为船舶的 MMSI 唯一识别码是由 9 位数组成的，不符合规定的删除。

（3）选择航行状态为正常航行中的数据。选择船舶的航行状态有发动机使用中、锚泊、未操纵、吃水受限、系泊、搁浅、捕捞、航行中，都统称为正常航行中。

（4）删除船长小于 3 和船宽小于 2 的船舶数据。由于较小的船只的运动轨迹更易受海流、风浪等环境因子的影响，其数据的存在也可能会造成模型出现过拟合，故需剔除过小的船。

（5）删除超出有效范围的经度、维度、对地航速、对地航向数据。根据AIS 数据有效范围可知超出参照数据的十进制表示有效范围是没有意义的，应当舍去。

经度(LON) -180.00000 ~ 180.00000

纬度(LAT) -90.00000 ~ 90.00000

对地航速(SOG) 0~51.2

对地航向(COG) -204.7~204.8

（6）删除对地航速连续 5 个及以上为 0 的数据点。根据初步判断，当对地航速出现连续 5 个时刻为 0 时，将该点视为停泊点，如果不删除对后续轨迹研究造成影响。

清洗后的数据，会输出到一个新文件夹。之后可以使用**箱状图.py**文件画AIS数据的箱状图查看数据分布情况。

<img src="README图片\箱状图.png" style="zoom: 50%;" />

##### 2.轨迹提取（轨迹提取.py）

将所有 AIS 数据按照 MMSI 分成多个组，并对每一组中的轨迹点按 BaseDateTime递增的顺序进行排列，便可得到同一 MMSI 代表的船舶在一个时间段内的航行轨迹数据集。

当同一条船舶相邻两个轨迹点的时间差超过 30min 时，将相邻两个轨迹点分别视为不同航迹的终点和起点，这就把一条船舶轨迹划分为了多条船舶子轨迹。

运行文件后会输出一个文件，文件夹下是以MMSI命名的多个文件夹，每个文件夹里是划出的船舶子轨迹。



<img src="README图片\船舶子轨迹.png" style="zoom:75%;" />

#### 二、特征相关性与轨迹平稳性分析

##### 1.特征相关性（关系矩阵_热力图.py）

读取船舶子轨迹，输出各特征之间的关系矩阵图，各数据特征值相关系数热力图。

<img src="README图片\关系矩阵图.png" style="zoom: 50%;" />

<img src="README图片\热力图.png" style="zoom:50%;" />

##### 2.轨迹平稳性（轨迹平稳性检验.py）

船舶轨迹序列是一个典型的多变量时间序列，在量化过程中使用时间序列分析工具时，经常需要先考察轨迹序列的平稳性，以便选择合适的轨迹预测方法。此代码直观地展示了船舶轨迹四个特征的序列分布情况。

<img src="README图片\序列分布情况.png" style="zoom: 50%;" />

#### 三、轨迹修复（插值处理.py）

在清洗和轨迹提取阶段，可能会舍弃一些轨迹点，这可能导致航迹序列中前后的时间间隔不均匀，此外，由于 AIS 设备的发送频率不等，不同频率下接收到的轨迹序列的时间间隔也会有明显的差异。对于一条航迹序列而言，如果时间间隔太长，将会对后续的算法模型的训练产生极大程度的影响，因此，为了平滑轨迹点，采用三次样条插值函数，加权移动平均插值函数作为参考方法。

插值处理.py会输出两种函数的插值结果到新的文件，之后会绘制散点图直观的对比结果。

<img src="README图片\插值结果.png" style="zoom: 67%;" />

如果需要AIS轨迹图，时间-经纬度图可以点击链接异步我另一个仓库 （[点这里](https://github.com/axyqdm/Track-visualization)）