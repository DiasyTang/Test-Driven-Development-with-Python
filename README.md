# Test-Driven-Development-with-Python（测试驱动开发）
## 配置环境（Mac）
### 1.安装Python
#### Mac有自带的Python，不过版本是Python2,如果大家习惯了Python3,可以自己下载最新的Python版本。这个可以自己去官网下载安装，这里就不做介绍了。
### 2.配置Python环境
#### 为了能够使用不同的Python环境，我们这里采用Python的虚拟环境
#### 1.安装virtualenv以及virtualenvwrapper
##### 使用命令（pip3 install virtualenv，pip3 install virtualenvwrapper），安装完成之后，在用户的当前目录下配置.bash_profile,\输入vim .bash_profile，在.bash_profile文件加入以下命令： 
```
export WORKON_HOME=~/env_workspaces #加入自己设定的所有虚拟环境的工作空间
source /usr/local/bin/virtualenvwrapper.sh #添加要激活的文件路径（重点virtualenvwrapper.sh这个的文件路径在你的Python3\安装目录下，这个可以在.bash_profile中查看到）
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3 #这里写的虚拟环境需要的Python版本
```
##### 最后输入 source .bash_profile来使文件成效，到此为止，虚拟环境已经配置完成了。
```
mkvirtualenv test #创建新的虚拟环境
rkvirtualenv test #删除虚拟环境 
workon #查看当前电脑中所有的Python虚拟环境 
workon test #进入test的Python的虚拟环境中，成功进入test的虚拟环境，命令行会有一个（test）的标志。
deactivate #退出当前Python的虚拟环境 
```
##### 2.在虚拟环境中安装django、selenium
```
pip3 install django
pip3 install selenium
```
##### 3.安装Firefox和Geckodriver（Firefox正常安装）
[Geckodriver](https://github.com/mozilla/geckodriver/releases)

下载后将Geckodriver放入Python的PATH中（Python的PATH可以在当前用户目录中的.bash_profile中查看到）
## 第一个项目
开始我们的第一个项目（functional_test.py），来检测我们的环境是否安装OK
```
from selenium import webdriver
browser = webdriver.Firefox() browser.get('http://localhost:8000')
assert 'Django' in browser.title
```
##### 这是我们第一个功能测试，我来详细说一下：
###### 1.开始了一个Selenium "webdriver"来推出一个真正的Firefox浏览器窗口
###### 2.使用它来打开我们期望的从本地PC提供的网页
###### 3.检查（使用断言）页面标题是否有“Django”
###### 4.python functional_tests.py
###### 我们应该会看到一个浏览器窗口被推出来，然后打开localhost:8000，最后你会看到一个“无法连接”的错误页面。在后面我们再修复
## 开始Django的运行
在这里，我们已经安装了Django，第一步是创建一个项目，这是我们站点的主要内容。Django为此提供了一个命令行工具：
$ django-admin.py startproject superlists
```
（这是文件结构）
├── functional_tests.py 
├── geckodriver.log 
└── superlists    
    ├── manage.py    
    └── superlists        
    ├── __init__.py        
    ├── settings.py        
    ├── urls.py       
    └── wsgi.py 
```
$cd superlists
$python manage.py runserver
Django的开发服务将会被开启并运行在我们的机器上。这个时候我们可以打开另一个命令行，再运行我们的第一个项目
$python functional_tests.py

    
   
