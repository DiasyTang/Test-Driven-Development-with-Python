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
##### 注意在VSCode中配置虚拟环境
在settings中进行设置,在配置搜索中输入python.venv，会看到python.venvFolders和python.venvPath（虚拟环境保存目录）
```
 "python.venvPath": "~/env_workspaces",
    "python.venvFolders": [
        "envs",
        ".pyenv",
        ".direnv",
        "test" //这是我的虚拟环境的名称
    ]
```
虚拟环境是通过名称识别的,例如在~/env_workspaces目录下test就不被".env"识别到.在这之后重启vscode，再通过命令面板Python: Select Interpreter来重新选择解释器。
```
"configurations": [
        {
            "name": "Python Experimental: Current File (Integrated Terminal)",
            "type": "pythonExperimental",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "env": {},
            "envFile": "${workspaceRoot}/test",
        }
 ```
 在launch.json中修改配置，来让python调试时，能够根据你需要的虚拟环境来进行调试。
## 第一个项目
开始我们的第一个项目（functional_test.py），来检测我们的环境是否安装OK
```
from selenium import webdriver
browser = webdriver.Firefox() 
browser.get('http://localhost:8000')
assert 'Django' in browser.title
```
##### 这是我们第一个功能测试，我来详细说一下：
###### 1.开始了一个Selenium "webdriver"来推出一个真正的Firefox浏览器窗口
###### 2.使用它来打开我们期望的从本地PC提供的网页
###### 3.检查（使用断言）页面标题是否有“Django”
###### 4.python functional_test.py
###### 我们应该会看到一个浏览器窗口被推出来，然后打开localhost:8000，最后你会看到一个“无法连接”的错误页面。在后面我们再修复
## 开始Django的运行
在这里，我们已经安装了Django，第一步是创建一个项目，这是我们站点的主要内容。Django为此提供了一个命令行工具：
$ django-admin.py startproject superlists
```
（这是文件结构）
├── functional_test.py 
├── geckodriver.log 
└── superlists    
    ├── manage.py    
    └── superlists        
    ├── __init__.py        
    ├── settings.py        
    ├── urls.py       
    └── wsgi.py 
```
```
$cd superlists 
$python manage.py runserver 
```
Django的开发服务将会被开启并运行在我们的机器上。这个时候我们可以打开另一个命令行，再运行我们的第一个项目
```
$python functional_test.py
```
## 使用unittest模块来扩展我们的功能测试
功能测试又被称为对验收测试或端到端测试。这些测试着眼于整个应用程序功能，但不了解被测系统的内部情况，另一个术语是黑盒测试。
我们来调整一下测试，该测试只是检查了默认的Django“有效”的页面，并检查了一些我们想在真实页面中看到的东西。这次我们来构建一个待办事项列表网站。
##### 我们来调整一下functional_test.py
```
from selenium import webdriver

browser=webdriver.Firefox()
browser.get("http://localhost:8000")
assert "To-Do" in browser.title
browser.quit() //浏览器关闭
```
请注意一下，我们修改了断言以查找“To-Do”而不是“Django”。这意味着现在的测试是失败的。我们来运行一下：
```
$python manage.py runserver（运行服务器）
$python functional_test.py
```
如预期一样我们测试失败了，这实际上是一个好消息，虽然不如测试通过，但至少是因为正确的原因而失败。这是一个好的开始。
### python标准库的单元测试模块
我们应该可以处理一些小的麻烦了。首先“AssertionError”不是很有帮助。如果测试告诉我们它实际找到的浏览器标题是什么，这就很好了。此外，火狐窗口依然停留在桌面上，如果可以被自动清理就比较好了。一种选择是对assert关键字使用第二个参数，如：
```
assert "To-Do" in browser.title, "Browser title was " + browser.title
```
我们还可以通过try/finally来清理旧的Firefox窗口。但是这些问题在测试中是很常见的，在python标准库的单元测试模块中有一些已经准备好的解决方案，让我们来试试。在functional_test.py：
```
from selenium import webdriver
import unittest


class NewVisitorTest(unittest.TestCase):
    def setUp(self):
        self.browser = webdriver.Firefox()

    def tearDown(self):
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        self.browser.get("http://localhost:8000")
        self.assertIn("To-Do", self.browser.title)
        self.fail("Finish the test!")


if __name__ == "__main__":
    unittest.main(warnings="ignore")
```
这段代码可能有几个不同点：</br>
1、测试被组织成了类，这些类继承自unittest.TestCase。</br>
2、测试的主体是在被称为test_can_start_a_list_and_retrieve_it_later的方法中。以test开头的方法是测试方法，并通过测试运行程序运行。你可以在一个类中有多个测试方法。</br>
3、setUp和tearDown是运行在测试之前和之后的特定方法。使用它们来开启和退出浏览器——有点像try/except，tearDown方法将会被运行即使在测试中出现了错误。将不会有多余的FireFox窗口停留在桌面上。</br>
4、我们使用self.assertIn来代替assert来进行测试断言。单元测试提供了许多像这样的帮助方法来进行断言测试，像assertEqual,assertTrue,assertFalse等等。你可以在单元测试文档中找到更多。</br>
5、无论如何，self.fail都会失败，并产生给定的错误消息。我是用它来提醒你完成测试。</br>
6、最后，我们有if __name__=="__main__" 子句（如果你之前没有见过，这就是python脚本检查是否是从命令行执行的，而不是通过其它脚本导入的）。我们调用unittest.main()，它会启动单元测试运行程序，这个单元测试程序会自动去寻找在文件中的测试类和测试方法然后运行它们。</br>
7、warning='ignore'会在编写时禁止发出多余的ResourceWarning。当你阅读时可能已经消失了；请尝试删除它！</br>
现在让我们来试一试！
```
$python functional_test.py
```
输出结果
```
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "d:\Test-Driven-Development-with-Python\functional_test.py", line 14, in test_can_start_a_list_and_retrieve_it_later
    self.assertIn("To-Do", self.browser.title)
AssertionError: 'To-Do' not found in 'Django: the Web framework for perfectionists with deadlines.'       

----------------------------------------------------------------------
Ran 1 test in 14.989s

FAILED (failures=1)
```
这样不是更好吗！它整理了Firefox窗口，提供给我们一个友好格式的报告，其中显示了运行了多少个测试，哪些失败了，assertIn给了我们更有帮助的错误信息以及有用的调试信息。
## 使用单元测试测试简单的页面
在上一章中，功能测试失败，告诉我们它希望站点的首页标题带有“To-Do”。现在我们开始创建我们的应用程序了
### 我们的第一个Django App，以及第一个单元测试
我们开始to-do lists的应用：
```
$ python manage.py startapp lists
```
文件结构变成了这样：
```
superlists/
├── db.sqlite3
├── functional_tests.py
├── lists
│ ├── admin.py
│ ├── apps.py
│ ├── __init__.py
│ ├── migrations
│ │ └── __init__.py
│ ├── models.py
│ ├── tests.py
│ └── views.py
├── manage.py
└── superlists
 ├── __init__.py
 ├── __pycache__
 ├── settings.py
 ├── urls.py
 └── wsgi.py
 ```
 ### 单元测试，以及它们与功能测试的区别
 单元测试和功能测试之间的界限有时候变成非常的模糊。最基本的分别就是功能测试是从用户的角度从外部来测试应用。单元测试是从程序员的角度从内部来测试应用。然而TDD方法就是希望我们能够覆盖这两种测试类型。工作流如下：<br>
 1、我们从写功能测试开始，从用户的角度描述新的功能。<br>
 2、一旦我们的功能测试失败了，我们就开始考虑怎么编写可以通过测试的代码。我们现在使用一个或者多个单元测试来定义我们希望代码的行为方式，这个想法是，你们编写的每一行生产代码都应该通过至少一个单元测试进行测试。<br>
 3、单元测试失败后，我们将编写尽可能少的应用程序代码，来让单元测试通过。我们可能会在第2步和第3步之间重复几次，直到我们认为功能测试可以更进一步。<br>
 4、现在我们重新运行功能测试，看看它们是否通过，或者进一步测试。这可能促使我们编写一些新的单元测试和一些新的代码，等等。<br>
 你可以看到，从头到尾，功能测试从高层次驱使着我们的工作发展，而单元测试则从低层次驱动着我们的工作。<br>
 这看起来有点多余？有时会这种感觉，但是功能测试和单元测试有着完全不同的目标，并且通常最终看起来完全不同。<br>
 注意：功能测试应该帮助你正确地建立应用程序功能，并确保你不会意外破坏它。单元测试应该帮助你来编写干净无错误的代码。<br>
 到目前为止已经有足够多的理论了，接下来我们看看怎么实践了。
 ### 在Django中的单元测试
 对于主页我们怎么来编写单元测试。我们来写一个小的单元测试，在lists/tests.py中
  ```
from django.test import TestCase

# Create your tests here.
class SmokeTest(TestCase):
    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
 ```
 执行命令：
 ```
 $python manage.py test
 ```
 运行结果：
 ```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_bad_maths (lists.tests.SmokeTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\Test-Driven-Development-with-Python\superlists\lists\tests.py", line 6, in test_bad_maths
    self.assertEqual(1 + 1, 3)
AssertionError: 2 != 3

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```
### Django的MVC，URL和查看功能
Django是按照经典的MVC模式来构建的。它肯定有模型，但是它的视图更像是一个控制器，而模板实际上是视图的一部分。无论采用哪种方式，就像使用任何web服务器一样，Django的主要工作都是决定当用户在我们网站上请求特定的URL时该怎么做。Django的工作流程如下：<br>
1、针对特定地址的HTTP请求<br>
2、Django使用一些规则来决定哪一个视图函数处理该请求（这是解析地址）<br>
3、视图函数处理请求并返回一个HTTP的响应<br>
所以我们将测试两件事：我们能不能将站点的根目录的URL解析成我们已经完成了的特定的视图函数和我们能不能让视图函数返回一些HTML来通过功能测试<br>
我们现在开始，在lists/tests.py中
```
from django.test import TestCase
from lists.views import home_page //这是我们接下来要编写的veiw函数，它实际上将返回我们想要的HTML。我们计划将它你别想在lists/views.py中。
from django.urls import resolve

# Create your tests here.
class HomePageTest(TestCase):
    def test_root_url_resolves_to_home_page_view(self):
        found = resolve("/") //resolve是Django内部用来解析URL并查找它们应映射到的视图函数。当调用“/”（网站的根目录）时，会找到名为home_page的函数。
        self.assertEqual(found.func, home_page)
```
 执行命令：
 ```
 $python manage.py test
 ```
 运行结果：
  ```
 System check identified no issues (0 silenced).
E
======================================================================
ERROR: lists.tests (unittest.loader._FailedTest)
----------------------------------------------------------------------
ImportError: Failed to import test module: lists.tests
Traceback (most recent call last):
  File "c:\users\tina\appdata\local\programs\python\python37\Lib\unittest\loader.py", line 436, in _find_test_path
    module = self._get_module_from_name(name)
  File "c:\users\tina\appdata\local\programs\python\python37\Lib\unittest\loader.py", line 377, in _get_module_from_name
    __import__(name)
  File "D:\Test-Driven-Development-with-Python\superlists\lists\tests.py", line 2, in <module>
    from lists.views import home_page
ImportError: cannot import name 'home_page' from 'lists.views' (D:\Test-Driven-Development-with-Python\superlists\lists\views.py)


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
 ```
 因为我们没有编写home_page，报了我们预期的错误。
 ### 最后我们编写一些应用程序代码
 之前我们的错误是无法从lists.views中导入home_page，现在我们来修复它，在lists/views.py中：
  ```
from django.shortcuts import render

# Create your views here.
home_page = None
 ```
  执行命令：
 ```
 $python manage.py test
 ```
 运行结果：
  ```
  Creating test database for alias 'default'...
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest) //
----------------------------------------------------------------------
Traceback (most recent call last):
  File "D:\Test-Driven-Development-with-Python\superlists\lists\tests.py", line 8, in test_root_url_resolves_to_home_page_view
    found = resolve("/")
  File "C:\Users\tina\Envs\venv\lib\site-packages\django\urls\base.py", line 24, in resolve
    return get_resolver(urlconf).resolve(path)
  File "C:\Users\tina\Envs\venv\lib\site-packages\django\urls\resolvers.py", line 567, in resolve
    raise Resolver404({'tried': tried, 'path': new_path})
django.urls.exceptions.Resolver404: {'tried': [[<URLResolver <URLPattern list> (admin:admin) 'admin/'>]], 'path': ''}

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Destroying test database for alias 'default'...
  ```
 测试结果告诉我们需要一个URL映射。Django使用urls.py的文件来将URL映射到视图函数。在superlists/superlists文件夹中有一个对于整个站点中主要的urls.py，在superlists/urls.py中，我们来看看：
```
"""superlists URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/2.2/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.urls import include, path
    2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
"""
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]
```
网址条目以正则表达式开始，该正则表达式定义了该网址适用于哪些 网址，并继续说明了这些请求发送到哪里，要么是你提供的视图函数，要么是其它地方的另一个urls.py文件。<br>
我们将移除admin的URL，因为我们到目前为止将不会使用Django admin站点，在superlists/urls.py中：
```
from django.contrib import admin
from django.urls import path
from lists import views
from django.conf.urls import url

urlpatterns = [
    url(r"^$", views.home_page, name="home")
]
```
执行命令：
```
 $python manage.py test
```
运行结果：
```
[...]
TypeError: view must be a callable or a list/tuple in the case of include().
```
现在已经不再报404了，bug信息是比较混乱的，但是最后的消息告诉我们发生了什么：单元测试实际上已经让URL“/”与在list/views.py中的home_page=None产生了关联，但是现在在抱怨home_page视图不可调用。回到list/views.py文件：
```
from django.shortcuts import render

# Create your views here.
def home_page():
    pass
```
执行命令：
```
 $python manage.py test
```
运行结果：
```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.002s

OK
Destroying test database for alias 'default'...
```
 ### 单元测试视图
 我们继续为视图编写测试，让它可以对浏览器页面做出真正的响应。打开list/test.py，增加一个新的测试方法，我将解释每一点：
 ```
from django.test import TestCase
from django.urls import resolve
from django.http import HttpRequest

from lists.views import home_page

# Create your tests here.
class HomePageTest(TestCase):
    def test_root_url_resolves_to_home_page_view(self):
        found = resolve("/")
        self.assertEqual(found.func, home_page)

    def test_home_page_returns_correct_html(self):
        request=HttpRequest()(1)
        response=home_page(request)(2)
        html=response.content.decode("utf8")(3)
        self.assertTrue(html.startswith("uft8"))(4)
        self.assertIn("<title>To-Do lists</title>",html)(5)
        self.assertTrue(html.endswith("</html>"))(4)

```
(1)我们创建了一个HttpRequest对象，当用户浏览器请求页面时，Django将会给我们看见这个对象 <br>
(2)我们将这个对象传给home_page视图，它将给我们一个响应。这个对象就是被称为HttpResponse类的实例<br>
(3)然后，我们提取响应的.content。这是一些原始字节，即发送给浏览器的0和1.我们通过.decode()将它们转换为发送给用户的HTML字符串<br>
(4)我们想要以<html>标记开始并以它结束 <br>
(5)我们想要在中间某个地方找到<title>标记，并在其中包含“To-Do lists”，这是因为它是我们功能测试中特写的内容<br>
 再一次提醒，单元测试通过功能测试驱动，但是它又更接近于实际代码，我们现在正像程序员一样思考。
 现在我们来运行这个单元测试，并看看会发生什么
 ```
 ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/tina/test/Test-Driven-Development-with-Python/superlists/lists/tests.py", line 15,in test_home_page_returns_correct_html
    response=home_page(request)
TypeError: home_page() takes 0 positional arguments but 1 was given

----------------------------------------------------------------------
Ran 2 tests in 0.004s

FAILED (errors=1)
Destroying test database for alias 'default'...
```
### 单元测试/代码周期
现在我们可以进入单元测试/代码周期了：
1､在终端，运行单元测试看它们是怎么失败的 <br>
2､在编辑器中，以最小的代码变动来解决当前测试的失败<br>
并且重复这些！<br>
我们对每个代码的变动越小且越少为好，我们的想法是绝对确保每一个代码段都可以通过测试证明是正确的，下面让我们来看看多快可以完成这个周期：
#### 最小代码变动：（lists/views.py）
```
def home_page(request):
    pass
```
#### 测试：
```
 html=response.content.decode("utf8")
AttributeError: 'NoneType' object has no attribute 'content'
```
#### 代码--我们使用django.http.HttpResponse，如下：（lists/views.py） 
```
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return  HttpResponse()
 ```
 #### 再次测试：
 ```
 self.assertTrue(html.startswith("<html>"))
AssertionError: False is not true
 ```
 #### 再次代码：（lists/views.py）
 ```
 def home_page(request):
    return  HttpResponse("<html>")
 ```
  #### 再次测试：
 ```
 self.assertIn("<title>To-Do lists</title>",html)
AssertionError: '<title>To-Do lists</title>' not found in '<html>'
 ```
  #### 再次代码：（lists/views.py）
 ```
 def home_page(request):
       return  HttpResponse("<html><title>To-Do lists</title>")
 ```
 #### 再次测试：
 ```
 self.assertTrue(html.endswith("</html>"))
AssertionError: False is not true
 ```
 #### 最后一次编码：（lists/views.py）
 ```
 def home_page(request):
       return  HttpResponse("<html><title>To-Do lists</title></html>")
 
 结果：
 Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..
----------------------------------------------------------------------
Ran 2 tests in 0.004s

OK
Destroying test database for alias 'default'...
 ```
 现在我们运行功能测试
 ```
 $ python functional_tests.py

     ---------------------------------------------------------------------
    Traceback (most recent call last):
      File "functional_tests.py", line 19, in
    test_can_start_a_list_and_retrieve_it_later
        self.fail('Finish the test!')
    AssertionError: Finish the test!
     ---------------------------------------------------------------------
    Ran 1 test in 1.609s
    FAILED (failures=1)
 ```
 功能测试并没有失败，只不过是一个小小的提示。还不错，我们掌握了：
 1､开启一个Django应用 <br>
 2､Django单元测试运行者 <br>
 3､功能测试与单元测试的区别<br>
 4､Django url解释与urls.py<br>
 5､Django视图方法，请求和响应对象 <br>
 6､返回一个基本的HTML<br>
 #### 一些有用的命令
 1､python manage.py runserver（运行Django服务器)<br>
 2､python functional_tests.py（运行功能测试)<br>
 3､python manage.py test（运行单元测试)<br>
 单元测试/代码周期：
 1､在终端运行单元测试。<br>
 2､在编辑中做最小的变动<br>
 3､重复进行1和2<br>
 ## 这些测试在做什么了（以及重构）
 ### 使用Selenium来测试用户交互
 在上一章结尾处在哪里？让我们重新运行测试并找出：
 ```
 FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "d:\Test-Driven-Development-with-Python\functional_test.py", line 15, in test_can_start_a_list_and_retrieve_it_later
    self.fail("Finish the test!")
AssertionError: Finish the test!

----------------------------------------------------------------------
Ran 1 test in 11.693s

FAILED (failures=1)
 ```
 你可以试试，如果你得到网页加载有问题或者无法链接？那是因为我忘记打开服务器了（pyhton manage.py runserver）。在这之后你将会提到上面的错误信息了。现在我们打开functional_tests.py并扩展这个方法：
 ```
 from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
import unittest


class NewVisitorTest(unittest.TestCase):
    def setUp(self):
        self.browser = webdriver.Firefox()

    def tearDown(self):
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        self.browser.get("http://localhost:8000")

        self.assertIn("To-Do", self.browser.title)
        header_text = self.browser.find_element_by_tag_name("h1").text (1)
        self.assertIn("To-Do", header_text)

        inputbox = self.browser.find_elements_by_id("id_new_item") (1)
        self.assertEqual(inputbox.__getattribute__("placeholder"), "Enter a to-do item")

        inputbox.send_keys("Buy peacock feathers") (2)

        inputbox.send_keys(Keys.ENTER) (3)
        time.sleep(1) (4)

        table = self.browser.find_elements_by_id("id_list_table")
        rows = table.find_element_by_tag_name("tr") (1)
        self.assertTrue(any(row.text == "1:Buy peacock feathers" for row in rows))

        self.fail("Finish the test!")


if __name__ == "__main__":
    unittest.main(warnings="ignore")

```
 (1)我们正在使用Selenium提供的几种方法来检查网页：find_element_by_tag_name,find_element_by_id和find_elements_by_tag_name（请注意：额外的s表示 它将返回多个元素，而不是一个元素）<br>
 (2)我们还使用send_keys，这是打字到输入框的selenium的方法。<br>
 (3)通过keys类（不是忘记导入），我们可以发送特殊键例如Enter键<br>
 (4)当我们点击Enter，这个页面将会重新刷新。这里的time.sleep是确保浏览器已经完成加载了在我们对于新页面进行断言之前。这个称为“显式断言”，我们将会在第六章做改善。<br>
现在我们来看看怎么测试：
```
$ python functional_tests.py
======================================================================
ERROR: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "d:\Test-Driven-Development-with-Python\functional_test.py", line 18, in test_can_start_a_list_and_retrieve_it_later
    header_text = self.browser.find_element_by_tag_name("h1").text
  File "C:\Users\tina\envs\venv\lib\site-packages\selenium\webdriver\remote\webdriver.py", line 530, in find_element_by_tag_name
    return self.find_element(by=By.TAG_NAME, value=name)
  File "C:\Users\tina\envs\venv\lib\site-packages\selenium\webdriver\remote\webdriver.py", line 978, in find_element
    'value': value})['value']
  File "C:\Users\tina\envs\venv\lib\site-packages\selenium\webdriver\remote\webdriver.py", line 321, in execute
    self.error_handler.check_response(response)
  File "C:\Users\tina\envs\venv\lib\site-packages\selenium\webdriver\remote\errorhandler.py", line 242, in check_response
    raise exception_class(message, screen, stacktrace)
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: h1


----------------------------------------------------------------------
Ran 1 test in 7.315s

FAILED (errors=1)
```
对此进行解码，这个测试是说无法在这个页面找到<h1>元素。现在看我们怎么添加这个元素到我们的主页面。
### “不需要测试常量”的规则，救援模板

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
