---
layout: post
title: Python+Requests接口测试框架
comments: true
toc: true
date: 2018-02-01 19:35:29
updated: 2018-02-01 19:35:29
categories:
tags:
---

#Util/util.py
# coding=utf-8
import md5 as _md5
from data_file_parser import *
# 对密码进行md5加密
def md5(password):
    m = _md5.new()
    m.update(password)
    return m.hexdigest()
# 从配置文件env.conf中读取环境信息，返回字典
def get_env(section):
    return ConfFile.load("../Conf/env.conf",section)
# 从配置文件env.conf中读取base_url信息，返回字符串
def get_env_url(section):
    return ConfFile.get("../Conf/env.conf",section,"base_url")
# 从配置文件user.conf读取user信息，返回字典
def get_user(section):
    return ConfFile.load("../Conf/user.conf",section)
def get_api(file_name):
    if '.json' not in file_name:
        file_name += ".json"
    return JsonFile.load("../API/"+file_name)
if __name__ == '__main__':
        print get_api("login.json")

#Util/data_file_parser.py
import json
import ConfigParser
import xlrd
class JsonFile:
    def __init__(self):
        pass
    @classmethod
    def get(cls, path, key):
        with open(path) as f:
            return json.load(f)[key]
    @classmethod
    def load(cls, path):
        with open(path) as f:
            return json.load(f)
class ConfFile:
    def __init__(self):
        pass
    @classmethod
    def get(cls, path, section, option):
        conf = ConfigParser.ConfigParser()
        conf.read(path)
        return conf.get(section, option)
    @classmethod
    def load(cls, path):
        _dict = {}
        conf = ConfigParser.ConfigParser()
        conf.read(path)
        for section in conf.sections():
            _dict[section] = {}
            for option in conf.options(section):
                _dict[section][option] = conf.get(section, option)
        return _dict
    @classmethod
    def load(cls, path, section):
        _dict = {}
        conf = ConfigParser.ConfigParser()
        conf.read(path)
        for option in conf.options(section):
            _dict[option] = conf.get(section, option)
        return _dict
class ExcelFile:
    def __init__(self):
        pass
    @classmethod
    def get(cls, path, sheet, row, col):
        wb = xlrd.open_workbook(path)
        if isinstance(sheet, int):
            sh = wb.sheet_by_index(sheet)
        else:
            sh = wb.sheet_by_name(sheet)
        return sh.cell_value(row, col)
    @classmethod
    def load(cls, path, sheet=0):
        wb = xlrd.open_workbook(path)
        if isinstance(sheet, int):
            sh = wb.sheet_by_index(sheet)
        else:
            sh = wb.sheet_by_name(sheet)
        cols = sh.ncols
        rows = sh.nrows
        data_list = []
        for row in range(1, rows):
            data = {}
            for col in range(0, cols):
                data[sh.cell_value(0, col)] = sh.cell_value(row, col)
            data_list.append(data)
        return data_list
class XMLFile:
    def __init__(self):
        pass
    @classmethod
    def get(cls):
        pass
if __name__ == '__main__':
    a=A
    a.test()

# Util/db.py

#Util/models.py
# coding=utf-8
from util import *
import json
import requests
import sys
reload(sys)
sys.setdefaultencoding("utf8")
class User:
    '''用户对象，初始化时，支持：
    0参：读取user.conf中的default选项下的用户；
    1参：读取user.conf中的参数对应的选项下的用户
    3参：按group_id,user_name,password,顺序读取参数
    字典类型参数：按{"group_id":"***","user_name":"***","password":"***"}
    注：配置文件中或所传password值使用原密码即可，不需要进行md5处理
    '''
    session = requests.Session()
    headers = {'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'}
    base_url = get_env_url("default")
    def __init__(self, *args, **kws):
        self.user_dict = {}
        # 0参，读取配置文件user.conf中的default用户
        if not args or kws:
            self.user_dict = get_user('default')
        elif args:
            # 1参：读取user.conf中的参数对应的选项下的用户
            if len(args) == 1:
                self.req = get_user(args[0])
            # 3参：按group_id,user_name,password,顺序读取参数
            elif len(args) == 3:
                self.user_dict['group_id'] = args[0]
                self.user_dict['user_name'] = args[1].decode('utf-8')
                self.user_dict['password'] = args[2]
            else:
                print 'args error'
        # 字典类型参数：按{"group_id":"***","user_name":"***","password":"***"}
        elif kws:
            self.user_dict = kws
        # 对password进行md5处理
        password = self.user_dict.get('password')
        if password:
            self.user_dict['password'] = md5(password)
    def login(self, *args):
        if not args:
            base_url = self.base_url
        elif isinstance(args[0], Env):
            base_url = args[0].base_url
        else:
            print "args error"
            return -1
        uri = base_url + "/api/Login/Login/login/"
        data = 'req=' + json.dumps(self.user_dict)
        res = self.session.post(uri, data=data, headers=self.headers)
        # print res.text
    def post(self, *args):
        '''发送请求方法,不包含login方法，要在login后使用
        1参：APIFile，使用默认环境,json格式，暂时不支持兼容.config格式的数据
        2参：env对象和APIFile
        2参，base_url和APIFile，base_url如：http://w-beta-1000.chemanman.com:7501
        2参：APIFile和DataFile
        3参：env对象APIFile,DataFile
        注：APIFile还有接口地址和所有必要字段，DataFile，含有多组数据，可以只包含部分变化字段
        '''
        if len(args) == 1:
            base_url = self.base_url
            api_dict = get_api(args[0])
        elif len(args) == 2:
            if isinstance(args[0], Env):
                base_url = args[0].base_url
                api_dict = get_api(args[1])
            elif "http" in args[0]:
                base_url = args[0]
                api_dict = get_api(args[1])
            else:
                base_url = self.base_url
                api_dict = get_api[args[0]]
        else:
            print "args error"
            return -1
        uri = base_url + api_dict.get("uri")
        data = 'req=' + json.dumps(api_dict.get("req"))
        # 尚未对data_file进行替换处理
        res = self.session.post(uri, data=data, headers=self.headers)
        print res.text
    def posts(self, full_url, data):
        '''根据接口地址和数据发送post请求
        full_url:如http://w-beta-1000.chemanman.com:7501/api/Login/Login/login/
        data：支持req={},"req":{}或{}，3种格式
        '''
        if isinstance(data, dict):
            if "req" in data.keys():
                data = "req=" + json.dumps(data.get("req"))
            else:
                data = "req=" + json.dumps(data)
        res = self.session.post(full_url, data=data, headers=self.headers)
        print res.text
    def post_all(self, path='.'):
        '''发送API文件或子文件夹下的所有json文件'''
        pass
    def list(self):
        pass
    def clean(self):
        pass
    def pre_data(self):
        pass
    def system_set(self):
        pass
    def retry(self):
        pass
class Env:
    '''环境对象，支持：
    0参：登录env.conf中的default环境
    1参：env.conf中的选项,不允许包含http
    1参：完整url,带端口号，如"http://w-beta-1000.chemanman.com:7501",结尾不包含/,必须包含http
    2参：host,port,如beta-1000,7501,如果只写beta,默认为beta-1000
    字典：{"env_name":hzc_7501,
    "base_url":"http://w-beta-1000.chemanman.com:7501";,
    "db":"tms_beta_rw",
    "index":"tms_beta_test"
    }
    '''
    def __init__(self, *args, **kws):  # 改为可变参数
        env_name = ''
        base_url = ''
        db = ''
        index = ''
        # 0参：登录env.conf中的default环境
        if not args:
            _env = get_env("default")
            self.base_url = _env.get("base_url")
            self.env_name = _env.get("env_name")
            self.db = _env.get("db")
            self.index = _env.get("index")
            # self.env_name
        elif len(args) == 1:
            if "http" in args[0]:  # 参数为完整url
                self.base_url = args[0]
            else:   # 应加上try
                _env = get_env(args[0])
                self.base_url = _env.get("base_url")
                self.env_name = _env.get("env_name")
                self.db = _env.get("db")
                self.index = _env.get("index")
        elif len(args) == 2:   # 2参情况
            if '-' in args[0]:
                self.base_url = "http://w-"; + \
                    args[0]+".chemanman.com:" + args[1]
            else:
                self.base_url = "http://w-"; + \
                    args[0]+"-1000.chemanman.com:" + args[1]
    def check(self):
        pass
    def deploy(self):
        pass
    def destroy(self):
        pass
    def list(self):
        pass
class DB:
    db_host = ''
    db_port = ''
    db_name = ''
    db_user = ''
    db_password = ''
    db_index = ''
    tables = []
    def get(self, table, key):
        pass
    def do_sql(self):
        pass
    # def filter(self,table,where):
    #   pass
    def select(self, key, table, where):
        pass
class Case:
    case_name = ''
    def run(self):
        pass
if __name__ == '__main__':
    env1 = Env("beta", "7501")
    user = User()
    user.login(env1)
    user.post('orgOp')
    user.login(env1)
    user.post('orgOp')

# Util/option_parser.py
from optparse import OptionParser
  
parser = OptionParser(usage="usage:%prog [optinos] filepath")  
parser.add_option("-a", "--api",  
                action = "store",  
                type = 'str',  
                dest = "api",  
                default = None,  
                help="Specify annalysis execution time limit"  
                )  
parser.add_option("-u", "--url",  
                action = "store_true",  
                dest = "url",  
                default = False,  
                help = "Specify if the target is an URL"  
                )  
(options, args) = parser.parse_args()  
  
# if options.url:  
#     print(args[0])  
# print options.api
#Conf

# Data

# API

# Case/base.py
# coding=utf-8
import sys
sys.path.append("..")
from Util.models import *
from Util.data_file_parser import *
class Base:
    """用例原型"""
    def __init__(self):
        env = Env("http://w-beta-1000.chemanman.com:7502";)
        user = User()
        user.login(env)
        user.post(env,"priceOpV2")
a = Base()
# Case/create_order.py
from base import Base
from Util.data import JsonFile
import json
# from Util.OptionParser import *
class CreateOrder(Base):
    
    def create_order(self):
        # print options.api
        # file = "../API/" + options.api + ".json"
        self.post('../API/create_order.json')
if __name__ == '__main__':
    a = CreateOrder()
    a.create_order()

