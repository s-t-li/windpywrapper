
# windpywrapper

- v0.6 / 2020-12-31 / under heavy development 
- 对Wind Python API进行封装，精简代码，整合功能，并进行一定拓展，简单易用
- √ 适用的场景：宏观研究、证券研究、组合业绩评估、资产配置等
- × 不适用的场景：需要快速数据响应和处理，如行情监测、高频交易等

### 重要提示

- 此工具包仅用于日常研究，请勿用于任何商业用途
- 使用此工具包需先自行购买开通Wind API服务权限
- 开发者未对工具包的功能进行全面、严格的测试，不保证各项功能在任何情况下都可以正常使用，不保证使用此工具包获取的数据的完整性和准确性，使用此工具包带来的任何后果和责任与开发者无关
- *如有任何建议或意见，欢迎留言，开发者将认真听取您的建议和意见，并进行持续更新*

### 开发初衷

Wind金融终端及相关数据服务是金融从业者日常工作中的重要工具，其数据API为读取各类数据提供了极大的便利。由于原始API的功能理应具有通用性和基础性，因此直接使用时会在一定程度上加大代码量并降低可读性。考虑金融从业者日常工作实际，对经常要使用到的wsd、wss、wsee、wset、edb以及各类日期相关函数等进行了功能整合。当前主要完成以Wind API为核心的数据读取功能，后期将在此基础上进一步整合其他统计分析功能，以期实现功能较为完备的综合型金融分析工具包。

### 功能特点

1. 整合原始API函数功能，将数据查询操作归为“时间序列数据”（time series data）查询和“截面数据”（cross-sectional data）查询，两者结合亦可实现“面板数据”（panel data）查询（见 *Z.提示*）
2. 通过自动循环实现单次多代码、多字段查询
3. 通过自动循环实现单次查询数据量可超过原始API函数的限制
4. 考虑Wind API存在7×24小时滚动数据查询量上限，当单次查询数据量过大时会自动提示
5. 全面优先使用pandas DataFrame/Series数据格式以及pandas Timestamp时间格式
6. 保留支持原Wind API的全部可选参数

### 工具包构成

工具包现包含datafeed_wind，datetools和utils三个文件，功能如下：

>  - datafeed_wind：读取时间序列数据、截面数据、面板数据等
>  - datetools：处理时间数据（目前主要依托Wind Python API）
>  - utils：支持运行的工具函数

以下功能在开发过程中（按优先度排序），后期将逐步整合至工具包中，并对现有文件架构进行调整：

>  - plottools：基于matplotlib/seaborn内置常用金融图表绘制工具
>  - pfpm：包含资产组合业绩分析所需的统计模型
>  - pfopt：包含进行资产配置所需的各类工具和最优化模型
>  - datafeed_bbg：实现对Bloomberg Python API功能的完善和升级

## 开始使用

### 0. 初始化

可使用如下语句调用：

``` python
from datafeed_wind import *
from datetools import *
# 或
from datatools import TODAY, to_pd_timestamp
```

---

### 1. 金融/经济数据查询

#### 1.1 指数/板块成份查询

**wind_components(wind_codes, date, show_name=False)**

**获取指数或板块成分的Wind代码（和名称）**

> 参数：
>  - **wind_codes：string，list**
>
>  Wind代码：普通代码（如沪深300指数：000300.SH）或板块代码（如Wind中国公募开放式基金板块代码：2001000000000000），目前仅支持单代码查询，若输入包含多个代码的字符串或列表，则只查询第一个代码
>
>  - **date：python datetime，pandas Timestamp，numpy datetime64，QuantLib date，支持Wind日期宏（用法详见Wind API文档），支持内置日期常量（后文详述），关于Wind日期宏的使用请参考*Z.提示***
>
>  查询日期，默认为最近交易日
>
>  - **show_name：bool，默认为False**
>
>  是否显示成分名称，默认不显示，True将以字典格式返回成份代码和成份名称（key为代码，value为名称）
>
> ---
>
> 返回值：
>
> - **list或dict**
>
> 若show_name = False，返回成份代码列表，可直接作为wind_series等函数的输入值；若show_name = True，返回成份代码和成份名称构成的字典

示例：
```python
# -------------------------------------------------
# 指数成分查询：
# 查询沪深300指数在最近交易日的成份股
wind_components("000300.sh") 

# 查询中证800指数在上月末的成份股，LME=last month end（上月末）
# 以下返回成份股代码列表
wind_components(["000906.sh"], LME)
# 以下返回成份股代码和成份股名称构成的字典
wind_components("000906.sh", show_name=True) 

# -------------------------------------------------
# 板块成分查询：
# 查询全部Wind中国公募开放式基金的代码
wind_components("2001000000000000", show_name=True)
```

#### 1.2 时间序列查询

**wind_series(wind_codes, fields, start_date, end_date=TODAY, ** kwargs)**

**查询常规时间序列数据**

> 参数：
>
>  - **wind_codes：string，list**
>
>  字符串或列表格式的Wind代码，包括普通代码（如沪深300指数：000300.SH），板块代码（如Wind中国公募开放式基金板块：2001000000000000），宏观经济数据代码（如中国GDP不变价当季同比：M0039354）；支持多代码查询，但不支持普通代码、板块代码或宏观经济数据代码混查（如["000300.SH", "M0039354"]）
>
>  - **fields：string，list**
>
>  字符串或列表格式的查询字段，支持多字段查询；如字段名为“edb”（不区分大小写），则自动查询经济数据库数据，功能等同于使用wind_edb()
>
>  - **start_date，end_date：python datetime，pandas Timestamp，numpy datetime64，QuantLib date，支持Wind日期宏（用法详见Wind API文档），支持内置日期常量（后文详述），关于Wind日期宏的使用请参考*Z.提示***
>
>  查询的起止时间，对于市场行情数据，默认截止时间为最近交易日，对于宏观经济数据等，默认截止时间为当日
>
>  - ** **kwargs**
>
>  支持原Wind API函数的各可选参数，具体用法请参考Wind Python API文档
>
> ---
>
> 返回值：
>
> - **pandas DataFrame**
> 同时查询多代码、多字段时，pandas DataFrame数据表的列标签自动分为field和code两级，field对应字段，code对应代码；由于此数据结构较为复杂，在引用时可读性低，故不建议使用多代码、多字段查询

示例：

``` python
# -------------------------------------------------
# 普通数据单代码+多字段查询：
# 查询沪深300指数自2020年1月1日至2020年9月30日区间内的收盘价和成交量（以下代码实现相同功能）
# 可使用字符串或列表作为输入参数，不区分大小写
wind_series("000300.SH", "CLOSE, VOLUME", "2020-01-01", "2020-09-30")
wind_series(["000300.sh"], ["close", "VOLUME"], "2020-01-01", "2020-09-30")
# 使用工具包内置常用时间宏，CYB=current year begin（当年年初）,CQ3E=current 3rd quarter end（当年3季度末）
wind_series("000300.sh", "CLOSE, volume", CYB, CQ3E)

# -------------------------------------------------
# 普通数据单多代码+多字段查询：
# 查询中证500指数和中证800指数自2020年1月1日至2020年9月30日区间内的开盘价和收盘价
wind_series(["000905.sh", "000906.sh"], "open, close", "2020-01-01", "2020-09-30")
# 查询美股通用电气过去5年每月末的估值（PE_TTM）
wind_series("ge.n", "pe_ttm", "ED-5Y", TODAY, period="m", tradingcalendar="NYSE")

# -------------------------------------------------
# 宏观经济数据库查询
# 查询过去2年至今的中国工业增加值当月同比和银行间同业拆借7天加权利率（以下代码实现功能相同）
wind_series("M0000545, M0041664", "EDB", "-2Y", TODAY)
wind_edb(["M0000545", "M0041664"], "-2Y", TODAY)
```

**wind_edb(wind_codes, start_date, end_date=TODAY, period=None, ** kwargs)**

**查询宏观经济库（Wind EDB）数据**

> 参数：
>  - **wind_codes：string，list**
>
> 字符串或列表格式的Wind EDB指标代码；支持多代码查询
>
>  - **start_date，end_date：python datetime，pandas Timestamp，numpy datetime64，QuantLib date，支持Wind日期宏（用法详见Wind API文档），支持内置日期常量（后文详述），关于Wind日期宏的使用请参考*Z.提示***
>
>  查询的起止时间，默认截止时间为当日
>
>  - **period：string，默认为None**
>
>  数据时间周期，y=年，q=季度，m=月，w=周，d=日；默认为None，即返回edb函数返回的原始数据，若指定周期，则对edb函数返回的原始时间序列进行变频（resampling）；**注意：在Wind EDB中读取市场行情数据（如沪深300指数收盘点位，代码M0020209）时，若不指定period，则返回数据对应的日期为“不规则的”交易所日历日，若设定period为某个特定时间周期，此函数会自动将交易所日历日与自然日对齐，并按照ffill方法（propagate last valid observation forward to next valid）填充非交易日的数据，这样处理是为了保证行情数据能够与同频率的宏观经济数据在日期上对齐，以便进行时间序列计算或画图**
>  - ** **kwargs**
>
>  支持原Wind API函数的各可选参数，具体用法请参考Wind Python API文档
>
> ---
>
> 返回值：
>
> - **pandas DataFrame**

示例：

``` python
# -------------------------------------------------
# 普通数据单代码+多字段查询：
# 查询沪深300指数自2020年1月1日至2020年9月30日区间内的收盘价和成交量（以下代码实现相同功能）
# 可使用字符串或列表作为输入参数，不区分大小写
wind_series("000300.SH", "CLOSE, VOLUME", "2020-01-01", "2020-09-30")
wind_series(["000300.sh"], ["close", "VOLUME"], "2020-01-01", "2020-09-30")
# 使用工具包内置常用时间宏，CYB=current year begin（当年年初）,CQ3E=current 3rd quarter end（当年3季度末）
wind_series("000300.sh", "CLOSE, volume", CYB, CQ3E)

# -------------------------------------------------
# 普通数据单多代码+多字段查询：
# 查询中证500指数和中证800指数自2020年1月1日至2020年9月30日区间内的开盘价和收盘价
wind_series(["000905.sh", "000906.sh"], "open, close", "2020-01-01", "2020-09-30")
# 查询美股通用电气过去5年每月末的估值（PE_TTM）
wind_series("ge.n", "pe_ttm", "ED-5Y", TODAY, period="m", tradingcalendar="NYSE")

# -------------------------------------------------
# 宏观经济数据库查询
# 查询过去2年至今的中国工业增加值当月同比和银行间同业拆借7天加权利率（以下代码实现功能相同）
wind_series("M0000545, M0041664", "EDB", "-2Y", TODAY)
wind_edb(["M0000545", "M0041664"], "-2Y", TODAY)
```

#### 1.3 截面数据查询

**wind_crosec(wind_codes, fields, ** kwargs)**

**查询截面数据查询（测试中，可能存在意外错误）**

> 参数：
>
>  - **wind_codes：string，list**
>
>  字符串或列表格式的Wind代码；支持多代码查询
>
>  - **fields：string，list**
>
>  字符串或列表格式的查询字段，支持多字段查询；不建议使用多代码、多指标查询，避免数据表结构过于复杂
>
>  - ** **kwargs**
>
>  支持原Wind API函数的各可选参数；查询截面数据的可选参数规则多样，请先查询Wind API文档或使用Wind终端的代码生成器（快捷键：CG）查询
>
> ---
>
> 返回值：
>
> - **pandas DataFrame**

示例：

```python
# -------------------------------------------------
# 板块成分查询：
# 查询全部Wind中国公募开放式基金的一级分类和二级分类
fund_code = wind_components("2001000000000000")[:100] # Wind中国公募开放式基金
fields = [
    "fund_firstinvesttype", # 基金类别-证监会规则
    "fund_investtype2", # 基金类别-Wind规则：证监会规则下的细类（二级：fundtype=2）
    ]
fund_type = wind_crosec(fund_code, fields, tradedate=TODAY, fundtype=2)
```

注：以上代码在使用wind_components查询成分基金代码时，考虑之后的查询量较大，后面加写[:100]仅取部分基金

#### 1.4 面板数据查询（开发中...）

**wind_panel(wind_code, fields, ** kwargs)**

---

### 2. 日期处理

#### 2.1 交易日处理：td(), td_next(), td_is(), td_offset(), td_count()

函数功能较为简单，不再具体说明参数，使用方法参考下方示例

功能说明：

 - 支持Wind日期宏和内置日期常量
 - 返回日期的格式均为pandas Timestamp
 - 默认按上海证券交易所交易日历（Wind参数名为“sse”）进行计算，可输入可选参数调整至不同交易所交易日历，具体方法参考Wind Python API文档

示例：

``` python
# -------------------------------------------------
# 返回最近交易日
td() # 默认为今日
td("2020-09-09")

# -------------------------------------------------
# 返回下一个交易日
td_next() # 默认为今日
td_next("2010-08-08")

# -------------------------------------------------
# 判断该日期是否为交易日
td_is() # 默认为今日
td_is("2020-01-01")

# 返回向前或向后指定距离的交易日
td_offset("-3y", "2020-09-09") # 2020年9月9日向前3年的最近交易日
td_offset("1M", YESTERDAY) # 昨天向后1月的最近交易日，YESTERDAY为内置日期常量

# 计算两个时间点之间的天数
td_count("-1Y", LQE) # 一年前至上季度末之间的交易日天数
td_count("2020-03-03", CMB, days="alldays") # 2020年3月3日至本月初的日历日天数
```

#### 2.2 内置日期常量

由于Wind日期宏为字符串格式，无法直接用于其他计算，故设置内置日期常量，功能与Wind日期宏近似，可作为上述各函数的输入值。内置日期常量的格式均为pandas Timestamp，值为日历日日期。

常量内容如下：
|常量名（均为大写）|对应日期（日历日）|
|--|-|
|TODAY|今日|
|YESTERDAY|昨日|
|TOMORROW|明日|
|LMB, LME|上月初, 上月末|
|LQB, LQE|上季度初, 上季度末|
|LYB, LYE|上年初, 上年末|
|CMB, CME|当月初, 当月末|
|CQB, CQE|当季初, 当季末|
|CYB, CYE|当年初, 当年末|
|LQ1B ... LQ4B|去年1季度初 ... 去年4季度初|
|LQ1E ... LQ4E|去年1季度末 ... 去年4季度末|
|CQ1B ... CQ4B|当年1季度初 ... 当年4季度初|
|CQ1E ... CQ4E|当年1季度末 ... 当年4季度末|
|E1W|1周前|
|E2W|2周前|
|E3W|3周前|
|E1M, E2M ... E12M|1月前, 2月前 ... 12月前（含各月）|
|E1Y, E2Y ... E10Y|1年前, 2年前 ... 10年前（含各年）|
|E15Y, E20Y ... E50Y|15年前, 20年前 ... 50年前（每5年递增）|

命名规则（助记规则）：

 - LME = Last Month End：L = Last, M = Month, Q = Quarter, Y = Year
 - CYB = Current Year Begin：C = Current, M = Month, Q = Quarter, Y = Year
 - CQ1E = Current 1st Quarter End
 - E3Y = Ealier 3 Year, E*n*P = Ealier *n* Periods：E = Ealier, W = Week, M = Month, Y = Year

#### 2.3 内置交易所每年交易日数常量

为实现精确的计算，内置主要交易所每年交易日数常量（格式为整数），该数据为交易所过去10年（样本为2010年-2019年）每年交易日数的算术平均数的近似整数，具体如下：

| 常量名（均为大写） | 交易所英文名称                   | 交易日数 |
| ------------------ | -------------------------------- | -------- |
| TDPYR_SSE          | Shanghai Stock Exchange          | 243      |
| TDPYR_SZSE         | Shenzhen Stock Exchange          | 243      |
| TDPYR_HKEX         | Stock Exchange of Hong Kong      | 256      |
| TDPYR_SHFE         | Shanghai Futures Exchange        | 243      |
| TDPYR_DCE          | Dalian Commodity Exchange        | 243      |
| TDPYR_ZCE          | Zhengzhou Commodity Exchange     | 243      |
| TDPYR_CFFE         | China Financial Futures Exchange | 243      |
| TDPYR_NYSE         | New York Stock Exchange          | 252      |
| TDPYR_NASDAQ       | Nasdaq                           | 252      |
| TDPYR_AMEX         | American Stock Exchange          | 252      |
| TDPYR_CME          | Chicago Mercantile Exchange      | 252      |
| TDPYR_COMEX        | New York Mercantile Exchange     | 252      |
| TDPYR_CBOT         | Chicago Board of Trade           | 252      |
| TDPYR_NYBOT        | New York Board of Trade          | 252      |
| TDPYR_LSE          | London Stock Exchange            | 253      |
| TDPYR_LME          | London Metal Exchange            | 253      |
| TDPYR_IPE          | International Petroleum Exchange | 256      |
| TDPYR_TSE          | Japan Exchange Group             | 245      |

说明：上表常量名称命名规则为"TDPYR_交易所简称"，TDPYR为Trading Days Per YeaR，交易所简称与Wind API内置的简称一致

---

### N. 其他功能

#### 数据查询错误

如遇数据查询错误，函数会自动返回Wind API的错误代码以及对应的错误含义

#### 数据量使用提示（测试中）
Wind API分为普通版、高级版、VIP版三种服务，分别对应7×24小时滚动数据查询上限为（按2020年产品手册）：500万单元格、1000万单元格、2000万单元格。当单次查询数据量过大时，工具包的函数会自动提示数据查询量。

### Z. 提示

#### 不推荐使用Wind日期宏（Date Macros）

Wind API提供日期宏功能，特定的字符串代表对应的日期（如"LME"表示上月末的日期），能够让使用者较简便地书写代码。但是不推荐使用Wind日期宏，主要原因在于日期宏为字符串格式，仅能够被Wind API内置函数识别，无法直接参与其他计算，并且在对Wind API内置函数进行功能封装时，容易造成难以预计的错误。推荐使用此工具包内置的日期常量，格式均为pandas Timestamp，能够参与各类计算。

#### 不推荐单次查询过大的数据量

Wind Python API内置函数有单次查询数据量上限，如（wsd单次查询数据量为8000单元格，其他详见Wind Python API文档），此工具包通过循环查询+自动拼接的方式可以实现“理论上”单次任意查询量，但仍不推荐单次查询过大的数据量（如全部A股在过去5年的收盘价数据对应约300万单元格的数据量），原因如下：1、由于进行循环查询，可能导致查询较慢，当网络不稳定时易造成查询失败；2、大量查询会占用过多数据流量，超限后会导致未来7*24小时无法再使用Wind Python API中的大部分函数查询功能；3、由于第2项原因，开发者难以对超大数据查询进行足够的测试，因此可能存在功能不稳定的情况。

#### 对代码格式的判断可能有误

为了进一步简化Wind Python API函数的使用操作，此工具包中部分函数会对输入的Wind代码进行自动识别。由于Wind未公开普通Wind代码（如股票代码、基金代码、债券代码等）、板块代码（如全部A股板块代码、中国开放式公募基金板块代码等）、宏观经济数据库代码（如工业增加值当月同比代码）的命名规则，此工具包仅依据可查的代码总结其命名规律。因此在通过代码格式判断其类别时，可能存在判断有误的情况（尽管已经进行了大量的测试）。
