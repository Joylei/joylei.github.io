+++
title = "linux内核文件系统监视"
date=2010-08-24

[taxonomies]
categories=["Programming"]
tags=["linux", "python"]
+++
pyinotify是一个基于linux内核的文件系统监视的python模块。

在ProcessEvent类的子类中定义对文件系统改变的事件的处理，形式为process+事件名称，如定义process_IN_CREATE方法来处理IN_CREATE事件，如果要处理所有的事件，则需要定义process_default方法。


下面这个程序监视文件或文件夹的创建或删除操作。
filewatcher.py

```py
#!/usr/bin/env python
#encoding=utf-8
import os
import sys
from pyinotify import WatchManager,Notifier,ProcessEvent

class EventHandler(ProcessEvent):
    def __init__(self,path):
        self.path = path
        self.file = file

    def process_IN_CREATE(self,event):
        path = self.path
        if event.name:
            self.file = '%s' % os.path.join(event.path,event.name)
        else:
            self.file = '%s' % event.path
        print '%s created' % self.file

    def process_IN_DELETE(self,event):
        path = self.path
        if event.name:
            self.file = '%s' % os.path.join(event.path,event.name)
        else:
            self.file = '%s' % event.path
        print '%s deleted' % self.file

def monitor(path='/home/daysky/'):
    handler = EventHandler(path)
    from pyinotify import IN_CREATE,IN_DELETE
    mask = IN_CREATE | IN_DELETE
    wm = WatchManager()
    notifier = Notifier(wm,handler)
    wm.add_watch(path,mask)
    print 'monitor starting...'
    while True:
        try:
            notifier.process_events()
            if notifier.check_events():              
                notifier.read_events()
        except KeyboardInterrupt:
            notifier.stop()
            print 'monitor stopped.'
            break

if __name__ == '__main__':
    monitor()
```

启动监视器`python filewatcher.py`

在命令行中执行以下操作：
```
mkdir test
rmdir test
touch test.txt
rm test.txt
```

可以看到执行结果如下：
```
monitor starting...
/home/daysky/test created
/home/daysky/test deleted
/home/daysky/test.txt created
/home/daysky/test.txt deleted
```

参考：http://pyinotify.sourceforge.net/

---
从我的百度空间导入