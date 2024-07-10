---
title: Python2 Tab Autocomplete
date: 2017-02-12 10:31:52
tags: [python, optimization]
categories: [Python, Python2]
---

# Linux Python2 Tab Autocomplete

① 新建 `vim ~/.pystartup` Python2 启动时的环境变量加载文件。
```python
# Add auto-completion and a stored history file of commands to your Python
# interactive interpreter. Requires Python 2.0+, readline. Autocomplete is
# bound to the Esc key by default (you can change it - see readline docs).
#
# Store the file in ~/.pystartup, and set an environment variable to point
# to it:  "export PYTHONSTARTUP=~/.pystartup" in bash.
import atexit
import os
import readline
import rlcompleter
readline.parse_and_bind('tab: complete')
historyPath = os.path.expanduser("~/.pyhistory")
def save_history(historyPath=historyPath):
    import readline
    readline.write_history_file(historyPath)
if os.path.exists(historyPath):
    readline.read_history_file(historyPath)
atexit.register(save_history)
del os, atexit, readline, rlcompleter, save_history, historyPath
```

<!--more-->


② 设置系统变量
```bash
# 即时生效，重启失效
export PYTHONSTARTUP=~/.pystartup  

# 永久生效
echo "export PYTHONSTARTUP=~/.pystartup" >> /etc/profile
```

# Windows Python2 Tab Autocomplete

① 安装 `pyreadline` 依赖
```
pip install pyreadline
```

② 在Python安装路径的Lib文件夹下新建一个 tab.py。
例如：`C:\Program Files (x86)\Python2\Lib`
```bash
# Add auto-completion and a stored history file of commands to your Python
# interactive interpreter. 
import atexit
import os
import readline
import rlcompleter
import sys
# Tab completion   
readline.parse_and_bind('tab: complete')   
# history file，这个路径可以自定义，但需要确保目录存在。
histfile = os.path.join("D:\\Tmp\\history\\", ".pythonhistory")
try:   
    readline.read_history_file(histfile)   
except IOError:   
    pass   
atexit.register(readline.write_history_file, histfile)   
           
del os, histfile, readline, rlcompleter
```

③ 和linux类似，如果想输入python就自动加载tab补全，在系统属性中的环境变量里增加PYTHONSTARTUP变量，值为绝对路径的 tab.py，
```
例如：
变量名：PYTHONSTARTUP
变量值：C:\Program Files (x86)\Python2\Lib\tab.py
```

注：不做这步的话，每次输入python进入交互界面后，需要手动import tab
