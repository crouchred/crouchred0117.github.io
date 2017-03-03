---
layout: post
title: python中import的解决方案
description: 平时我自己是如何处理python中的import的
category: 技术
tags: [python]
---

## 引言
我在python方面也还算是新手，平时工作的时候，如果直接抄老司机import的代码，倒是基本没什么问题，但是自己新写一个项目的时候，由于python的import有两种方式，且加载系统路径的方式也不一样，难免会产生困惑。
比如在看这篇[structing your project](http://docs.python-guide.org/en/latest/writing/structure/#test-suite)的时候，`from .context import sample`就一直不知道怎么跑起来，故记录。

## 正文
### 绝对引用与相对应用
python中import有绝对引用和相对引用两种形式。
对于对于两种引用的方式选择，在PEP8中，Python官方推荐的是绝对引用,详细理由可以参考[pep8](https://www.python.org/dev/peps/pep-0008/#imports)。
1. 绝对引用
    + `import module`
    + `from module import submodule`
2. 相对引用: 用`.`来表示和当前路径的关系
    + `import .module`
    + `from .module import submodule`

### 相对引用
相对引用的好处是可以清晰的看到引入的模块和当前目录的关系。但是我自己写的时候不大喜欢。因为相对引用比如以module的方式运行脚本。方法有二：
1. `python -m package.moduleX`
2. 在另一个，pakcage文件夹以外的py文件中，调用该module

### 绝对引用
绝对引用的问题在于如果执行的路径不对，会抛出ImportError的错误。一般都要指定sys.path才靠谱，在工作中我会采用两种办法指定PYTHONPATH
1. 在代码中加载根目录
```
import os
import sys
sys.path.append(os.path.abspath(os.path.dirname(__file__)).rsplit('/',1)[0])
```
这样就能加载该文件所在的文件夹的再上一层文件夹了。
其实我自己是比较喜欢这种方法的。。因为根据文件定位的话，怎么执行都能准确的加载sys.path。但是问题是写起来要三行比较长，而且根据文件夹的深浅，代码也会有所不同
2. 设置环境变量
    ```export  PYTHONPATH=<your directory>```
这样的缺点是。。如果通过~/.bash_profile把好几个项目的目录都加到PYTHONPATH里面，可能会出现两个重名的包的情况。

#### 我的解决办法：使用autoenv包
简单的说[autoenv](https://github.com/kennethreitz/autoenv)的作用是，在根目录下创建一个.env文件，在进入文件夹的时候，将自动执行.env中的代码,比如设置PYTHONPATH。
还有很重要的作用是：如果我们使用[虚拟环境virtualenv](http://pythonguidecn.readthedocs.io/zh/latest/dev/virtualenvs.html)，那么进入文件夹的同时，我们可以激活虚拟环境，而不用每次都去active一下
```
    git clone git://github.com/kennethreitz/autoenv.git ~/.autoenv   //官方文档上说可以用pip install，反正我在用pip装了之后，总是没法activate
    echo 'source ~/.autoenv/activate.sh' >> ~/.bashrc   
```
在.env中写入以下代码：
```
	DIR=$(dirname "${BASH_SOURCE[0]}")
	source $DIR/venv/bin/activate
	export PYTHONPATH=$DIR
	echo '.env has been executed'
```

## 参考
+  [structing your project](http://docs.python-guide.org/en/latest/writing/structure/)
+ [pep8](https://www.python.org/dev/peps/pep-0008/#imports)
+ [刘畅的博客](https://github.com/Liuchang0812/slides/tree/master/pycon2015cn)
+ [BrenBarn's answer in stackoverflow](http://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time#answer-14132912)
+ [stackoverflow:get the source directory](http://stackoverflow.com/questions/59895/getting-the-source-directory-of-a-bash-script-from-within)

#### changeLog
+ 20170303: .env中的代码从手动输入文件路径，改为在脚本中获取文件路径

