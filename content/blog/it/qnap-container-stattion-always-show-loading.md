+++
title = "QNAP Container Station打开时一直loading"
date=2020-03-27

[taxonomies]
categories=["NAS"]
tags=["nas", "qnap", "container"]
+++

`Container Station`打开时一直loading。
ssh连上QTS后，也不能通过命令进行`docker`操作，报错：

```
Can't connect to docker daemon.
```

参考了官方文档和论坛里的帖子，尝试了以下操作都不起任何作用：

- 重启`container station`
- 重启所有服务
- 卸载重装`container station`
- 删除虚拟交换机，安装`container station`
- 机器重启后重装`container station`
- 手动安装`container station`旧版本
- 系统固件更新到旧版本，安装`container station`
- 系统固件从旧版本更新到新版本，安装`container station`

继续查找原因。`Container Stattion`的日志在：`/share/CACHEDEV1_DATA/.qpkg/container-station/var/log/container-station`，查看日志都挺正常，只在`ctstation.log`中发现了异常，都是打不开数据库:

```
[2020-03-27 22:29:14,605] ERROR in app: Exception on /api/v1/wizard/workspace/status [GET]
Traceback (most recent call last):
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask/app.py", line 1982, in wsgi_app
    response = self.full_dispatch_request()
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask/app.py", line 1614, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask/app.py", line 1517, in handle_user_exception
    reraise(exc_type, exc_value, tb)
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask/app.py", line 1612, in full_dispatch_request
    rv = self.dispatch_request()
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask/app.py", line 1598, in dispatch_request
    return self.view_functions[rule.endpoint](**req.view_args)
  File "/home/ubuntu/src/build/amd64/container-station/target/ctstation/api/exception.py", line 37, in wrapper
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask_login.py", line 756, in decorated_view
    elif not current_user.is_authenticated():
  File "/usr/local/container-station/python/lib/python2.7/site-packages/werkzeug/local.py", line 348, in __getattr__
    return getattr(self._get_current_object(), name)
  File "/usr/local/container-station/python/lib/python2.7/site-packages/werkzeug/local.py", line 307, in _get_current_object
    return self.__local()
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask_login.py", line 46, in <lambda>
    current_user = LocalProxy(lambda: _get_user())
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask_login.py", line 794, in _get_user
    current_app.login_manager._load_user()
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask_login.py", line 340, in _load_user
    deleted = self._session_protection()
  File "/usr/local/container-station/python/lib/python2.7/site-packages/flask_login.py", line 373, in _session_protection
    if '_id' not in sess:
  File "/usr/local/container-station/python/lib/python2.7/_abcoll.py", line 348, in __contains__
    self[key]
  File "/home/ubuntu/src/build/amd64/container-station/target/ctstation/common/session.py", line 35, in __getitem__
  File "/home/ubuntu/src/build/amd64/container-station/target/ctstation/common/session.py", line 68, in _get_conn
OperationalError: unable to open database file
```

既然是数据库文件打不开，就尝试删除数据库文件。首先停止container station，然后重命名`ctstation.sqlite`为`ctstation.sqlite.bak`，让系统生成新的文件。重新启用container station。结果还是不工作。

继续查找原因， 根据日志里的路径去查看`python`代码，发现有些是pyc文件，反编译pyc文件，查看session相关的代码，没有发现有什么问题，觉得是系统内部状态出了问题。

最后不得不重置系统设置来解决问题。
步骤：

- 在控制台 -> 备份/恢复，备份系统设置
- 在控制台 -> 备份/恢复 -> 恢复出厂默认设置， 选择重置设置，然后等待系统重启
- 重启登录后，在控制台 -> 备份/恢复，浏览之前备份的系统设置文件，点击恢复，等待系统恢复设置和安装App

这次`container station`终于正常工作了。
