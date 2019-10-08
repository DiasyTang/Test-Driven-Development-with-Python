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

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
