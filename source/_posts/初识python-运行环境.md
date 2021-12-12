---
title: '初识python: 运行环境'
author: 不好听
img: /2018/12/04/chu-shi-python-yun-xing-huan-jing/images/timg.jpg
categories: python
tags:
  - python
date: 2018-12-04 19:50:31
---

此过程为<font face="微软雅黑">windows7</font>上的开发环境  

安装<font color=#0099ff face="微软雅黑">[anaconda](https://www.anaconda.com/)</font>(开源的Python发行版本，其包含了conda、Python等180多个科学包及其依赖项)

用pip命令安装开发Web App需要的第三方库：
异步框架aiohttp：{% codeblock %}pip install aiohttp{% endcodeblock %} (下同)
前端模板引擎jinja2：pip3 install jinja2
MySQL的Python异步驱动程序aiomysql：pip3 install aiomysql

### 运行环境 
<font face="微软雅黑">virtualenv</font>：  
每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境。  
**1.安装虚拟运行环境  **  
{% codeblock %}pip install virtualenv{% endcodeblock %}  
**2.cd进入工作目录，创建虚拟运行环境**  
{% codeblock %}virtualenv --no-site-packages venv{% endcodeblock %}  
 参数<font color=#0099ff>--no-site-packages</font>表示不带任何第三方包的“干净”的Python运行环境  
 dir命令可以查看当前目录  
**3.启动环境**  
进入venv\Scripts下，执行activate.bat启动环境(call activate.bat)后就可以安装各种包了，执行deactivate.bat可以关闭环境

启动后，注意到命令提示符变了，有个(<font color=red>venv</font>)前缀，表示当前环境是一个名为<font color=red>venv</font>的Python环境。
在<font color=red>venv</font>环境下，用<font color=#0099ff>pip</font>安装的包都被安装到<font color=red>venv</font>这个环境下，系统Python环境不受任何影响。也就是说，<font color=red>venv</font>环境是专门针对这个应用创建的。  
如果使用idea工具，如<font color=#0099ff>pycharm</font>等，以上步骤可以省略，创建项目勾选相应的选项后，会自动创建项目独立的运行环境

**问题与细节**  
每次需要启动虚拟环境的时候都需要进入虚拟环境的文件夹的Scripy的目录下，非常不方便，可以将Scripts的路径添加到系统环境变量中。  
如果系统还要安装其他的python版本，如python2.7.13；将其路径添加到系统环境变量后，进入安装文件目录，将python.exe文件改为python2.exe文件，防止命令冲突；
如果同时存在多个python版本，那么pip也有多个版本，这个时候使用pip安装需要指定python版本。  
{% codeblock %}  
python -m pip install xxx   # python3版本安装包  
python2 -m pip install xxx  # python2版本安装包  
{% endcodeblock %}  
每次添加了系统环境变量以后，需要关闭当前的cmd窗口，重新启动一个窗口才会生效。