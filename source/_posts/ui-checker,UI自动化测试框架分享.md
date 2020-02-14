---
layout: post
title: uc-checker,UI自动化测试框架分享
comments: true
toc: false
date: 2017-12-21 15:30:05
updated: 2017-12-21 15:30:05
categories: 自动化测试
tags: selenium
permalink url:
---
> 北京麻辣诱惑有限公司 信息技术部 测试组 韩志超

## UI自动化测试的意义
1. 验证页面交互的一致性
2. 验证前后端分离项目验证前端数据的正确性
3. 补充覆盖接口测试无法覆盖的业务场景

## UI自动化测试的策略
1. 验证所有页面是否都能正常显示
2. 验证核心业务流程，如下单，查询订单，开具发票，申购，调拨，盘点等

## 测试框架的本质及实现要点
### 测试框架的本质
1. 对重复的操作进行封装
2. 有效的组织，配置、用例、数据、报告等

### 测试框架的实现要点
1. 稳定，健壮性
2. 易用，可配置，兼容性
5. 运行控制，灵活性，效率
6. 方便debug，日志，截图，报告及邮件
7. 参数化

### UI自动化测试基础框架对比
> 由于QTP只支持IE和Firefox,且为商业工具，方案选择时选择了selenium + python
> 基础框架对比基于python单元测试框架:unittest,nose,pytest 及 robot framework(RF)

|对比项|unittest|nose|pytest|RF|
|--|--|--|--|--|
|执行器|无|nosetest|pytest/py.test|pybot|
|discover|支持<br>用例类需要继承unittest.TestCase|支持<br>模块/类/用例以test/Test开头|支持<br>同nose|支持| 
|skip/make fail|@unittest.skip()<br>@unittest.skipIf()<br>...|from nose.plugins.skip import SkipTest<br>raise SkipTest|@pytest.mark.skipif()<br>@pytest.mark.xfail|无|
|fixture|setUp/tearDown<br>setUpClass/tearDownClass<br>setUpMoudle/tearDownMoudle|同unittest(继承unittest.TestCase类)<br>使用装饰器）|@pytest.fixture(session="session"autouser=True)<br>作用域支持function\module\session<br>autouser=Ture默认执行|[SetUp] ... <br>[Teardown]... 支持project,suite,case作用域
|参数化|无|无|@pytest.mark.parametrize("a,b,expected", testdata)|[Template] 1 2 3|
|tags|无|attrib标签<br>from nose.plugins.attrib import attr<br>@attr(type='smoke')<br>def test_demo1():<br>...<br>nostests -a type=smoke|@pytest.mark.smoke<br>pytest -m smoke|[Tags] smoke<br>pybot -i/-e smoke|
|timeout/time limit|无|from nose.tools import timed<br>@timed(30)<br>def test_demo()...|pip install pytest-timeout<br>@pytest.mark.timeout(30)或pytest --timeout=30|[Timeout]  30 seconds|
|list case|无|nosetests --collect-only|pytest -v --collect-only|未知|
|rerun fail case|无|nosetests --failed|pip install pytest-rerunfailures<br>@pytest.mark.flaky(rerun=1)或pytest --rerun=1|robot --rerunfailed|
|log|无|支持，运行参数|支持，运行参数|支持，自动生成log和出错截图|
|xml报告/Jenkins集成|无|nosetests --with-xunit --xunit-file=支持allure报告<br>pip install pytest-nose-adaptor|pytest --junit-xml=<br>支持allure报告<br>pip install pytest-allure-adaptor|支持，自动生成|
|html报告|三方HtmlTestRunner(python2.0)|pip install nose-htmloutput<br>nosetests --with-html --html-file=|pip install pytest-html<br>pytest --html=|支持，自动生成|
|baseline对比|无|无|无|无|
|发送email|无|无|无|无|无|
|并发/多线程|无|无|无|无|
|Selenium支持|原生库|原生库|pytest-selenium|robotframework-seleniumlibrary<br>robotframework-selenium2library|

** 选择unittest,是因为其为python的核心库，提供了用例组织的基础功能，二次开发的灵活性更好 **

## 框架结构及要点实现原理
> 基于 python2/3 + unittest + selenium
> 需要安装Chrome

### 框架结构
```
-ui_checker                                         ---项目目录
    -conf                                           ---配置文件目录
        default.conf
    -data                                           ---数据文件目录
        test_customer_ccustomer_index.ini           ---数据文件
    -driver                                         ---浏览器驱动目录
    -page_obj                                       ---页面对象目录
        base_page.py                                ---页面对象原型
        -customer                                   ---模块
            -CCustomer                              ---子模块
                index.py                            ---页面对象
                index.property                      ---页面元素
    -report                                         ---运行报告目录
        -log                                        ---报告目录
        -snapshot                                   ---截图目录
        customer_2017-11-29_162338_result.html      ---测试报告
    -test_case                                      ---测试用例目录
        base_case.py                                ---用例原型
        -customer
            test_ccustomer_index.py                 ---测试用例
    -tools                                          ---其他工具目录
    -util                                           ---公共方法目录
        browser.py                                  ---封装浏览器对象，支持Chrome Headless和Grid分布式
        config.py                                   ---解析default.conf配置
        data_file_parser.py                         ---解析各种数据文件
        db.py                                       ---数据库操作封装
        decorator.py                                ---运行控制装饰器
        HTMLTestRunnerCN.py                         ---生成报告，美化页面，支持python2/3
        log.py                                      ---log配置
        option_parser.py                            ---run_all_test 运行参数解析
        root.py                                     ---解析项目目录，用于拼装绝对路径
        selenium_easy.py                            ---基于xpath,增加元素定位及操作方法
    run_all_test.py                                 ---执行入口文件
```
### 要点实现原理
1. 健壮性---采用PageObject模式，页面对象与元素分离，元素写到单独的文件中
2. 易用---封装页面对象原型和用例原型，通过继承简化操作，selenium_easy模式
3. 效率---Chrome Headless模式，按模块多线程并发
4. 参数化---通过python装饰器实现，数据文件采用config文件，支持多组数据
装饰器
```
def data(data_file, section='default'):    # 使用该装饰器的方法必须具有data参数
    data_path = '../data/'
    _data_file = data_path + data_file + ".config"
    data = ConfFile.load_section(_data_file, section)
    def _data(func):
        @wraps(func)
        def wrapper(*_args, **_kwargs):
            _kwargs['data']=data
            return func(*_args, **_kwargs)
        return wrapper
    return _data
```
数据文件 test_customer_ccustomer_index.ini
```
[customer_not_exists]
phone=18010181268
```
用例
```
@data('test_customer_ccustomer_index','customer_not_exists')
def test_search_not_exist_customer(self,data):
    self.search_phone(data['phone'])
```
5. 方便debug---log及通过装饰器，显示项selenium操作时间
6. 用例执行
6.1. 用例level(tags)
装饰器
```
def level(level):
    test_level = Config.option('runtime','test_level')   # 通过配置文件中的test_level控制
    if test_level == 0 or test_level == level:
        return lambda func: func
    return unittest.skip("skip this level cases")
```
配置文件 default.con
```
[runtime]
test_level=1  # 默认test_level=0 ，执行所有用例
```
用例
```
@level(1)
@data('test_customer_ccustomer_index','customer_not_exists')
def test_search_not_exist_customer(self,data):
    self.search_phone(data['phone'])
```
6.2. timeout控制---装饰器(略)
6.3. 命令行参数解析---option_parser
7. 数据对比---在页面配置文件中配好各元素的db_map，支持db_compare_all
8. 环境清理---需要自行调用db.py中的exec_sql方法，通过执行sql清理环境

## 主要特性
1. 采用PageObject模式封装每个操作页面，支持页面与元素分离
2. 支持selenium_easy模式，按人类习惯，通过链接/按钮文字，Form前置标签识别/操作控件，支持多个相同标签控件的识别
3. 支持Json/XML/CSV/Excel/Config类型的数据文件解析，支持将数据文件完整解析成dict格式，或从中获取某个值
4. 支持数据连接及对比数据库的值
5. 支持Chrome Headless无界面模式运行，支持Grid分布式运行或Remote模式
6. 支持按模块并发执行用例，每个模块生成各自的报告
7. 支持邮件/截图/log
8. 支持显示webdriver各项操作消耗时间，支持time_out设置，支持多线程重复某一操作， 支持制定test case level
9. 支持 Windows/Linux + python2.*/python3.* 

## 使用方法
### 运行环境
>推荐 Windows 7/10 + Python 3.* + selenium3.5 + Chrome 62以上
> 或 CentOS 7.* + Python2.7 + selenium3.5 + google-chrome-stable最新版

以Windows为例：

1. 下载安装Chrome最新版
2. 下载安装Git Windows客户端
3. 下载安装python3.6.3（自动安装pip和配置好环境变量）
4. 安装python第三方库，打开cmd
```
pip install selenium
pip install pymysql
pip install xlrd
```

### 使用项目
1. 克隆项目，打开cmd
```
git@192.168.100.240:hanzhichao/ui_checker.git
cd ui_checker
```
2. 配置项目
```
打开conf/default.conf
修改[env]下base_url及登录用户名和密码,保存
```
3. 执行所有用例
```
python run_all_test.py
```

### 编写用例
1. 完善page_obj对象
```
1.1 根据页面菜单url确定所在模块，如"综合管理系统>部门管理>新增部门" 页面链接为.../admin/ADepartment/create
1.2 打开page_obj/目录下对应的模块下对应的py文件，如page_obj/admin/ADepartment/create.py
1.3 完善menu和subject：
    如menu='综合管理系统','部门管理','新增部门'（用于加载页面，注意：中间为英文逗号），
    subject='部门信息'（页面主体，第一个头部主题，用于判断页面是否加载成功）
1.4 封装页面操作，3种方式，参考page_obj/customer/CCustumer/下的：
    index.py(selenium_easy模式)
    index2.py(标准PageObj模式)
    index3.py+index3.property(元素分离模式)
```
2. 编写测试用例
```
进入test_case/相应模块下，如test_case/admin
新建test_adepartment_create.py文件
编写该页面相关的用例，参考test_case/customer/test_ccustomer_index.py
```
3. page_obj示例
```
# !/usr/bin/env python
# -*- coding=utf-8 -*-

from time import sleep

from page_obj.base_page import BasePage
from page_obj.index.index.login import LoginPage
from util.browser import Chrome
from util.db import DB
from util.selenium_easy import Element, Input


class IndexPage(BasePage):
    menu = ('客服管理系统', '综合管理', '综合信息')
    subject = '会员信息'
 
    def search_phone(self, phone):
        self.type('电话搜索：', phone)
        self.click_btn('搜索')
        self.sleep()
        
    def save_work_bill(self):
        self.click("显示更多内容")
        
        self.select("来电原由", "订单")
        self.select("来电原由", "创建订单", 2)
        self.type_area("备注", "自动化测试", 2)
        self.click_btn("保存", 2)
        
    def create_order(self, phone, station, code):
        self.search_phone(phone)
        self.save_work_bill()
        self.click("显示更多内容", 4)
        self.select("配送站点", station)
        self.type("请输入要查询商品的简码", code)
        sleep(1)
        self.click_text("麻酱烧饼")
        self.select("订单渠道：", "电话")
        self.select("订单区域", "北京")
        self.click_input("是否预定送餐时间", 4)
        self.click_btn("当前")
        self.click_btn("确认")
        self.check("支付方式", 1)
        self.select("支付方式", "支付宝")
        self.type_area("备注", "自动化测试下单", 3)
        self.click_btn("保存", 3)
```
4. 测试用例示例
test_case/customer/test_ccustomer_index.py
```
import sys
sys.path.append("..")
from page_obj.customer.CCustomer.index import IndexPage
from test_case.base_case import BaseCase
from util.decorator import data,level

class TestCcustomerIndex(BaseCase):
    @classmethod
    def setUpClass(cls):
        super(TestCcustomerIndex, cls).setUpClass()
        cls.page = IndexPage(cls.driver)
        cls.page.load()
    
    def test_search_exist_customer(self):
        """
        pre-condition: 18010181267 customer exists
        no cleaning need
        """
        phone = '18010181267'
        self.page.search_phone(phone)
        # assert page_obj value and search value
        customer_phone = self.page.get_input_value('会员电话：')
        self.assertEqual(customer_phone, phone)

    @level(1)
    @data(test_customer_ccustomer_index)
    def test_search_not_exist_customer(self):
        """
            pre-condition: 18010181261 customer not exists
            no cleaning need
        """
        phone = '18010181261'
        self.page.search_phone(phone)
        customer_phone = self.page.get_input_value('会员电话：')
        self.assertFalse(customer_phone)

if __name__ == '__main__':  
    unittest.main(verbosity=2)
```

## Future
1. 切换到robot framework, 和收银机UI自动化项目保持一致
2. 绿色化，封装依赖环境，优化可移植性
3. 补充主要业务的测试用例
4. 集成数据库对比
5. 集成到jenkin中