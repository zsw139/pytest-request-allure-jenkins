# pytest-request-allure-jenkins
接口自动化
#接口自动化

代码框架选择以requests+pytest+allure 技术为底层框架进行编写。

##所需要安装第三方库

```shell script
pip install requests
pip install pytest
pip install pytest-html
```
allure安装：https://blog.csdn.net/chenfei_5201213/article/details/80982929

##一、框架基础原理

requests：python常用的http接口请求第三方库

使用简介：

```python
import requests

url = "http://192.168.1.188:8001/meeu/admin/aliyunRpb/cancelRpb"
header = {
    "Connection": "keep-alive",
    "Pragma": "no-cache",
    "Cache-Control": "no-cache",
    "Accept": "*/*",
    "X-Requested-With": "XMLHttpRequest",
    "Content-Type": "application/json;charset=utf-8",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1MjY0Njg1MDMzNTEsInBheWxvYWQiOiJudWxsIn0.1Rjp50pq7R-Wj-7n_po7f5arxt3nUpjesQbHM1Tzr_Q"
}
data = {
      "appId": "meeu",
      "appType": 0,
      "appVersion": "string",
      "cancelReasonType": 0,
      "cancelUserId": 0,
      "languageType": 1,
      "sex": 0,
      "skipMaintenance": 0,
      "terminalType": 0,
      "token": "cf74cfcd-a41a-43e9-95ea-27d21157bc23",
      "userId": 0
    }
#一个http接口请求，resp就是获取到了接口的返回
resp = requests.post(url=url, headers=header, json=data)
print(resp.text)

```


pytest：是一个非常成熟的全功能的Python测试框架，类似ununittest，通用测试框架，提供丰富插件，相对与ununittest代码编写简单，功能更加丰富。

使用简介：

```python
import pytest
import requests
import json

def test_认证():
    url = "http://192.168.1.188:8001/meeu/admin/aliyunRpb/cancelRpb"
    header = {
        "Connection": "keep-alive",
        "Pragma": "no-cache",
        "Cache-Control": "no-cache",
        "Accept": "*/*",
        "X-Requested-With": "XMLHttpRequest",
        "Content-Type": "application/json;charset=utf-8",
        "Accept-Encoding": "gzip, deflate",
        "Accept-Language": "en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7",
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1MjY0Njg1MDMzNTEsInBheWxvYWQiOiJudWxsIn0.1Rjp50pq7R-Wj-7n_po7f5arxt3nUpjesQbHM1Tzr_Q"
    }
    data = {
          "appId": "meeu",
          "appType": 0,
          "appVersion": "string",
          "cancelReasonType": 0,
          "cancelUserId": 0,
          "languageType": 1,
          "sex": 0,
          "skipMaintenance": 0,
          "terminalType": 0,
          "token": "cf74cfcd-a41a-43e9-95ea-27d21157bc23",
          "userId": 0
        }
    resp = requests.post(url=url, headers=header, json=data)
    resps = json.loads(resp.text)
    assert resps["data"] == 1


if __name__ =="__main__":
    pytest.main()
```

在当前文件目录下执行输入：pytest，代码就会运行。获取数据并检查。

```shell script
hulbdeMacBook-Air:demo dtwave$ pytest 
========================================================================================================================== test session starts ==========================================================================================================================
platform darwin -- Python 3.7.0, pytest-5.2.4, py-1.8.0, pluggy-0.13.1
sensitiveurl: .*
rootdir: /Users/dtwave/Documents/mc_robot/mc-robot, inifile: pytest.in
plugins: html-2.0.1, zytest-0.1.9, variables-1.9.0, base-url-1.4.1, rerunfailures-8.0, selenium-1.17.0, allure-pytest-2.8.11, metadata-1.8.0
collected 1 item                                                                                                                                                                                                                                                        

test_demo.py .                                                                                                                                                                                                                                                    [100%]

=========================================================================================================================== 1 passed in 3.20s ===========================================================================================================================
```

allure：是一种灵活的轻量级多语言测试报告工具，它不仅能够以简洁的web报告形式显示已测试的内容，而且允许参与开发过程的每个人从测试的日常执行中提取最大限度的有用信息。

allure是个生成可视化报告第三方库，需要本地安装。

代码核心原理就是使用request请求接口，再集成pytest框架触发接口请求，断言，最后使用allure或pytest-html来生成报告。

##二、request的封装

按照上述request请求方式进行请求，很不便捷，代码冗余，使用操作困难。在PackageRequest.py文件中，对request请求进行封装成函数，将接口的信息保存在InterfaceData.py文件中，在用例文件中请求接口，就是调用PackageRequest.py里的函数来处理InterfaceData.py文件里的数据。

```python
from common.PackageRequest import *
from common.InterfaceData import *

def test_认证():
    """
    登录接口
    """
    resp = httpclent.http_client(cancel_certification_args)  #httpclent.http_client()是一个处理接口数据的函数，cancel_certification_args是一个字典，里面都是接口的信息，包括请求方法，url，params等信息
    assert resp["data"] == 1
```

##启动方式

由于使用pytest直接运行，命令长，不易用，例如想生成一个html报告，需要执行命令  pytest --html=../report_html/2020.html --self-contained-html，文件名称也需要自己定义。假如想运行固定测试用例文件更加痛苦。针对于该问题，在bin目录下封装了一个启动脚本run.py。
支持运行方式

```shell script
./run.py   #关键字运行，会根据在case文件放的关键字去运行代码顺序，或不运行哪些文件等。
./run.py -all #运行所有用例文件
./run.py -mark main  #会根据在pytest.ini文件中设置的tag，去运行打了标记的用例。mark介绍：https://www.cnblogs.com/xingyunqiu/p/11734226.html
./run.py -cata /test_实人认证 #运行指定目录下的文件
./run.py -t   #上述命令后加-t，不会生成allure报告，会生成html报告，并发送邮件报告（还未实现）
```

启动文件代码实现原理：

通过运行./run.py文件，生成固定的pytest命令，然后使用sys.system()函数来运行命令

代码运行大致原理：

关键字运行： 遍历/test_case目录下所有用例文件，放在一个列表中，再去读取case文件中的关键字进行匹配，匹配上后就会去生成命令运行用例

全部运行： 直接使用sys.system()函数，跳转到/test_case目录下执行命令，运行所有用例文件

指定mark tag运行：在pytest全局配置文件 pytest.ini文件中配置mark tag， 写接口用例时，可以将用例打上tag，指定tag运行时，就会去运行所有打上tag的用例。

指定目录运行： 通过传入参数目录名称，去运行指定目录下的用例。

##参数化运行

参数化运行是使用pytest的一个内置装饰器@pytest.mark.parametrize实现，装饰器原理：https://cloud.tencent.com/developer/article/1527541

```python
@pytest.mark.parametrize('data', AppId)    #AppId是从InterfaceData.py数据文件中获取的一个列表，装饰器会读取列表中的元素，每个元素都会作为入参传入用例中，每次传入运行一遍用例，列表中有多少入参，就会运行多少次用例。
def test_检查_appId(data):
    """
    appId
    """
    resp = httpclent.http_client(cancel_certification_args, appId=data[0])  #appId字段或获取列表中的入参数据，作为传参，其他参数会去默认数据
    assert resp["data"] == data[1]         #断言也会去取列表中相应的数据

```
```python
cancel_certification_args = {
    "method": "post",
    "url": "/meeu/admin/aliyunRpb/cancelRpb",
    "params": {
      "appId": "meeu",
      "appType": 0,
      "appVersion": "string",
      "cancelReasonType": 0,
      "cancelUserId": 0,
      "languageType": 1,
      "sex": 0,
      "skipMaintenance": 0,
      "terminalType": 0,
      "token": "cf74cfcd-a41a-43e9-95ea-27d21157bc23",
      "userId": 0
    }
}
AppId = [["aa7", 1], ["aa6", 5],["aa5",2],["aa4",1],["aa3",1]]
AppType = [["aa7", 1], ["aa6", 5],["aa5",2],["aa4",1],["aa3",1]]

```

##写法demo

例如：需要新写一个实人认证中的取消认证接口

1、在test_case目录下新建模块『test_实人认证』的目录，在实人认证目录下新建一个『test_取消认证.py』的接口用例文件。必须都已『test_』为开头。

2、在『case/case』文件中，添加该用例文件关键字，接口用例文件名为『test_取消认证.py』，那关键字就是『取消认证』，要与用例文件名称对应。

3、在InterfaceData.py文件中，添加取消认证接口的默认基础信息，标好备注，接口数据名称变量不能重复

```python
#取消认证接口
cancel_certification_args = {
    "method": "post",
    "url": "/meeu/admin/aliyunRpb/cancelRpb",
    "params": {
      "appId": "meeu",
      "appType": 0,
      "appVersion": "string",
      "cancelReasonType": 0,
      "cancelUserId": 0,
      "languageType": 1,
      "sex": 0,
      "skipMaintenance": 0,
      "terminalType": 0,
      "token": "cf74cfcd-a41a-43e9-95ea-27d21157bc23",
      "userId": 0
    }
}
```

4、在创建的『test_取消认证.py』文件中，调用PackageRequest.py和InterfaceData.py模块中的所有方法，然后开始写接口用例
```python
from common.PackageRequest import *
from common.InterfaceData import *
import pytest
```

5、开始编写用例，定义一个函数，函数名称以『test_』为开头，后面加上这个接口用例作用，例如『test_取消认证()』，在函数下面直接编写用例内容请求接口，然后再进行断言。
```python
def test_取消认证():
    """
    取消认证
    """
    resp = httpclent.http_client(cancel_certification_args)
    assert resp["data"] == 1                #返回的resp已经处理成了一个dict格式http接口请求返回数据
```

6、保存，进入命令行，在『test_实人认证』目录下执行 pytest test_取消认证.py 该用例就会执行。

7、参数化运行，在实际接口测试中，需要对一个接口的入参进行校验，需要传入多种不同参数对接口进行校验。覆盖接口不同的场景。
```python
def test_取消认证():
    """
    登录接口
    """
    resp = httpclent.http_client(cancel_certification_args)
    assert resp["data"] == 1

AppId = [["aa7", 1], ["aa6", 5],["aa5",2],["aa4",1],["aa3",1]]
@pytest.mark.parametrize('data', AppId)    #将AppId的列表数据赋值给变量data，该装饰器会根据data里元素的个数来运行用例，每一个元素就使用该元素数据运行一次用例。
def test_取消认证_appId(data):
    """
    appId
    """
    resp = httpclent.http_client(cancel_certification_args, appId=data[0])
    assert resp["data"] == data[1]
```

将AppId的列表数据赋值给变量data，该装饰器会根据data里元素的个数来运行用例，每一个元素就使用该元素数据运行一次用例。例如AppId传入的数据，每个元素是["aa7", 1]，每次取的时候，将"aa7"作为入参，1作为断言。直接运行用例就会根据参数运行不同场景用例。

由于数据和用例分离原则，需要将AppId值放在InterfaceData.py文件中。

8、用例打tag运行。在测试过程中，有时候冒烟和回归测试，只需要跑主干核心场景，不需要跑太多异常场景，一次性跑太多，时间久，数据不容易分析，这个时候可以给用例打上tag，运行同一个tag的用例。

在pytest.ini文件中，定义mark，写用例时，在上方加上该mark tag，运行时指定tag就会跑指定tag的用例

```ini
[pytest] # 固定的section名

markers= # 固定的option名称

　　main: 主流程场景
```

```python
@pytest.mark.main      #main就是tag
def test_取消认证():
    """
    登录接口
    """
    resp = httpclent.http_client(cancel_certification_args)
    assert resp["data"] == 1
```

9、运行代码，在上述代码框架介绍中提及了不同的运行方式，可根据不同的方式去运行

```shell script
./run.py   #关键字运行，会根据在case文件放的关键字去运行代码顺序，或不运行哪些文件等。
./run.py -all #运行所有用例文件
./run.py -mark main  #会根据在pytest.ini文件中设置的tag，去运行打了标记的用例。mark介绍：https://www.cnblogs.com/xingyunqiu/p/11734226.html
./run.py -cata /test_实人认证 #运行指定目录下的文件
./run.py -t   #上述命令后加-t，不会生成allure报告，会生成html报告，并发送邮件报告（还未实现）
```

注：
    使用allure生成报告，需要本地安装allure插件，上面有安装教程链接
    
    本次代码为初版，后续可能会有些许更改，但大致写法不会变

##暂时未完成模块


测试准备与数据清理(需要结合具体业务看这么做比较合适)



