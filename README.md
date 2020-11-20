# windpywrapper

- v0.6 / 对Wind Python API进行封装，拓展功能，简单易用
- 部分功能仍在完善中，或在某些特定使用情境下存在功能异常，欢迎及时反馈、交流

### 重要提示

- 此工具包仅用于日常研究，请勿用于任何商业用途
- 开发者未对工具包的功能进行严格测试，不保证各项功能在任何情况下都可以正常使用，不保证使用此工具包获取的数据的完整性和准确性，使用此工具包带来的任何后果和责任与开发者无关
- 使用此工具包需先自行购买开通Wind API服务权限
- 由于进一步对原Wind API函数封装，代码运行效率会有所下降，此工具包不适合高频交易等需要快速数据响应和处理的场景
- *如有任何建议或意见，欢迎留言*

### 开发初衷
Wind金融终端及相关数据服务是金融从业者日常工作中的重要工具，其数据API为读取各类数据提供了极大的便利。由于原始API的功能理应具有通用性和基础性，因此直接使用时会在一定程度上加大代码量并降低可读性。考虑金融从业者日常工作实际，对经常要使用到的wsd、wss、wsee、wset、edb以及各类日期相关函数等进行了功能整合。当前主要完成以Wind API为核心的数据读取功能，后期将在此基础上进一步整合其他统计分析功能，以期实现功能较为完备的综合型金融分析工具包。

### 功能特点

1. 整合原始API函数功能，将数据查询操作归为“时间序列数据”（time series data）查询和“截面数据”（cross-sectional data）查询，两者结合亦可实现“面板数据”（panel data）查询
2. 通过自动循环实现单次多代码、多字段查询
3. 通过自动循环实现单次查询数据量可超过原始API函数的限制
4. 考虑Wind API存在7×24小时滚动数据查询量上限，内置查询量记录功能，接近查询量上限时自动提示
5. 全面优先使用pandas DataFrame/Series数据格式以及pandas Timestamp时间格式
6. 支持原Wind API的全部可选参数

### 工具包构成

工具包现包含datafeed_wind，datetools和utils三个文件，功能如下：

>  - datafeed_wind：读取时间序列数据、截面数据、面板数据等
>  - datetools：处理时间数据
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
#### 指数/板块成份查询

获取指数或板块成份的Wind代码（和名称）：wind_components(wind_codes, date, show_name)
> 参数说明：
>  - wind_codes：字符串或列表格式的Wind代码（普通代码或板块代码），仅支持单代码
>  - date：查询日期，默认为最近交易日，支持以下格式：字符串日期（格式为YYYY-MM-DD），python内置datetime，pandas Timestamp，numpy datetime64，QuantLib date；支持Wind日期宏（详见Wind API文档），支持内置日期常量（后文详述）
>  - show_name：布尔值，默认为False；True将返回成份代码和成份名称，以字典格式返回（key为代码，value为名称）
> 
> 返回值：
> - list或dict：若show_name = False，反馈成份代码列表，可直接作为wind_series等函数的输入至；若show_name = True，反馈成份代码和成份名称构成的字典

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

#### 时间序列查询

获取常规时间序列数据：wind_series(wind_codes, fields, start_date, end_date, **kwargs)

获取宏观经济时间序列数据：wind_edb(wind_codes, fields, start_date, end_date, **kwargs)

获取板块时间序列数据（开发中...）：wind_sector_series(wind_codes, fields, start_date, end_date, **kwargs)

> 参数说明：
>  - wind_codes：字符串或列表格式的Wind代码（普通代码，或板块代码），可以为单代码或多代码
>  - fields：字符串或列表格式的查询字段，可以为单字段或多字段；如字段名为“edb”（不区分大小写），则自动查询经济数据库数据，功能等同于使用wind_edb()
>  - start_date，end_date：查询起止日期，支持以下格式：字符串日期（格式为YYYY-MM-DD），python内置datetime，pandas Timestamp，numpy datetime64，QuantLib date；支持Wind日期宏，支持内置日期常量（后文详述）
>  - **kwargs：支持原始Wind API函数的各可选参数
> 
> 返回值
> - pandas DataFrame数据表：查询不同发布频率的宏观经济数据时，低频数据（如季度数据）自动通过线性插值按高频数据（如月度数据）的频率补齐中间数据；查询多代码、多字段时，pandas DataFrame数据表的列标签自动分为field和code两级

示例：
``` python
# -------------------------------------------------
# 普通数据单代码+多字段查询：
# 查询沪深300指数自2020年1月1日至2020年9月30日区间内的收盘价和成交量（以下代码实现相同功能）
# 可使用字符串或列表作为输入参数，不区分大小写
wind_series("000300.SH", "CLOSE, VOLUME", "2020-01-01", "2020-09-30")
wind_series(["000300.sh"], ["close", "VOLUME"], "2020-01-01", "2020-09-30")
# 使用工具包内置常用时间宏，CYB=current year begin（今年年初）
wind_series("000300.sh", "CLOSE, volume", CYB, "2020-09-30")

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

#### 截面数据查询
获取截面数据：wind_crosec(wind_code, fields, **kwargs)

获取板块截面数据（开发中...）：(wind_codes, fields, start_date, end_date, **kwargs)
> 参数说明：
>  - wind_codes：字符串或列表格式的Wind代码（普通代码或板块代码），支持单代码或多代码
>  - fields：字符串或列表格式的查询字段，可以为单字段或多字段
>  - **kwargs：支持原始Wind API函数的各可选参数，查询截面数据的可选参数规则多样，建议先查询Wind API文档或使用Wind终端的代码生成器（快捷键：CG）查询
> 
> 返回值：
> - pandas DataFrame数据表：不建议使用多代码、多指标查询，避免数据表结构过于复杂

示例：
```python
# -------------------------------------------------
# 板块成分查询：
# 查询全部Wind中国公募开放式基金的一级分类和二级分类
fund_code = wind_component("2001000000000000")[:100] # Wind中国公募开放式基金
fields = [
    "fund_firstinvesttype", # 基金类别-证监会规则
    "fund_investtype2", # 基金类别-Wind规则：证监会规则下的细类（二级：fundtype=2）
    ]
fund_type = wind_crosec(fund_code, fields, tradedate=TODAY, fundtype=2)
```
注：以上代码在使用wind_component查询成份基金代码时，考虑之后的查询量较大，后面加写[:100]仅取部分基金

#### 面板数据查询（开发中...）
获取面板数据：wind_panel(wind_code, fields, **kwargs)

---

### 2. 日期处理
#### 交易日处理：td(), td_next(), td_is(), td_offset(), td_count()
 
函数功能较为简单，不再具体说明参数

功能说明：

 - 支持Wind日期宏和内置日期常量
 - 返回日期的格式均为pandas Timestamp
 - 默认按上海证券交易所交易日历（Wind参数名为“sse”）进行计算，可输入可选参数调整至不同交易所交易日历

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

#### 内置日期常量

由于Wind日期宏为字符串格式，无法直接用于其他计算，故设置内置日期常量，功能与Wind日期宏近似，可作为上述各函数的输入值。内置日期常量的格式均为pandas Timestamp，值为日历日日期。

常量内容如下：
| 常量名（均为大写） | 对应日期（日历日） |
|--|--|
| TODAY | 今日 |
| YESTERDAY | 昨日 |
| TOMORROW | 明日 |
| LME | 上月末 |
| LQE | 上季度末 |
| LYE | 上年末 |
| CMB | 当月初 |
| CQB | 当季初 |
| CYB | 当年初 |
| E1W | 1周前 |
| E2W | 2周前 |
| E3W | 3周前 |
| E1M、E2M ... E12M | 1月前、2月前 ... 12月前（含各月） |
| E1Y、E2Y ... E10Y | 1年前、2年前 ... 10年前（含各年） |
| E15Y、E20Y ... E50Y | 15年前、20年前 ... 50年前（每5年递增） |

命名规则（助记规则）：

 - LME = Last Month End：L = Last, M = Month, Q = Quarter, Y = Year
 - CYB = Current Year Begin：C = Current, M = Month, Q = Quarter, Y = Year
 - E3Y = Ealier 3 Year, E*n*P = Ealier *n* Periods：E = Ealier, W = Week, M = Month, Y = Year

#### 内置交易所每年交易日数常量
为实现精确的计算，内置主要交易所每年交易日数常量（格式为整数），该数据为交易所过去10年（样本为2010年-2019年）每年交易日数的算术平均数的近似整数，具体如下：

| 常量名（均为大写） | 交易所英文名称 | 交易日数 |
|--|--|--|
| TDPYR_SSE | Shanghai Stock Exchange | 243 |
| TDPYR_SZSE | Shenzhen Stock Exchange | 243 |
| TDPYR_HKEX | Stock Exchange of Hong Kong | 256 |
| TDPYR_SHFE | Shanghai Futures Exchange | 243 |
| TDPYR_DCE | Dalian Commodity Exchange | 243 |
| TDPYR_ZCE | Zhengzhou Commodity Exchange | 243 |
| TDPYR_CFFE | China Financial Futures Exchange | 243 |
| TDPYR_NYSE | New York Stock Exchange | 252 |
| TDPYR_NASDAQ | Nasdaq | 252 |
| TDPYR_AMEX | American Stock Exchange | 252 |
| TDPYR_CME |  Chicago Mercantile Exchange | 252 |
| TDPYR_COMEX | New York Mercantile Exchange | 252 |
| TDPYR_CBOT | Chicago Board of Trade | 252 |
| TDPYR_NYBOT | New York Board of Trade | 252 |
| TDPYR_LSE | London Stock Exchange | 253 |
| TDPYR_LME | London Metal Exchange | 253 |
| TDPYR_IPE | International Petroleum Exchange | 256 |
| TDPYR_TSE | Japan Exchange Group | 245 |

说明：上表常量名称命名规则为"TDPYR_交易所简称"，TDPYR为Trading Days Per YeaR，交易所简称与Wind API内置的简称一致

---

### Z. 其他
#### 数据查询错误
如遇数据查询错误，函数会自动返回Wind API的错误代码以及对应的错误含义
#### 数据量使用提示
Wind API分为普通版、高级版、VIP版三种服务，分别对应7×24小时滚动数据查询上限为：500万单元格、1000万单元格、2000万单元格。此工具包内置数据查询量监测功能，会累计单次查询量，当7×24小时内的查询量接近上限时会自动提示。
