# SAM-Terminal-SDK-Python
> 项目功能简述：调用终端sdk接口

## 1. 运行v

> 开发平台是Pycharm，可以用VS Code等IDE平台均可运行

## 2. 开发环境配置

系统：Win10

语言：Python3.7.3（需添加环境变量） 

下载地址：https://www.python.org/downloads/

相关包/库：numpy, pymysql

开发工具：Pycharm

## 3. 安装方法

用 pip 安装，当前最新版本为1.4.7
> pip install samdata_terminal_dev

## 4. 执行回测

1.通过 pip 安装 sdk。

2.下载策略模板，在模板对应位置填入用户名，密码和策略ID。

3.根据SDK API在策略模板中的onData函数实现策略逻辑。

4.完成策略，运行python代码，执行回测。

```python 
# coding=utf-8
from samdata_terminal_dev.backtest_sdk import SAMTraderEngine, SAMTrader
from datetime import datetime

def initial():
    userid = "779d34ccde3846ee85ea95e32233251e"    # 用户id
    strategyID = "9fdf5563-97be-43e7-90db-a90ca8fed4de"    # 策略id
    symbol_list = "BTC/USDT.BN"    # 订阅交易对
    for symbol in symbol_list.split(','):
        history_data[symbol] = []
    time_type = "1m"     # 1:1m 2:5m 3:15m 4:30m 5:1h
    data_type = "kline"    # 数据类型：kline(k线)、depth(盘口)
    get_data_way = "1"    # 获取数据方式：1（本地csv）、2（api）、3（同步数据库） 
    api_key = ''    # 如果通过api获取数据，则需要传api_key 和 api_secret 
    api_secret = ''
    cash = 100000     #initial cash
    starttime = "2019-12-01"    # 回测开始时间
    endtime = "2019-12-02"    # 回测结束时间，不包含最后一天数据
    account_type = "USD"    #  账户类型 USD：美元,CNY:人民币

    excuation = 2    # 订单成交方式，1：当前bar价格成交，2：下一bar的价格成交
    running_type = 1    # 0：手动模式，返回数据库地址；1：自动模式，自动拉取行情数据；

    e = SAMTraderEngine(userid = userid, strategyID = strategyID,symbol_list = symbol_list,time_type = time_type,cash = cash,\
    starttime = starttime,endtime = endtime,fill_bar_type = excuation,running_type = running_type,account_type = account_type,\
    data_type = data_type, get_data_way = get_data_way, api_key = api_key, api_secret = api_secret)
    
    return e

def on_data(bar, e):
    #strategy
    dt = bar['BTC/USD.OK.Q'][0].bar_time

    if 'BTC/USD.OK.Q' not in e.get_holdings().keys() or e.get_holdings()['BTC/USD.OK.Q'][0].quantity < 1:
        e.place_order(symbol = 'RB1705.SHF.future', direction = 'Buy', order_type = 0,  order_time = dt,open_or_close = 'Open', quantity = 100, fillprice = bar['BTC/USD.OK.Q'][0].close)

    return

sdk = SAMTrader(initial = initial,on_data = on_data)
sdk.start()
```




## 5. 函数功能

## 1. SAMTrader **入口函数类**

任何用户自定义的策略回测都需要创建一个 SAMTrader 对象，用于配置策略运行过程中需要的参数。

实现内部策略引擎SAMTraderEngine，包含函数：

- **__init__**：构造函数
- **start**：策略回测开始

在调用SAMTrader入口函数类中的方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.backtest_sdk import SAMTrader
```




### 1.1 构造函数 - **init**

函数说明： 构造函数，传入初始化参数

函数原型：   

``` python
def _init_(self,initial,onData)
```




### 1.1.1  初始化函数 - initial

函数说明：初始化策略引擎， 其中初始化策略引擎所需参数，请参考 **2.1 策略引擎构造函数**

函数原型：def initial()

示例代码：

```python 
def initial():
    userid = "779d34ccde3846ee85ea95e32233251e"    # 用户id
    strategyID = "9fdf5563-97be-43e7-90db-a90ca8fed4de"    # 策略id
    symbol_list = "BTC/USDT.BN"    # 订阅交易对
    for symbol in symbol_list.split(','):
        history_data[symbol] = []
    time_type = "1m"     # 1:1m 2:5m 3:15m 4:30m 5:1h
    data_type = "kline"    # 数据类型：kline(k线)、depth(盘口)
    get_data_way = "1"    # 获取数据方式：1（本地csv）、2（api）、3（同步数据库） 
    api_key = ''    # 如果通过api获取数据，则需要传api_key 和 api_secret 
    api_secret = ''
    cash = 100000     # initial cash
    starttime = "2019-12-01"    # 回测开始时间
    endtime = "2019-12-02"    # 回测结束时间，不包含最后一天数据
    account_type = "USD"    #  账户类型 USD：美元,CNY:人民币

    excuation = 2    # 订单成交方式，1：当前bar价格成交，2：下一bar的价格成交
    running_type = 1    # 0：手动模式，返回数据库地址；1：自动模式，自动拉取行情数据；

    e = SAMTraderEngine(userid = userid, strategyID = strategyID,symbol_list = symbol_list,time_type = time_type,cash = cash,\
    starttime = starttime,endtime = endtime,fill_bar_type = excuation,running_type = running_type,account_type = account_type,\
    data_type = data_type, get_data_way = get_data_way, api_key = api_key, api_secret = api_secret)
    
    return e
```




### 1.1.2 策略回调函数 - **on_data**

函数说明： 策略回调函数,编写策略回测逻辑，实现根据实时推送的数据进行策略回测

函数原型： on_data(bar, e)

参数说明： 

bar：一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个KlineData(K线数据) / DepthData(盘口数据) 的list数组

e: 策略引擎

KlineData 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | 品种   |
| bar_time    | string   | 报价时间   |
| high    | float   | 最高价   |
| low    | float   | 最低价   |
| open    | float   | 开盘价   |
| close    | float   | 收盘价   |
| quote_volume    | float   | 基础币种成交量   |
| quoteasset_volume    | float   | 计价币种成交量（okex期货合约表示合约张数，单位为张）   |
| trade_num    | float   | 交易笔数   |



DepthData 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | 品种   |
| bar_time    | string   | 报价时间   |
| bids_price_list    | list（float）   | 买20价 |
| bids_quantity_list    | list（float）   | 买20量   |
| asks_price_list    | list（float）   | 卖20价   |
| asks_quantity_list    | list（float）  | 卖20量   |


示例代码：

```python 
def on_data(bar, e):
    print(bar["BTC/USD.OK.Q"][0].bar_time)
```




### 1.3  策略回测开始函数 - **start**

函数说明：完成SAMTrader 对象初始化后，通过调用该对象的start(), 开始运行策略回测

示例代码：

```python 
sdk = SAMTrader(initial = initial,on_data = on_data)
sdk.start()
```




## 2. 策略引擎 SAMTraderEngine

实现内部策略引擎SAMTraderEngine，包含函数：

- **__init__**：构造函数
- **get_all_bars**：获取所有回测数据bar
- **pull_bar**：获取下一个数据bar
- **place_order**: 下单
- **get_orders**: 获取订单
- **get_account**：获取账户信息，包含可用资金等
- **get_holdings**：获取持仓
- **client_close**：结束回测
- **get_local_csvpath**： 获取数据文件路径

在调用SAMTraderEngine策略引擎类中的方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.backtest_sdk import SAMTraderEngine
```

### 2.1 构造函数 - **init**

函数说明：构造函数，传入初始化参数

函数原型：   

``` python
def __init__(self,userid,strategyID,symbol_list,time_type,cash,starttime,endtime,fill_bar_type,\
    running_type,account_type, data_type, get_data_way, api_key, api_secret)
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                                                         |
| ------------- | -------- | ---------------- | ------------------------------------------------------------ |
| userid    | string   | 用户id           | 用户id                            |
| strategyID    | string   | 策略ID           | 根据用户创建策略给出的唯一ID,                            |
| symbol_list   | string   | 回测品种         | 支持多品种，多个品种用英文半角逗号(",")隔开                      |
| time_type     | string   | 时间精度类型     | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时）                                  |
| data_type     | string   | 数据类型         | kline(k线)、depth(盘口)                               |
| get_data_way     | string   | 获取数据方式     | 1（本地csv）、2（api）、3（同步数据库）                         |
| api_key          | string   | 访问api的key         |  当获取数据方式为2，即通过api获取数据时，需要传入该值，其具体值请从官网中的个人中心获取                                                            |
| api_secret          | string   | 访问api的secret         |  当获取数据方式为2，即通过api获取数据时，需要传入该值，其具体值请从官网中的个人中心获取                                                            |
| cash          | float   | 初始资金         |                                                              |
| starttime     | string   | 回测开始时间     |  格式：'%Y-%m-%d %H:%M:%S' 或者  '%Y-%m-%d'                                                          |
| endtime       | string   | 回测结束时间     | 格式同上                                                           |
| fill_bar_type | string   | 成交方式类型     | 1：当前bar价格，2：下一bar的价格 ，目前暂时不处理该字段，请传入空字符''                            |
| running_type  | string   | 数据源类型       | 0： 手动模式，返回数据文件路径；1：自动模式，自动拉取行情数据 , 目前只支持自动模式|
| account_type  | string   | 结算币种         | 账户类型 USD：美元,CNY:人民币，保留                          |

示例代码：

```python
    SAMTraderEngine(userid = "779d34ccde3846ee85ea95e32233251e", strategyID = "9fdf5563-97be-43e7-90db-a90ca8fed4de",symbol_list = "BTC/USDT.BN",time_type = "1h",cash = 100000, starttime = "2019-11-01",\
        endtime = "2019-12-01",fill_bar_type = "2",running_type = "1",account_type = "USD", data_type = "kline", get_data_way = "1", api_key = "", api_secret = "")
```


**注意**:

1、 当starttime 和 endtime的格式为"YYYY-mm-dd", 则会自动补全时间字符串为"YYYY-mm-dd 00:00:00"。



### 2.2 获取回测所有数据bar - **get_all_bars**

函数说明：根据指定获取数据的方式，获取回测开始时间~结束时间的所有k线（盘口）数据bar

返回参数： 一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个KlineData(K线数据)/DepthData(盘口数据)的list数组返回 list列表，其中每个元素为一个KlineData / DepthData对象, KlineData 和 DepthData类的具体属性请参考 **1.1.2  on_data**


示例代码：

```python
data = e.get_all_bars()
print(data["BTC/USD.OK.Q"][0].close)     # 打印第一个k线数据的收盘价,
```


**注意**：

### 2.3 获取下一个数据bar - **pull_bar**

函数说明：通过指定获取数据方式，获取下一个数据bar的k线（盘口）数据

返回参数：返回两个参数

参数 1 ：一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个list列表，其中每个元素为一个KlineData /DepthData(盘口数据)对象，如果无数据，该value值为一个空list列表, KlineData 和 DepthData类的具体属性请参考 **1.1.2  on_data**

参数 2 ：数据获取状态


示例代码：

```python
data, status = e.pull_bar()    # data 为返回的数据， status 为获取数据的状态信息
print(data["BTC/USD.OK.Q"][0].open)     # 获取下一个k线数据的开盘价
```

### 2.4 下单 - **place_order**

函数说明：下单函数，根据策略回测信号主动触发下单

函数原型：

```python
def place_order(self,symbol,direction,order_time,quantity,open_or_close,order_type ,fillprice ,limit_price=0):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| symbol        | string   | 交易品种         | BTC/USDT.BN             |
| direction     | string   | 交易方向         |  Buy（买）、Sell（卖）                        |
| order_time    | string   | 交易时间         |                         |
| quantity      | float   | 交易手数或下单量 |                         |
| open_or_close | string   | 开仓或平仓       | Open（开仓）, Close（平仓）             |
| order_type    | string   | 订单类型         | 0:市价单，1：限价单     |
| fillprice   | float   | 订单成交价         |  |
| limit_price   | string   | 订单限价         | order_type填0此处可不填 |


返回参数：无

示例代码：

```python
e.place_order(symbol = 'BTC/USD.OK.Q', direction = 'Buy', order_type = 0,  order_time = '2019-10-01 00:00:00',open_or_close = 'Open', quantity = 100, fillprice = 1000)
``` 




### 2.5 获取所有订单 - **get_orders**

函数说明：获取当前策略的所有订单

返回参数：返回 list列表，其中每个元素为一个 OrderModel 对象

OrderModel 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | 品种   |
| direction   | string   | 订单方向（Buy 和 Sell）   |
| offset    | string   | 交易类型（Open 开仓，Close平仓）   |
| ordertype    | int   | 订单类型（0：市价单，1：限价单）   |
| limitprice    | float   | 限制成交价   |
| fillprice    | float   | 成交价   |
| quote_volume    | float   | 基础币种成交量   |
| volumemultiple    | float   | 合约乘数   |
| quantity    | int   | 交易手数   |
| strategyid   | string   | 策略id   |
| ordertime    | datetune   | 下单时间   |
| jobid    | string   | 回测id   |
| orderfee    | float   | 订单手续费   |
| status    | string   | 订单状态（fill:成交、invalid: 无效）   |
| filltime    | datetime   | 成交时间   |
| orderrid    | string   | 订单id   |

示例代码：

```python
data = e.get_orders()
print(data[0].symbol)
```




### 2.6 获取当前账户的可用资金 - **get_account**

函数说明：获取当前账户的可用资金

返回参数：可用资金（float类型）

示例代码：

```python
print(e.get_account())
```


### 2.7 获取持仓量 - **get_holdings**

函数说明：获取当前账户的持仓量

返回参数：一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个 Holding(持仓量)的list数组

参数说明：

Holding 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | 品种   |
| direction    | string   | 方向   |
| quantity    | float   | 持仓手数   |
| volumemultiple    | float   | 合约乘数   |
| averageprice    | float   | 开仓平均成本   |

示例代码：

```python
holds = e.get_holdings()
if 'BTC/USD.OK.Q' in e.get_holdings().keys():
    for h in holds['BTC/USD.OK.Q']:
        print(h.direction)   # 输出持仓方向
```




### 2.8 结束回测 - **client_close**

函数说明： 结束回测

返回参数：无

示例代码：

```python
e.client_close()
```




### 2.9 获取数据文件存放的路径 - **get_local_csvpath**

函数说明：返回存放数据的文件路径

返回参数：string 类型

示例代码：

```python
e.get_local_csvpath()
```





## 3. 数据获取(通过api)

在购买访问数据api的权限后，可直接通过调用sdk中的方法，获取现货的历史k线数据、现货交易对信息、期货交易对信息、指数交易对信息以及期权交易对信息。其相对应的函数命名如下：

- **get_history_kline_data_by_api**：获取现货的历史k线数据
- **get_spot_symbol_by_api**：获取现货交易对信息
- **get_future_symbol_by_api**：获取期权交易对信息
- **get_index_symbol_by_api**：获取指数交易对信息
- **get_option_symbol_by_api**：获取期权交易对信息


在调用以上数据获取方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.datagetting_sdk import *
```


### 3.1 获取现货的历史k线数据 - **get_history_kline_data_by_api**

函数说明：通过提供Api Key、Api Secret以及相关参数获取指定时间范围的现货K线数据

函数原型：

```python
def get_history_kline_data_by_api(api_key, api_secret, symbol, frequency, starttime='', endtime=''):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| api_key        | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取     |             |
| api_secret     | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取         |                          |
| symbol    | string   | 交易品种         |  格式为：基础币种/计价币种.交易对简称，如：BTC/USDT.BN                        |
| period      | string   | k线周期 | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时）                       |
| starttime | string   | 开始时间       | 时间格式：'%Y-%m-%d %H:%M:%S'          |
| endtime    | string   | 结束时间         | 同上    |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_history_kline_data_by_api(api_key = '***', api_secret = '***', symbol = 'BTC/USDT.BN', period =  '1m', starttime='2019-10-10 00:00:00', endtime='2019-10-01 00:00:00')
``` 

注意：

1、如果starttime和endtime参数同时为空，则返回最近的200条k线数据;starttime和endtime参数要么同时为空，要么同时不为空。


### 3.2 获取现货交易对信息 - **get_spot_symbol_by_api**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_spot_symbol_by_api(api_key, api_secret,exchange, symbol):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| api_key        | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取     |             |
| api_secret     | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取         |                          |
| exchange    | string   | 交易所        |  交易所全称或交易所简称                        |
| symbol      | string   | 交易品种 | 格式为：基础币种/计价币种，如：BTC/USDT                    |

返回参数：list数组，其中每一项为SymbolModel对象

参数说明：

SymbolModel 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | matrixdata统一交易对   |
| baseasset    | string   | 基础币种   |
| quoteasset    | string   | 计价币种   |
| type    | SymbolTypeModel   | 交易对类型，具体属性参考 SymbolTypeModel类属性说明  |
| status    | string   | 状态   |
| displaysymbol    | string   | 交易所显示交易对   |
| tradesymbol    | string   | 交易所接口交易对   |
| exchange    | string   | 交易所   |
| listedtime    | string   | 上市时间   |
| delistedtime    | string   | 退市时间   |


SymbolTypeModel 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symboltype    | string   | 交易对类型   |
| subsymboltype    | string   | 交易对子类型   |

示例代码：

```python
from samdata_terminal_dev.datagetting_sdk import *

data = get_spot_symbol_by_api('***', '***', 'BN', 'BTC/USDT')
``` 




### 3.3 获取期货交易对信息 - **get_future_symbol_by_api**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_future_symbol_by_api(api_key, api_secret,exchange, symbol):
```

参数说明：具体参考 获取交易对信息 - **get_spot_symbol_by_api**的参数说明

返回参数：list数组，其中每一项为FuturesModel对象

参数说明：

FuturesModel 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | matrixdata统一交易对   |
| baseasset    | string   | 基础币种   |
| quoteasset    | string   | 计价币种   |
| contractmonth    | int   | 合约乘数   |
| contractmultiple    | int   |    |
| listedtime    | string   | 上市时间   |
| delistedtime    | string   | 退市时间   |
| deliverydate    | string   | 交割日   |
| deliverymonth    | int   | 交割月份   |
| deliveryprice    | float   | 交割价   |
| minprice    |  float  | 最小变动价位  |
| type    | SymbolTypeModel   | 交易对类型，具体属性参考 SymbolTypeModel类属性说明  |
| status    | string   | 状态   |
| displaysymbol    | string   | 交易所显示交易对   |
| tradesymbol    | string   | 交易所接口交易对   |
| exchange    | string   | 交易所   |
| latestcontract    | int   | 最新合约   |

示例代码：

```python
from samdata_terminal_dev.datagetting_sdk import *

data = get_spot_symbol('***', '***', 'BN', 'BTC/USD')
``` 




### 3.4 获取指数交易对信息 - **get_index_symbol_by_api**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_index_symbol_by_api(api_key, api_secret,exchange, symbol):
```

参数说明：具体参考 获取交易对信息 - **get_spot_symbol_by_api**的参数说明

返回参数：list数组，其中每一项为IndexModel对象

参数说明：

IndexModel 类属性说明

| 属性        | 数据类型 | 描述             |
| ------------- | -------- | ---------------- | 
| symbol    | string   | matrixdata统一交易对   |
| baseasset    | string   | 基础币种   |
| quoteasset    | string   | 计价币种   |
| type    | SymbolTypeModel   | 交易对类型，具体属性参考 SymbolTypeModel类属性说明  |
| status    | string   | 状态   |
| displaysymbol    | string   | 交易所显示交易对   |
| tradesymbol    | string   | 交易所接口交易对   |
| exchange    | string   | 交易所   |
| listedtime    | string   | 上市时间   |
| delistedtime    | string   | 退市时间   |

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk

data = get_index_symbol_by_api('***', '***', 'BN', 'BTC/USDT')
``` 




### 3.5 获取期权交易对信息 - **get_option_symbol_by_api**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_option_symbol_by_api(api_key, api_secret,exchange, symbol):
```

参数说明：具体参考 获取交易对信息 - **get_spot_symbol_by_api**的参数说明

返回参数：list数组，其中每一项为IndexModel对象，其属性介绍请参考 获取指数交易对信息 - **get_index_symbol_by_api**的IndexModel对象介绍

示例代码：

```python
from samdata_terminal_dev.datagetting_sdk import *

data = get_option_symbol_by_api('***', '***', 'BN', 'BTC/USDT')
``` 

## 4. 数据获取(本地数据库)

从本地postgresql数据库中获取k线数据。其相对应的函数命名如下：

- **get_sync_spot_kline_data**：获取现货的k线数据
- **get_sync_future_kline_data**：获取期货的k线数据

在调用以上数据获取方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.datagetting_sdk import *
```




### 4.1 获取现货的k线数据 - **get_sync_spot_kline_data**

函数说明：通过提供Api Key、Api Secret以及相关参数获取指定时间范围的现货K线数据

函数原型：

```python
def def get_sync_spot_kline_data(server_url, port, username, password, exchange, base_symbol, asset_symbol, period, starttime, endtime):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| servel_url        | string   | 数据库url     |             |
| port        | string   |数据库端口号     |             |
| username        | string   |数据库登录名     |             |
| password        | string   | 数据库密码     |             |
| exchange        | string   |交易所     |             |
| base_symbol     | string   | 基础币种         |                          |
| asset_symbol    | string   | 计价品种         |  格式为：基础币种/计价币种.交易对简称，如：BTC/USDT.BN                        |
| period      | string   | k线周期 | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时）                       |
| starttime | string   | 开始时间       | 时间格式如：'2019-09-09 10:00:00'          |
| endtime    | string   | 结束时间         | 时间格式如：'2019-09-09 10:00:00'     |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_sync_spot_kline_data('127.0.0.1', '3306', '123', '123','BN', 'BTC', 'USDT', '1m', '2019-08-01 00:00:00', '2019-08-01 04:00:00')
``` 

注意：

1、starttime和endtime都不能为空




### 4.2 获取期货的k线数据 - **get_sync_future_kline_data**

函数说明：通过提供Api Key、Api Secret以及相关参数获取指定时间范围的现货K线数据

函数原型：

```python
def def get_sync_future_kline_data(server_url, port, username, password, exchange, base_symbol, asset_symbol, future_type, period, starttime, endtime):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| servel_url        | string   | 数据库url     |             |
| port        | string   |数据库端口号     |             |
| username        | string   |数据库登录名     |             |
| password        | string   | 数据库密码     |             |
| exchange        | string   |交易所     |             |
| base_symbol     | string   | 基础币种         |                          |
| asset_symbol    | string   | 计价品种         |  格式为：基础币种/计价币种.交易对简称，如：BTC/USDT.BN                        |
| period      | string   | k线周期 | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时）                       |
| starttime | string   | 开始时间       | 时间格式如：'2019-09-09 10:00:00'          |
| endtime    | string   | 结束时间         | 时间格式如：'2019-09-09 10:00:00'     |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_sync_future_kline_data('127.0.0.1', '3306', '123', '123','BN', 'BTC', 'USDT', '1m', '2019-08-01 00:00:00', '2019-08-01 04:00:00')
``` 

注意：

1、starttime和endtime都不能为空


## 5. 数据获取(本地csv)

从数据同步后的本地csv路径中获取k线数据、盘口数据。其相对应的函数命名如下：

- **get_spot_kline_data**：获取现货的 k线 数据
- **get_future_kline_data**：获取期货的 k线 数据
- **get_spot_depth_data**：获取现货的 盘口 数据

在调用以上数据获取方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.datagetting_sdk import *
```




### 5.1 获取现货的k线数据 - **get_spot_kline_data**

函数说明：从数据同步后的本地csv路径获取指定时间范围的现货K线数据

函数原型：

```python
def def get_spot_kline_data(exchange, base_symbol, asset_symbol, period, starttime, endtime):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| exchange        | string   |交易所     |             |
| base_symbol     | string   | 基础币种         |                          |
| asset_symbol    | string   | 计价品种         |                         |
| period      | string   | k线周期 | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时）                       |
| starttime | string   | 开始时间       | 时间格式如：'2019-09-09 10:00:00'          |
| endtime    | string   | 结束时间         | 时间格式如：'2019-09-09 10:00:00'     |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_spot_kline_data('BN', 'BTC', 'USDT', '1m', '2019-08-01 00:00:00', '2019-08-01 04:00:00')
``` 

注意：

1、starttime和endtime都不能为空




### 5.2 获取期货的k线数据 - **get_future_kline_data**

函数说明：从数据同步后的本地csv路径获取指定时间范围的现货K线数据

函数原型：

```python
def get_future_kline_data(exchange, base_symbol, asset_symbol, future_type, period, starttime, endtime):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| exchange        | string   |交易所     |             |
| base_symbol     | string   | 基础币种         |                          |
| asset_symbol    | string   | 计价品种         |                          |
| period      | string   | k线周期 | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时）                       |
| future_type | string   | 期货类型       | Q（季度）、TW（当周）、NW（次周）          |
| starttime | string   | 开始时间       | 时间格式如：'2019-09-09 10:00:00'          |
| endtime    | string   | 结束时间         | 时间格式如：'2019-09-09 10:00:00'     |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_future_kline_data('BN', 'BTC', 'USDT', 'Q','1m', '2019-08-01 00:00:00', '2019-08-01 04:00:00')
``` 

注意：

1、starttime和endtime都不能为空



### 5.3 获取现货的盘口数据 - **get_spot_depth_data**

函数说明：从数据同步后的本地csv路径获取指定时间范围的现货盘口数据

函数原型：

```python
def def get_spot_depth_data(exchange, base_symbol, asset_symbol, starttime, endtime):
```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                    |
| ------------- | -------- | ---------------- | ----------------------- |
| exchange        | string   |交易所     |             |
| base_symbol     | string   | 基础币种         |                          |
| asset_symbol    | string   | 计价品种         |                        |
| starttime | string   | 开始时间       | 时间格式如：'2019-09-09 10:00:00'          |
| endtime    | string   | 结束时间         | 时间格式如：'2019-09-09 10:00:00'     |

返回参数：list数组，其中每一项为KlineData对象，DepthData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_spot_depth_data('BN', 'BTC', 'USDT', '2019-08-01 00:00:00', '2019-08-01 04:00:00')
``` 

注意：

1、starttime和endtime都不能为空


# End
