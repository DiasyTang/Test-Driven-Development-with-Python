# Test-Driven-Development-with-Python
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
    
   
