# 量化数据下载更新工具

## 交流QQ群
群号：519597639，加群请说明理由

## 介绍
在量化策略中，数据是最重要的一环。市场上也已经有很专业的商业数据库。主要分两类：
1. 落地数据库。速度快，但价格贵，运维成本高，需要搭建数据库服务器。
2. 在线接口。速度很慢，还可能有流量限制。

- TuShare是一个很优秀的免费开源项目，但它的数据来源方是财经网站，无法满足大数据量下载的需求。
- Wind是万得公司的财经软件，在专业金融机构上占有率极高。但落地数据库太贵，在线API又有流量限制。
- Choice是东方财富出品，价格比Wind便宜。
- 其它。如天软、朝阳永续等

就算我们已经购买了收费的接口，但要将数据用起来还是有些麻烦：
1. 从接口下载数据总是占去不少时间
2. 数据下载可能超流量限制
3. 下载过来的数据需要一般要转换成矩阵才方便使用

## 期望目标
1. 能比较快的获得股票行情数据
2. 数据格式调整过，基本可以直接在Python/MATLAB中使用
3. 可增量下载财务数据，减小下载时间和流量限制
4. 提供一些量化数据处理的函数

## 实现方式
1. 从通达信软件获取日线、5分钟线、1分钟线
2. 从大智慧网站获取除权除息信息
3. 多进程运行，将所有股票行情文件合并成一个，方便给alphalens这类的工具做分析
4. 提供脚本，设置计划任务后可每天收盘后自动下载数据和转换

## 目录说明
- auto: 全自动化运行所需脚本（需要按情况修改）
- demo_load: 数据加载示例
- demo_stock: 股票数据下载转换示例（需要按情况修改）
- api.py: `get_price`函数所在文件
- config.py: 各种配置信息（需要按情况修改）
- kquant_data.pth: 库路径文件（需要按情况修改后，复制到Anaconda3\Lib\site-packages\下），如果出现site.py编码问题，请将.pth中的中文注释删除
- DATA_STK.zip: 测试用数据目录。最新行情通过前面的脚本生成，由于数据量大，万得因子数据可从群、网盘下载最新数据覆盖，其它时间自己更新即可。

## 安装
1. 只支持Python 3，因为在数据处理时为了加快进度，使用了多进程，Python 2版多进程时不支持参数
2. pandas.read_csv在中文路径下可能有问题，目前测试Python 64位下有问题，32位下正常。可以两版都装。板块、ST这类的信息文件名中会出现中文
3. 目前没有提供pip安装方式，可以将项目放到合适地方后，在Anaconda3\Lib\site-packages\中添加kquant_data.pth文件
4. 安装通达信股票软件，每天收盘后下载数据。可配置成自动化
5. 通达信最新版不能在虚拟机中运行，有部署到云服务器需求的用户可以考虑安装2017年6月份以前的版本

## 全自动下载转换
1. 安装AutoIt3
2. 如在云服务器上使用，不要使用远程桌面，改用RealVNC，不然AutoIt的鼠标键盘模拟失效。
3. 计划任务中添加15点45以后的任务，run_in_taskschd.bat为入口，入口bat不要有会阻塞任务的操作，如pause，否则下一次计划任务不会执行。
4. 编辑auto/run_tdx.au3文件，配置通达信软件的目录和窗口标题，配错了无法自动下载数据
5. run_after_market_close.bat/run_for_wind.bat/run_for_5min.bat请根据自己的实际情况进行配置

## 使用方法
- 数据准备，先bat下载数据。日线、5分钟、1分钟都可以通达信中直接下载。注意，已经退市股票无法下载。
- 5分钟、1分钟数据只能下载近期的，如想下载以前的，可以从通达信官网下载，然后转换成指定格式。以后就每次最新的与历史的合并即可。
- 参考demo_load/load_from_api.py和load_from_N_files.py学习Python的调用方法
- 参考demo_load/load_from_All_in_One_file.m学习在MATLAB中的使用方法

## api与矩阵数据
少量数据的获取用`get_price`，它的接口设计成与目前流行的在线量化平台基本类似

在做全市场股票策略时，希望获取的是根据时间对齐的矩阵，这样就可以直接交由Python/MATLAB进行各项运算，
但通过`get_price`来获取全市场数据，再合成矩阵耗时太久，占用内存太大，如果是32位程序崩溃是常有的事。如能提前将数据准备好，这些麻烦就不存在了。
所以demo_stock目录下提供了一些脚本，能将几千个包含开高低收数据的h5文件再分别合并成单独的几个h5文件。使用时只要加载对应文件即可。
