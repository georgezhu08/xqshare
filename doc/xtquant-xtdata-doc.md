# XtQuant xtdata 模块接口文档

> 生成时间: 2026-03-02
> 基于源码: `/opt/homebrew/Caskroom/miniconda/base/envs/dev/lib/python3.13/site-packages/xtquant/xtdata.py`

---

## 目录

- [连接管理](#连接管理)
- [行情订阅](#行情订阅)
- [历史行情数据](#历史行情数据)
- [实时行情数据](#实时行情数据)
- [财务数据](#财务数据)
- [板块管理](#板块管理)
- [合约信息](#合约信息)
- [交易日历](#交易日历)
- [期权相关](#期权相关)
- [数据下载](#数据下载)
- [策略/模型](#策略模型)
- [工具函数](#工具函数)
- [数据周期](#数据周期)
- [数据字段](#数据字段)

---

## 连接管理

### connect(ip='', port=None, remember_if_success=True)

连接到 xtquant 服务。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| ip | str | '' | 服务器IP地址 |
| port | int/tuple | None | 端口号或端口范围 |
| remember_if_success | bool | True | 是否记住连接参数 |

**返回:** client 对象

**异常:** 连接失败时抛出异常

```python
# 连接本地服务
client = xtdata.connect()

# 连接远程服务
client = xtdata.connect('192.168.1.100', 58610)
```

---

### reconnect(ip='', port=None, remember_if_success=True)

重新连接服务。

**参数:** 同 `connect()`

---

### disconnect()

断开与服务端的连接。

```python
xtdata.disconnect()
```

---

### get_client()

获取当前连接的客户端对象，如果未连接则自动重连。

**返回:** client 对象

---

### get_data_dir()

获取数据目录路径。

**返回:** str - 数据目录路径

---

## 行情订阅

### subscribe_quote(stock_code, period='1d', start_time='', end_time='', count=0, callback=None)

订阅单只股票行情数据。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| stock_code | str | - | 股票代码，如 "000001.SZ" |
| period | str | '1d' | 周期：tick/1m/5m/1d 等 |
| start_time | str/datetime | '' | 开始时间，如 "20240101" |
| end_time | str/datetime | '' | 结束时间 |
| count | int | 0 | 数据数量，-1为全部 |
| callback | function | None | 回调函数 |

**回调函数格式:**
```python
def on_subscribe(datas):
    # datas: {stock: [data1, data2, ...]}
    pass
```

**返回:** int - 订阅序号

```python
def on_data(datas):
    print(datas)

seq = xtdata.subscribe_quote("000001.SZ", "1m", callback=on_data)
```

---

### subscribe_quote2(stock_code, period='1d', start_time='', end_time='', count=0, dividend_type=None, callback=None)

订阅行情数据（支持复权）。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| dividend_type | str | None | 除权类型：none/front/back/front_ratio/back_ratio |

其他参数同 `subscribe_quote()`

---

### subscribe_whole_quote(code_list, callback=None)

订阅全推行情数据。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| code_list | list | 市场列表 ['SH', 'SZ'] 或股票列表 ['600000.SH'] |
| callback | function | 回调函数 |

**返回:** int - 订阅序号

```python
def on_data(datas):
    # datas: {stock1: data1, stock2: data2, ...}
    print(datas)

# 订阅整个市场
seq = xtdata.subscribe_whole_quote(['SH', 'SZ'], on_data)

# 订阅指定股票
seq = xtdata.subscribe_whole_quote(['600000.SH', '000001.SZ'], on_data)
```

**返回数据字段:**
| 字段 | 说明 |
|------|------|
| time | 时间戳 |
| lastPrice | 最新价 |
| open | 开盘价 |
| high | 最高价 |
| low | 最低价 |
| lastClose | 前收盘价 |
| amount | 成交额 |
| volume | 成交量 |
| askPrice | 卖价档位 [1-5] |
| bidPrice | 买价档位 [1-5] |
| askVol | 卖量档位 [1-5] |
| bidVol | 买量档位 [1-5] |

---

### subscribe_l2thousand(stock_code, gear_num=None, callback=None)

订阅千档盘口。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| stock_code | str | 股票代码 |
| gear_num | int | 档位数，None 表示全部 |
| callback | function | 回调函数 |

**返回:** int - 订阅号

```python
def on_data(data):
    print(data)
seq = xtdata.subscribe_l2thousand('000001.SZ', gear_num=10, callback=on_data)
```

---

### subscribe_l2thousand_queue(stock_code, callback=None, gear_num=None, price=None)

订阅千档盘口队列。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| stock_code | str | 股票代码 |
| callback | function | 回调函数 |
| gear_num | int | 档位数 |
| price | float/list/tuple | 价格档位：单个价格/多个价格/价格范围 |

```python
# 订阅买卖2档
seq = xtdata.subscribe_l2thousand_queue('000001.SZ', callback=on_data, gear_num=2)

# 订阅指定价格范围
seq = xtdata.subscribe_l2thousand_queue('000001.SZ', callback=on_data, price=(11.3, 11.4))
```

---

### unsubscribe_quote(seq)

取消订阅。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| seq | int | 订阅接口返回的订阅号 |

```python
xtdata.unsubscribe_quote(seq)
```

---

### run()

阻塞当前线程，持续接收行情回调。

```python
xtdata.subscribe_quote("000001.SZ", "1m", callback=on_data)
xtdata.run()  # 阻塞接收回调
```

---

## 历史行情数据

### get_market_data(field_list=[], stock_list=[], period='1d', start_time='', end_time='', count=-1, dividend_type='none', fill_data=True)

获取历史行情数据。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| field_list | list | [] | 字段列表，[]为全部字段 |
| stock_list | list | [] | 股票代码列表 |
| period | str | '1d' | 周期 |
| start_time | str | '' | 开始时间 "20200101" |
| end_time | str | '' | 结束时间 "20201231" |
| count | int | -1 | 数量，-1全部，n从结束时间向前数n个 |
| dividend_type | str | 'none' | 除权类型 |
| fill_data | bool | True | 是否填充缺失数据 |

**返回:**
- period 为 'tick' 时: `{stock: np.ndarray}`
- 其他周期: `{field: pd.DataFrame}` - index 为股票代码，columns 为时间

```python
data = xtdata.get_market_data(
    field_list=['open', 'high', 'low', 'close', 'volume'],
    stock_list=['000001.SZ', '600000.SH'],
    period='1d',
    start_time='20240101',
    end_time='20240131'
)
print(data['close'])
```

---

### get_market_data_ex(field_list=[], stock_list=[], period='1d', start_time='', end_time='', count=-1, dividend_type='none', fill_data=True)

获取历史行情数据（扩展版）。

**返回:** `{stock: pd.DataFrame}` - 每只股票一个 DataFrame，index 为时间

```python
data = xtdata.get_market_data_ex(
    field_list=[],
    stock_list=['000001.SZ'],
    period='1d',
    start_time='20240101',
    end_time='20240131'
)
print(data['000001.SZ'])
```

---

### get_local_data(field_list=[], stock_list=[], period='1d', start_time='', end_time='', count=-1, dividend_type='none', fill_data=True, data_dir=None)

从本地文件获取历史数据（不请求服务器）。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| data_dir | str | None | 数据目录，None 使用默认目录 |

其他参数同 `get_market_data_ex()`

---

### get_market_data3(...)

`get_market_data_ex()` 的别名，参数相同。

---

## 实时行情数据

### get_full_tick(code_list)

获取最新 tick 数据。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| code_list | list | 股票代码列表 |

**返回:** dict - `{stock: {tick_data}}`

```python
ticks = xtdata.get_full_tick(['000001.SZ', '600000.SH'])
print(ticks['000001.SZ'])
```

---

### get_l2_quote(field_list=[], stock_code='', start_time='', end_time='', count=-1)

获取 Level2 实时行情快照。

**返回:** np.ndarray

---

### get_l2_order(field_list=[], stock_code='', start_time='', end_time='', count=-1)

获取 Level2 逐笔委托数据。

**返回:** np.ndarray

---

### get_l2_transaction(field_list=[], stock_code='', start_time='', end_time='', count=-1)

获取 Level2 逐笔成交数据。

**返回:** np.ndarray

---

### get_l2thousand_queue(stock_code, gear_num=None, price=None)

获取千档盘口队列数据。

**返回:** dict

---

### get_transactioncount(code_list)

获取大单统计数据。

**返回:** dict

---

### get_fullspeed_orderbook(code_list)

获取全速盘口数据。

**返回:** dict

---

### get_full_kline(field_list=[], stock_list=[], period='1m', start_time='', end_time='', count=1, dividend_type='none', fill_data=True)

K线全推获取最新交易日数据。

**返回:** `{field: pd.DataFrame}`

---

## 财务数据

### get_financial_data(stock_list, table_list=[], start_time='', end_time='', report_type='report_time')

获取财务数据。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| stock_list | list | - | 股票代码列表 |
| table_list | list | [] | 报表名称列表 |
| start_time | str | '' | 开始时间 |
| end_time | str | '' | 结束时间 |
# report_type | str | 'report_time' | 时段筛选：announce_time/report_time |

**可选报表:**
| 表名 | 说明 |
|------|------|
| Balance | 资产负债表 |
| Income | 利润表 |
| CashFlow | 现金流量表 |
| Capital | 股本结构 |
| HolderNum | 股东人数 |
| Top10Holder | 十大股东 |
| Top10FlowHolder | 十大流通股东 |
| PershareIndex | 每股指标 |

**返回:** `{stock: {table: pd.DataFrame}}`

```python
data = xtdata.get_financial_data(
    stock_list=['000001.SZ'],
    table_list=['Balance', 'Income'],
    start_time='20230101',
    end_time='20231231'
)
print(data['000001.SZ']['Balance'])
```

---

### download_financial_data(stock_list, table_list=[], start_time='', end_time='', incrementally=None)

下载财务数据到本地。

---

### download_financial_data2(stock_list, table_list=[], start_time='', end_time='', callback=None)

下载财务数据（带回调）。

---

## 板块管理

### get_sector_list()

获取板块列表。

**返回:** list[str] - 板块名称列表

```python
sectors = xtdata.get_sector_list()
print(sectors)  # ['沪深A股', '上证A股', '深证A股', ...]
```

---

### get_sector_info(sector_name='')

获取板块信息。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| sector_name | str | '' | 板块名称，空则获取所有 |

**返回:** pd.DataFrame - 包含 sector 和 category 列

---

### get_stock_list_in_sector(sector_name, real_timetag=-1)

获取板块成分股。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| sector_name | str | - | 板块名称 |
| real_timetag | str/int | -1 | 时间戳或日期字符串，-1为最新 |

**返回:** list[str] - 股票代码列表

```python
# 获取最新成分股
stocks = xtdata.get_stock_list_in_sector('沪深A股')

# 获取历史成分股
stocks = xtdata.get_stock_list_in_sector('沪深A股', '20230101')
```

---

### get_index_weight(index_code)

获取指数权重。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| index_code | str | 指数代码 |

**返回:** dict - `{stock: weight}`

```python
weights = xtdata.get_index_weight('000300.SH')
```

---

### create_sector_folder(parent_node, folder_name, overwrite=True)

创建板块目录节点。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| parent_node | str | 父节点，''为'我的'目录 |
| folder_name | str | 目录名称 |
| overwrite | bool | True跳过，False自动编号 |

---

### create_sector(parent_node, sector_name, overwrite=True)

创建板块。

---

### add_sector(sector_name, stock_list)

增加自定义板块。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| sector_name | str | 板块名称 |
| stock_list | list | 股票代码列表 |

```python
xtdata.add_sector('我的自选', ['000001.SZ', '600000.SH'])
```

---

### remove_stock_from_sector(sector_name, stock_list)

从板块中移除股票。

---

### remove_sector(sector_name)

删除自定义板块。

---

### reset_sector(sector_name, stock_list)

重置板块成分股。

---

### download_sector_data()

下载行业板块数据。

---

## 合约信息

### get_instrument_detail(stock_code, iscomplete=False)

获取合约信息。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| stock_code | str | - | 股票代码 |
| iscomplete | bool | False | 是否返回完整信息 |

**返回:** dict

**主要字段:**
| 字段 | 类型 | 说明 |
|------|------|------|
| ExchangeID | str | 市场代码 |
| InstrumentID | str | 合约代码 |
| InstrumentName | str | 合约名称 |
| ProductID | str | 品种ID |
| ProductName | str | 品种名称 |
| CreateDate | str | 上市日期 |
| ExpireDate | str | 到期日 |
| PreClose | float | 前收盘价 |
| UpStopPrice | float | 涨停价 |
| DownStopPrice | float | 跌停价 |
| FloatVolume | float | 流通股本 |
| TotalVolume | float | 总股本 |
| PriceTick | float | 最小变价单位 |
| VolumeMultiple | int | 合约乘数 |
| MainContract | int | 主力合约标记 |
| IsTrading | bool | 是否可交易 |

```python
info = xtdata.get_instrument_detail('000001.SZ')
print(info['InstrumentName'])
```

---

### get_instrument_detail_list(stock_list, iscomplete=False)

批量获取合约信息。

**返回:** `{stock: info_dict}`

---

### get_instrument_type(stock_code, variety_list=None)

判断证券类型。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| stock_code | str | 股票代码 |
| variety_list | list | 品种列表，None返回全部 |

**返回:** dict - `{类型名: 是否属于该类型}`

---

### get_markets()

获取所有可选市场。

**返回:** dict - `{市场代码: 市场名称}`

```python
markets = xtdata.get_markets()
# {'SH': '上交所', 'SZ': '深交所', 'BJ': '北交所', ...}
```

---

### get_wp_market_list()

获取外盘市场列表。

---

### get_ipo_info(start_time='', end_time='')

获取IPO信息。

**返回字段:**
| 字段 | 说明 |
|------|------|
| securityCode | 证券代码 |
| codeName | 代码简称 |
| publishPrice | 发行价格 |
| startDate | 申购开始日期 |
| listedDate | 上市日期 |
| lwr | 中签率 |

---

## 交易日历

### get_trading_dates(market, start_time='', end_time='', count=-1)

获取交易日列表。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| market | str | 市场代码：SH/SZ/IF/DF/SF/ZF 等 |
| start_time | str | 开始时间 |
| end_time | str | 结束时间 |
| count | int | 数量，-1为全部 |

**返回:** list[int] - 毫秒时间戳列表

```python
dates = xtdata.get_trading_dates('SH', '20240101', '20240131')
```

---

### get_trading_calendar(market, start_time='', end_time='')

获取交易日历（包含未来交易日）。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| market | str | 市场，仅支持 SH/SZ |
| start_time | str | 开始时间 |
| end_time | str | 结束时间 |

**返回:** list[str] - 日期字符串列表

---

### get_holidays()

获取节假日列表。

**返回:** list[str] - 日期字符串列表

```python
holidays = xtdata.get_holidays()
```

---

### get_divid_factors(stock_code, start_time='', end_time='')

获取除权除息因子。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| stock_code | str | 股票代码 |
| start_time | str | 开始时间 |
| end_time | str | 结束时间 |

**返回:** pd.DataFrame

```python
factors = xtdata.get_divid_factors('000001.SZ', '20200101', '20241231')
```

---

## 期权相关

### get_option_detail_data(optioncode)

获取期权合约详情。

**返回:** dict

**主要字段:**
| 字段 | 说明 |
|------|------|
| optType | 期权类型：CALL/PUT |
| OptExercisePrice | 行权价 |
| OptUndlCode | 标的代码 |
| EndDelivDate | 到期日 |

---

### get_option_undl_data(undl_code_ref)

获取标的对应的所有期权列表。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| undl_code_ref | str | 标的代码，空则返回全部 |

**返回:** list 或 dict

---

### get_option_list(undl_code, dedate, opttype='', isavailavle=False)

获取指定日期可交易的期权列表。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| undl_code | str | 标的代码 |
| dedate | str | 日期 |
| opttype | str | 期权类型：C/CALL/P/PUT |
| isavailavle | bool | 是否只返回可交易的 |

**返回:** list[str]

---

### get_his_option_list(undl_code, dedate)

获取历史上某日的期权信息列表。

**返回:** pd.DataFrame

---

### get_his_option_list_batch(undl_code, start_time='', end_time='')

批量获取历史期权信息。

**返回:** `{date: pd.DataFrame}`

---

## 数据下载

### download_history_data(stock_code, period, start_time='', end_time='', incrementally=None)

下载历史数据。

**参数:**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| stock_code | str | - | 品种代码 |
| period | str | - | 数据周期 |
| start_time | str | '' | 开始时间 |
| end_time | str | '' | 结束时间 |
| incrementally | bool | None | 是否增量下载，None根据start_time判断 |

```python
# 下载全部历史数据
xtdata.download_history_data('000001.SZ', '1d')

# 下载指定时间段
xtdata.download_history_data('000001.SZ', '1d', '20230101', '20231231')
```

---

### download_history_data2(stock_list, period, start_time='', end_time='', callback=None, incrementally=None)

批量下载历史数据（支持回调）。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| stock_list | list | 股票代码列表 |
| callback | function | 进度回调函数 |

**回调格式:**
```python
def on_progress(data):
    # data: {'finished': n, 'total': m}
    print(f"{data['finished']}/{data['total']}")
```

---

### download_index_weight()

下载指数权重数据。

---

### download_history_contracts(incrementally=True)

下载过期合约数据。

---

### download_holiday_data(incrementally=True)

下载节假日数据。

---

### download_etf_info()

下载 ETF 申赎信息。

---

### download_cb_data()

下载可转债数据。

---

### download_his_st_data()

下载历史 ST 数据。

---

## 策略/模型

### subscribe_formula(formula_name, stock_code, period, start_time='', end_time='', count=-1, dividend_type=None, extend_param={}, callback=None)

订阅策略模型。

**返回:** int - 模型ID (request_id)

---

### get_formula_result(request_id, start_time='', end_time='', count=-1, timeout_second=-1)

获取模型结果。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| request_id | int | 模型ID |
| timeout_second | int | 等待时间，-1无限，0立即返回 |

**返回:** dict

---

### call_formula(formula_name, stock_code, period, start_time='', end_time='', count=-1, dividend_type=None, extend_param={})

同步调用策略模型。

**返回:** dict

---

### bind_formula(request_id, callback=None)

绑定模型回调。

---

### unsubscribe_formula(request_id)

取消订阅模型。

---

### create_formula(formula_name, formula_content, formula_params={})

创建策略。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| formula_name | str | 策略名称 |
| formula_content | str | 策略内容 |
| formula_params | dict | 策略参数 |

---

### import_formula(formula_name, file_path)

导入策略文件（.rzrk）。

---

### del_formula(formula_name)

删除策略。

---

### get_formulas()

查询所有策略。

---

## 工具函数

### get_main_contract(code_market, start_time='', end_time='')

获取主力合约。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| code_market | str | 主力连续合约代码，如 "IF00.IF" |
| start_time | str | 开始时间 |
| end_time | str | 结束时间 |

**返回:**
- 无时间参数: str - 当前主力合约
- 有 start_time: str - 指定日期主力合约
- 有 start_time 和 end_time: pd.Series - 时间序列

```python
# 当前主力
code = xtdata.get_main_contract("IF00.IF")

# 历史主力序列
series = xtdata.get_main_contract("IF00.IF", "20230101", "20240101")
```

---

### get_sec_main_contract(code_market, start_time='', end_time='')

获取次主力合约。参数同 `get_main_contract()`。

---

### get_cb_info(stockcode)

获取可转债信息。

**返回:** dict

---

### get_etf_info()

获取 ETF 申赎信息。

**返回:** dict

---

### get_his_st_data(stock_code)

获取历史 ST 状态数据。

**返回:** dict - `{status: [[start, end], ...]}`

---

### timetag_to_datetime(timetag, format)

将毫秒时间戳转换为日期时间字符串。

**参数:**

| 参数 | 类型 | 说明 |
|------|------|------|
| timetag | int | 毫秒时间戳 |
| format | str | 时间格式，如 "%Y%m%d" |

**返回:** str

```python
date_str = xtdata.timetag_to_datetime(1704067200000, '%Y%m%d')
```

---

### datetime_to_timetag(datetime, format='%Y%m%d%H%M%S')

将日期时间字符串转换为毫秒时间戳。

**返回:** int

---

### read_feather(file_path)

读取 feather 格式文件。

**返回:** (param_dict, pd.DataFrame)

---

### write_feather(dest_path, param, df)

写入 feather 格式文件。

---

## 数据周期

### K线周期

| 周期 | 说明 |
|------|------|
| tick | 分笔数据 |
| 1m | 1分钟K线 |
| 5m | 5分钟K线 |
| 15m | 15分钟K线 |
| 30m | 30分钟K线 |
| 60m/1h | 60分钟K线 |
| 1d | 日K线 |
| 1w | 周K线 |
| 1mon | 月K线 |

### Level2 周期

| 周期 | 说明 |
|------|------|
| l2quote | Level2行情快照 |
| l2quoteaux | Level2行情快照补充 |
| l2order | Level2逐笔委托 |
| l2transaction | Level2逐笔成交 |
| l2transactioncount | Level2大单统计 |
| l2orderqueue | Level2委买委卖队列 |

### 特殊周期

| 周期 | 说明 |
|------|------|
| transactioncount1m | Level1逐笔成交统计1分钟 |
| transactioncount1d | Level1逐笔成交统计日线 |
| warehousereceipt | 期货仓单 |
| futureholderrank | 期货席位 |
| interactiveqa | 互动问答 |

---

## 数据字段

### 1m / 5m / 1d K线字段

| 字段 | 说明 |
|------|------|
| time | 时间戳 |
| open | 开盘价 |
| high | 最高价 |
| low | 最低价 |
| close | 收盘价 |
| volume | 成交量 |
| amount | 成交额 |
| settlementPrice | 今结算 |
| preClose | 前收价
| openInterest | 持仓量 |
| suspendFlag | 停牌标记 0 - 正常 1 - 停牌 -1 - 当日起复牌 |

### tick分笔字段

| 字段 | 说明 |
|------|------|
| time | 时间戳 |
| lastPrice | 最新价 |
| open | 开盘价 |
| high | 最高价 |
| low | 最低价 |
| lastClose | 前收盘价 |
| amount | 成交总额 |
| volume | 成交总量 |
| pvolume | 原始成交总量 |
| stockStatus | 证券状态 |
| openInt | 持仓量 |
| lastSettlementPrice | 前结算 |
| askPrice1-5 | 卖一价~卖五价 |
| bidPrice1-5 | 买一价~买五价 |
| askVol1-5 | 卖一量~卖五量 |
| bidVol1-5 | 买一量~买五量 |

---

## 除权类型

| 类型 | 说明 |
|------|------|
| none | 不复权 |
| front | 前复权 |
| back | 后复权 |
| front_ratio | 等比前复权 |
| back_ratio | 等比后复权 |

---

## 除权数据

| 字段 | 说明 |
|------|------|
| interest|   每股股利（税前，元） |
| stockBonus |   每股红股（股） |
| stockGift |   每股转增股本（股） |
| allotNum |   每股配股数（股） |
| allotPrice |   配股价格（元） |
| gugai |   是否股改，对于股改，在算复权系数时，系统有特殊算法 |
| dr |   除权系数 |

---

## l2quote - level2实时行情快照

| 字段 | 说明 |
|------|------|
|time|   时间戳|
|lastPrice|   最新价|
|open|   开盘价|
|high|   最高价|
|low|   最低价|
|amount|   成交额|
|volume|   成交总量|
|pvolume|   原始成交总量|
|openInt|   持仓量|
|stockStatus|   证券状态|
|transactionNum|   成交笔数|
|lastClose|   前收盘价|

| 字段 | 说明 |
|------|------|
|lastSettlementPrice|前结算|
|settlementPrice|今结算|
|pe|市盈率|
|askPrice|多档委卖价|
|bidPrice|多档委买价|
|askVol|多档委卖量|
|bidVol|多档委买量|

---

## l2order - level2逐笔委托

| 字段 | 说明 |
|------|------|
|time|    时间戳 |
|price|    委托价 |
|volume|    委托量 |
|entrustNo|    委托号 |
|entrustType|    委托类型 |
|entrustDirection|    委托方向 |

---

## l2transaction - level2逐笔成交

| 字段 | 说明 |
|------|------|
|time|    时间戳 |
|price|    成交价 |
|volume|    成交量 |
|amount|    成交额 |
|tradeIndex|    成交记录号 |
|buyNo|    买方委托号 |
|sellNo|    卖方委托号 |
|tradeType|    成交类型 |
|tradeFlag|    成交标志 |

---

## l2quoteaux - level2实时行情补充（总买总卖）

| 字段 | 说明 |
|------|------|
|time|    时间戳 |
|avgBidPrice|    委买均价 |
|totalBidQuantity|    委买总量 |
|avgOffPrice|    委卖均价 |
|totalOffQuantity|    委卖总量 |
|withdrawBidQuantity|    买入撤单总量 |
|withdrawBidAmount|    买入撤单总额 |
|withdrawOffQuantity|    卖出撤单总量 |
|withdrawOffAmount|    卖出撤单总额 |

---

## l2orderqueue - level2委买委卖一档委托队列

| 字段 | 说明 |
|------|------|
|time|    时间戳 |
|bidLevelPrice|    委买价 |
|bidLevelVolume|    委买量 |
|offerLevelPrice|    委卖价 |
|offerLevelVolume|    委卖量 |
|bidLevelNumber|    委买数量 |
|offLevelNumber|    委卖数量 |

---

# 数据字典

## 证券状态

```
0,10 - 默认为未知
11 - 开盘前S
12 - 集合竞价时段C
13 - 连续交易T
14 - 休市B
15 - 闭市E
16 - 波动性中断V
17 - 临时停牌P
18 - 收盘集合竞价U
19 - 盘中集合竞价M
20 - 暂停交易至闭市N
21 - 获取字段异常
22 - 盘后固定价格行情
23 - 盘后固定价格行情完毕
```

## 委托类型

- level2逐笔委托 - entrustType 委托类型
- level2逐笔成交 - tradeType 成交类型

```
0 - 未知
1 - 正常交易业务
2 - 即时成交剩余撤销
3 - ETF基金申报
4 - 最优五档即时成交剩余撤销
5 - 全额成交或撤销
6 - 本方最优价格
7 - 对手方最优价格
```

## 委托方向

- level2逐笔委托 - entrustDirection 委托方向

注：上交所的撤单信息在逐笔委托的委托方向，区分撤买撤卖

```
1 - 买入
2 - 卖出
3 - 撤买（上交所）
4 - 撤卖（上交所）
```

## 成交标志

- level2逐笔成交 - tradeFlag 成交标志

注：深交所的在逐笔成交的成交标志，只有撤单，没有方向

```
0 - 未知
1 - 外盘
2 - 内盘
3 - 撤单（深交所）
```

---

# 财务数据字段列表

## Balance - 资产负债表

| 字段 | 说明 |
|------|------|
|m_anntime|	披露日期|
|m_timetag|	截止日期|
|internal_shoule_recv|	内部应收款|
|fixed_capital_clearance|	固定资产清理|
|should_pay_money|	应付分保账款|
|settlement_payment|	结算备付金|
|receivable_premium|	应收保费|
|accounts_receivable_reinsurance|	应收分保账款|
|reinsurance_contract_reserve|	应收分保合同准备金|
|dividends_payable|	应收股利|
|tax_rebate_for_export|	应收出口退税|
|subsidies_receivable|	应收补贴款|
|deposit_receivable|	应收保证金|
|apportioned_cost|	待摊费用|
|profit_and_current_assets_with_deal|	待处理流动资产损益|
|current_assets_one_year|	一年内到期的非流动资产|
|long_term_receivables|	长期应收款|
|other_long_term_investments|	其他长期投资|
|original_value_of_fixed_assets|	固定资产原值|
|net_value_of_fixed_assets|	固定资产净值|
|depreciation_reserves_of_fixed_assets|	固定资产减值准备|
|productive_biological_assets|	生产性生物资产|
|public_welfare_biological_assets|	公益性生物资产|
|oil_and_gas_assets|	油气资产|
|development_expenditure|	开发支出|
|right_of_split_share_distribution|	股权分置流通权|
|other_non_mobile_assets|	其他非流动资产|
|handling_fee_and_commission|	应付手续费及佣金|
|other_payables|	其他应交款|
|margin_payable|	应付保证金|
|internal_accounts_payable|	内部应付款|
|advance_cost|	预提费用|
|insurance_contract_reserve|	保险合同准备金|
|broker_buying_and_selling_securities|	代理买卖证券款|
|acting_underwriting_securities|	代理承销证券款|
|international_ticket_settlement|	国际票证结算|
|domestic_ticket_settlement|	国内票证结算|
|deferred_income|	递延收益|
|short_term_bonds_payable|	应付短期债券|
|long_term_deferred_income|	长期递延收益|
|undetermined_investment_losses|	未确定的投资损失|
|quasi_distribution_of_cash_dividends|	拟分配现金股利|
|provisions_not|	预计负债|
|cust_bank_dep|	吸收存款及同业存放|
|provisions|	预计流动负债|
|less_tsy_stk|	减:库存股|
|cash_equivalents|	货币资金|
|loans_to_oth_banks|	拆出资金|
|tradable_fin_assets|	交易性金融资产|
|derivative_fin_assets|	衍生金融资产|
|bill_receivable|	应收票据|
|account_receivable|	应收账款|
|advance_payment|	预付款项|
|int_rcv|	应收利息|
|other_receivable|	其他应收款|
|red_monetary_cap_for_sale|	买入返售金融资产|
|agency_bus_assets|	以公允价值计量且其变动计入当期损益的金融资产|
|inventories|	存货|
|other_current_assets|	其他流动资产|
|total_current_assets|	流动资产合计|
|loans_and_adv_granted|	发放贷款及垫款|
|fin_assets_avail_for_sale|	可供出售金融资产|
|held_to_mty_invest|	持有至到期投资|
|long_term_eqy_invest|	长期股权投资|
|invest_real_estate|	投资性房地产|
|accumulated_depreciation|	累计折旧|
|fix_assets|	固定资产|
|constru_in_process|	在建工程|
|construction_materials|	工程物资|
|long_term_liabilities|	长期负债|
|intang_assets|	无形资产|
|goodwill|	商誉|
|long_deferred_expense|	长期待摊费用|
|deferred_tax_assets|	递延所得税资产|
|total_non_current_assets|	非流动资产合计|
|tot_assets|	资产总计|
|shortterm_loan|	短期借款|
|borrow_central_bank|	向中央银行借款|
|loans_oth_banks|	拆入资金|
|tradable_fin_liab|	交易性金融负债|
|derivative_fin_liab|	衍生金融负债|
|notes_payable|	应付票据|
|accounts_payable|	应付账款|
|advance_peceipts|	预收账款|
|fund_sales_fin_assets_rp|	卖出回购金融资产款|
|empl_ben_payable|	应付职工薪酬|
|taxes_surcharges_payable|	应交税费|
|int_payable|	应付利息|
|dividend_payable|	应付股利|
|other_payable|	其他应付款|
|non_current_liability_in_one_year|	一年内到期的非流动负债|
|other_current_liability|	其他流动负债|
|total_current_liability|	流动负债合计|
|long_term_loans|	长期借款|
|bonds_payable|	应付债券|
|longterm_account_payable|	长期应付款|
|grants_received|	专项应付款|
|deferred_tax_liab|	递延所得税负债|
|other_non_current_liabilities|	其他非流动负债|
|non_current_liabilities|	非流动负债合计|
|tot_liab|	负债合计|
|cap_stk|	实收资本(或股本)|
|cap_rsrv|	资本公积|
|specific_reserves|	专项储备|
|surplus_rsrv|	盈余公积|
|prov_nom_risks|	一般风险准备|
|undistributed_profit|	未分配利润|
|cnvd_diff_foreign_curr_stat|	外币报表折算差额|
|tot_shrhldr_eqy_excl_min_int|	归属于母公司股东权益合计|
|minority_int|	少数股东权益|
|total_equity|	所有者权益合计|
|tot_liab_shrhldr_eqy|	负债和股东权益总计|

---

## Income - 利润表

| 字段 | 说明 |
|------|------|
|m_anntime|	披露日期|
|m_timetag|	截止日期|
|revenue_inc|	营业收入|
|earned_premium|	已赚保费|
|real_estate_sales_income|	房地产销售收入|
|total_operating_cost|	营业总成本|
|real_estate_sales_cost|	房地产销售成本|
|research_expenses|	研发费用|
|surrender_value|	退保金|
|net_payments|	赔付支出净额|
|net_withdrawal_ins_con_res|	提取保险合同准备金净额|
|policy_dividend_expenses|	保单红利支出|
|reinsurance_cost|	分保费用|
|change_income_fair_value|	公允价值变动收益|
|futures_loss|	期货损益|
|trust_income|	托管收益|
|subsidize_revenue|	补贴收入|
|other_business_profits|	其他业务利润|
|net_profit_excl_merged_int_inc|	被合并方在合并前实现净利润|
|int_inc|	利息收入|
|handling_chrg_comm_inc|	手续费及佣金收入|
|less_handling_chrg_comm_exp|	手续费及佣金支出|
|other_bus_cost|	其他业务成本|
|plus_net_gain_fx_trans|	汇兑收益|
|il_net_loss_disp_noncur_asset|	非流动资产处置收益|
|inc_tax|	所得税费用|
|unconfirmed_invest_loss|	未确认投资损失|
|net_profit_excl_min_int_inc|	归属于母公司所有者的净利润|
|less_int_exp|	利息支出|
|other_bus_inc|	其他业务收入|
|revenue|	营业总收入|
|total_expense|	营业成本|
|less_taxes_surcharges_ops|	营业税金及附加|
|sale_expense|	销售费用|
|less_gerl_admin_exp|	管理费用|
|financial_expense|	财务费用|
|less_impair_loss_assets|	资产减值损失|
|plus_net_invest_inc|	投资收益|
|incl_inc_invest_assoc_jv_entp|	联营企业和合营企业的投资收益|
|oper_profit|	营业利润|
|plus_non_oper_rev|	营业外收入|
|less_non_oper_exp|	营业外支出|
|tot_profit|	利润总额|
|net_profit_incl_min_int_inc|	净利润|
|net_profit_incl_min_int_inc_after|	净利润(扣除非经常性损益后)|
|minority_int_inc|	少数股东损益|
|s_fa_eps_basic|	基本每股收益|
|s_fa_eps_diluted|	稀释每股收益|
|total_income|	综合收益总额|
|total_income_minority|	归属于少数股东的综合收益总额|
|other_compreh_inc|	其他收益|

---

## CashFlow - 现金流量表

| 字段 | 说明 |
|------|------|
|m_anntime|    披露日期|
|m_timetag|    截止日期|
|cash_received_ori_ins_contract_pre|	收到原保险合同保费取得的现金|
|net_cash_received_rei_ope|	收到再保险业务现金净额|
|net_increase_insured_funds|	保户储金及投资款净增加额|
|net_increase_in_disposal| 处置交易性金融资产净增加额|
|cash_for_interest|	收取利息、手续费及佣金的现金|
|net_increase_in_repurchase_funds|	回购业务资金净增加额|
|cash_for_payment_original_insurance|	支付原保险合同赔付款项的现金|
|cash_payment_policy_dividends|	支付保单红利的现金|
|disposal_other_business_units|	处置子公司及其他收到的现金|
|cash_received_from_pledges|	减少质押和定期存款所收到的现金|
|cash_paid_for_investments|	投资所支付的现金|
|net_increase_in_pledged_loans|	质押贷款净增加额|
|cash_paid_by_subsidiaries|	取得子公司及其他营业单位支付的现金净额|
|increase_in_cash_paid|	增加质押和定期存款所支付的现金|
|cass_received_sub_abs|	其中子公司吸收现金|
|cass_received_sub_investments|	其中:子公司支付给少数股东的股利、利润|
|minority_shareholder_profit_loss|	少数股东损益|
|unrecognized_investment_losses|	未确认的投资损失|
|ncrease_deferred_income|	递延收益增加(减:减少)|
|projected_liability|	预计负债|
|increase_operational_payables|	经营性应付项目的增加|
|reduction_outstanding_amounts_less|	已完工尚未结算款的减少(减:增加)|
|reduction_outstanding_amounts_more|	已结算尚未完工款的增加(减:减少)|
|goods_sale_and_service_render_cash|	销售商品、提供劳务收到的现金|
|net_incr_dep_cob|	客户存款和同业存放款项净增加额|
|net_incr_loans_central_bank|	向中央银行借款净增加额(万元|
|net_incr_fund_borr_ofi|	向其他金融机构拆入资金净增加额|
|net_incr_fund_borr_ofi|	拆入资金净增加额|
|tax_levy_refund|	收到的税费与返还|
|cash_paid_invest|	投资支付的现金|
|other_cash_recp_ral_oper_act|	收到的其他与经营活动有关的现金|
|stot_cash_inflows_oper_act|	经营活动现金流入小计|
|goods_and_services_cash_paid|	购买商品、接受劳务支付的现金|
|net_incr_clients_loan_adv|	客户贷款及垫款净增加额|
|net_incr_dep_cbob|	存放中央银行和同业款项净增加额|
|handling_chrg_paid|	支付利息、手续费及佣金的现金|
|cash_pay_beh_empl|	支付给职工以及为职工支付的现金|
|pay_all_typ_tax|	支付的各项税费|
|other_cash_pay_ral_oper_act|	支付其他与经营活动有关的现金|
|stot_cash_outflows_oper_act|	经营活动现金流出小计|
|net_cash_flows_oper_act|	经营活动产生的现金流量净额|
|cash_recp_disp_withdrwl_invest|	收回投资所收到的现金|
|cash_recp_return_invest|	取得投资收益所收到的现金|
|net_cash_recp_disp_fiolta| 	处置固定资产、无形资产和其他长期投资收|
|other_cash_recp_ral_inv_act|	收到的其他与投资活动有关的现金|
|stot_cash_inflows_inv_act|	投资活动现金流入小计|
|cash_pay_acq_const_fiolta|	购建固定资产、无形资产和其他长期投资支付的|
|other_cash_pay_ral_oper_act|	支付其他与投资的现金|
|stot_cash_outflows_inv_act|	投资活动现金流出小计|
|net_cash_flows_inv_act|	投资活动产生的现金流量净额|
|cash_recp_cap_contrib|	吸收投资收到的现金|
|cash_recp_borrow|	取得借款收到的现金|
|proc_issue_bonds|	发行债券收到的现金|
|other_cash_recp_ral_fnc_act|	收到其他与筹资活动有关的现金|
|stot_cash_inflows_fnc_act|	筹资活动现金流入小计|
|cash_prepay_amt_borr|	偿还债务支付现金|
|cash_pay_dist_dpcp_int_exp|	分配股利、利润或偿付利息支付的现金|
|other_cash_pay_ral_fnc_act|	支付其他与筹资的现金|
|stot_cash_outflows_fnc_act|	筹资活动现金流出小计|
|net_cash_flows_fnc_act|	筹资活动产生的现金流量净额|
|eff_fx_flu_cash|	汇率变动对现金的影响|
|net_incr_cash_cash_equ|	现金及现金等价物净增加额|
|cash_cash_equ_beg_period|	期初现金及现金等价物余额|
|cash_cash_equ_end_period|	期末现金及现金等价物余额|
|net_profit|	净利润|
|plus_prov_depr_assets|	资产减值准备|
|depr_fa_coga_dpba|	固定资产折旧、油气资产折耗、生产性物资折旧|
|amort_intang_assets|	无形资产摊销|
|amort_lt_deferred_exp|	长期待摊费用摊销|
|decr_deferred_exp|	待摊费用的减少|
|incr_acc_exp|	预提费用的增加|
|loss_disp_fiolta|	处置固定资产、无形资产和其他长期资产的损失|
|loss_scr_fa|	固定资产报废损失|
|loss_fv_chg|	公允价值变动损失|
|fin_exp|	财务费用|
|invest_loss|	投资损失|
|decr_deferred_inc_tax_assets|	递延所得税资产减少|
|incr_deferred_inc_tax_liab|	递延所得税负债增加|
|decr_inventories|	存货的减少|
|decr_oper_payable|	经营性应收项目的减少|
|others|	其他|
|im_net_cash_flows_oper_act|	经营活动产生现金流量净额|
|conv_debt_into_cap|	债务转为资本|
|conv_corp_bonds_due_within_1y|	一年内到期的可转换公司债券|
|fa_fnc_leases|	融资租入固定资产|
|end_bal_cash|	现金的期末余额|
|less_beg_bal_cash|	现金的期初余额|
|plus_end_bal_cash_equ|	现金等价物的期末余额|
|less_beg_bal_cash_equ|	现金等价物的期初余额|
|im_net_incr_cash_cash_equ|	现金及现金等价物的净增加额|
|tax_levy_refund|	收到的税费返还|

---

## PershareIndex - 主要指标

| 字段 | 说明 |
|------|------|
|s_fa_ocfps|    每股经营活动现金流量|
|s_fa_bps|    每股净资产|
|s_fa_eps_basic|    基本每股收益|
|s_fa_eps_diluted|    稀释每股收益|
|s_fa_undistributedps|    每股未分配利润|
|s_fa_surpluscapitalps|    每股资本公积金|
|adjusted_earnings_per_share|    扣非每股收益|
|du_return_on_equity|    净资产收益率|
|sales_gross_profit|    销售毛利率|
|inc_revenue_rate|    主营收入同比增长|
|du_profit_rate|    净利润同比增长|
|inc_net_profit_rate|    归属于母公司所有者的净利润同比增长|
|adjusted_net_profit_rate|    扣非净利润同比增长|
|inc_total_revenue_annual|    营业总收入滚动环比增长|
|inc_net_profit_to_shareholders_annual|    归属净利润滚动环比增长|
|adjusted_profit_to_profit_annual|    扣非净利润滚动环比增长|
|equity_roe|    加权净资产收益率|
|net_roe|    摊薄净资产收益率|
|total_roe|    摊薄总资产收益率|
|gross_profit|    毛利率|
|net_profit|    净利率|
|actual_tax_rate|    实际税率|
|pre_pay_operate_income|    预收款 / 营业收入|
|sales_cash_flow|    销售现金流 / 营业收入|
|gear_ratio|    资产负债比率|
|inventory_turnover|    存货周转率|
|m_anntime|    公告日|
|m_timetag|    报告截止日|

---

## CapitalStructure - 股本表

| 字段 | 说明 |
|------|------|
|total_capital|    总股本|
|circulating_capital|    已上市流通A股|
|restrict_circulating_capital|    限售流通股份|
|m_timetag|    报告截止日|
|m_anntime|    公告日|

---

## 市场代码

| 代码 | 市场 |
|------|------|
| SH | 上交所 |
| SZ | 深交所 |
| BJ | 北交所 |
| HK | 港交所 |
| HGT | 沪港通 |
| SGT | 深港通 |
| IF | 中金所 |
| SF/SHFE | 上期所 |
| DF/DCE | 大商所 |
| ZF/CZCE | 郑商所 |
| GF/GFEX | 广期所 |
| INE | 能源交易所 |
| SHO | 上证期权 |
| SZO | 深证期权 |

---

## 简写函数

xtdata 提供了以下简写别名：

| 简写 | 原函数 |
|------|--------|
| gmd | get_market_data |
| gmd2 | get_market_data_ex |
| gmd3 | get_market_data3 |
| gld | get_local_data |
| t2d | timetag_to_datetime |
| gsl | get_stock_list_in_sector |

```python
# 使用简写
data = xtdata.gmd([], ['000001.SZ'], '1d')
stocks = xtdata.gsl('沪深A股')
```
