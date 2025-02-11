# IPProxyTool
Используйте Scrapy Cracker для сканирования прокси-сайтов и получения большого количества бесплатных IP-адресов прокси. Отфильтруйте все доступные IP-адреса и сохраните их в базе данных для дальнейшего использования. Вы можете ознакомиться с другими моими интересными проектами, посетив мой личный сайт awolfly9.

Спасибо YoungJeff за поддержку этого проекта вместе со мной.

## Рабочая среда
Установите базу данных Python3 и MySQL.

среда установки модуля шифрования:
```
sudo yum install gcc libffi-devel python-devel openssl-devel
```


```
$ pip install -r requirements.txt
```



## Загрузите и используйте
Клонировать проект локально

```
$ git clone https://github.com/awolfly9/IPProxyTool.git
```

Введите каталог проекта

```
$ cd IPProxyTool
```
Измените имя пользователя и пароль базы данных_config в конфигурации базы данных mysql [config.py](https://github.com/awolfly9/IPProxyTool/blob/master/config.py) 中 database_config на имя пользователя и пароль базы данных.

```
$ vim config.py
---------------

database_config = {
	'host': 'localhost',
	'port': 3306,
	'user': 'root',
	'password': '123456',
	'charset': 'utf8',
}
```

MYSQL: 导入数据表结构
```
$ mysql> create database ipproxy;
Query OK, 1 row affected (0.00 sec)
$ mysql> use ipproxy;
Database changed
$ mysql> source '/你的项目目录/db.sql'

```


Запустите сценарий запуска ipproxytool.py. Вы также можете запустить сценарии сканирования, проверки и интерфейса сервера отдельно. Метод запуска см. в описании проекта.

```
$ python ipproxytool.py 
```

Добавлен асинхронный метод проверки, метод запуска выглядит следующим образом.

```
$ python ipproxytool.py async
```
<br>

## инструкция по проекту
#### Сканировать прокси-сайт
Весь код для сканирования прокси-сайтов находится в [proxy](https://github.com/awolfly9/IPProxyTool/tree/master/ipproxytool/spiders/proxy)<br/>
##### Разверните, чтобы сканировать другие прокси-сайты
1.Создайте новый скрипт в каталоге прокси и наследуйте его от BaseSpider <br/>
2.настраивать name、urls、headers<br/>
3.Переопределить метод parse_page для извлечения данных прокси.<br/>
4.Хранить данные в базе данных.Подробнее см. [ip181](https://github.com/awolfly9/IPProxyTool/blob/master/ipproxytool/spiders/proxy/ip181.py)                 [kuaidaili](https://github.com/awolfly9/IPProxyTool/blob/master/ipproxytool/spiders/proxy/kuaidaili.py)<br/>
5.如果需要抓取特别复杂的代理网站，可以参考[peuland](https://github.com/awolfly9/IPProxyTool/blob/master/ipproxytool/spiders/proxy/peuland.py)<br/>

##### 修改 run_crawl_proxy.py 导入抓取库，添加到抓取队列

可以单独运行 run_crawl_proxy.py 脚本开始抓取代理网站

```
$ python run_crawl_proxy.py
```

#### 验证代理 ip 是否有效
目前验证方式：<br>
1.从上一步抓取并存储的数据库中取出所有的代理 IP <br>
2.利用取出的代理 IP 去请求 [httpbin](http://httpbin.org/get?show_env=1)<br>
3.根据请求结果判断出代理 IP 的有效性，是否支持 HTTPS 以及匿名度，并存储到表 httpbin 中<br>
4.从 httpbin 表中取出代理去访问目标网站，例如 [豆瓣](https://www.douban.com/)<br>
5.如果请求在合适的时间返回成功的数据，则认为这个代理 IP 有效。并且存入相应的表中<br>

一个目标网站对应一个脚本，所有验证代理 ip 的代码都在 [validator](https://github.com/awolfly9/IPProxyTool/tree/master/ipproxytool/spiders/validator)
##### 扩展验证其他网站
1.在 validator 目录下新建脚本并继承 Validator <br>
2.设置 name、timeout、urls、headers <br>
3.然后调用 init 方法,可以参考 [baidu](https://github.com/awolfly9/IPProxyTool/blob/master/ipproxytool/spiders/validator/baidu.py) [douban](https://github.com/awolfly9/IPProxyTool/blob/master/ipproxytool/spiders/validator/douban.py)<br>
4.如果需要特别复杂的验证方式，可以参考 [assetstore](https://github.com/awolfly9/IPProxyTool/blob/master/ipproxytool/spiders/validator/assetstore.py)<br>
##### 修改 run_validator.py 导入验证库，添加到验证队列
可以单独运行 run_validator.py 开始验证代理ip的有效性

```
$ python run_validator.py
```

### 获取代理 ip 数据服务器接口
在 config.py 中修改启动服务器端口配置 data_port，默认为 8000
启动服务器

```
$ python run_server.py
```

服务器提供接口
#### 获取
<http://127.0.0.1:8000/select?name=httpbin&anonymity=1&https=yes&order=id&sort=desc&count=100>

参数

| Name    | Type   | Description   | must |
| ----    | ----   | ----          | ---- |
| name    | str    | 数据库名称      | 是   |
| anonymity | int  | 1:高匿 2:匿名 3:透明 | 否 |
| https     | str  | https:yes http:no  | 否 |
| order     | str  | table 字段  | 否 |
| sort      | str | asc 升序，desc 降序 | 否 |
| count | int | 获取代理数量，默认 100 | 否 |




#### 删除
<http://127.0.0.1:8000/delete?name=httpbin&ip=27.197.144.181>

参数

| Name | Type | Description | 是否必须|
| ----| ---- | ---- | --- |
| name | str | 数据库名称 |  是 |
| ip | str | 需要删除的 ip | 是 |

#### 插入
<http://127.0.0.1:8000/insert?name=httpbin&ip=555.22.22.55&port=335&country=%E4%B8%AD%E5%9B%BD&anonymity=1&https=yes&speed=5&source=100>

参数

| Name | Type | Description | 是否必须|
| ----| ---- | ---- | ----|
| name | str | 数据库名称 |是 |
| ip | str | ip 地址 | 是|
| port | str | 端口 |是|
| country | str | 国家 |否|
| anonymity | int | 1:高匿,2:匿名,3:透明  |否|
| https | str | yes:https,no:http |否|
| speed | float | 访问速度 |否|
| source | str | ip 来源 |否|


## TODO
* 添加多数据库支持
  * mysql
  * redis TODO...
  * sqite TODO...
* 添加抓取更多免费代理网站，目前支持的抓取的免费代理 IP 站点，目前有一些国外的站点连接不稳定
  * (国外) <http://www.freeproxylists.net/>
  * (国外) <http://gatherproxy.com/>
  * (国内) <https://hidemy.name/en/proxy-list/>
  * (国内) <http://www.ip181.com/>
  * (国内) <http://www.kuaidaili.com/>
  * (国外) <https://proxy.peuland.com/proxy_list_by_category.htm>
  * (国外) <https://list.proxylistplus.com/>
  * (国内) <http://m.66ip.cn>
  * (国外) <http://www.us-proxy.org/>
  * (国内) <http://www.xicidaili.com>
* 分布式部署项目
* ~~添加服务器获取接口更多筛选条件~~
* ~~多进程验证代理 IP~~
* ~~添加 https 支持~~
* ~~添加检测 ip 的匿名度~~


## 参考
* [IPProxyPool](https://github.com/qiyeboy/IPProxyPool)


## 项目更新
-----------------------------2017-6-23----------------------------<br>
1.python2 -> python3<br>
2.web.py -> flask<br>
<br>
-----------------------------2017-5-17----------------------------<br>
1.本系统在原来的基础上加入了docker。操作见下方，关于docker的相关知识可以上官网看看http://www.docker.com.<br>
<br>
-----------------------------2017-3-30----------------------------<br>
1.修改完善 readme<br>
2.数据插入支持事务<br>
<br>
-----------------------------2017-3-14----------------------------<br>
1.更改服务器接口，添加排序方式<br>
2.添加多进程方式验证代理 ip 的有效性<br>
<br>
-----------------------------2017-2-20----------------------------<br>
1.添加服务器获取接口更多筛选条件<br>
<br>

-----------------------------2017-2-16----------------------------<br>
1.验证代理 IP 的匿名度<br>
2.验证代理 IP HTTPS 支持<br>
3.添加 httpbin 验证并发数设置，默认为 4












## 在系统中安装docker就可以使用本程序：

下载本程序
```
git clone https://github.com/awolfly9/IPProxyTool
```

然后进入目录：
```
cd IPProxyTool
```

创建镜像：
```
docker build -t proxy .
```

运行容器：
```
docker run -it proxy
```

## 在config.py中按照自己的需求修改配置信息
```
database_config = {
    'host': 'localhost',
    'port': 3306,
    'user': 'root',
    'password': 'root',
    'charset': 'utf8',
}
```
