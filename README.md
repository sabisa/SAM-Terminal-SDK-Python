# SAM-Terminal-SDK-Python
# samdata 终端 python sdk 说明

> 项目功能简述：调用终端sdk接口

## 1. 运行

> 开发平台是Pycharm，可以用VS Code等IDE平台均可运行

## 2. 开发环境配置

系统：Win10

语言：Python3.7.3（需添加环境变量） 

下载地址：https://www.python.org/downloads/

相关包/库：numpy, pymysql

开发工具：Pycharm

## 3. 安装方法

用 pip 安装，当前最新版本为0.0.8

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
    server_url = '127.0.0.1'#'192.168.193.10'
    port = 9007
    username = "lily"
    password = "1111"
    strategyID = "7bcb1173-110a-3ef0-5745-54f641d631d6"  
    symbol_list = 'BTC/USD.OK.Q'  #订阅交易对
    time_type = '1m' #1:1m 2:5m 3:15m 4:30m 5:1h
    cash = 100000000  #initial cash
    starttime = "2019-10-01"
    endtime = "2019-10-02"
    account_type = 'CNY'
    excuation = 1   # 1:current bar  2: next bar
    running_type = 1
    e = SAMTraderEngine(server_url = server_url, port = port, username =username,password = password,\
            strategyID = strategyID,symbol_list = symbol_list,time_type = time_type,cash = cash,\
            starttime = starttime,endtime = endtime,fill_bar_type = excuation,running_type = running_type,account_type = account_type)
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

在调用SAMTrader入口函数类中的方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.backtest_sdk import SAMTrader
```

SAMTrader 对象内部初始函数：**<u>def _init_(self,initial,onData)</u>**

### 1.1  初始化函数 - initial

函数说明：初始化策略引擎

函数原型：def initial()

示例代码：

```python 
def initial():
    server_url = '127.0.0.1' #  数据库地址
    port = 9007  #  端口号
    userid = '3213'    # 用户id
    username = "lily"  #  用户名
    password = "1111"  #  用户密码
    strategyID = "7bcb1173-110a-3ef0-5745-54f641d631d6"  #  策略id
    symbol_list = 'BTC/USD.OK.Q'  #  订阅交易对
    time_type = '1m'  #  分钟精度：1m、5m、15m、30m、1h
    cash = 100000000  #  初始资金
    starttime = "2019-10-01"  #  回测开始时间
    endtime = "2019-10-02"  #  回测结束时间，不包含最后一天数据
    account_type = 'CNY'  #  账户类型 USD：美元,CNY:人民币
    excuation = 1   #  订单成交方式，1：当前bar价格成交，2：下一bar的价格成交
    running_type = 1  #  0：手动模式，返回数据库地址；1：自动模式，自动拉取行情数据；
    e = SAMTraderEngine(server_url = server_url, port = port, userid = userid， username =username,password = password,\
            strategyID = strategyID,symbol_list = symbol_list,time_type = time_type,cash = cash,\
            starttime = starttime,endtime = endtime,fill_bar_type = excuation,running_type = running_type,account_type = account_type)

    return e
```




### 1.2 策略回调函数 - **on_data**

函数说明： 策略回调函数（包含数据推送，策略下单，策略资金等等）

函数原型： on_data(bar, e)

参数说明： 

bars：一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个KlineData(K线数据)的list数组

e: 策略引擎

KlineData 类属性说明

| 属性              | 数据类型 | 描述                                                 |
| ----------------- | -------- | ---------------------------------------------------- |
| symbol            | string   | 品种                                                 |
| bar_time          | string   | 报价时间                                             |
| high              | float    | 最高价                                               |
| low               | float    | 最低价                                               |
| open              | float    | 开盘价                                               |
| close             | float    | 收盘价                                               |
| quote_volume      | float    | 基础币种成交量                                       |
| quoteasset_volume | float    | 计价币种成交量（okex期货合约表示合约张数，单位为张） |
| trade_num         | float    | 交易笔数                                             |


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
- **get_all_bars**：获取所以回测数据bar
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

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                                                         |
| ------------- | -------- | ---------------- | ------------------------------------------------------------ |
| server_url    | string   | 本地数据库地址   | 策略回测数据库                                               |
| port          | string   | 本地数据库端口号 |                                                              |
| username      | string   | 本地数据库用户名 |                                                              |
| password      | string   | 本地数据库密码   |                                                              |
| strategyID    | string   | 策略ID           | 根据用户创建策略给出的唯一ID,Guid                            |
| symbol_list   | string   | 回测品种         | 支持多品种，多个品种用英文半角逗号隔开                       |
| time_type     | string   | 数据类型         | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时） |
| cash          | string   | 初始资金         |                                                              |
| starttime     | string   | 回测开始时间     | 格式：'YYYY-mm-dd'                                           |
| endtime       | string   | 回测结束时间     | 不包含最后一天数据，格式同上                                 |
| fill_bar_type | string   | 成交方式类型     | 1：当前bar价格，2：下一bar的价格                             |
| running_type  | string   | 数据源类型       | 0： 手动模式，返回数据文件路径；1：自动模式，自动拉取行情数据 |
| account_type  | string   | 结算币种         | 账户类型 USD：美元,CNY:人民币，保留                          |
|               |          |                  |                                                              |


示例代码：

```python
SAMTraderEngine(self,"127.0.0.1", "3306", "1111", "12345" ,"7bcb1173-110a-3ef0-5745-54f641d631d6",\
    'BTC/USD.OK.Q','1m',10000,'2019-10-01','2019-10-02',1,1,"USD")
```




### 2.2 获取回测所有数据bar - **get_all_bars**

函数说明：根据回测引擎构造函数传入的参数，获取回测开始时间~结束时间所需的所有数据bar,导入内存变量

返回参数： 一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个KlineData(K线数据)的list数组返回 list列表，其中每个元素为一个KlineData 对象

示例代码：

```python
data = e.get_all_bars()
print(data["BTC/USD.OK.Q"][0].close)     # 打印第一个k线数据的收盘价,
```




### 2.3 获取下一个数据bar - **pull_bar**

函数说明：获取下一个数据bar的k线数据

返回参数：返回两个参数，一个为一个<string, list>的key-value对象，其中每一项的key是品种名，value是一个list列表，其中每个元素为一个KlineData 对象，但该列表只包含下一个数据 bar 的k线数据，如果无数据，该value值为一个空list列表


示例代码：

```python
data, status = e.pull_bar()
print(data["BTC/USD.OK.Q"][0].open)     # 获取下一个线数据的开盘价

```





### 2.4 下单 - **place_order**

函数说明：下单函数，根据策略回测信号主动触发下单

函数原型：

```python
def place_order(self,symbol,direction,order_time,quantity,open_or_close,order_type ,fillprice ,limit_price=0):

```

参数说明：

| 参数名        | 数据类型 | 说明             | 备注                        |
| ------------- | -------- | ---------------- | --------------------------- |
| symbol        | string   | 交易品种         | BTC/USDT.BN                 |
| direction     | string   | 交易方向         | Buy（买）、Sell（卖）       |
| order_time    | string   | 交易时间         |                             |
| quantity      | string   | 交易手数或下单量 |                             |
| open_or_close | string   | 开仓或平仓       | Open（开仓）, Close（平仓） |
| order_type    | string   | 订单类型         | 0:市价单，1：限价单         |
| fillprice     | string   | 订单成交价       |                             |
| limit_price   | string   | 订单限价         | order_type填0此处可不填     |


返回参数：无

示例代码：

```python
e.place_order(symbol = 'BTC/USD.OK.Q', direction = 'Buy', order_type = 0,  order_time = dt,open_or_close = 'Open', quantity = 100, fillprice = "1000")

```




### 2.5 获取所有订单 - **get_orders**

函数说明：获取当前策略的所有订单

返回参数：返回 list列表，其中每个元素为一个 OrderModel 对象

OrderModel 类属性说明

| 属性           | 数据类型 | 描述                                 |
| -------------- | -------- | ------------------------------------ |
| symbol         | string   | 品种                                 |
| direction      | string   | 订单方向（Buy 和 Sell）              |
| offset         | string   | 交易类型（Open 开仓，Close平仓）     |
| ordertype      | int      | 订单类型（0：市价单，1：限价单）     |
| limitprice     | float    | 限制成交价                           |
| fillprice      | float    | 成交价                               |
| quote_volume   | float    | 基础币种成交量                       |
| volumemultiple | float    | 合约乘数                             |
| quantity       | int      | 交易手数                             |
| strategyid     | string   | 策略id                               |
| ordertime      | datetune | 下单时间                             |
| jobid          | string   | 回测id                               |
| orderfee       | float    | 订单手续费                           |
| status         | string   | 订单状态（fill:成交、invalid: 无效） |
| filltime       | datetime | 成交时间                             |
| orderrid       | string   | 订单id                               |

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

| 属性           | 数据类型 | 描述         |
| -------------- | -------- | ------------ |
| symbol         | string   | 品种         |
| direction      | string   | 方向         |
| quantity       | float    | 持仓手数     |
| volumemultiple | float    | 合约乘数     |
| averageprice   | float    | 开仓平均成本 |

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





## 3. 数据获取

在购买访问数据api的权限后，可直接通过调用sdk中的方法，获取现货的历史k线数据、现货交易对信息、期货交易对信息、指数交易对信息以及期权交易对信息。其相对应的函数命名如下：

- **get_history_kline_data**：获取现货的历史k线数据
- **get_spot_symbol**：获取现货交易对信息
- **get_future_symbol**：获取期权交易对信息
- **get_index_symbol**：获取指数交易对信息
- **get_option_symbol**：获取期权交易对信息

在调用以上数据获取方法前，需要先执行下面代码引用相关库

```python 
from samdata_terminal_dev.datagetting_sdk import *

```


### 3.1 获取现货的历史k线数据 - **get_history_kline_data**

函数说明：通过提供Api Key、Api Secret以及相关参数获取指定时间范围的现货K线数据

函数原型：

```python
def get_history_kline_data(api_key, api_secret, symbol, frequency, start='', end=''):

```

参数说明：

| 参数名     | 数据类型 | 说明                                            | 备注                                                         |
| ---------- | -------- | ----------------------------------------------- | ------------------------------------------------------------ |
| api_key    | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取 |                                                              |
| api_secret | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取 |                                                              |
| symbol     | string   | 交易品种                                        | 格式为：基础币种/计价币种.交易对简称，如：BTC/USDT.BN        |
| frequency  | string   | k线周期                                         | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时） |
| start      | string   | 开始时间                                        | UTC国际毫秒时间戳（13位）                                    |
| end        | string   | 结束时间                                        | UTC国际毫秒时间戳（13位）                                    |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_history_kline_data('***', '***', 'BTC/USDT.BN', '1m', start='1584953212000', end='1585039612000')

```

注意：

1、如果start和end参数同时为空，则返回最近的200条k线数据;start和end参数要么同时为空，要么同时不为空。


### 3.2 获取现货交易对信息 - **get_spot_symbol**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_spot_symbol(api_key, api_secret,exchange, symbol):

```

参数说明：

| 参数名     | 数据类型 | 说明                                            | 备注                                    |
| ---------- | -------- | ----------------------------------------------- | --------------------------------------- |
| api_key    | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取 |                                         |
| api_secret | string   | 访问获取数据的Api Key ,具体请从官网个人中心获取 |                                         |
| exchange   | string   | 交易所                                          | 交易所全称或交易所简称                  |
| symbol     | string   | 交易品种                                        | 格式为：基础币种/计价币种，如：BTC/USDT |

返回参数：list数组，其中每一项为SymbolModel对象

参数说明：

SymbolModel 类属性说明

| 属性          | 数据类型        | 描述                                               |
| ------------- | --------------- | -------------------------------------------------- |
| symbol        | string          | matrixdata统一交易对                               |
| baseasset     | string          | 基础币种                                           |
| quoteasset    | string          | 计价币种                                           |
| type          | SymbolTypeModel | 交易对类型，具体属性参考 SymbolTypeModel类属性说明 |
| status        | string          | 状态                                               |
| displaysymbol | string          | 交易所显示交易对                                   |
| tradesymbol   | string          | 交易所接口交易对                                   |
| exchange      | string          | 交易所                                             |
| listedtime    | string          | 上市时间                                           |
| delistedtime  | string          | 退市时间                                           |


SymbolTypeModel 类属性说明

| 属性          | 数据类型 | 描述         |
| ------------- | -------- | ------------ |
| symboltype    | string   | 交易对类型   |
| subsymboltype | string   | 交易对子类型 |

示例代码：

```python
from samdata_terminal_dev.datagetting_sdk import *

data = get_spot_symbol('***', '***', 'BN', 'BTC/USDT')

```




### 3.3 获取期货交易对信息 - **get_future_symbol**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_future_symbol(api_key, api_secret,exchange, symbol):

```

参数说明：具体参考 获取交易对信息 - **get_spot_symbol**的参数说明

返回参数：list数组，其中每一项为FuturesModel对象

参数说明：

FuturesModel 类属性说明

| 属性             | 数据类型        | 描述                                               |
| ---------------- | --------------- | -------------------------------------------------- |
| symbol           | string          | matrixdata统一交易对                               |
| baseasset        | string          | 基础币种                                           |
| quoteasset       | string          | 计价币种                                           |
| contractmonth    | int             | 合约乘数                                           |
| contractmultiple | int             |                                                    |
| listedtime       | string          | 上市时间                                           |
| delistedtime     | string          | 退市时间                                           |
| deliverydate     | string          | 交割日                                             |
| deliverymonth    | int             | 交割月份                                           |
| deliveryprice    | float           | 交割价                                             |
| minprice         | float           | 最小变动价位                                       |
| type             | SymbolTypeModel | 交易对类型，具体属性参考 SymbolTypeModel类属性说明 |
| status           | string          | 状态                                               |
| displaysymbol    | string          | 交易所显示交易对                                   |
| tradesymbol      | string          | 交易所接口交易对                                   |
| exchange         | string          | 交易所                                             |
| latestcontract   | int             | 最新合约                                           |

示例代码：

```python
from samdata_terminal_dev.datagetting_sdk import *

data = get_spot_symbol('***', '***', 'BN', 'BTC/USD')

```




### 3.4 获取指数交易对信息 - **get_index_symbol**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_index_symbol(api_key, api_secret,exchange, symbol):

```

参数说明：具体参考 获取交易对信息 - **get_spot_symbol**的参数说明

返回参数：list数组，其中每一项为IndexModel对象

参数说明：

IndexModel 类属性说明

| 属性          | 数据类型        | 描述                                               |
| ------------- | --------------- | -------------------------------------------------- |
| symbol        | string          | matrixdata统一交易对                               |
| baseasset     | string          | 基础币种                                           |
| quoteasset    | string          | 计价币种                                           |
| type          | SymbolTypeModel | 交易对类型，具体属性参考 SymbolTypeModel类属性说明 |
| status        | string          | 状态                                               |
| displaysymbol | string          | 交易所显示交易对                                   |
| tradesymbol   | string          | 交易所接口交易对                                   |
| exchange      | string          | 交易所                                             |
| listedtime    | string          | 上市时间                                           |
| delistedtime  | string          | 退市时间                                           |

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk

data = get_index_symbol('***', '***', 'BN', 'BTC/USDT')

```




### 3.5 获取期权交易对信息 - **get_option_symbol**

函数说明：通过提供Api Key、Api Secret以及相关参数获取交易对信息

函数原型：

```python
def get_index_symbol(api_key, api_secret,exchange, symbol):

```

参数说明：具体参考 获取交易对信息 - **get_spot_symbol**的参数说明

返回参数：list数组，其中每一项为IndexModel对象，其属性介绍请参考 获取指数交易对信息 - **get_index_symbol**的IndexModel对象介绍

示例代码：

```python
from samdata_terminal_dev.datagetting_sdk import *

data = get_option_symbol('***', '***', 'BN', 'BTC/USDT')

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
def get_sync_spot_kline_data(exchange, base_symbol, asset_symbol, period, starttime, endtime):

```

参数说明：

| 参数名       | 数据类型 | 说明     | 备注                                                         |
| ------------ | -------- | -------- | ------------------------------------------------------------ |
| exchange     | string   | 交易所   |                                                              |
| base_symbol  | string   | 基础币种 |                                                              |
| asset_symbol | string   | 计价品种 | 格式为：基础币种/计价币种.交易对简称，如：BTC/USDT.BN        |
| period       | string   | k线周期  | 1m（1分钟）、5m（5分钟）、15m（15分钟）、30m（30分钟）、1h（1小时） |
| starttime    | string   | 开始时间 | 时间格式如：'2019-09-09 10:00:00'                            |
| endtime      | string   | 结束时间 | 时间格式如：'2019-09-09 10:00:00'                            |

返回参数：list数组，其中每一项为KlineData对象，KlineData类的具体属性请参考 策略回调函数 - **on_data** 的返回参数

示例代码：

```python
import samdata_terminal_dev.datagetting_sdk
data = get_sync_spot_kline_data('BN', 'BTC', 'USDT', '1m', '2019-08-01 00:00:00', '2019-08-01 04:00:00')

```

注意：

1、starttime和endtime都不能为空

# End
