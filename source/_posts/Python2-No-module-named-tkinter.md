---
title: python2 No module named _tkinter of matplotlib
date: 2017-02-20 10:18:06
tags: [python, matplotlib, troubleshooting]
categories: [python, troubleshooting]
---

# Environment
```bash
Python：2.7.13
Pip：9.0.1
matplotlib：2.0.0  (pip install matplotlib)
```


# Error Message
```bash
>>> import matplotlib.pyplot
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
 File "/opt/soft/python2/lib/python2.7/site-packages/matplotlib/pyplot.py", line 115, in <module>
 _backend_mod, new_figure_manager, draw_if_interactive, _show = pylab_setup()
 File "/opt/soft/python2/lib/python2.7/site-packages/matplotlib/backends/__init__.py", line 32, in pylab_setup
 globals(),locals(),[backend_name],0)
 File "/opt/soft/python2/lib/python2.7/site-packages/matplotlib/backends/backend_tkagg.py", line 6, in <module>
 from six.moves import tkinter as Tk
 File "/opt/soft/python2/lib/python2.7/site-packages/six.py", line 203, in load_module
 mod = mod._resolve()
 File "/opt/soft/python2/lib/python2.7/site-packages/six.py", line 115, in _resolve
 return _import_module(self.mod)
 File "/opt/soft/python2/lib/python2.7/site-packages/six.py", line 82, in _import_module
 __import__(name)
 File "/opt/soft/python2/lib/python2.7/lib-tk/Tkinter.py", line 39, in <module>
 import _tkinter # If this fails your Python may not be configured for Tk
ImportError: No module named _tkinter
```

<!--more-->

# Solution
Recompile python

1. Extract the source file
    ```bash
    [root@arno Python-2.7.13]# tar xzf Python-2.7.13.tgz
    [root@arno Python-2.7.13]# cd Python-2.7.13
    ```

2. Check the tk/tcl version，Step 4 need the version information
    ```bash
    [root@arno Python-2.7.13]# rpm -qa | grep ^tk
    tk-8.5.7-5.el6.x86_64
    tkinter-2.6.6-66.el6_8.x86_64
    [root@arno Python-2.7.13]# rpm -qa | grep ^tcl
    tcl-8.5.7-6.el6.x86_64
    ```

3. Install tk/tcl devel
    ```bash
    [root@arno Python-2.7.13]# yum install tk-devel tcl-devel -y
    ```

4. Find these lines and cancel the comment
    ```bash
    [root@arno Python-2.7.13]# vim Modules/Setup
    # *** Always uncomment this (leave the leading underscore in!):
    _tkinter _tkinter.c tkappinit.c -DWITH_APPINIT \
    -L/usr/local/lib \
    -I/usr/local/include \
    -ltk8.5 -ltcl8.5 \      # this version from step 2.
    -lX11
    ```

5. Start complie & install python
    ```bash
    [root@arno Python-2.7.13]# ldconfig
    [root@arno Python-2.7.13]# ./configure --prefix=/opt/soft/python2 --with-ensurepip=install
    [root@arno Python-2.7.13]# make
    [root@arno Python-2.7.13]# make install
    ```

6. Config sys env.
    ```bash
    [root@arno Python-2.7.13]# mv /usr/bin/python /usr/bin/pythonbak
    [root@arno Python-2.7.13]# unlink /usr/bin/python2
    [root@arno Python-2.7.13]# vim /usr/bin/yum
    #!/usr/bin/python2.6
    [root@arno Python-2.7.13]# vim ~/.bash_profile
    # User specific environment and startup programs
    PYPATH=/opt/soft/python2/bin
    PATH=$PATH:$HOME/bin:$PYPATH
    export PATH
    [root@arno Python-2.7.13]# source ~/.bash_profile
    ```

7. Verification result
    ```bash
    Python 2.7.13 (default, Feb 20 2017, 20:35:07)
    [GCC 4.4.7 20120313 (Red Hat 4.4.7-16)] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import matplotlib.pyplot
    >>>
    ```
