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
